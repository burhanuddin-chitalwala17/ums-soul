---
name: ums-sdlc
description: Operational SDLC flows for ums-watchkeeper. Three subcommands — spec-start (scaffold a feature folder with spec.md + trd.md in soul/docs/specs/), pr-finish (walk the Lite PR checklist before merge), adr-new (add a decision-record file to soul/adr/ or the relevant service's adr/). Invoke during the SDLC cycle. Examples — "/ums-sdlc spec-start oil-detection-model", "/ums-sdlc pr-finish", "/ums-sdlc adr-new switch to asyncpg".
---

# /ums-sdlc — Operational SDLC flows

Three subcommands. Pick the one that matches the user's invocation.

## Where docs live (refresher)

- **Project-wide** (cross-service): in `soul/` — `ARCHITECTURE.md`, `adr/` (one file per ADR + `README.md` index), `docs/specs/` (one **feature folder** per feature, each with `spec.md` + `trd.md` — no numbering; soul ADR-05), `CONTRACTS.md`, `BACKLOG.md`, `HARDWARE_NOTES.md`, `CHANGELOG.md`.
- **Service-internal**: in each service folder — its own `CLAUDE.md`, `ARCHITECTURE.md`, `adr/`, `CHANGELOG.md`, `DEPENDENCIES.md`, `TESTING.md`.

When something below says "the relevant `ARCHITECTURE.md`", choose by scope.

---

## Subcommand: `spec-start <feature-name>`

Scaffolds a **feature folder** under `soul/docs/specs/` with a `spec.md` (requirements + product approach) and a `trd.md` template (technical approach/decisions). Layout convention: soul **ADR-05** — feature folders, no numbering, spec-vs-TRD split, ≤100 readable lines.

### Steps

1. **Pick the feature name.** `kebab-case`, descriptive, **no number** (e.g. `oil-detection-model`). One feature per folder. If the work is really part of an existing feature, add a **subfeature subfolder** under that feature instead of a new top-level folder.
2. **Check it doesn't already exist.** Scan `soul/docs/specs/` for a folder of that (or an obviously overlapping) name. If the feature already has a folder, extend it — don't create a second.
3. **Frame the product first.** `spec.md` is **requirements + product approach ONLY** — what the feature must do and why, from a product standpoint. No technical approach, package/tool choices, data-model or algorithm decisions — those all belong in `trd.md`.
4. **Create `soul/docs/specs/<feature-name>/spec.md`:**

```markdown
# Spec — <feature>

- **Status:** Draft / Accepted / Implemented / Superseded
- **Implementing service:** sentinel | ui | cross-service
- **Phase:** <MVP-doc phase>
- **References:** <relative links to trd.md, supporting docs, ADRs, other features>

> Requirements and product approach only. Technical approach/decisions live in trd.md.

## What this feature is
One or two lines, in product terms.

## Why
The product/user reason it exists.

## Requirements
- R1 — …

## Product acceptance
- A1 — how we know it works, from the product/user side.

## Out of scope (this feature)
What belongs to other features/phases.
```

5. **Confirm a spike exists before the TRD.** `trd.md` captures technical decisions and must draw on a spike (a `/ums-brainstorm`, a data/R&D report, or a relevant ADR). If none exists and the feature is non-trivial, push back and offer `/ums-brainstorm` first (Golden Rule: no TRD without a Spike). Then create `soul/docs/specs/<feature-name>/trd.md`:

```markdown
# TRD — <feature>

- **Status:** Draft / Accepted / Implemented / Superseded
- **Implementing service:** sentinel | ui | cross-service
- **Phase:** <MVP-doc phase>
- **References:** spec.md, the spike/brainstorm, ADRs

> Technical approach and decisions. NO code — only references to existing code (path + symbol) to justify a decision.

## Approach
The chosen approach, referencing the spike/ADR that justified it.

## Technical decisions
- D1 — decision + rationale (+ reference).

## Packages
New dependencies (cross-ref the service's DEPENDENCIES.md). "None" if none.

## Data model
Schemas / dataclasses / tables added or modified. "None" if none.

## API / contract changes
Touches soul/CONTRACTS.md? If yes, the merge PR carries [API-BREAKING] and CONTRACTS.md updates in lockstep. "None" if none.

## Test plan
The tests/evaluation that will exist when done (cross-ref the service's TESTING.md).

## Delivery (PRs)
Small PRs, each independently demoable.

## Open questions
Must be closed before Draft → Accepted.

## Invariants check (soul/CLAUDE.md)
- [ ] Sensor independence · [ ] alert constants in config · [ ] detection enum, not magic strings · [ ] in MVP scope
```

6. **Keep both ≤100 readable lines.** *Readable content* = body prose (sentences, bullets); **excludes** front-matter, headings, tables, fenced code/diagram blocks, link-only lines, and blanks. If either would exceed 100, **break the feature into subfeatures** — subfolders each with their own `spec.md`/`trd.md`, and a short overview `spec.md` in the parent.
7. **Supporting artifacts** (data reports, checklists, diagrams) go in the same folder, named for their purpose (`curation.md`, …) — never named `spec`/`trd`, and not line-capped.
8. **Confirm** feature name, implementing service, and phase with the user; offer to pre-fill the product requirements (spec) and known technical decisions (trd) from the spike. Do **not** start writing code — implementation happens in normal flow, gated by `/ums-review` and `/ums-sdlc pr-finish`.

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
| Service's `adr/` | If a service-internal architectural decision was made — offer `/ums-sdlc adr-new` (adds a new file + index row). |
| `soul/adr/` | If a cross-service / workspace-level decision was made. |
| `<TRD>` | If status changed to Implemented, update the TRD's status field. |

### Step 6 — API contract coordination

If this PR is `[API-BREAKING]`, remind the user: **"ums-watchkeeper-ui needs a coordinated bump. soul/CONTRACTS.md must be updated to describe the new schema. Note the change so the UI repo's TRD can reference it."**

### Step 7 — Green light

Summarize: which checks passed, which gates were waived (with reason), and a one-line "ready to merge" or "blocked on X".

---

## Subcommand: `adr-new <title>`

Adds a new ADR as its own file. Run when a real architectural choice was just made and needs durable recording. (ADRs are one-file-per-decision in an `adr/` folder — see soul ADR-04.)

### Steps

1. **Determine the scope.** Cross-service / workspace-level decision → `soul/adr/`. Service-internal → that service's `adr/`. If unsure, ask the user.
2. **Read the target `adr/README.md` index** to find the highest existing `ADR-XX` number. New number = highest + 1.
3. **Confirm with the user** that this rises to ADR-level. Heuristic: an ADR is warranted when the decision (a) changes how future code is organized, (b) involves picking one package/pattern over named alternatives, or (c) explicitly defers a constraint (like sentinel's ADR-03 deferring abstract interfaces). If the answer is "this is just a tactical choice," log it in the relevant TRD or as a CHANGELOG note instead, not an ADR.
4. **Create a new file** `adr/NNNN-<kebab-slug>.md` (NNNN = the new number, zero-padded to 4 digits) using the standard format: a top-level `# ADR-NN: Title` heading, then Date / Status / Context / Decision / Alternatives considered / Consequences. Do **not** edit existing ADR files.
5. **Add a row to `adr/README.md`** (ADR · title · status · date), in number order. Do not reorder or edit existing rows.
6. **Status:** default to `Accepted` if the user is recording a decision they've already made. Use `Proposed` only if they want to socialize before committing.
7. If this ADR **supersedes** an existing one, edit only the old file's `Status` line to `Superseded by ADR-XX` and add a one-line pointer (this is the *only* allowed edit to a historical ADR — a forward-pointer, not a content change); update its row in the index too.
