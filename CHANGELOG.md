# CHANGELOG — soul (project-wide context)

## Format
- **Append-only.** Never edit old entries. Corrections go in as new entries.
- Newest entries at the top.
- Entry header: `## [PR-XX] YYYY-MM-DD — Short title`
- Add `[API-BREAKING]` to the header if the cross-service contract (`CONTRACTS.md`) changed — this is the signal for all service repos that consume the contract.
- Soul tracks **cross-service** changes (workspace structure, contracts, project-wide ADRs, BACKLOG updates). Service-internal changes go in that service's own CHANGELOG.
- One entry per merged PR.

## Entries

## [PR-01] 2026-06-09 — Workspace context bootstrap
Project promoted from a single Python repo (`UMS-watch-keeper`) to a polyrepo workspace under `ums-watchkeeper-master/`. This repo (`soul`) is created to hold project-wide context: cross-service HLD (`ARCHITECTURE.md`), cross-service contract spec (`CONTRACTS.md`, stub), phased backlog (`BACKLOG.md`), workspace-level rules (`CLAUDE.md`), cross-cutting ADRs (`ADR.md` — workspace structure, Lite rigor, UI-in-separate-repo), and project-wide hardware notes (`HARDWARE_NOTES.md`, moved from the pre-split repo). Canonical home for the `/ums-brainstorm`, `/ums-sdlc`, `/ums-review` skills established at `.claude/skills/` — service repos symlink into this directory. The sentinel service repo (Python detection engine) is now `../ums-watchkeeper-sentinel/`; its own SDLC docs live there. The `ums-watchkeeper-ui` repo is not yet created.
