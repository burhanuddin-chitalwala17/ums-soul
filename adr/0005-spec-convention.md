# ADR-05: Spec layout — feature folders, no numbering, spec-vs-TRD, ≤100 readable lines

- **Date:** 2026-07-12
- **Status:** Accepted
- **Supersedes:** the **spec-layout portion** of ADR-04 (the "numbered flat files, one global SPEC sequence" part). ADR-04's *ADR-folder* decision is unaffected and stands.
- **Context:** Specs began as numbered flat files (`SPEC-02-*.md`) per ADR-04. In practice: (a) numbering a feature's requirements is meaningless and caused confusion (the "SPEC-01 phantom"); (b) one file mixing product requirements with technical approach, growing unbounded, loses review quality; (c) supporting artifacts (data reports, review checklists) had no natural home.
- **Decision:**
  1. **Feature folders, no numbers.** Each feature gets a `kebab-case` folder under `soul/docs/specs/`, named for the feature. **One feature per folder.**
  2. **Spec vs TRD separation.** `spec.md` = **requirements + product approach only** — what the feature must do and why, from a product standpoint; **no** technical approach or decisions. `trd.md` = **technical approach + decisions** — and **no code**: at most a few references to *existing* code to justify a decision, never the code we intend to write.
  3. **≤100 readable lines** per spec (and per TRD, by the same review-quality logic). **"Readable content"** = body prose lines (sentences, bullets); **excluded from the count:** YAML front-matter, Markdown headings, table rows/separators, fenced code/diagram blocks, link-reference-only lines, and blank lines. If a spec needs more, **break the feature into subfeatures**: nested subfolders each with their own `spec.md`, and a short overview `spec.md` in the parent.
  4. **Supporting artifacts live in the feature folder**, named for their purpose (`curation.md`, `brother-review.md`, …) — never named `spec`/`trd` — and are **not** subject to the line cap.
  5. **Cross-reference by relative path / feature name.** A spec or TRD may reference another feature's spec/TRD. Numbering is never used for reference.
  6. **Lightweight header** on every spec/TRD: `status · implementing service · phase · references`. The index keys on **feature name**.
- **Alternatives considered:**
  - *Keep numbered flat files (ADR-04)* — rejected: numbering is meaningless for feature requirements, and mixing product + technical in one growing file kills reviewability.
  - *One combined spec+TRD doc per feature* — rejected: the product/technical split keeps each short and lets product requirements be reviewed independently of technical choices.
  - *Cap only the spec, leave TRDs unbounded* — rejected for now: the review-quality rationale applies equally; extending the cap to TRDs is flagged as an extension of the stated rule, revisit if TRDs prove to need more room.
- **Consequences:**
  - ✅ Self-describing folders; supporting artifacts co-located; numbering confusion gone (resolves the SPEC-01 phantom).
  - ✅ Short, single-concern docs stay reviewable; product requirements are validated without wading through technical detail.
  - ✅ Feature/subfeature nesting scales cleanly.
  - ⚠️ Prior `SPEC-02/03/04` references are de-numbered. Live docs remapped to feature names; append-only CHANGELOG history left intact, with the mapping recorded here: **SPEC-02 → `oil-detection-model/`**, **SPEC-03 → `oil-detection-integration/`** (planned), **SPEC-04 → `smoke-detection/`** (planned, deferred).
