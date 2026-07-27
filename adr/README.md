# Architecture Decision Records — project-wide

These ADRs cover **cross-service** decisions: workspace shape, process rigor, inter-service coordination. Service-internal ADRs (package choices, internal patterns) live in each service's own `adr/` folder.

**One decision per file**, named `NNNN-kebab-title.md` (zero-padded to the ADR number). This folder replaced the single `ADR.md` on 2026-07-07 — see [0004](0004-adr-folder-and-spec-layout.md).

> **Renumbering note:** before the workspace split (2026-06-09), all ADRs lived in a single `ADR.md` in the pre-split repo. They were re-allocated by scope: project-wide ADRs here, service-internal ADRs to the relevant service. Each `adr/` folder is its own sequence starting at ADR-01. The original numbering is recorded inline in each record for traceability.

## Format
Each ADR file follows:
```
# ADR-NN: Title
- **Date:** YYYY-MM-DD
- **Status:** Proposed / Accepted / Superseded by ADR-YY / Deprecated
- **Context:** Why was this decision needed? What forces are at play?
- **Decision:** What was decided?
- **Alternatives considered:** What else was on the table, and why was it not chosen?
- **Consequences:** What becomes easier / harder because of this decision?
```

## How to add an ADR
- Create a **new file** `NNNN-<slug>.md`. New number = highest existing + 1. Never edit an old ADR's decision.
- Add a row to the index below.
- To change a past decision: write a new ADR, then edit only the old one's **Status** line to `Superseded by ADR-YY` and add a pointer. (`/ums-sdlc adr-new` walks this.)

## Index
| ADR | Title | Status | Date |
|-----|-------|--------|------|
| [0001](0001-workspace-polyrepo.md) | Workspace structure — polyrepo under a shared parent directory | Accepted | 2026-06-09 |
| [0002](0002-lite-sdlc-rigor.md) | Lite SDLC rigor for solo MVP | Accepted | 2026-05-20 |
| [0003](0003-ui-separate-repo.md) | UI lives in a separate polyrepo sibling | Accepted | 2026-05-03 |
| [0004](0004-adr-folder-and-spec-layout.md) | ADRs as one-file-per-decision; specs in `docs/specs/` | Accepted (spec-layout part superseded by 0005) | 2026-07-07 |
| [0005](0005-spec-convention.md) | Spec layout — feature folders, no numbering, spec-vs-TRD, ≤100 readable lines | Accepted | 2026-07-12 |
