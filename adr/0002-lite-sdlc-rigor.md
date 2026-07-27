# ADR-02: Lite SDLC rigor for solo MVP

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
