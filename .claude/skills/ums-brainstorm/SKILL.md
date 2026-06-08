---
name: ums-brainstorm
description: Spike / R&D mode for ums-watchkeeper. Invoke BEFORE writing a TRD when scoping a new spec, evaluating a package, or investigating a hardware/environment quirk. Surveys options, weighs tradeoffs, outputs to soul/HARDWARE_NOTES.md or proposes an ADR (in soul or in the relevant service folder, depending on scope). Writes no code — research and recommendation only. Examples — "/ums-brainstorm RTSP reconnect strategy", "/ums-brainstorm should we add a state machine for alert persistence", "/ums-brainstorm why is yolo11n slower on mps than expected".
---

# /ums-brainstorm — Spike / R&D phase

You are entering **R&D mode** for the ums-watchkeeper project. This skill runs **before** any TRD is written for a topic.

## Where docs live (project layout)

```
ums-watchkeeper-master/
├── ums-watchkeeper-soul/        ← project-wide context
│   • CLAUDE.md, ARCHITECTURE.md, ADR.md, HARDWARE_NOTES.md, CONTRACTS.md, BACKLOG.md
├── ums-watchkeeper-sentinel/    ← Python detection service
│   • CLAUDE.md, ARCHITECTURE.md, ADR.md, DEPENDENCIES.md, TESTING.md, src/, config/, tests/
└── ums-watchkeeper-ui/          ← Next.js dashboard (future, when scoped)
```

Cross-service / workspace-level concerns → look in **soul**.
Service-internal concerns → look in the relevant service folder.

## Hard constraints

1. **No code changes.** Do not edit any file under any service's `src/`, `config/`, or `tests/`. Reading is fine. You may *propose* code in your response, but writing it belongs in implementation, not here.
2. **No new dependencies installed.** You may research a package and recommend it, but do not run `pip install`, `npm install`, or modify any `requirements.txt` / `package.json`.
3. **Stay inside MVP scope** as defined in `soul/CLAUDE.md` and the MVP doc. If the user's brainstorm topic crosses into out-of-scope territory (fleet-wide, multi-camera, edge deployment, sensor integration, ATEX certification), flag it explicitly and ask whether to continue or scope down — do not silently expand scope.
4. **Respect the project invariants** in `soul/CLAUDE.md` (vision is advisory only, alert constants from config, detection classes as enums, separate UI repo). Any recommendation that conflicts with these must call out the conflict and either resolve it or flag it as a question for the user.

## Process

### Step 1 — Frame the question
Restate the brainstorm topic in one sentence so it's unambiguous. If the topic is too broad ("brainstorm Phase 3"), narrow it with the user before continuing — a brainstorm with no edges produces a wandering output.

### Step 2 — Check what's already known
Read **before** researching. Pick the right docs based on scope:

| If the topic is … | Read these |
|-------------------|-----------|
| Cross-service or workspace-level | `soul/ARCHITECTURE.md`, `soul/ADR.md`, `soul/CONTRACTS.md`, `soul/BACKLOG.md` |
| Service-internal (e.g. sentinel inference, FastAPI routes) | The service's `ARCHITECTURE.md`, `ADR.md`, `DEPENDENCIES.md`, and the relevant `src/` modules |
| Hardware / camera / GPU / engine-room environment | `soul/HARDWARE_NOTES.md` (always, since this file is project-wide) |
| Package choice | `soul/ADR.md` AND the relevant service's `ADR.md` AND `DEPENDENCIES.md` |

State what you found before proposing anything new. This avoids reinventing or contradicting prior decisions.

### Step 3 — Survey options
Present **2 or 3** distinct approaches. More than 3 dilutes the comparison; only 1 isn't a comparison.

For each, give:
- **Approach name** — one line.
- **How it works** — 2–4 sentences.
- **Pros** — bullet list, project-specific (not generic).
- **Cons** — bullet list, project-specific.
- **What this means for the certifiability story** — does it preserve sensor independence? Does it require any change to the alert-constants discipline?

Tradeoffs must be concrete. "Faster" is not a tradeoff; "~3x faster on a 4-min clip per a 2024 Ultralytics blog post" is.

### Step 4 — Recommendation
Pick one. Say why in one paragraph. If the recommendation is "wait — we don't have enough information yet," that's a valid output; list what would have to be true to decide.

### Step 5 — Output destination
The brainstorm must land somewhere durable. Pick the right one:

| Output goes to | When |
|----------------|------|
| **`soul/HARDWARE_NOTES.md`** (new dated entry) | The finding is about hardware, camera, GPU, RTSP, or engine-room environment behavior. |
| **A proposed new ADR-XX in `soul/ADR.md`** | A *cross-service* / *workspace-level* decision was made (workspace shape, process rigor, contract approach). |
| **A proposed new ADR-XX in the relevant service's `ADR.md`** | A *service-internal* decision was made (package choice, internal pattern). |
| **A short note in the response only** | Pure exploration that didn't reach a decision; no durable artifact needed yet. |

If proposing an ADR, write it in the standard format from the relevant `ADR.md` (Date / Status / Context / Decision / Alternatives / Consequences) and ask the user to confirm before you append. Set status to **Proposed** until they confirm — then **Accepted**.

### Step 6 — Handoff to TRD
End the brainstorm with a one-liner: **"Next step: `/ums-sdlc spec-start SPEC-XX` to write the TRD using this brainstorm as the input."** (If the brainstorm concluded "we shouldn't do this," say that explicitly instead.)

## Response shape

Keep it skimmable. Use headers for each step. The full brainstorm shouldn't be more than ~500–700 words unless the topic is genuinely deep — depth comes from concrete claims, not paragraph length.
