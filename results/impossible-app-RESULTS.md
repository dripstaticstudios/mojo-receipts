# Impossible App — Results

Run 2026-11-23 on the Duckie VM (Mojo 1.2.0 (2016d0cd) via pixi).
Engine: `corpus_search.mojo`. Receipt: `receipts/corpus_search-mojo-24f1ff8d`
— **PASS verify.py** (rehash → rebuild → re-run → output-hash match).
All numbers below are taken from that receipt's output. Receipts are
evaluation artifacts, never training data.

## Corpus

| Book | Passages |
|---|---:|
| Pride and Prejudice (Gutenberg #1342) | 956 |
| Frankenstein, 1818 edition (archive.org `Frankenstein1818Edition`, djvu OCR) | 583 |
| The Art of War, Giles translation (Gutenberg #132) | 317 |
| **Total** | **1,856** |

Vectors: hashed TF-IDF, 256 dims (method specified exactly in README.md).
Layout: 39 blocks × 48 passages, per-block top-10 cache, K = 10.

## Correctness evidence

1. **Incremental gate (in the receipt):** 60 prepared edit batches —
   real text edits (a sentence appended to 1–3 passages per batch,
   re-vectorized; schedule in `data/edits.json`). After every batch,
   the incremental (repair + merge) top-10 was compared against a
   brute-force full scan over the current corpus state:
   **60/60 batches exact, 0 mismatches** (indices identical, scores ≤ 1e-5).
2. **Cross-language query check (outside the receipt, run 2026-11-23):**
   an independent Python implementation of the same vectorization and
   full scan was compared against the Mojo engine's `--query` mode on
   5 test queries — **5/5 identical top-10 lists**. This pins the
   FNV-1a hash, tokenization, and idf weighting as agreeing exactly
   across the Python prep and the Mojo engine.
3. **Probe queries (in the receipt):** "elizabeth darcy marriage" →
   top hits all Pride and Prejudice (ids 949, 116, 1); "spies enemy
   deception war" → top hits all from *The Art of War*, Chapter XIII
   "The Use of Spies" (ids 1834, 1830, 1833).

An honest lexical note: for "monster creature creator revenge" the top
hit is an Art of War passage (Chapter XII, "The Attack by Fire") on
shared vocabulary, with Frankenstein passages at #2 and #3. That is
what a lexical method does — it is reported here rather than tuned away.

## Crossover sweep (from the receipt, ns per operation, 25 reps per level)

Sweep edits swap in another passage's base vector (a cost-equivalent
stand-in for a content edit; the correctness gate above uses the real
prepared text edits).

| Blocks touched T | % of corpus | Repair | Full scan | Speedup |
|---:|---:|---:|---:|---:|
| 1 | 2.6% | 54,266 | 1,615,580 | 29.8× |
| 2 | 5.1% | 98,836 | 1,616,996 | 16.4× |
| 4 | 10.3% | 184,380 | 1,624,306 | 8.8× |
| 8 | 20.5% | 357,424 | 1,612,378 | 4.5× |
| 16 | 41.0% | 700,334 | 1,623,534 | 2.3× |
| 32 | 82.1% | 1,396,559 | 1,622,691 | 1.16× |
| 39 | 100% | 1,699,628 | 1,620,623 | 0.95× |

**Crossover ≈ 37 of 39 blocks (~95% of the corpus).** Repair wins at
T ≤ 32 and loses at T = 39 (1,699,628 vs 1,620,623 ns). Interpolated
from the receipt values: repair slope ≈ 43,296 ns/block between T=32
and T=39, crossing the ~1.62M full-scan line at T ≈ 37. At T = 39 the
repair degenerates to rebuild-everything + merge at ~1.05× scan cost —
a much smaller degeneration penalty than the Tier 1 arena's ~1.27×,
because here the block (48) is much larger than K (10), so the merge
touches 390 cached pairs instead of 512 for a corpus 3.6× the size.

## Caveats

- One machine, one compiler version, shared-VM timing noise (full-scan
  readings ranged 1,612,378–1,624,306 ns across levels for identical
  work). Crossover is an interpolation between measured levels.
- The engine reads data files from a fixed absolute path (see
  README.md); the receipt's verification is valid on this machine with
  the corpus data in place.
- Frankenstein's text is OCR of a reprint: occasional OCR noise tokens
  survive into the vectors. The search is over the text as obtained,
  and the edition record in README.md says exactly which text that is.
- Chapter locators are "last heading seen"; Frankenstein's three
  volumes reuse chapter numbers (Volume II Chapter I and Volume I
  Chapter I share a locator string). Cosmetic only; passage ids are
  the stable reference.
