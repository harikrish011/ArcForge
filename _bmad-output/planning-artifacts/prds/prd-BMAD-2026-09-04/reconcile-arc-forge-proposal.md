# Reconciliation: ARC_Forge_Proposal_Parking_Spot_Finder.md vs. prd.md + addendum.md

**Scope of this pass:** per the confirmed scope decision, the Agentic SDLC process content (orchestrator, five human gates, specialist agent roles, guardrails, cost/token discipline, evaluation-criteria mapping, execution schedule, demo script) is intentionally condensed or absent — that is expected, not flagged below. This document covers only product-relevant facts, constraints, or requirements from the source that are missing from both prd.md and addendum.md.

**Overall finding:** coverage is strong. Product scope (§3, §13), the full dataset schema (§3, §12), the technical stack and error-handling behavior (§12), the map/route fallback logic (§12, Survival Plan), and the product-relevant risks (§18: map API unavailable, external API unavailable) are all faithfully carried into prd.md's FRs/NFRs/Constraints and addendum.md's Technical-How section. Tone/voice items (the "honest, narrow, not overclaiming" framing from §2 Problem Statement and the Final Pitch) are preserved in the PRD's Vision section and in the SM-C1 counter-metric, which specifically guards against badges appearing more confident than the underlying data warrants. No FRs contradict the source.

Two minor, borderline gaps were found — both low-severity scope/fidelity notes rather than missing requirements.

---

## Gap 1 — MVP visual-polish bar not carried forward

**What's missing:** The source's Hackathon Survival Plan explicitly lists "polished UI styling beyond basic usability" under **"Can be skipped"** — i.e., a deliberate, stated decision that v1 should be functionally complete but not visually refined. This scope-setting fact doesn't appear in prd.md's Non-Goals, MVP Scope (§6), or Cross-Cutting NFRs, nor in addendum.md.

**Where in source:** Hackathon Survival Plan (final section, "Can be skipped" row).

**Why it matters:** It's minor — a PM/architect could reasonably infer "basic usability only" from the MVP's narrow FR list and the one-day-build framing already excluded from the PRD by scope decision. But it's a concrete, explicit quality-bar statement that would usefully bound downstream design effort (e.g., signal to a design/architecture agent not to over-invest in visual polish for v1). Worth a one-line addition to Non-Goals or Constraints if the PM wants it preserved; not a blocking omission.

## Gap 2 — Map-unavailability fallback: two source variants, only one carried forward (addendum overstates the match)

**What's missing:** The source describes map-unavailability fallback two different ways in two places: §12 Technical Architecture says the app "falls back to the list view only" (map dropped entirely), while the Survival Plan says the map provider is "swapped for a static embedded map image with pins" (a lesser but still-visual fallback). prd.md's FR-5 and addendum.md's "Hackathon fallback ladder" bullet adopt only the §12 variant (list-only degradation) — a reasonable and defensible choice since §12 is the canonical architecture section. However, addendum.md's fallback-ladder bullet parenthetically claims this is "already reflected as a hard requirement in PRD FR-5, FR-8," which slightly overstates the match: the static-image-with-pins variant specifically is not what FR-5 implements, only named as an alternative.

**Where in source:** §12 Technical Architecture (map bullet) vs. Hackathon Survival Plan ("Can be mocked" row).

**Why it matters (borderline):** This is not a missing fact — the static-image fallback is explicitly named in addendum.md's own text — so it's more an internal-fidelity nuance than a gap. Flagging only because the addendum's parenthetical could be misread as "PRD FR-5 implements the static-image fallback" when it actually implements the simpler list-only fallback. No action needed unless the team wants the static-image variant preserved as a distinct, lower-priority fallback option.

---

## Not flagged as gaps (explicitly checked and found adequately covered)

- §12 Technical Architecture (frontend/backend/DB/maps/error-handling): fully mirrored in addendum.md's Technical-How, including exact DB field list and "no auth layer for MVP."
- §18 Risks and Mitigations, product-relevant rows ("Map API unavailable/limited," "External parking API unavailable"): captured via FR-5's degrade-to-list-only requirement and the Glossary's "Mock Dataset ... Not live or sensor-fed" definition establishing it as the primary data source, not a contingency.
- §9 Privacy/location-handling guardrail language ("not stored, not transmitted... appropriate for a hackathon prototype, explicitly not a production-grade privacy design"): carried near-verbatim into prd.md's Cross-Cutting NFRs and Constraints > Privacy.
- Executive Summary / Problem Statement "honest, narrow, not overclaiming" framing and Final Pitch's implicit warning against implying real-time accuracy: preserved in prd.md's Vision and directly operationalized as SM-C1 (counter-metric guarding against misleadingly confident badges).
- §20 Future Enhancements list (occupancy, reservations, payments, predictions, reviews, admin, fleet management): fully covered by prd.md §5 Non-Goals and §2.2 Non-Users (fleet/commercial managers named as non-users rather than a future-feature line, an equivalent framing).
