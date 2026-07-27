# ARCHITECTURE.md — Project-Wide Source of Truth

## Purpose
This document describes the **ums-watchkeeper product** at the HLD level — how the services fit together, what they collectively do, and what the contract surfaces between them are. Service-internal designs live in each service's own `ARCHITECTURE.md`.

It is *not* a plan — it reflects current state. The MVP doc (v1.0) is the plan; this is the present.

> Last reflected change: workspace split (2026-06-09). Update on next cross-service merge.

---

## Workspace layout

```
ums-watchkeeper-master/        (parent dir, no git)
├── ums-watchkeeper-soul/      ← this folder (project-wide context)
├── ums-watchkeeper-sentinel/  ← Python detection service
└── ums-watchkeeper-ui/        ← Next.js dashboard (future)
```

Each service folder is an independent git repo. See `adr/0001-workspace-polyrepo.md` for the workspace-shape decision.

---

## Services

| Service | Folder | Status | Role |
|---------|--------|--------|------|
| **sentinel** | `../ums-watchkeeper-sentinel/` | Phase 1 live (offline pipeline) | Observe (video → soon RTSP cameras → later read-only sensors), judge (alert state machine), alert (DB + WebSocket + Telegram) |
| **ui** | `../ums-watchkeeper-ui/` | Not started | Display: live feed with bbox overlay, alert history, confirm / false-positive feedback |

Service-internal architecture for each lives in that service's own `ARCHITECTURE.md`.

---

## System context — current (Phase 1)

```
┌──────────────────┐    ┌────────────────────┐    ┌────────────────┐
│ data/videos/*mp4 ├───▶│ sentinel (CLI)     ├───▶│ logs/*.log     │
│ (operator drops  │    │ Phase 1 pipeline   │    │                │
│  a file in)      │    └────────────────────┘    └────────────────┘
└──────────────────┘
```

No network surface. No persistent state. No UI yet — sentinel is the only service that exists.

---

## System context — planned full MVP (target by Phase 5)

```
┌──────────┐  RTSP   ┌────────────────┐  WS/REST  ┌──────────────────────┐
│  Camera  ├────────▶│   sentinel     ├──────────▶│         ui           │
└──────────┘         │   (FastAPI)    │           │   (separate repo)    │
                     │   ├── YOLO     │           └──────────────────────┘
                     │   ├── Alert    │  HTTP     ┌──────────────────────┐
                     │   │   logic    ├──────────▶│   Telegram Bot API   │
                     │   └── Postgres │           └──────────────────────┘
                     └────────────────┘
```

The future sensor channel (Phase 2+ of post-MVP) adds a **read-only** input to sentinel; it never modifies sensor state. See soul/CLAUDE.md §"Project Invariants" #1.

---

## Cross-service contracts

The contract surface between sentinel and ui is documented in **`CONTRACTS.md`**. Headline points:

- **WebSocket** — sentinel pushes live detections and alert state updates to ui.
- **REST** — ui queries alert history, posts confirm / false-positive feedback.
- **Schema versioning** — any breaking change is flagged `[API-BREAKING]` in the sentinel CHANGELOG so ui can be bumped in lockstep.

`CONTRACTS.md` is a stub today; it will be filled when Phase 3 designs the WebSocket/REST schema.

---

## What's intentionally absent (cross-product)

The following are *not yet built* anywhere in the product. Listed here so they're visible from the top.

| Capability | Owner service | Phase |
|------------|---------------|-------|
| Fine-tuned model (`oil`, `smoke`) | sentinel | 2 |
| Alert logic (threshold + persistence + cooldown per §6.3) | sentinel | 3 |
| FastAPI service | sentinel | 3 |
| PostgreSQL (`detection_events`, `camera_health`) | sentinel | 3 |
| Telegram bot | sentinel | 3 |
| React/Next.js dashboard | ui | 4 |
| Confirm / false-positive feedback loop | ui ↔ sentinel | 5 |
| Camera health monitoring | sentinel | 5 |
| Docker Compose packaging | (workspace-level) | 5 |

See `BACKLOG.md` for the phased milestone list pulled from the MVP doc.
