---
name: ums-review
description: Project-specific implementation review for ums-watchkeeper. Checks the project invariants (sensor independence, alert constants in config, detection enums), Python guidelines, file size, dependency injection, and doc-update gates. DIFFERENT from the built-in /review — this one knows the maritime-CV-specific rules. Invoke before merging any PR. Example — "/ums-review" on the current branch.
---

# /ums-review — Project-specific implementation review

You are reviewing a PR against ums-watchkeeper's project-specific invariants. This is **not** generic code review — for that, the built-in `/review` is fine. This skill checks the things that, if broken, would break the *certifiability story* or the *MVP discipline*.

## Where invariants are defined
- Project-wide invariants: `soul/CLAUDE.md` §"Project Invariants".
- Service-specific rules (Python style, test conventions): the relevant service's `CLAUDE.md`.
- Cross-service contract: `soul/CONTRACTS.md`.

## Step 0 — Determine scope

Run `git diff` against the base branch (or `git diff HEAD~1` if no base is obvious) — but **run it in the right repo**. Each service is its own git repo; `git` in the wrong working dir reports an empty diff. Identify:
- Which repo you're in (`soul`, `sentinel`, `ui`)
- Files changed
- Whether the change touches `src/`, `config/`, `tests/`, docs at the repo root, or `.claude/`
- Whether the change adds, removes, or bumps a dependency

If the diff is empty or the branch is clean, say so and stop.

---

## Step 1 — Project invariants (HARD BLOCKS)

For each violation, **block** the PR — these are non-negotiable per `soul/CLAUDE.md`.

### 1.1 Sensor independence (the certifiability story)
- Search the diff for any code path that reads, writes, or otherwise interacts with sensor state, sensor alarms, IAS, MODBUS, NMEA, or anything resembling existing-ship-system input/output.
- Phase 1–4 should produce ZERO such hits. If anything appears, block and require explicit user justification + an ADR.
- When sensor *integration* eventually lands (Phase 2+ of the post-MVP roadmap), it must be **read-only** — flag any write path immediately.

### 1.2 Alert constants live in config (sentinel-specific)
- Grep `sentinel/src/` for the literals `0.65`, `0.75`, `5`, `120` (the MVP §6.3 constants and common adjacent numbers). Any inline magic number related to thresholding, frame counts, or seconds → block and require it move to `sentinel/config/settings.py`.
- Acceptable: literals in `sentinel/config/settings.py` itself, in `sentinel/tests/` (intentional fixtures), in docstrings.

### 1.3 Detection classes as enum / Literal
- If the diff introduces or compares detection class names, the comparison must use an enum or `typing.Literal` — never a bare string equality (`if det.class_name == "oil_leak":`).
- Existing concession: `Detector.infer` returns `class_name: str` from Ultralytics; that's the boundary. The conversion to enum should happen at the first project-owned consumer.

### 1.4 MVP scope guard
- If the diff implies fleet-wide, multi-camera, multi-vessel, edge-deploy, ATEX, or Phase 2+ work — block and require an explicit user confirmation that scope has been intentionally expanded.

---

## Step 2 — Cross-service contract coordination (HARD FLAG)

If the diff touches a FastAPI route, WebSocket handler, Pydantic schema sent on the wire, or any field name/type returned to / received from another service:
- Block until the user confirms the CHANGELOG entry header (in the changing service's CHANGELOG) includes `[API-BREAKING]`.
- Block until `soul/CONTRACTS.md` has been updated to describe the new schema.
- Remind the user that **ums-watchkeeper-ui** needs a coordinated bump (or sentinel, if reviewing a ui PR).

---

## Step 3 — Service-specific guidelines (SOFT BLOCKS)

These apply within each service's own conventions. For sentinel (Python), the checks below; for ui (Next.js, future), substitute the analogous TypeScript rules.

| Check | How to verify (sentinel) |
|-------|--------------------------|
| Type hints on every public function/method | Grep the diff for `def ` without `->` in `sentinel/src/` |
| No `print()` in `sentinel/src/` (CLI status to `stderr` in `main.py` is the only allowed exception) | Grep diff for `print(` outside `main.py` |
| Dataclasses for cross-module data (not bare dicts) | Look for `dict[str, Any]` return types |
| Services are constructor-injected, not instantiated inside business logic | Look for `YOLO(...)` or `cv2.VideoCapture(...)` inside functions that aren't constructors |
| No hardcoded strings for paths / keys / table names | Look for string literals outside `sentinel/config/` that look like config |
| Max file length: 300 lines | `wc -l` on changed files in `sentinel/src/` |

Block unless the user explicitly accepts the deviation in the PR description.

---

## Step 4 — Tests (HARD BLOCK)

- Any new function in `sentinel/src/` with branching logic → must have a test.
- Pure alert logic (when present) → must be at 100% branch coverage. Block if not.
- Run `pytest` from `sentinel/` (or ask the user to). Block on any failure.

---

## Step 5 — Documentation gates (SOFT BLOCKS)

Cross-check the diff against the docs. The choice of service-level vs soul-level doc is by scope:

| If the diff … | Then … |
|---------------|--------|
| Adds / removes / restructures a module in a service's `src/` | That service's `ARCHITECTURE.md` must be updated. |
| Changes how services connect (new contract field, new event type) | `soul/ARCHITECTURE.md` AND `soul/CONTRACTS.md` must be updated. |
| Adds / removes / bumps a package | The service's `DEPENDENCIES.md` must reflect it. |
| Surfaces a hardware/camera/GPU/RTSP/environment finding | `soul/HARDWARE_NOTES.md` must have a new dated entry. |
| Makes a real service-internal architectural decision | The service's `ADR.md` must have a new entry (offer `/ums-sdlc adr-new`). |
| Makes a real cross-service / workspace decision | `soul/ADR.md` must have a new entry. |
| Implements a TRD | The TRD's `Status` field should be updated to `Implemented`. |
| Anything | The relevant `CHANGELOG.md` must have a new top-line entry. |

---

## Step 6 — Output

Produce a structured report:

```
## /ums-review report

### Hard blocks
- (list, or "None")

### Hard flags
- (list, or "None")

### Soft blocks
- (list, or "None")

### Documentation gaps
- (list, or "None")

### Verdict
READY TO MERGE  |  BLOCKED  |  READY WITH NOTED CAVEATS

### Suggested next action
- (one-line: "Move the threshold literal to sentinel/config/settings.py and re-run review",
   or "Update soul/CONTRACTS.md with the new field, then merge",
   or similar)
```

Be terse. The report should fit on one screen.
