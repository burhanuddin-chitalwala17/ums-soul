# CONTRACTS.md — Cross-service contract spec

## Purpose
The wire-level contract between **sentinel** (Python detection engine) and **ui** (Next.js dashboard, future). When either side changes the contract, the change is flagged `[API-BREAKING]` in the changing repo's CHANGELOG and this file is updated in lockstep.

## Status
**Stub.** No contract exists yet — sentinel is at Phase 1 (offline pipeline, no network surface). The contract is designed in Phase 3 (per MVP doc §6), when FastAPI + WebSocket land in sentinel. This file will be filled then.

---

## Planned shape (from MVP doc — orientation only, NOT specified yet)

### Transport
- **WebSocket** — sentinel pushes live detection events and alert state updates to ui. Single multiplexed channel; subscription model TBD.
- **REST** — ui queries alert history, posts confirm / false-positive feedback. JSON over HTTPS.

### Event categories (sentinel → ui)
- `detection` — raw per-frame detections (class, confidence, bbox).
- `alert` — alert state machine output (when threshold + persistence are met and cooldown is elapsed). Includes annotated frame URL or inline base64.
- `camera_health` — periodic health updates (FPS, last-frame-time, dropout count). Phase 5.

### REST surfaces (ui → sentinel)
- `GET /alerts?since=…` — paginated alert history. Powers the "last 10 alerts" panel.
- `POST /alerts/<id>/feedback` — body: `{ "verdict": "confirmed" | "false_positive", "notes": "..." }`. Feeds the Phase 5 training-label loop.
- `GET /healthz` — liveness.

### Schema versioning
- Wire schema is versioned at the **message** level (each event has a `schema_version` field), not the endpoint level. Lets either side upgrade independently within a compatibility window.
- Breaking changes (rename a field, remove an event type, change a type) → `[API-BREAKING]` flag + bumped `schema_version` + coordinated PR in the other repo.

---

## Design rules (apply when this file is filled)

1. **No leaking of internal state.** ui consumes a *projection* of what sentinel knows. The sentinel alert state machine, model internals, raw OpenCV frames — none of that crosses the wire. Only emitted events do.
2. **Sensor channel is invisible to ui.** When sensor integration eventually lands in sentinel, the ui sees the *same* event shape; the source (vision / sensor / both) is a tag on the event, not a separate channel. Reinforces the certifiability invariant.
3. **Field changes prefer additive.** Add new optional fields; never silently change semantics. Removal goes through a deprecation window.
4. **Annotated frames are URLs by default**, inline base64 only when the consumer has no other way to get them (Telegram). Keeps the WebSocket payload small.
