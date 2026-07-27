# Specs — project-wide

Feature-based specs. **One folder per feature**, `kebab-case`, named for the feature — no numbering. Each folder holds a `spec.md` (requirements + product approach) and, once the technical approach is decided, a `trd.md` (technical decisions, no code), plus any supporting artifacts. This is the home for per-feature detail; the cross-phase status *map* is `../../BACKLOG.md`; decisions are in `../../adr/`.

## Convention (soul [ADR-05](../../adr/0005-spec-convention.md))
- **Folder per feature**, kebab-case, no numbers. One feature per folder.
- **`spec.md`** = requirements + product approach **only** — no technical approach or decisions.
- **`trd.md`** = technical approach + decisions; **no code**, only references to *existing* code to justify a decision.
- **≤100 readable lines** per spec and per TRD. *Readable content* = body prose (sentences, bullets); **excludes** front-matter, headings, tables, fenced code/diagram blocks, link-only lines, and blanks. Over 100 → **break into subfeatures**: nested subfolders each with their own `spec.md`, and a short overview `spec.md` in the parent.
- **Supporting artifacts** live in the folder, named for their purpose (`curation.md`, `brother-review.md`, …); never named `spec`/`trd`; not line-capped.
- **Cross-reference by relative path / feature name** — a spec/TRD may reference another feature's. Numbering is never used.
- **Lightweight header** on each: status · implementing service · phase · references.
- Scaffold with `/ums-sdlc spec-start <feature-name>`. A TRD is required per the Lite SDLC when a feature starts, changes architecture/data flow, adds a package, or changes the cross-service contract (see `../../CLAUDE.md`, soul ADR-02).

## Features
| Feature | What | Phase | Service | Status |
|---------|------|-------|---------|--------|
| [oil-detection-model/](oil-detection-model/) | Fine-tune the `oil` detector (label → train → evaluate → `best.pt`) | 2a | sentinel | 🚧 spec + TRD draft; curation done; blocked on brother-review |
| `oil-detection-integration` *(planned)* | Wire `best.pt` into sentinel — enum `{oil, normal}`, re-tune `CONFIDENCE_THRESHOLD` → 0.65, run on real video | 2a | sentinel | ⏳ Not started |
| `smoke-detection` *(planned)* | `smoke` class | 2b | sentinel | ⏳ Deferred (pending real smoke imagery) |

> The former `SPEC-02/03/04` numbering is retired (ADR-05). Mapping: **SPEC-02 → oil-detection-model**, **SPEC-03 → oil-detection-integration**, **SPEC-04 → smoke-detection**. See sentinel ADR-04 for the 2a/2b oil-vs-smoke scoping.
