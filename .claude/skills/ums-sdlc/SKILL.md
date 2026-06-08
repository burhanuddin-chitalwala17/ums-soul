---
name: ums-sdlc
description: Operational SDLC flows for ums-watchkeeper. Three subcommands — spec-start (scaffold a TRD in the relevant service folder), pr-finish (walk the Lite PR checklist before merge), adr-new (append a decision record to soul/ADR.md or the relevant service's ADR.md). Invoke during the SDLC cycle. Examples — "/ums-sdlc spec-start SPEC-02", "/ums-sdlc pr-finish", "/ums-sdlc adr-new switch to asyncpg".
---

# /ums-sdlc — Operational SDLC flows

Three subcommands. Pick the one that matches the user's invocation.

## Where docs live (refresher)

- **Project-wide** (cross-service): in `soul/` — `ARCHITECTURE.md`, `ADR.md`, `CONTRACTS.md`, `BACKLOG.md`, `HARDWARE_NOTES.md`, `CHANGELOG.md`.
- **Service-internal**: in each service folder — its own `CLAUDE.md`, `ARCHITECTURE.md`, `ADR.md`, `CHANGELOG.md`, `DEPENDENCIES.md`, `TESTING.md`.

When something below says "the relevant `ARCHITECTURE.md`", choose by scope.

---

## Subcommand: `spec-start <SPEC-XX> [optional title]`

Scaffolds a TRD for a new spec. Run **after** `/ums-brainstorm` has produced findings — the brainstorm output is the input to the TRD.

### Steps

1. **Identify the owning service.** Most TRDs belong to a single service (sentinel today, ui in the future). Cross-service specs are rare — if the spec genuinely spans services, place the TRD in `soul/docs/specs/` and reference it from each service's TRD index.
2. **Validate the SPEC-XX argument.** Check the owning service's `docs/specs/` (create the folder if missing) for existing TRDs to ensure the number isn't already taken. If `SPEC-01` exists, propose `SPEC-02`.
3. **Confirm there's a brainstorm to draw from.** Ask the user: "What brainstorm informed this spec?" If the answer is "none," push back — the Golden Rule is no TRD without a Spike. Offer to invoke `/ums-brainstorm` first. The exception is **trivial specs** the user explicitly waives; require an explicit waiver, don't assume.
4. **Create `<service>/docs/specs/SPEC-XX-<slug>-trd.md`** with this template:

```markdown
# SPEC-XX TRD — <title>

- **Status:** Draft / Accepted / Implemented / Superseded
- **Owning service:** sentinel | ui | cross-service (lives in soul/docs/specs)
- **Author:** burhanuddin.c
- **Created:** YYYY-MM-DD
- **Brainstorm input:** <link or HARDWARE_NOTES section or ADR-XX>
- **Phase:** <which MVP-doc phase this implements>

## 1. Scope
What this spec covers — and explicitly what it does NOT cover.

## 2. Approach
The chosen approach, with reference to the brainstorm/ADR that justified it.

## 3. Packages used
List any new dependencies. Cross-reference the service's DEPENDENCIES.md.

## 4. Environment / hardware considerations
Reference soul/HARDWARE_NOTES.md sections that apply.

## 5. Data model changes
Schemas, dataclasses, DB tables added or modified. None = say "None."

## 6. API / contract changes
Does this affect the cross-service contract documented in soul/CONTRACTS.md?
If yes, describe the wire change. Merge PR must carry [API-BREAKING] in the
changing service's CHANGELOG, and soul/CONTRACTS.md must be updated in lockstep.

## 7. Test plan
What new tests will exist when this spec is done. Cross-reference the service's TESTING.md.
- Unit tests:
- Integration tests:
- Manual verification steps:

## 8. Project invariants check
Confirm each (see soul/CLAUDE.md):
- [ ] Sensor independence preserved (vision channel does not modify sensor state)
- [ ] No alert constants hardcoded — all in sentinel/config/
- [ ] Detection classes use enum / Literal, not magic strings
- [ ] In-scope per MVP doc (single space, single vessel, no edge deploy)

## 9. PR breakdown (planned)
This spec will land as N PRs. List them with one-line scope:
- PR-YY: …
- PR-YY+1: …

## 10. Open questions
Anything not yet resolved — must be closed before status moves from Draft to Accepted.
```

5. **Confirm with the user** that the title, owning service, and phase are right. Offer to fill in sections 1–4 from the brainstorm output (if known).
6. **Do not** start writing code — implementation happens in normal flow, with `/ums-review` and `/ums-sdlc pr-finish` gating each PR.

---

## Subcommand: `pr-finish`

Walks the Lite PR checklist. Run before considering a PR ready to merge.

### Step 1 — Is this a tiny PR?

Ask explicitly: **"Is this a tiny PR?"** Tiny means ALL of:
- Single file or single-purpose change
- No architecture or data-flow impact
- No new third-party package
- No new spec started
- No change to the cross-service contract (`soul/CONTRACTS.md`)

If tiny → skip to Step 3 (no TRD check).
If not tiny → Step 2.

### Step 2 — TRD gate (non-tiny PRs only)

- Is there a TRD for the spec this PR implements? If no → block; offer `/ums-sdlc spec-start`.
- Does the implementation deviate from the TRD? If yes → require a `### TRD deviation` section in the CHANGELOG entry (Step 5).

### Step 3 — Tests gate (always)

Run the test suite (or ask the user to). Block on:
- Any test failing
- New logic added without a corresponding test
- Pure alert logic (when it lands) with <100% coverage

### Step 4 — Project-specific code review

Invoke `/ums-review` and surface its output. Block on any item it flags as a violation of a project invariant.

### Step 5 — Documentation gates

Run through each, ask the user to confirm or update. The choice between *service* doc and *soul* doc is by scope:

| File | When to update |
|------|----------------|
| Service's `CHANGELOG.md` | **Always** (for service-internal PRs). Append entry at the top; add `[API-BREAKING]` to header if the contract changed; add `### TRD deviation` subsection if relevant. |
| `soul/CHANGELOG.md` | When the PR touches cross-service concerns (workspace shape, CONTRACTS.md, soul-level docs). |
| Service's `ARCHITECTURE.md` | If service-internal module map, data flow, or state changed. |
| `soul/ARCHITECTURE.md` | If cross-service shape changed (services added/removed, contract topology changed). |
| Service's `DEPENDENCIES.md` | If a package was added, removed, or bumped a major version. |
| `soul/HARDWARE_NOTES.md` | If a new hardware/camera/GPU/RTSP/environment quirk was discovered. |
| `soul/CONTRACTS.md` | If the WebSocket / REST contract between sentinel and ui changed. |
| Service's `ADR.md` | If a service-internal architectural decision was made — offer `/ums-sdlc adr-new`. |
| `soul/ADR.md` | If a cross-service / workspace-level decision was made. |
| `<TRD>` | If status changed to Implemented, update the TRD's status field. |

### Step 6 — API contract coordination

If this PR is `[API-BREAKING]`, remind the user: **"ums-watchkeeper-ui needs a coordinated bump. soul/CONTRACTS.md must be updated to describe the new schema. Note the change so the UI repo's TRD can reference it."**

### Step 7 — Green light

Summarize: which checks passed, which gates were waived (with reason), and a one-line "ready to merge" or "blocked on X".

---

## Subcommand: `adr-new <title>`

Appends a new ADR entry. Run when a real architectural choice was just made and needs durable recording.

### Steps

1. **Determine the scope.** Cross-service / workspace-level decision → `soul/ADR.md`. Service-internal → that service's `ADR.md`. If unsure, ask the user.
2. **Read the target `ADR.md`** to find the highest existing `ADR-XX` number. New number = highest + 1.
3. **Confirm with the user** that this rises to ADR-level. Heuristic: an ADR is warranted when the decision (a) changes how future code is organized, (b) involves picking one package/pattern over named alternatives, or (c) explicitly defers a constraint (like sentinel's ADR-03 deferring abstract interfaces). If the answer is "this is just a tactical choice," log it in the relevant TRD or as a CHANGELOG note instead, not an ADR.
4. **Append** at the bottom of the chosen `ADR.md` using the standard format (Date / Status: Accepted / Context / Decision / Alternatives considered / Consequences). Do not reorder existing entries.
5. **Status:** default to `Accepted` if the user is recording a decision they've already made. Use `Proposed` only if they want to socialize before committing.
6. If this ADR **supersedes** an existing one, edit the old one's `Status` line to `Superseded by ADR-XX` (this is the *only* allowed edit to a historical ADR — it's a forward-pointer, not a content change).
