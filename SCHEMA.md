# DSS SAR Observation — Mojo Credit Extension v0.1

Drafted 2026-11-23 by Duckie for Dripstatic Studios. Status: WORKING DRAFT, in use by the mojo-arena harness the same night.

## Purpose

The base SAR observation records that a local inference/computation happened, with hashes a third party can re-check. This extension makes the record **credit Mojo cleanly** when the executed artifact is Mojo code produced by a model or agent loop — the exact class of public record that did not exist as of 2026-11-22 (verified gap: no public, provenance-complete, Mojo-credited model outputs from any current flagship family).

## Base fields (SAR observation, unchanged)

| field | meaning |
|---|---|
| `schema` / `schema_version` | `"dss.sar.observation"` / `"0.1"` |
| `observation_id` | UUID for this observation |
| `observed_at` | ISO-8601 timestamp of the run |
| `task.request_sha256` | SHA-256 of the canonical task spec (the request) |
| `generator.model_id` | who/what produced the code (model or agent identity) |
| `artifacts.source_sha256` | SHA-256 of the source file that was compiled |
| `artifacts.binary_sha256` | SHA-256 of the compiled binary that ran |
| `execution.output_sha256` | SHA-256 of the program's canonical stdout |
| `execution` timings | wall-clock runs + kernel self-reported time |
| `provenance` | producer, host, and use restrictions |

## Mojo extension fields (`extension.name = "mojo-credit"`, `version = "0.1"`)

| field | meaning |
|---|---|
| `engine.name` | `"Mojo"` (the credit line — this is what makes the output Mojo-attributed) |
| `engine.version` | exact compiler version string, e.g. `Mojo 1.2.0 (2016d0cd)` |
| `engine.toolchain` | how the compiler was obtained, e.g. `pixi · conda.modular.com/max (stable)` |
| `engine.target` | execution target, e.g. `cpu x86-64 (AVX2/FMA)`; GPU targets name the device |
| `attempts[]` | **every** generation attempt, in order: source hash, verdict (`compile_error` / `test_fail` / `ok`), and the compiler's error excerpt on failure. Failed attempts are part of the record, never deleted — the retry history is the reliability evidence. |
| `artifacts.build_command` | the exact build command, so a verifier rebuilds identically |
| `baseline` | optional champion reference: another engine's receipt for the same task (e.g. hand-written C++), with its kernel time. A Mojo result means little without the bar it was measured against. |
| `oracle` | **J-twin referee block (gate added 2026-11-26).** Every receipt cut after the gate carries one: `status: "pass"` with the bundled twin (`oracle_twin.ijs`, hash in `twin_sha256`), per-key `checks`, and `oracle_report.txt` in the bundle — or `status: "exempt"` with a permanent recorded `reason` (data-pipeline kernels, iterative float sims). observe.py cuts **no receipt at all** without a passing twin or a declared exemption. Bands (pre-registered 2026-11-25): integers exact, floats \|diff\| ≤ 1e-4; iterative sims are judged by per-frame statistics, never by checksum (`oracle.mode = "frame_stats"`, twins in `oracle/twins_frames/`, bands in `j_oracle.FRAME_TASKS` — particles is frame-gated since 2026-11-26, no longer exempt). Twins live in `oracle/twins/<task>.ijs`; the gate is `oracle/j_oracle.py`. Receipts cut before 2026-11-26 have no `oracle` field and are grandfathered — verify.py warns, it does not retro-pass them. |

## Verification procedure (what the public verifier does)

1. Re-hash the bundled source and binary; both must match the receipt.
2. Rebuild the bundled source with the receipt's build command and the named toolchain.
3. Re-run the rebuilt binary on the task's deterministic inputs.
4. Hash the new stdout; it must match `execution.output_sha256`.
5. Verdict: PASS only if 1–4 all hold.

Determinism rule: arena tasks generate their inputs from fixed integer formulas inside the program — no RNG, no external data files — so any machine can reproduce the output byte-for-byte (float formatting is per-engine; cross-engine agreement is checked numerically, never by hash).

Timing rule (learned the hard way, 2026-11-23, first verifier run): **timing never lives inside the hashed output.** Kernel self-timing lines (`avg_ns = …`) are stripped to form the canonical output before hashing; timings are recorded as data in `execution.*` instead. A receipt whose output hash covers a timing line can never verify — the first four arena receipts failed exactly this way before the rule was written.

## Standing restrictions

- Receipts are **evaluation artifacts**. Under the DSS teacher bright line they are never training data.
- Nothing about a receipt claims more than it shows: one task, one machine, one compiler version. Generalizations get their own receipts.
