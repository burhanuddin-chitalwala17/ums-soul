# CLAUDE.md — Project-Wide Rules & SDLC

## Purpose
Single source of rules for the **ums-watchkeeper** product as a whole.
**Claude Code reads this at the start of every session when working in `soul/`.**
Each service folder has its own `CLAUDE.md` covering service-specific rules (Python style, test conventions, etc.) — this file covers what applies across all services.

## How to use this file
- Always-on **project-wide** rules live here.
- Always-on **service-specific** rules live in each service's `CLAUDE.md` (e.g. `../ums-watchkeeper-sentinel/CLAUDE.md`).
- Operational workflows (start a spec, finish a PR, log a decision) live in the project skills at `.claude/skills/`. Invoke them rather than asking Claude to remember the steps.
- When a rule and a skill say different things, the skill wins for that workflow — then update this file to match.

## Workspace layout
```
ums-watchkeeper-master/        (parent dir, no git)
├── ums-watchkeeper-soul/      ← project-wide context (you are here)
├── ums-watchkeeper-sentinel/  ← Python detection service
└── ums-watchkeeper-ui/        ← Next.js dashboard (future)
```
Each service folder is an **independent git repo** (polyrepo — see ADR-01).

---

## SDLC — Lite

> Solo personal project. Rigor is intentionally calibrated for one developer building muscle memory, not a team coordinating across PRs. (See ADR-02.)

### Cycle per spec
```
SPEC DEFINED
   ↓
SPIKE / R&D PHASE       — invoke `/ums-brainstorm <topic>`
   - Research packages, hardware behavior, known gotchas
   - Output: soul/HARDWARE_NOTES.md addition and/or proposed ADR (in the relevant repo)
   ↓
TRD WRITTEN             — invoke `/ums-sdlc spec-start SPEC-XX`
   - Technical Requirements Document for this spec
   - Lives in the service folder it primarily belongs to (sentinel/docs/specs/ or ui/docs/specs/)
   - Covers: scope, approach, packages used, environment considerations, test plan
   - Deviations during implementation logged in that service's CHANGELOG.md
   ↓
IMPLEMENTATION (fragmented PRs)
   - Each PR small, focused, independently verifiable
   - Each PR includes tests for any new logic
   - No invisible plumbing PRs — each must demo something
   ↓
PR REVIEW               — invoke `/ums-review`
   ↓
ARCHITECTURE.md UPDATED — only if structural change
   - Service-internal change → that service's ARCHITECTURE.md
   - Cross-service change → soul/ARCHITECTURE.md
   ↓
CHANGELOG.md ENTRY ADDED — always (in the relevant repo)
   ↓
MERGE                   — `/ums-sdlc pr-finish` walks the gate
```

### TRD requirement (Lite)
A TRD is **required** when a PR:
- changes a service's architecture or data flow,
- adds a new third-party package,
- starts a new spec, **or**
- changes the cross-service contract (see `CONTRACTS.md`) — flagged `[API-BREAKING]` in CHANGELOG to coordinate with the other service repo.

A TRD is **not required** for tiny PRs:
- typo / comment / log-message fix
- config tweak with no behavior change
- single-function refactor with no public-API impact
- doc-only changes

**Even tiny PRs still require:** a CHANGELOG entry (in the relevant repo) and tests for any new or changed logic.

### Golden Rules
1. **Architecture docs are source of truth, not a plan** — they reflect what currently exists. Update after, never before. (Both soul and per-service.)
2. **Changelogs are append-only** — never edit old entries. Append corrections as new entries.
3. **TRD deviations are logged in the relevant CHANGELOG.md before merging.**
4. **`/ums-review` runs before every PR is considered done.**

---

## Project Invariants (non-negotiable)

These exist because of the certifiability story and the locked-in MVP doc (v1.0). Violating any of them is a hard block, not a tradeoff.

### 1. The vision channel is advisory only
The vision pipeline **never modifies, suppresses, delays, or otherwise interacts with sensor alarms or sensor state**. It runs alongside, emits its own events, and writes to its own channels (DB, Telegram, dashboard). When sensor integration eventually lands, it lands as a **read-only** input. This is the entire reason this project is allowed near a ship — do not propose architectures that violate it.

### 2. Alert constants live in config
`DETECTION_THRESHOLD`, `PERSISTENCE_FRAMES`, `COOLDOWN_SECONDS` (MVP doc §6.3) — and any future tuning parameter — come from `sentinel/config/settings.py` (env-var overridable). Never inline a literal in service `src/` code. The dashboard "confirm / false positive" feedback loop will eventually tune these — they must be one place to change.

### 3. Scope = single machinery space, single vessel
MVP target is the HFO Purifier area on the user's brother's ship. Do not propose fleet-wide, multi-camera, multi-vessel, or edge-deployment work unless explicitly asked. Those are Phase 2+ and out of MVP scope.

### 4. UI is a separate repo
Frontend (Next.js dashboard, planned Phase 4) lives in **ums-watchkeeper-ui** — a separate polyrepo sibling. Any change in **sentinel** that affects the WebSocket or REST contract **must** be flagged `[API-BREAKING]` in the sentinel CHANGELOG so the UI repo can be bumped in lockstep. The cross-service contract is documented in `CONTRACTS.md` here in soul.

### 5. Detection classes are an enum / Literal, never magic strings
Currently the model is stock `yolo11n.pt` (COCO classes). When fine-tuned classes land (`oil_leak`, `smoke`, `normal`, optionally `motion`), they live as an `enum.Enum` or `typing.Literal` in sentinel — never as inline string comparisons.

---

## Extensibility Principles

- **External dependencies sit behind an interface** — the app core depends on a contract, not on a package. Swapping a package = writing a new implementation, nothing else changes.
  - Current state in sentinel: `Detector` and `FrameReader` are concrete classes. That's fine for Phase 1 — see sentinel/ADR-03 (defer abstract interfaces).
  - Future services (`AlertSink`, `StorageBackend`, `NotificationChannel`) must be introduced as ABCs from day one because multiple implementations are already foreseen.
- New screens / endpoints must be addable without modifying existing ones (Open/Closed).
- Business logic is fully decoupled from I/O — adding a new detection class should require only adding a class entry and tuning logic, not modifying the HTTP handler or the OpenCV loop.

---

## Development Principles

- **SOLID:**
  - *Single Responsibility* — every module / class does one thing
  - *Open/Closed* — open for extension, closed for modification
  - *Liskov Substitution* — implementations are fully substitutable for their interface
  - *Interface Segregation* — focused interfaces; don't force classes to implement what they don't need
  - *Dependency Inversion* — depend on abstractions, never on concretes or packages directly
- **DRY** — no duplicated logic; extract shared behavior into utilities
- **YAGNI** — don't build what isn't in the current spec; extensibility comes from good seams, not pre-built features
- **KISS** — simplest solution that satisfies the spec; complexity must be justified
- **Separation of Concerns** — UI knows nothing about inference; inference knows nothing about HTTP; services know nothing about each other beyond `CONTRACTS.md`

---

## Service folders

| Service | Folder | Role |
|---------|--------|------|
| **sentinel** | `../ums-watchkeeper-sentinel/` | Python detection engine — observe (cameras + later read-only sensors), judge (alert state machine), alert (DB, WebSocket, Telegram). |
| **ui** (future) | `../ums-watchkeeper-ui/` | Next.js dashboard — display live feed, alert history, confirm / false-positive feedback. |

Each service has its own `CLAUDE.md`, `ARCHITECTURE.md`, `ADR.md`, `CHANGELOG.md`, and `DEPENDENCIES.md`, scoped to that service.

---

## Skills (project-local)

| Skill | When to invoke |
|-------|---------------|
| `/ums-brainstorm <topic>` | Before writing a TRD. Research-only mode — no code. Outputs to `soul/HARDWARE_NOTES.md` or proposes an ADR (in the relevant repo). |
| `/ums-sdlc <subcommand>` | During the SDLC cycle. `spec-start <SPEC-XX>` scaffolds a TRD in the relevant service folder. `pr-finish` walks the Lite checklist. `adr-new <title>` appends a decision record. |
| `/ums-review` | Before merging a PR. Project-specific checks (sensor independence, alert constants in config, API contract flagging, doc updates). Different from the built-in `/review`. |

Skills live canonically at `soul/.claude/skills/`. Each service folder symlinks `.claude/skills` to `../ums-watchkeeper-soul/.claude/skills` so the skills are discoverable regardless of which repo you open in Claude Code.
