# Hardware & Environment Notes — project-wide

## Purpose
Living document for hardware-, camera-, GPU-, and engine-room–environmental quirks discovered during spikes, testing, or demos. **Project-wide** — relevant to any service that touches cameras, GPUs, displays, or the engine room (sentinel today; ui when it lands; any future service). When a finding is service-internal (e.g. a pytest fixture quirk), record it in that service's own docs instead.

## Update rules
- **Append-only within a section.** Never delete an old finding; if a finding is reversed, append a follow-up note pointing back to it.
- A finding is worth recording if it would have saved you (or a future contributor) an hour. If it's obvious to anyone reading the code, leave it out.
- Format: `### YYYY-MM-DD — Short title` then body.

---

## Dev / demo machines

### 2026-05-03 — Two-machine setup (dev + demo)
- **Dev (Apple Silicon MacBook):** runs Phase 1 against test video, `UMS_DEVICE=mps`. Set in `.env`. Inference works but `mps` backend in Ultralytics occasionally falls back to CPU on certain ops — verify with `--device cuda` on the demo machine before the brother demo.
- **Demo (Helios 300, NVIDIA GPU):** target for the brother demo. `UMS_DEVICE=cuda`. Not yet bring-up tested.
- The `UMS_DEVICE` env var is the single switch. Don't hardcode device anywhere else.

---

## GPU

### 2026-06-10 — Phase 2a fine-tuning is a small job; train anywhere
- yolo11n (nano) on ~100–150 labelled images is **minutes, not hours**. Training hardware is not a bottleneck at this scale — don't over-invest in it.
- **Mac (Apple Silicon, `mps`):** fine for training this. The real caveat is MPS-backend *maturity*, not speed — some ops fall back to CPU (`PYTORCH_ENABLE_MPS_FALLBACK=1`) and numerics can differ from CUDA. **Mitigation: don't trust the loss curve — run the trained `best.pt` on a held-out real image and confirm it actually detects oil before believing the metrics.**
- **No Helios 300 (cuda)?** Google **Colab free T4** is the cleanest path: real CUDA, zero setup, and you upload only the *labelled subset* (a few hundred MB), **not** the 1.8 GB raw set — Roboflow exports straight into the notebook. **Kaggle** (free T4/P100) is the backup. Pure CPU works as a last resort (tens of minutes). **Skip** hosted/rented GPU — unnecessary at this scale, and ADR-01 already rejected hosted Roboflow *inference* for the offline-demo requirement (training need not be offline, but the lock-in still buys nothing here).
- Scope: this concerns *training* only. The 2026-05-03 note about `mps` falling back to CPU on some ops still stands for the live *inference* pipeline.

Concerns to watch for, to be confirmed/refuted:
- Laptop GPU thermal throttling during multi-hour inference (a real-world watch is 4 hours)
- VRAM pressure if a second model (sensor-fusion later) shares the GPU
- Frame-drop behavior when inference is slower than camera FPS

---

## Camera

(no entries yet — populate when a real RTSP camera is added in Phase 3)

Concerns to watch for, to be confirmed/refuted:
- **Low light** — engine rooms vary; some areas are well-lit, oil-purifier rooms can be dim. May need IR or low-light tuning.
- **Glare / reflection** — polished metal surfaces, oil films, water on floors all produce specular reflection that confuses object detectors.
- **Mounting angle** — overhead vs eye-level changes the apparent shape of leak puddles dramatically. Document the angle used for the training set.
- **Vibration** — ship engine rooms vibrate constantly. Bracket must dampen, or frames will be motion-blurred.
- **Lens contamination** — oil mist and salt spray will fog the lens over weeks. Out of MVP scope but worth noting for pilot.

---

## RTSP streaming

(no entries yet — populate when RTSP source is added in Phase 3)

Concerns to watch for, to be confirmed/refuted:
- `cv2.VideoCapture` doesn't expose RTSP reconnect — need a wrapper that detects EOF and reopens.
- Codec compat between camera vendors and OpenCV's bundled FFmpeg.
- Network drops on shipboard wifi — must degrade gracefully, not crash.
- Latency budget: how far behind real-time is acceptable before an alert is useless?

---

## Engine-room environment (out of MVP scope, but documented for pilot)

These are explicitly **not** in MVP scope per the project memory. Recorded here so they aren't forgotten when the pilot conversation starts.

- Ambient temperatures routinely 40°C+, sometimes 50°C+ near machinery.
- Humidity high, often condensing on cooler surfaces.
- Salt spray ingress in some spaces.
- Vibration constant.
- Oil mist coats every surface over time.
- Electrical interference from large motors / VFDs.
- Power quality on ship mains can be poor — UPS or DC supply recommended for any edge device.
- ATEX (explosion-proof) requirements in some machinery spaces — would mandate certified enclosures for a real deployment.

---

## Detection model behavior

### 2026-05-03 — Stock yolo11n.pt detects COCO classes only
- `MODEL_WEIGHTS = "yolo11n.pt"` is the stock Ultralytics download — trained on COCO. It will detect `person`, `car`, `cell phone`, etc. — **not** `oil` / `smoke`.
- Phase 1 log output therefore reads as nonsense (e.g. `class=person` on factory footage). That is expected.
- `CONFIDENCE_THRESHOLD` is currently **0.75**, tuned to suppress most stock-model noise. The MVP doc §6.3 production target of **0.65** applies only once the fine-tuned model lands in Phase 2.
- When the fine-tuned model arrives, re-evaluate the threshold against the validation set — do not blindly carry 0.75 over.
