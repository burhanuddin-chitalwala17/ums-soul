# Oil Review Checklist — for the engineer (oil-detection-model)

**Who:** the ship's engineer (ground-truth expert).
**Why:** we're training a camera to detect oil in the engine room. Of 159 photos, ~90 are clearly clean and ~17 clearly show oil. The **~52 below are the ones we genuinely can't call** — mostly "is this a fresh/active oil presence, or just old staining / water / grease / preservative film / shadow?" Your call turns these into training labels, and tells us how many real oil examples exist *outside* the one spill event.

**How to use:** for each file, open the photo (folder path given) and mark one box. It's fast — most are yes/no.

- **[N]ot oil** — no meaningful oil (old dry stain, water sheen, grease, shadow, paint, preservative film all count as *not oil* for our purposes).
- **[O]il — fresh/active** — wet, glossy, recently leaked/dripping/pooling oil.
- **[S]tale oil** — real oil but old/dry/soaked (still useful, tagged differently).

★ = highest priority (most likely to change the outcome). If short on time, do the ★ rows first.

> Paths are under `data-set/INCINERATOR WO TANKS/`. Filenames are the phone's originals.

---

## ER BOTTOM PLATFORM tree (13)

**TAIL SHAFT/**
- [ ] ★ `IMG_20260524_213041` — glossy wet film on shaft coupling. Fresh oil, or applied preservative/running film?  → N / O / S
- [ ] ★ `IMG_20260524_213050` — wet smears + drips on coupling bolts (best active-leak candidate we have). → N / O / S

**MISCELANIOUS PUMPS/**
- [ ] ★ `IMG_20260524_213934` — glossy black soaked pump/gearbox, drips to coaming. → N / O / S
- [ ] `IMG_20260524_213147` — dark wet patch on pump base coaming. → N / O / S
- [ ] `IMG_20260524_213859` / `IMG_20260524_213900` — dark deposits around brass valve. → N / O / S
- [ ] `IMG_20260524_213940` — dark greasy mechanism. → N / O / S
- [ ] `BILGE AND SLUDGE PUMPS.jpg` — dark grime; sludge vs oil? → N / O / S

**MAIN LO PUMPS/**
- [ ] `IMG_20260524_213310` / `IMG_20260524_213420` — glossy film on pump shaft (running-oil film?). → N / O / S

**SW COOLING PUMPS/**
- [ ] `IMG_20260524_213841` — deck sheen below pump (water?). → N / O / S

**BWTS FILTERS BOTTOM PLATFORM/**
- [ ] `IMG_20260524_213746` — dark pool by valve (oil vs paint). → N / O / S

## ER LOWER PLATFORM (8)
- [ ] `FO TRANSFER PUMPS.jpg` — dark streaks on white pipe. → N / O / S
- [ ] `FWG & HT COOLER.jpg` — dark deck patch + stained flange. → N / O / S
- [ ] `FWG1.jpg` — grimy pipe joints. → N / O / S
- [ ] `IMG_20260524_215311` — dark floor between skids. → N / O / S
- [ ] `ME HT PUMPS.jpg` — dark tray front-left. → N / O / S
- [ ] `ME LO AUTOBACKWASH FILTERS.jpg` — dark patches under housings. → N / O / S
- [ ] `ME PREHEATER.jpg` — stained flange lagging. → N / O / S
- [ ] `VRCS.jpg` — dark film in save-all tray. → N / O / S

## PURIFIER ROOM (6) — *this is the MVP target space; extra care here*
- [ ] ★ `IMG_20260517_215737` — oily residue in save-all tray. → N / O / S
- [ ] ★ `IMG_20260517_215552` — oil-soaked lagging. → N / O / S
- [ ] `IMG_20260517_215530` / `_215541` / `_215627` / `_215632` / `_215649` — separator-base / save-all sheen. → N / O / S

## MAIN ENGINE LOWER PLATFORM (6)
- [ ] ★ `IMG_20260520_214938` — glossy streaks down HCU column (active-leak candidate). → N / O / S
- [ ] `IMG_20260520_214815` / `_214827` — HCU deck dark patches/sheen. → N / O / S
- [ ] `IMG_20260520_214909` — HP accumulator greasy lagging. → N / O / S
- [ ] `IMG_20260520_215045` — deck wet patch below valve. → N / O / S
- [ ] `IMG_20260520_215134` — brown streak on cylinder casing. → N / O / S

## ER OIL SPILL (5) — *these are from the real spill; mostly confirming edge cases*
- [ ] ★ `IMG_20260427_070333` — large oil pool, just very dark. → N / O / S
- [ ] `IMG_20260427_063732` / `_063736` / `_071318` — **cleanup scenes** (pads/drums). Real oil, but should we train on cleanup-staging shots? Your view: usable, or misleading? → keep / drop
- [ ] `IMG_20260427_070312` — mostly clean skid, dark base. → N / O / S

## GENERATOR PLATFORM (4)
- [ ] ★ `IMG_20260520_214128` — heavy oil-soaked lagging + deck residue. → N / O / S
- [ ] `IMG_20260520_214033` / `_214120` / `_214216` — dark smears / streaks / drips. → N / O / S

## STEERING GEAR (2)
- [ ] `IMG_20260517_214806` / `_214833` — dark recessed masses (oil vs rag/shadow). → N / O / S

## Small folders (5)
- [ ] INCINERATOR `IMG_20260517_213648` / `_213700` — sludge-tank staining. → N / O / S
- [ ] BOILER GAUGE GLASS `IMG_20260517_213729` — dry-looking brown streak. → N / O / S
- [ ] BOILER BURNER `IMG_20260517_213907` — base spots + oil drum. → N / O / S
- [ ] BOILER BURNER `IMG_20260517_214108` — fuel-train residue (image is sideways). → N / O / S

## ER UPPER PLATFORM (1)
- [ ] `IMG_20260517_215418` — deck smear. → N / O / S

---

### Two summary questions (most useful of all)
1. Of everything you marked **O (fresh/active)**, which single photo best shows *"the kind of leak that would have caught the incident early"*? (Helps us weight the model.)
2. Are there spots on this ship where oil leaks **recur** that we did *not* photograph? If so, a few targeted photos next round would help more than anything.

> Optional: I can generate visual contact sheets (thumbnail grids) of these so you can review on a phone without opening files one by one — ask and I'll produce them.
