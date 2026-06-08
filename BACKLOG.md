# BACKLOG.md — Project-wide phased backlog

## Purpose
Top-down view of where the product is in its phased build (per MVP doc v1.0 §6). Each phase is a *cross-service* milestone — the work is allocated to services below. Update when a phase starts, completes, or its scope shifts.

> Service-internal sprint planning lives in each service's own TRD / SPEC files (e.g. `../ums-watchkeeper-sentinel/docs/specs/`). This file is the milestone tracker, not the task list.

---

## Phase status

| Phase | Goal | Lead service(s) | Status | Notes |
|-------|------|-----------------|--------|-------|
| **1** — Pipeline | Video → OpenCV → YOLOv11n → console/log | sentinel | ✅ **Done** | Stock model, no alert logic, no DB. Tested on `factory_man.mp4` / `test.mp4` (generic footage). |
| **2** — Detection model | Fine-tune YOLOv11n on ~300 labelled images of `oil_leak` / `smoke` | sentinel | ⏳ **Blocked on brother's photos** | 100–300 phone photos from engine room rounds. Public Roboflow Universe datasets can bootstrap. Re-tune `CONFIDENCE_THRESHOLD` against the fine-tuned validation set (target §6.3 = 0.65). |
| **3** — Alerts + persistence + Telegram | FastAPI service, Postgres (`detection_events`, `camera_health`), Telegram bot, alert state machine (threshold + persistence ≥ 5 frames + 120s cooldown) | sentinel | ⏳ Not started | Lots of design ahead — brainstorm before TRD. Will introduce the cross-service contract (`CONTRACTS.md`). |
| **4** — UI dashboard | Live feed + bbox overlay + alert panel + confirm / false-positive buttons | ui | ⏳ Not started | UI repo (`ums-watchkeeper-ui`) doesn't exist yet — created at the start of this phase. |
| **5** — Packaging + camera health + feedback loop | Docker Compose, camera health indicator, confirm/FP clicks → training labels | sentinel + ui | ⏳ Not started | First cross-service work — `[API-BREAKING]` flag will start mattering here. |
| **6** — Demo + pitch | 10-minute demo recording for brother; one-pager for class-society conversations | (workspace-level) | ⏳ Not started | The "brother watches and says 'this would have caught it'" success criterion lives here. |

---

## Cross-service watch list

These items aren't a phase of their own but cross services and matter:

- **Cross-service contract** (`CONTRACTS.md`) — designed in Phase 3, evolves through Phase 5. Breaking changes flag `[API-BREAKING]`.
- **Workspace tooling** — currently just symlinked skills + `master/README.md`. Phase 5 may add `master/docker-compose.yml` for local end-to-end runs.
- **Brother engagement** — Phase 2 blocked on real engine-room photos. Phase 6 success criterion is brother's verdict. He is also the operational user from Phase 4 onward.

---

## Out of MVP scope (do not promote into a phase without explicit decision)

| Item | Why deferred |
|------|--------------|
| Multi-camera correlation | Phase 2+ post-MVP |
| Fleet / multi-vessel | Phase 2+ post-MVP |
| Edge deployment (Jetson / RPi) | Pilot-only |
| ATEX / oil-mist enclosures | Pilot-only |
| IAS / MODBUS / NMEA sensor integration | Phase 2+ post-MVP; would land as read-only input only |
| Class-society certification | Pilot-only; certifiability *story* is built in but cert *itself* is post-pilot |

---

## How to update this file
- When a phase changes status (start, complete, block, unblock): edit the row in the status table.
- When scope shifts inside a phase: add a sub-bullet in `Notes`.
- When a new cross-service item arrives: add to `Cross-service watch list`.
- This file is **append-edit** rather than append-only — phases change status legitimately. Major scope shifts inside a phase should also leave a CHANGELOG entry in this repo.
