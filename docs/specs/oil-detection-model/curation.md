# Data Curation Report — oil dataset (oil-detection-model)

- **Status:** Draft (data spike — feeds this feature's [trd.md](trd.md))
- **Date:** 2026-07-12
- **Dataset:** `soul/data-set/` (git-ignored) — 159 still images across 21 machinery-space folders, one ship, phone photos ~4608×2076.
- **Method:** 7 parallel curation passes, one shared rubric. Each image labelled: oil (yes/no/ambiguous), form (active_leak / pooled_spill / staining_or_soaked / cleanup_scene / none), quality, clutter, recommend (positive / negative / drop / brother-review), near-duplicate grouping.
- **Note on counts:** provisional. Reconciled from the per-image tables. Positives/negatives here are *candidate* labels — the brother-review queue below must be adjudicated before any label is final. Nothing is dropped.

---

## 1. Headline numbers

| Bucket | Count | Meaning |
|--------|-------|---------|
| **Positives (candidate)** | ~17 | Oil visible with reasonable confidence |
| **Negatives (candidate)** | ~90 | Clean/normal machinery (incl. valuable *hard* negatives — grimy but oil-free) |
| **Brother-review** | ~52 | Genuinely ambiguous — stain-vs-fresh, glossy-shaft-film, save-all deposits, cleanup scenes, dark frames |
| **Drops** | 0 | — |
| **Total** | 159 | ✓ |

**The single most important fact:** of ~17 candidate positives, **13 come from one correlated event** (the ER OIL SPILL walkthrough, one space, ~40 minutes). The rest of the ship contributes negatives + a large ambiguous pile. The whole dataset contains only **~1–2 clear *active-leak* frames**; the positive signal is overwhelmingly *pooled/soaked* morphology.

---

## 2. Per-folder breakdown

| Folder | Total | Pos | Neg | Review |
|--------|------|-----|-----|--------|
| ER OIL SPILL | 18 | 13 | 0 | 5 |
| MAIN ENGINE LOWER PLATFORM | 29 | 3 | 20 | 6 |
| ER LOWER PLATFORM | 22 | 1 | 13 | 8 |
| ER BOTTOM PLATFORM (+7 subfolders) | 30 | 0 | 17 | 13 |
| PURIFIER ROOM | 11 | 0 | 5 | 6 |
| STEERING GEAR | 10 | 0 | 8 | 2 |
| GENERATOR PLATFORM | 8 | 0 | 4 | 4 |
| ER UPPER PLATFORM | 7 | 0 | 6 | 1 |
| BOILER BURNER | 6 | 0 | 4 | 2 |
| 6 small folders (HT TANK, COMPOSITE BOILER TOP, INCINERATOR, BOILER GAUGE GLASS, SEWAGE, AFT MOORING WINCH) | 18 | 0 | 13 | 5 |
| **Total** | **159** | **~17** | **~90** | **~52** |

---

## 3. Candidate positives (~17) — need brother confirmation

**ER OIL SPILL (13)** — prop-free oil directly on machinery/bilge:
`070059, 070106, 070112, 070122` (pooled bilge / pipework) · `070134, 070144, 070155` (oil-soaked lagging) · `070208` (oil down motor flange) · `070223` (**the one clear active leak** — oil streaming down pipe) · `070228` (oil sheeting off pipe) · `070245` (runs on beam) · `070341` (glossy bilge flood) · `071535` (pooled below grating). *(IMG_20260427_ prefix.)*

**MAIN ENGINE LOWER (3):** `214843, 214851, 214924` — oil-soaked mat / staining around lubricator & HCU hydraulic blocks.

**ER LOWER (1):** `215307` — oil pooled in a DESMI pump save-all tray.

> Even these should be eyeballed by the brother, but they're the confident set. 13 of 17 are one event → see the split problem in §7.

---

## 4. Brother-review queue (~52) — the critical path

Grouped by folder for efficient triage. ★ = highest priority (most likely real oil / decision-critical). For each: does it show **fresh/active oil** (→ positive), or **old stain / water / grease / preservative film / shadow** (→ negative)?

**ER BOTTOM PLATFORM tree (13):**
- ★ TAIL SHAFT `213041`, `213050` — glossy wet film + fresh drips on shaft coupling bolts. *Best active-leak candidate in the whole dataset* — but could be normal applied preservative/running LO film. Your call is decisive.
- ★ MISCELANIOUS PUMPS `213934` — glossy black soaked pump/gearbox, drips to coaming.
- MISCELANIOUS PUMPS `213147, 213859, 213900, 213940`, `BILGE AND SLUDGE PUMPS.jpg` — dark deposits, sludge vs fresh.
- MAIN LO PUMPS `213310, 213420` — glossy film on pump shaft (likely running-oil film?).
- SW COOLING PUMPS `213841` — deck sheen (likely water).
- BWTS FILTERS `213746` — dark pool by valve (oil vs paint).

**ER LOWER PLATFORM (8):** `FO TRANSFER PUMPS.jpg`, `FWG & HT COOLER.jpg`, `FWG1.jpg`, `215311`, `ME HT PUMPS.jpg`, `ME LO AUTOBACKWASH FILTERS.jpg`, `ME PREHEATER.jpg`, `VRCS.jpg` — mostly dark deck patches / save-all trays / stained flanges.

**PURIFIER ROOM (6):** ★ `215737` (save-all tray residue), ★ `215552` (oil-soaked lagging) · `215530, 215541, 215627, 215632, 215649` (separator-base sheen / save-all). *Note: this is the MVP target space, so its "normal" state matters most — worth extra care.*

**MAIN ENGINE LOWER (6):** ★ `214938` (glossy streaks running down HCU column — active-leak candidate) · `214815, 214827, 214909, 215045, 215134` (HCU deck sheen / accumulator lagging / casing streak).

**ER OIL SPILL (5):** ★ `070333` (large oil pool, just very dark) · `063732, 063736, 071318` (**cleanup scenes** — oil is real but recognizable partly via pads/drums; decide keep-as-positive or drop to avoid teaching "pads = oil") · `070312` (mostly clean skid, dark base ambiguous).

**GENERATOR PLATFORM (4):** ★ `214128` (heavy oil-soaked lagging + deck residue — closest to positive here) · `214033, 214120, 214216`.

**STEERING GEAR (2):** `214806, 214833` — dark recessed masses (oil vs black rag/shadow).

**6 small folders (5):** INCINERATOR `213648, 213700` (sludge-tank staining) · BOILER GAUGE GLASS `213729` (dry-looking brown streak) · BOILER BURNER `213907` (base spots + oil drum), `214108` (fuel-train residue — also **rotated 90°**).

**ER UPPER PLATFORM (1):** `215418` (deck smear).

> Practical suggestion: the brother only needs to sort each into **oil / not-oil**, and for the "oil" ones note **fresh-leak vs old-stain**. That single pass converts this pile into final labels and, crucially, tells us how many positives exist *outside* the one spill event.

---

## 5. Negatives (~90)

Not enumerated (bulk clean). Distributed across every folder — see §2. These are genuinely valuable: many are **hard negatives** (grimy, greasy, rusty, dark machinery that is *not* leaking), which is exactly what teaches the model that dirty ≠ oil. Strongest clean sources: MAIN ENGINE LOWER (20), ER BOTTOM tree (17), ER LOWER (13), the 6 small folders (13), STEERING GEAR (8).

---

## 6. Near-duplicate / burst groups (must not split across train/val)

Group-aware splitting is mandatory. Known burst/same-scene groups:
- **ER OIL SPILL:** the entire `07:00–07:03` run is one correlated sequence; treat all 18 as one group.
- **MAIN ENGINE:** {214753, 214756}, {215110, 215114, 215123}, {211907, 211922, 211927}, {214815, 214827}.
- **ER LOWER:** {215159, 215200}, {215307, 215311}.
- **GENERATOR:** {214120, 214128}.
- **BOILER BURNER:** {214041, 214058} (near-identical panel — consider keeping only one).
- **PURIFIER:** {215530, 215541, 215552}, {215627, 215632}, {215737, 215741}.
- **STEERING:** {214658, 214817}, {214941, 214957}.
- **ER BOTTOM:** {213859, 213900}, {213934, 213940}, {213041, 213050}, {213310, 213420}, {213841/213844/213848}.
- **Small folders:** {HT 213230, 213237}, {COMP BOILER 213533, 213552}, {SEWAGE 214514, 214536}, {AFT WINCH 215044, 215047}, {INCINERATOR 213648, 213700}.

---

## 7. Key challenges surfaced (feed the TRD)

1. **Positives are one correlated event.** 13 of ~17 confident positives are the ER OIL SPILL walkthrough (one space, one session). A naive train/val split either starves val of positives or starves train. **Mitigation options:** (a) brother-triage promotes positives from *other* spaces (TAIL SHAFT, MAIN LO, HCU, save-all trays) to spread the positive class → makes a group-aware hold-out viable; (b) k-fold cross-validation instead of a single split; (c) frame-extract more positives from the videos (§9). Decide in the TRD **after** triage.
2. **Almost no active-leak morphology.** ~1–2 frames. If the product is sold as "leak" detection, this is thin — the model will mostly learn pooled/soaked oil. Directly informs the one-vs-two-class + naming decision (§8).
3. **Volume is low.** ~17 confident positives is proof-of-concept scale, not production. Triage + video extraction are the levers to grow it.

## 8. Data-quality issues to fix before training

- **Burnt-in date/time overlay on ~every image** (lower-left). Overfitting trap — crop the corner or ensure negatives carry the same overlay. **Required prep step.**
- **Orientation:** `BOILER BURNER/IMG_20260517_214108.jpg` is rotated 90°; normalize EXIF/orientation across the set.
- **Portrait/landscape mix** and **dark/backlit/green-cast night shots** (many low-platform frames) — normalize; consider exposure augmentation.

## 9. False-positive & spurious-correlation risks (guard in evaluation)

- **Green epoxy deck** specular reflection reads as oil sheen; **brass/bronze** fittings under low light read amber-glossy; **water** sheen vs oil.
- Negatives are dominated by the **green non-slip deck** → risk the model learns "green deck = safe." Positives are dominated by **dark bilge** → risk it learns "dark = oil." Evaluation must include off-distribution checks for both shortcuts.

## 10. Video files (7 clips, ~1.5 GB, not in the 159 — assessed 2026-07-12)

Seven `.mp4` clips, all 1920×1080, ~12 min total, sampled at 9 frames each. Findings:

| Clip | Space | Len | Content | Use |
|------|-------|-----|---------|-----|
| video_20260528_213005 / _213124 | **PURIFIER ROOM** (MVP target) | 72s / 68s | walkthrough; some save-all-tray staining | ⭐ prime integration test footage + dense-sample for brother |
| video_20260524_215108 | ER LOWER | 124s | walkthrough; one deck stain | integration test footage |
| video_20260524_215336 | MAIN ENGINE LOWER | 67s | cylinder/HCU area, grimy | integration test footage |
| video_20260524_213444 | ER BOTTOM | 313s | walkthrough, mostly clean | integration test footage (longest) |
| video_20260524_215005 | GENERATOR | 44s | clean gensets | integration test footage |
| video_20260422_153718 | (open deck) | 12s | **ships at sea from the deck** | ❌ irrelevant — drop |

**Verdict:** the spill event (0427) was never filmed; the clips are clean-machinery walkthroughs. They do **not** meaningfully grow the oil-positive class. Real value: (a) **integration "run on real video"** — the two PURIFIER clips are the actual MVP target space and are the highest-value test assets we now have (previously only a generic `test.mp4`); (b) a *minor* positive/ambiguous source if the PURIFIER + ER-LOWER clips are densely sampled (~1 fps) and reviewed by the brother. Recommend: register the 6 engine-room clips as integration test fixtures; drop the 0422 deck clip; optional dense-sample of PURIFIER/ER-LOWER for a few extra review frames.

---

## 11. Recommendations for the TRD

1. **One class, named `oil`** (not `oil_leak`). The data is overwhelmingly pooled/soaked with ~1–2 active-leak frames; a separate `oil_leak` class would have too few samples, and `oil_leak` as the single-class name misrepresents what it detects. (Confirms the earlier naming concern; supersede the class name in soul/CLAUDE.md invariant #5 wording via the TRD.)
2. **Brother triage is the linchpin — do it before finalizing the split and before training.** It converts ~52 ambiguous frames into labels and reveals how many positives exist outside the one spill event. Everything downstream depends on that number.
3. **Split strategy: decide after triage.** Default to group-aware (by space + burst group); fall back to k-fold CV if positives stay concentrated in the one event.
4. **Prep pipeline:** crop/neutralize the timestamp overlay, fix orientation, normalize portrait/landscape, downscale from 4608px to a sane `imgsz`, heavy augmentation given low volume.
5. **Grow the positives:** brother-promoted review images + frame extraction from the videos; flag whether a targeted follow-up photo request to the brother is warranted.
6. **Evaluation protocol:** held-out *space* (not random split) + a false-positive rate measured on clean/normal footage, with explicit checks against the green-deck and dark-bilge shortcuts. mAP alone is insufficient at this scale.
