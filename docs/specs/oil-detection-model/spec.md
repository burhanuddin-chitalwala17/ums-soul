# Spec — Oil detection (engine-room vision)

- **Status:** Draft
- **Implementing service:** sentinel
- **Phase:** 2a
- **References:** [./trd.md](trd.md) · [./curation.md](curation.md) · [./brother-review.md](brother-review.md) · sentinel ADR-04 · soul ADR-05

> Requirements and product approach only. Technical approach/decisions live in [trd.md](trd.md).

## What this feature is
The system watches the engine room and flags the **presence of oil** (leaks/spills) as an **advisory** signal, so an unmanned machinery space gets an early warning before a leak becomes a major cleanup.

## Why
Motivated by a real HFO oil-leak incident aboard the developer's brother's ship, where an undetected leak became a large, hazardous clean-up. Early visual warning is the product's reason to exist.

## Requirements
- **R1 — Detect visible oil.** Any visible oil presentation (active leak, pooled spill, oil-soaked surfaces) is surfaced as a single product concept: *"oil present."*
- **R2 — Advisory only.** The vision channel never reads, modifies, suppresses, or delays sensor alarms or sensor state. It runs alongside and emits its own signal. (Product invariant — the reason the product is allowed near a ship.)
- **R3 — Low false-alarm rate.** Normal engine-room reality — grime, grease, rust, water sheen, wet-looking painted decks — must not trigger it. A nuisance alert (especially overnight) is a product failure, not a minor annoyance.
- **R4 — Prove it in the real target environment first.** Primary target space is the HFO Purifier area on the brother's ship.

## Product acceptance
- **A1** — On held-out real footage of the target space, it flags genuine oil and stays quiet on clean scenes.
- **A2** — The engineer's verdict: *"this would have caught the incident."*

## Out of scope (this feature)
Alerting, persistence, and notification (separate features / later phases); smoke detection (separate feature, `smoke-detection/`); multi-vessel, multi-camera, and edge deployment (post-MVP).
