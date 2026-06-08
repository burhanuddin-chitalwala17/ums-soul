# Architecture Decision Records — project-wide

These ADRs cover **cross-service** decisions: workspace shape, process rigor, inter-service coordination. Service-internal ADRs (package choices, internal patterns) live in each service's own `ADR.md`.

> **Renumbering note:** before the workspace split (2026-06-09), all ADRs lived in a single `ADR.md` in the pre-split repo. They have been re-allocated by scope: project-wide ADRs to this file, service-internal ADRs to the relevant service. Each `ADR.md` is its own sequence starting at ADR-01. The original numbering is recorded inline below for traceability.

## Format
```
### ADR-XX: Title
- **Date:** YYYY-MM-DD
- **Status:** Proposed / Accepted / Superseded by ADR-YY / Deprecated
- **Context:** Why was this decision needed? What forces are at play?
- **Decision:** What was decided?
- **Alternatives considered:** What else was on the table, and why was it not chosen?
- **Consequences:** What becomes easier / harder because of this decision?
```

Append new ADRs at the bottom. Never edit an old ADR's decision — supersede with a new ADR and mark the old one `Superseded by ADR-YY`.

---

## Records

### ADR-01: Workspace structure — polyrepo under a shared parent directory
- **Date:** 2026-06-09
- **Status:** Accepted
- **Origin:** New decision recorded at the workspace split.
- **Context:** The project is growing into at least two services (sentinel + ui). The original single `UMS-watch-keeper` repo conflated project-wide context (cross-service architecture, process rules) with Python-service implementation, leaving no natural home for cross-service ADRs and no obvious anchor point for the future UI repo to reference.
- **Decision:** Adopt a **polyrepo under a shared parent directory** layout:
  - `ums-watchkeeper-master/` is a parent directory only (no `.git`).
  - `ums-watchkeeper-soul/` is an independent git repo containing project-wide context (this folder).
  - Each service (`ums-watchkeeper-sentinel`, future `ums-watchkeeper-ui`, …) is its own independent git repo as a sibling under master.
  - Skills live canonically in `soul/.claude/skills/` and are symlinked from each service's `.claude/skills`.
- **Alternatives considered:**
  - *Monorepo (single .git at master)* — simpler for solo dev, but conflates Python and Node toolchains, makes per-service CI awkward, and undermines the existing decision (now ADR-03) to keep the UI in a separate repo for focus.
  - *Single repo with subfolders, no master* — what we had. Doesn't scale to multiple services; nowhere for cross-service docs to live with their own version history.
  - *Hybrid where soul is a docs-only directory inside one of the service repos* — couples soul's lifecycle to that service's lifecycle, which is exactly what we're trying to avoid.
- **Consequences:**
  - ✅ Each service can iterate independently; clean GitHub repo boundaries.
  - ✅ Cross-service decisions have a durable home (`soul/ADR.md`).
  - ✅ The skill triad (`/ums-brainstorm`, `/ums-sdlc`, `/ums-review`) has a single canonical source, symlinked into each service.
  - ⚠️ Cross-service changes touch multiple repos as one logical change — discipline lives in the `[API-BREAKING]` CHANGELOG flag and `CONTRACTS.md`.
  - ⚠️ Symlinks are committed in the service repos; they break on Windows or if the relative layout changes. Solo Mac dev — acceptable for now.

### ADR-02: Lite SDLC rigor for solo MVP
- **Date:** 2026-05-20
- **Status:** Accepted
- **Origin:** Migrated from pre-split `UMS-watch-keeper/ADR.md` ADR-04 on 2026-06-09. Content unchanged.
- **Context:** Considered three rigor levels (full / lite / docs-only). Full rigor (Spike→TRD→PR for every change) is the right discipline for a team but is overhead for a solo developer on small changes. Docs-only invites rot.
- **Decision:** Adopt **Lite** rigor (see CLAUDE.md). TRD required only for PRs that change architecture, add a package, start a new spec, or change the cross-service contract. Tiny PRs skip TRD but still require a CHANGELOG entry and tests for any logic.
- **Alternatives considered:**
  - *Full rigor* — best discipline-building, but the TRD ceremony would slow down the typo-fix loop and discourage small frequent PRs.
  - *Docs-only* — predictably rots; tried-and-failed in many personal projects.
- **Consequences:**
  - ✅ Small PRs stay small; momentum preserved.
  - ✅ Architecturally meaningful PRs still get the TRD treatment.
  - ⚠️ The line between "tiny" and "real" is judgment-based — `/ums-sdlc pr-finish` enforces a checklist to remove the wiggle room.
  - ⚠️ If TRDs start getting skipped on real changes, escalate to Full rigor — record that as a new ADR superseding this one.

### ADR-03: UI lives in a separate polyrepo sibling
- **Date:** 2026-05-03 (decision), reframed 2026-06-09 (extracted from former ADR-03 at the workspace split)
- **Status:** Accepted
- **Origin:** Migrated from pre-split `UMS-watch-keeper/ADR.md` ADR-03 on 2026-06-09. The original ADR bundled this *workspace* decision with the *sentinel-internal* stack choice (Python+FastAPI). Only the workspace decision is recorded here; the sentinel stack choice will be recorded in `sentinel/ADR.md` when FastAPI actually lands in Phase 3.
- **Context:** Two user-facing surfaces: the engineer's phone (Telegram push) and a workstation dashboard. The dashboard is workstation-first (live RTSP video with bbox overlay), so it's best served by a web app. Question: should the UI live in the same repo as the detection engine, or in its own repo?
- **Decision:** UI lives in **`ums-watchkeeper-ui`** — a separate polyrepo sibling under master. The two services coordinate through a versioned WebSocket / REST contract documented in `soul/CONTRACTS.md`; the sentinel CHANGELOG flags `[API-BREAKING]` when that contract changes so ui can be bumped in lockstep. The choice of UI framework (Next.js vs cross-platform mobile, etc.) is recorded in the ui repo's own `ADR.md` when that repo is created.
- **Alternatives considered:**
  - *Monorepo (one repo with `backend/` + `frontend/`)* — easier for one developer, but mixes Python/Node toolchains and CI; supersedes the focus argument.
  - *UI lives inside the sentinel repo as a subfolder* — same problem as monorepo at the sentinel scope; couples UI release cadence to sentinel's.
- **Consequences:**
  - ✅ Each repo iterates independently within the contract.
  - ✅ Each repo's `/ums-review` surface stays small.
  - ⚠️ Contract drift is the main risk; the `[API-BREAKING]` discipline + `CONTRACTS.md` are the mitigations.
