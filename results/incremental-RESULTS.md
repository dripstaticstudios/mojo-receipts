# Tier 1 — Incremental Exact Top-k Search: Results

Run 2026-11-23 on the Duckie VM (Mojo 1.2.0 (2016d0cd) via pixi · g++ 13.3.0 -O3).
Sources: `kernels/incremental_search.mojo`, `kernels/incremental_search.cpp` — the SAME
design implemented in both languages, run through the same schedules.
Receipts: `receipts/incremental_search-mojo-61164680/`, `receipts/incremental_search-cpp-6679e92c/`
— **both PASS verify.py** (rehash → rebuild → re-run → output-hash match).
All numbers below are taken from the receipt outputs. Receipts are evaluation
artifacts, never training data (SCHEMA.md standing restriction).

## Design (identical in both languages)

- Corpus: 512 vectors × 64 dims, cosine similarity vs a fixed query — the
  embed_search arena setup (deterministic in-program formulas, no RNG, no data files).
- Vectors split into **64 blocks of 8**. Each block caches its top-8 (idx, score) pairs.
- An edit replaces a vector's value formula (salt 0 = original formula; salt > 0 = edited formula).
- **Repair path:** recompute only the blocks containing edited vectors, then merge the
  64 block caches (512 cached pairs) into the global top-8.
- **Reference path:** brute-force full scan over the current state.
- **Correctness gate:** after every edit batch, incremental result must match the full
  scan exactly — same indices in the same order, scores within 1e-5.

## Correctness evidence

| Check | Mojo | C++ |
|---|---|---|
| Edit batches checked | 150 | 150 |
| Mismatches | **0** | **0** |
| Final top-8 indices | 23, 46, 69, 92, 115, 138, 161, 184 | 23, 46, 69, 92, 115, 138, 161, 184 (identical) |

Batch schedule: 1–6 edits per batch, spread over distinct blocks by a fixed
formula; edits accumulate across batches (a vector edited once stays edited),
so later batches test repair over a heavily-edited corpus, not a pristine one.

### Attempt 1 — FAILED, root-caused, preserved

The first version of both implementations reported **104/150 mismatches**. Root
cause was in the test harness's edit formula, not the incremental design:
`(i*13 + j*29 + salt*7) % 29` makes the j-term vanish (29j ≡ 0 mod 29), so every
edited vector was constant across dims — and for vec 498 / salt 465 the constant
was exactly 0, giving a zero-norm vector whose cosine is **NaN**. NaN breaks the
total order inside insertion, so the repair and scan paths diverged. The cache
audit passed throughout (NaN − NaN comparisons are false, so the staleness check
was blind to it) — caught by printing the failing block's cache against fresh
scores. Fix: edit formula changed to the base formula's family,
`(i*7 + j*11 + salt*5) % 23 − 11) / 11`, which varies over j and cannot produce
a zero vector. Failed sources preserved as `kernels/incremental_search_attempt1.{mojo,cpp}`.

## Crossover sweep — side by side (ns per operation, from the receipts)

Each level: T blocks touched by fresh edits (1 vector per block), repair timed
over 200 reps; full scan re-measured at the same level over 200 reps.

| Blocks touched T | % of corpus | Mojo repair | Mojo full scan | Mojo speedup | C++ repair | C++ full scan | C++ speedup |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 1 | 1.6% | 14,538 | 123,242 | 8.5× | 8,208 | 154,076 | 18.8× |
| 2 | 3.1% | 17,279 | 128,475 | 7.4× | 11,402 | 153,268 | 13.4× |
| 4 | 6.3% | 23,302 | 138,798 | 6.0× | 17,207 | 153,871 | 8.9× |
| 8 | 12.5% | 35,135 | 155,282 | 4.4× | 28,400 | 156,194 | 5.5× |
| 16 | 25% | 57,755 | 155,304 | 2.7× | 52,105 | 153,085 | 2.9× |
| 32 | 50% | 107,145 | 149,468 | 1.4× | 97,461 | 158,963 | 1.6× |
| 64 | 100% | 194,156 | 150,798 | 0.78× | 196,301 | 156,196 | 0.80× |

## Crossover points

Repair cost grows ~linearly in T (block recompute) on top of a fixed merge cost
(the 512-pair cache merge runs at every level). Full scan is ~flat in T.

- **Mojo: crossover ≈ 48 blocks touched (~75% of the corpus).** Repair wins at
  T ≤ 32, loses at T = 64 (194,156 vs 150,798 ns). Interpolated from the receipt
  values: repair slope ≈ 2,719 ns/block between T=32 and T=64, crossing the
  ~150.8k full-scan line at T ≈ 48.
- **C++: crossover ≈ 51 blocks touched (~80% of the corpus).** Repair wins at
  T ≤ 32, loses at T = 64 (196,301 vs 156,196 ns). Repair slope ≈ 3,089 ns/block,
  crossing the ~156.2k line at T ≈ 51.

At T = 64 repair degenerates to "rebuild everything + merge", costing ~1.27×
(Mojo) / ~1.26× (C++) the plain full scan in both languages — the merge overhead
is the entire penalty, as the design predicts.

## Mojo vs C++ read

- **Full scan:** Mojo ~123–155k ns vs C++ ~153–159k ns — Mojo at parity to
  slightly faster, consistent with the embed_search bout (134,079 vs 179,702 ns).
- **Repair at small T:** C++ is faster (8,208 vs 14,538 ns at T=1). At low edit
  counts the fixed merge dominates, and C++'s small-vector handling beats Mojo's
  `List` allocation pattern in this implementation. The gap narrows as T grows;
  at T=64 the two are indistinguishable (194,156 vs 196,301 ns).
- **The incremental thesis holds in both languages:** for edit loads up to half
  the corpus, repair is strictly cheaper than rescan, with exact — not
  approximate — results, verified batch-by-batch against the full scan.

## Caveats

- One machine, one compiler version each, shared-VM timing noise (Mojo full-scan
  readings ranged 123k–155k across levels for identical work). Crossover points
  are interpolations between measured levels, not direct measurements.
- Block size 8 = K means each block cache holds the whole block; a production
  design with block size ≫ K would show a cheaper merge per block and a later
  crossover. That variant is future work, not claimed here.
