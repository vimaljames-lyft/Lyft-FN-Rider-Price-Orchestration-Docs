# Rider Pricing Unification — Metered Fares (Phase 2)

**Part of:** [Full Base-Price Orchestration (Phase 2)](rider-pricing-unification-phase2-tech-spec.md) — shared architecture, endpoint & graphs, cardinality, connectivity, rollout live on the hub page.
**Siblings:** [Predefined fares & surcharges](rider-pricing-unification-phase2-predefined.md) · [Upfront fares](rider-pricing-unification-phase2-upfront.md)
**Adoption order:** step 3 — the first field PC *computes* rather than fetches.

---

## Component: metered fares

Metered has a bounded calculation, making it the first field where PC computes rather than fetches. `EstimatedFareService.findMeteredFare` picks the tariff regulation by fleet from `meteredFareConfigurations` (FareConfigurationService), runs `calculatePriceByDistance(distance, basePriceInMinor, pricesPerDistance)` (base + per-distance-band accumulation), adds surcharge, and constructs `EstimatedFareCalculationResult` (provenance `TariffFareEstimation`).

**Ownership boundary:**

| Stage | Component (`class` · `method`) | Owner |
| :---- | :---- | :---- |
| Enrich | `FareCalculationRequestEnricherService` → polygons, office, distance | TFC — passed to PC; metered requires distance |
| Fetch config | metered tariff regulation (`FareConfigurationService.getMeteredConfigurations`) | PC |
| Compute | `EstimatedFareService.calculatePriceByDistance` → base fare price | PC |
| Surcharge + build | `fareSurchargeSelector` + `surchargePostProcessor.applyAdjustments` (elastic) → total | TFC until the [surcharge relay](rider-pricing-unification-phase2-predefined.md#component-surcharges) is in PC (total fuses surcharge) |
| Construct result | `EstimatedFareCalculationResult` (`TariffFareEstimation`) | TFC |
| Payment / persist / represent | payment fan-out, persistence, representation | TFC (as all components) |

The estimated slot is filled by a data-science ML prediction (`FareEstimationService`, preferred) or the metered tariff (fallback), both producing `EstimatedFareCalculationResult`. This component owns the metered tariff path; owning the DS path additionally requires threading `FareEstimationService`.

Metered reuses the surcharge relay documented on the [predefined & surcharges page](rider-pricing-unification-phase2-predefined.md#component-surcharges) — surcharge is a shared prerequisite, not metered-specific.
