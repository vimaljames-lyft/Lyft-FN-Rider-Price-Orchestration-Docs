# Rider Pricing Unification — Upfront Fares (Phase 2)

**Part of:** [Full Base-Price Orchestration (Phase 2)](rider-pricing-unification-phase2-tech-spec.md) — shared architecture, endpoint & graphs, cardinality, connectivity, rollout live on the hub page.
**Siblings:** [Predefined fares & surcharges](rider-pricing-unification-phase2-predefined.md) · [Metered fares](rider-pricing-unification-phase2-metered.md)
**Adoption order:** step 4 — the coupled formula, adopted last because it depends on the fields above.

---

## Component: upfront

Upfront is the coupled formula — base × surge with interleaved rounding and a two-pass surcharge on the surged price (`UpfrontFareVariantCalculator`) — so it is adopted **last**, after the fields it depends on are trusted. Discounts migrate to Lyft-native coupons rather than porting FreeNow vouchers. The minimum-fare clamp interacts with upfront (see [Open questions](rider-pricing-unification-phase2-tech-spec.md#open-questions)).

**Why last.** Upfront consumes the earlier steps: it multiplies the base (predefined/metered) by surge, rounds mid-formula, and applies surcharge in two passes over the surged price. A trusted upfront field would therefore depend on already-trusted base + surcharge fields — so upfront can only be adopted once predefined (steps 1–2) and metered (step 3) are at parity. It is the point at which PC owns the whole coupled pass rather than a single fetched or computed field.

**Coupling constraint.** Because base × surge, rounding, and the two-pass surcharge are one interleaved computation, upfront is migrated and shadowed as a **bundle**, not field-by-field — unlike the earlier fetch/fuse steps. This is the boundary at which TFC becomes a true thin wrapper for the fare.
