# mojo-receipts

**Mojo-verified, browser-delivered.** This repository is the public evidence record of the Dripstatic Studios Mojo Arena: Mojo kernels benchmarked head-to-head against hand-written C++ baselines, with every run recorded as a signed-off receipt. The 15-slide presentation in [`deck/`](deck/index.html) is the browser-delivered summary; the receipts underneath it are the proof.

This is a **proof-only** repository. The scoreboard is public. The playbook is not.

## Headline results

| Result | Mojo | C++ baseline | Evidence |
|---|---|---|---|
| softmax over 1,024 inputs | **7,589 ns** | 8,039 ns | [`receipts/softmax-mojo-6520ec25`](receipts/softmax-mojo-6520ec25/receipt.json) vs [`receipts/softmax-cpp-7cc26262`](receipts/softmax-cpp-7cc26262/receipt.json) |
| embed_search | **134,079 ns** | 179,702 ns (~1.34×) | [`receipts/embed_search-mojo-d083e8a4`](receipts/embed_search-mojo-d083e8a4/receipt.json) vs [`receipts/embed_search-cpp-95f25de4`](receipts/embed_search-cpp-95f25de4/receipt.json) |
| The Impossible App — incremental exact search over 1,856 passages | **60/60 edit batches exact, 0 mismatches** | — | [`results/impossible-app-RESULTS.md`](results/impossible-app-RESULTS.md) |

Full summaries: [`results/`](results/) · Receipt format: [`SCHEMA.md`](SCHEMA.md) · Every file's size and SHA-256: [`MANIFEST.md`](MANIFEST.md)

## What a receipt is

Each `receipt.json` is a DSS SAR observation (schema in [`SCHEMA.md`](SCHEMA.md)) recording one run: the task, the engine (Mojo 1.2.0), **every** generation attempt in order — compile failures included, never deleted — the SHA-256 of the source that was compiled, the SHA-256 of the binary that ran, the program's output and its hash, timings, and provenance. Receipts cut after 2026-11-26 also carry a referee verdict from an independent J-language twin implementation (the oracle block: per-key checks, twin hash, pass/exempt status).

## What is not here — plainly

**The source code and the compiled binaries are withheld.** No Mojo, C++, Python, J, or Uiua source and no binaries are published in this repository — 155 source/binary/script files and 47 text companions from the original evidence bundles were excluded (counts in [`MANIFEST.md`](MANIFEST.md)). Each receipt's hashes name exactly what ran, so if the source is ever disclosed, anyone can check it against these records.

Because the source is withheld, the full rebuild-and-rerun verification described in `SCHEMA.md` cannot be performed from this repository alone. What anyone *can* check here is the internal consistency and completeness of the records themselves: the hashes, the attempt histories, the cross-engine baselines, and the independent oracle verdicts. Nothing about a receipt claims more than it shows: one task, one machine, one compiler version.

Reading and citing these records is welcome. Reuse is not licensed — see [`LICENSE`](LICENSE). All rights reserved.

## Standing restriction (carried verbatim from the receipts)

Every receipt's provenance field reads:

> `training_use: "none — evaluation artifact (teacher bright line)"`

And the schema's standing restrictions state:

> Receipts are **evaluation artifacts**. Under the DSS teacher bright line they are never training data.

These receipts are published as evidence. They are not a dataset, and they are not training material.

## The deck

[`deck/index.html`](deck/index.html) is the arena's opening presentation — 15 slides, visually signed off at desktop and phone widths on 2026-11-27 (SHA-256 `0b2a5d024c7f4b1668448d0e20e2642e71f9abb0f126720460364f3038c9879c`). [`deck/sourceink.html`](deck/sourceink.html) is its sanctioned-facts companion page. The deck is a presentation of the results; the receipts in this repository are the evidence behind its claims.

---

Dripstatic Studios — [github.com/dripstaticstudios](https://github.com/dripstaticstudios) · Sister repository: [carbon-wasm-repro](https://github.com/dripstaticstudios/carbon-wasm-repro) · [dripstaticstudios.com](https://dripstaticstudios.com)
