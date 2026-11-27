# Receipt-Verified Video Pipeline v1 — RESULTS (RO3 + P3, 2026-11-24)

**Deliverable:** `mojo-receipts-v1.mp4` — 2,227,402 B · 72.53 s · H.264 1280×720
yuv420p @30 fps · AAC narration (Helio) · SHA-256
`07ba0f92efea5cbe654ae974026fefc94bac536a96c1d918e06b389744706009`.
A ~73 s data-story of the Mojo lane's verified results (bout bars, the
crossover curve drawing itself, the corpus-app stats), produced end-to-end by
the wrapper pattern: deterministic director → stage commands → a SAR receipt
per stage → gates between stages. Nothing published; local/private.

## Final receipt chain (on disk, re-verified after production)

| Stage | Bundle | Verification |
|---|---|---|
| Compute — crossover polyline (Mojo kernel) | `video_crossover-mojo-2bbbff08` | **verify.py PASS** (rehash → rebuild → re-run → hash match). Kernel-computed crossovers: Mojo 47.81 blocks, C++ 51.37 — matching the incremental RESULTS interpolations (≈48 / ≈51) |
| Compute — bout bars (Uiua) | `video_bars-render-9e6a3c41` | check PASS; Uiua run twice in-stage, stdout identical (deterministic) |
| Narration (TTS per-sentence + concat) | `video_narration-render-707f7427` | check PASS · 70.9 s |
| Frames (PIL, 2,176 frames) | `video_frames-render-d8c44830` | check PASS · Pass 3 rehash **2,176/2,176 frames match the manifest** |
| Assembly (ffmpeg + loudnorm finish) | `video_assembly-render-96bc53ec` | check PASS · facts carry the exact encode command + measured output loudness |

## RO3 Pass 1 — Audit + scope: DONE
Full audit in PLAN.md. Headlines: Demo 2's prep is BUILT+VERIFIED (kernel,
server, takes) but was never receipted (its receipts/ folder is empty) — this
pipeline borrows its determinism discipline, not its code. Lab map applied:
Uiua does whole-table math (bar geometry), Mojo does the kernel lane
(crossover points), ffmpeg assembles (field-notes discipline), PIL draws.
Maple build route live-checked before any native promise was made.

## RO3 Pass 2 — Build + produce: DONE (with three named failures en route)
1. **TTS single-request render FAILED** — backend stopped partway (76,065 B
   sent, then error); the sentence "Softmax: Mojo, 7,589 nanoseconds." fails
   even alone (colon form trips the backend; em-dash form renders). The
   director's gate stopped the line — no bypass. Fix: per-sentence rendering
   + concat with 0.12 s gaps; narration.txt carries the em-dash form (same
   words, same numbers).
2. **Frames stage crashed twice on a PIL trap** — `rounded_rectangle` with
   float coordinates truncates internally and raises `x1 must be >= x0` for
   narrow bars (the corpus repair bar growing through ~3–13 px). Root-caused
   empirically (int coords pass where float coords raise), fixed with int
   widths + adaptive radius + plain-rectangle fallback, then proven by a
   full sweep of every scene function across its whole time range before
   the final run.
3. **A service restart killed one director incarnation mid-frames**, and one
   background completion report arrived from a discarded incarnation whose
   bundles (927ffb47 / 97ce0f34 / 8d7a5f52 / 45c2ba68) **never existed on
   disk**. They are cited nowhere; only on-disk bundles count. The final
   chain above is from one coherent run, re-verified after the fact.
First production (run 1) also completed all gates; it was superseded by the
Pass 3 defect fixes, not by any failure of its own.

## RO3 Pass 3 — Perfect: DONE
Independent verification of the finished MP4 (frames extracted FROM the
shipped file and read, per the field notes — never off the render):
- ffprobe: H.264, 1280×720, yuv420p, 30 fps, AAC, 72.53 s, 2,227,402 B ✓
- Frames read at 7 timestamps. **3 defects found, fixed, re-verified in the
  re-produced file:** (a) crossover footnote ran off the right edge and the
  C++ crossover label clipped — footnote shortened, labels now flip to the
  left of their line when they'd overflow; (b) corpus card sub-lines
  overflowed their cards — auto-fit font sizing; (c) narration measured
  **−27.5 LUFS** integrated (too quiet) — loudnorm finish added at assembly;
  shipped file measures **−17.49 LUFS, true peak −1.28 dBTP**.
- All stage receipts re-checked PASS; frame manifest rehash 2,176/2,176.
- **Deviation on the record:** concept length was ~45–55 s; the delivered
  clip is 72.5 s because the TTS pace (~117 wpm on this script) is slower
  than estimated. Content is complete; the length is a pacing fact, not a
  defect claim either way.

## P3 — Native: DONE
`native/MojoVideoPipeline.exe` — 2,240,512 B, SHA-256
`1c864be0c74e32f52839f51fd8b9888bfe9a8d7428b92b5d1534472ab495e587`.
C# WinForms + WebView2 (sibling pattern), built on MAPLE with csc; embeds the
viewer page (gzip, SHA-checked: page 4,627 B, SHA-256 82d039193c…) and the
MP4 itself (SHA-checked against the production file). Viewer page generated
from the actual receipt.json files — no hand-typed numbers (one generator
bug caught in review: bundle "newest" was name-sorted, which selected
superseded bundles; fixed to receipt-write-time order, chain re-checked).
**--smoke-test ON MAPLE: SMOKE PASS, exit 0** — 6/6 TITLE_OK, VIDEO_OK,
WebView2 runtime 158.0.4370.33. Exe pulled back to the VM; SHA-256 identical
to Maple's build hash (parity ✓). Copy also at
`C:\Users\kylel\mojo-video-native-build\` on Maple. GUI pixels not
screen-driven (sibling precedent — Kyle's visual gate). One catch inside
the smoke output worth naming: the page stamp in the shipped exe is the
generator's second-run hash, consistent end-to-end (local viewer.html
SHA matches the embedded decompressed SHA).

## Director seat / LFM deferral (on the record)
The director is a deterministic script + Duckie driving. **No LFM or trained
model was wired in**: the lab's pruning data scored the LFM3 director
candidate 0–1/12 vs 4/12 for the original — not evidence-ready. The stage
interface is exactly the surface a future model director would emit (one
deterministic command per stage, receipt/ffprobe read-back — the ffmpeg
field-notes architecture), so the seat can be swapped without touching the
stages. Staged path, recorded — not a silent omission.

## Scores (honest, self-graded against the receipts)
- Receipt integrity / gating: **10/10** — every stage gated + receipted; the
  gates demonstrably stopped the line twice on real failures.
- Content accuracy: **10/10** — every number on screen traces to a
  verify.py-PASS receipt; the Mojo kernel's crossovers independently match
  the RESULTS interpolations.
- Audio: **8/10** — clear Helio narration, loudness normalized; per-sentence
  concat seams are inaudible in practice but the pace stretches the runtime.
- Visual polish: **7/10** — clean, legible data-story; PIL flat graphics,
  motion limited to bar growth / curve reveal / fades.
- Length vs concept: **72.5 s delivered vs ~45–55 s concept** — deviation
  recorded above.
- Native packaging: **9/10** — smoke PASS + hash parity; pixels unverified.

**FAILED items (final state): none.** Failed en route and fixed: TTS
single-request render, frames-stage PIL crash (×2), viewer generator bundle
selection. Receipts are evaluation artifacts — never training data.
