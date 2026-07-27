# ADR-03: UI lives in a separate polyrepo sibling

- **Date:** 2026-05-03 (decision), reframed 2026-06-09 (extracted from former ADR-03 at the workspace split)
- **Status:** Accepted
- **Origin:** Migrated from pre-split `UMS-watch-keeper/ADR.md` ADR-03 on 2026-06-09. The original ADR bundled this *workspace* decision with the *sentinel-internal* stack choice (Python+FastAPI). Only the workspace decision is recorded here; the sentinel stack choice will be recorded in `sentinel/adr/` when FastAPI actually lands in Phase 3.
- **Context:** Two user-facing surfaces: the engineer's phone (Telegram push) and a workstation dashboard. The dashboard is workstation-first (live RTSP video with bbox overlay), so it's best served by a web app. Question: should the UI live in the same repo as the detection engine, or in its own repo?
- **Decision:** UI lives in **`ums-watchkeeper-ui`** — a separate polyrepo sibling under master. The two services coordinate through a versioned WebSocket / REST contract documented in `soul/CONTRACTS.md`; the sentinel CHANGELOG flags `[API-BREAKING]` when that contract changes so ui can be bumped in lockstep. The choice of UI framework (Next.js vs cross-platform mobile, etc.) is recorded in the ui repo's own `adr/` when that repo is created.
- **Alternatives considered:**
  - *Monorepo (one repo with `backend/` + `frontend/`)* — easier for one developer, but mixes Python/Node toolchains and CI; supersedes the focus argument.
  - *UI lives inside the sentinel repo as a subfolder* — same problem as monorepo at the sentinel scope; couples UI release cadence to sentinel's.
- **Consequences:**
  - ✅ Each repo iterates independently within the contract.
  - ✅ Each repo's `/ums-review` surface stays small.
  - ⚠️ Contract drift is the main risk; the `[API-BREAKING]` discipline + `CONTRACTS.md` are the mitigations.
