# TRD — Oil detection model (build + evaluate)

- **Status:** Draft
- **Implementing service:** sentinel
- **Phase:** 2a
- **References:** [./spec.md](spec.md) · [./curation.md](curation.md) · [./brother-review.md](brother-review.md) · sentinel ADR-01, ADR-04 · soul HARDWARE_NOTES

> Technical approach and decisions for producing and evaluating the detector (`best.pt`). Integrating it into the running service is a separate feature (`oil-detection-integration/`). **No code here** — only references to existing code where they justify a decision.

## Approach
Fine-tune stock `yolo11n.pt` (sentinel ADR-01) to a **single class `oil`** (sentinel ADR-04) on our curated dataset; group-aware split; evaluate on a held-out space.

## Technical decisions
- **D1 — Class = single `oil`** (not `oil_leak`). The data is overwhelmingly pooled/soaked with only ~1–2 active-leak frames ([curation.md](curation.md) §2, §7); a separate leak class would be untrainably small and `oil_leak` misnames a mostly-spill detector. The enum `{oil, normal}` is realized in the integration feature.
- **D2 — Split = group-aware** by machinery space + burst group ([curation.md](curation.md) §6). Single held-out space vs k-fold is decided once [brother-review.md](brother-review.md) sets the positive count and spatial spread.
- **D3 — Prep pipeline** = neutralize the burnt-in timestamp overlay (overfit trap), fix the one rotated image, normalize orientation, downscale from 4608 px to a standard `imgsz`, augment for low volume ([curation.md](curation.md) §8).
- **D4 — Environment** = train on Apple `mps` with a CPU/Colab sanity check (HARDWARE_NOTES).
- **D5 — Evaluation** = held-out *space* (not random split) + false-positive rate on clean stills and the walkthrough videos ([curation.md](curation.md) §10) + explicit probes for the "green deck = safe" and "dark bilge = oil" shortcuts ([curation.md](curation.md) §9). mAP reported, not the sole gate.

## Packages
`ultralytics`, `torch`, `roboflow` — all already present; no new dependencies.

## Code references (justification only)
The eventual integration seam is the existing `Detector` wrapper (sentinel `src/inference/detector.py`); it is untouched by this feature.

## Delivery (PRs)
prep → train (`best.pt`) → evaluate (report). The split-strategy call is made at the top of prep, gated on brother-review.

## Open questions
- **Brother-review outcome** — how many of the ~52 become positives, and spread across spaces or still one event? *Blocking* — sets the split strategy.
- **More data?** If confident positives stay < ~30 and single-event, dense-sample the Purifier/ER-Lower videos and/or request a targeted follow-up photo set.
- **`imgsz` + augmentation recipe** against `mps` memory/speed.

## Invariants check (soul/CLAUDE.md)
Sensor independence — N/A (vision only). Alert constants — N/A here (belong to integration). Detection enum — lands in integration. MVP scope — single space, single vessel, no edge.
