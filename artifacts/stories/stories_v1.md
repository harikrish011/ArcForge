---
stage: stories
version: v1
status: APPROVED
agent: user-story-agent
approver: lekshmi.lelithambika@experionglobal.com
timestamp: 2026-09-04T00:00:00
supersedes: none
---

# Stories: Parking Spot Finder

## Sources

- **Planning backlog (primary — defines exactly which stories to produce):** `artifacts/planning/backlog_v1.md` (frontmatter `status: APPROVED`). `[RESOLVED]` This artifact originally flagged a self-contradiction (frontmatter said `APPROVED`, closing line said `DRAFT`) — fixed at the source; the backlog's closing line now reads `APPROVED` consistently.
- **Requirements/PRD (personas, Glossary, FR detail):** `_bmad-output/planning-artifacts/prds/prd-BMAD-2026-09-04/prd.md` (`status: final`).
- **Design (screen-level precondition/postcondition detail):** `artifacts/design/design_v1.md` (`status: APPROVED`).

Personas used throughout (never a generic "user," except D1 by deliberate, resolved decision — see below): **Ritu** (UJ-1 — grants location, finds a Free spot, routes to it) and **Arjun** (UJ-2 — declines location, uses Manual Area Entry, hits a zero-result state, expands radius, finds a Paid spot). D1 (free-text filter) is framed as "a driver browsing results," not tied to either named persona — `[RESOLVED]` this corrects an original conflict between the backlog's Epic D header ("realizes UJ-1") and PRD FR-9's own text ("not exercised by either named journey"); the PRD's framing was confirmed correct and the backlog header fixed to match. Glossary terms (Parking Spot, Free/Paid Classification / `parkingType`, Radius, Mock Dataset, Active Location, Manual Area Entry, Spot Detail Panel, Route Handoff) are used exactly as defined in PRD §3.

Two feasibility flags the backlog carries forward as **still open** (not Gate-2-resolved) apply across multiple stories below and are noted inline wherever they affect Acceptance Criteria: the accessibility/safety best-effort framing, and A2's curated-list-vs-free-text assumption.

Order below matches the backlog's Priority Split: Must-have A1, A2, B1, B2, B3, B4, C1, C2, then Nice-to-have D1, D2.

---

## A1. Epic A — Location & Area Selection: Capture current location via browser geolocation

| Field | Content |
|---|---|
| **Story Title** | `Epic A — Location & Area Selection: Capture current location via browser geolocation` |
| **User Story** | As Ritu, I want to grant browser location permission so the app captures my current coordinates as my Active Location, so that I can immediately see nearby Parking Spots without manually specifying where I am. |
| **Precondition** | The app has loaded and no Active Location has been set yet for this session. The user's browser supports the geolocation API. |
| **Acceptance Criteria** | 1. On app load (or via an explicit "Use my location" action), the system requests browser location permission.<br>2. When permission is granted, the captured coordinates become the Active Location for the session.<br>3. The captured Active Location is held only for the current session — never persisted across sessions and never transmitted beyond what the nearby-parking query requires.<br>4. When permission is denied or unavailable, the system proceeds to Manual Area Entry (A2) without displaying a blocking error state.<br>5. Once the Active Location is set, the system proceeds to the nearby-parking results. |
| **Edge Cases** | 1. Permission prompt is dismissed without an explicit grant or deny (left pending) — the system does not hang indefinitely and offers a path to Manual Area Entry.<br>2. Geolocation API is unsupported by the browser entirely — treated the same as denial, routing to Manual Area Entry.<br>3. Permission is granted but the browser returns a location error (e.g., signal unavailable) — treated the same as denial, routing to Manual Area Entry. |
| **Post Condition** | An Active Location (browser-captured coordinates) exists for the session, or the flow has routed to Manual Area Entry (A2) without error. |
| **Validation** | Captured coordinates must be valid latitude/longitude pairs before being used as the Active Location; session-only handling (no persistence) applies on every path through this story. `[confidence: high — directly derived from PRD FR-1 consequences]` |

---

## A2. Epic A — Location & Area Selection: Manual area entry fallback (curated preset list)

| Field | Content |
|---|---|
| **Story Title** | `Epic A — Location & Area Selection: Manual area entry fallback (curated preset list)` |
| **User Story** | As Arjun, I want to manually select an area from a curated list when I decline the location permission prompt, so that I can still see nearby Parking Spots without granting browser location access. |
| **Precondition** | Browser location permission was denied or is unavailable (from A1), and no Active Location has been set yet for this session. |
| **Acceptance Criteria** | 1. Immediately on denial/failure of location permission, the system presents a curated, selectable list of named areas covered by the Mock Dataset — not a dead-end screen. `[confidence: low — inherits the PRD's own low-confidence assumption that Manual Area Entry uses a curated preset list rather than free-text geocoding; still open, not resolved at Gate 2]`<br>2. Selecting an area from the list sets that area as the Active Location for the session.<br>3. Once set, a manually-selected Active Location behaves identically to a captured one for all downstream nearby-parking behavior.<br>4. The system proceeds to the nearby-parking results after an area is selected. |
| **Edge Cases** | 1. User reaches this screen with no prior interaction (e.g., routed here directly after a second consecutive zero-result state per B4) — the same curated list is presented, behaving identically regardless of entry path.<br>2. Every area offered in the curated list corresponds to an area actually covered by the Mock Dataset — there is no selectable area that would trivially return zero results purely by construction (ties to backlog Dependency #2). |
| **Post Condition** | A manually-selected Active Location exists for the session, equivalent in downstream behavior to a captured one. |
| **Validation** | The curated area list must stay in agreement with the Mock Dataset's actual area/neighborhood coverage (backlog Dependency #2) — an area offered here that the dataset doesn't cover would silently break the flow. `[confidence: medium — reasonable inference from the backlog's explicit dependency note, not separately restated as a testable rule in the PRD]` |
| **Flag** | `[carried-forward, still-open — not Gate-2-resolved]` This story's core mechanism (curated list vs. free-text) rests on a PRD-level `[ASSUMPTION · confidence: low]`. If a reviewer overturns it in favor of free-text geocoding, this story's Acceptance Criteria and effort estimate both change materially (per backlog: S/M → at least M), and the Manual Area Entry screen in the design artifact would need a follow-up revision. Not resolved here — flagged, not silently fixed. |

---

## B1. Epic B — Nearby Parking Discovery: `/parking/nearby` query endpoint + Mock Dataset

| Field | Content |
|---|---|
| **Story Title** | `Epic B — Nearby Parking Discovery: /parking/nearby query endpoint + Mock Dataset` |
| **User Story** | As Ritu, I want the system to return the Parking Spots within my Radius of my Active Location — each with its Free/Paid Classification, address, price info, and operating hours — so that I can see a complete, trustworthy set of nearby options. `[confidence: high — this capability is shared infrastructure: it equally underlies Arjun's UJ-2 path, since both journeys' Active Location (captured or manually-selected) feeds the identical query per backlog Dependencies §1, §3]` |
| **Precondition** | An Active Location exists for the session (from A1 or A2). The Mock Dataset (~12-15 hand-curated rows, Gate 2 decision) is seeded and available to query. |
| **Acceptance Criteria** | 1. Given an Active Location and a Radius of 1km or 2km, the system returns every Parking Spot in the Mock Dataset whose distance from the Active Location (computed via the haversine formula, per PRD FR-3) is within that Radius.<br>2. A Radius value other than 1 or 2 (km) is rejected rather than silently defaulted.<br>3. A latitude outside [-90, 90] or a longitude outside [-180, 180] is treated as malformed and rejected rather than silently processed.<br>4. Each returned Parking Spot includes id, name, coordinates, address, Free/Paid Classification (`parkingType`), price info, and operating hours.<br>5. `parkingType` is present and non-empty on every returned spot — no "unclassified" spot exists in the Mock Dataset.<br>6. The query is served only over HTTPS.<br>7. The Mock Dataset contains approximately 12-15 hand-curated Parking Spots, covering one demo area with a mix of Free and Paid spots, plus at least one area that returns zero results at both 1km and 2km (to exercise B4). |
| **Edge Cases** | 1. Active Location coordinates are valid but no Mock Dataset spot falls within the requested Radius — an empty result set is returned (not an error), to be handled by B4.<br>2. A Radius value the system doesn't recognize (e.g., 0, 3, or a non-numeric value) is requested — rejected, not silently coerced to 1 or 2.<br>3. Latitude/longitude are syntactically well-formed numbers but outside the valid geographic bounds — rejected as malformed, distinct from a merely-empty result. |
| **Post Condition** | A response containing zero or more matching Parking Spots (each with the full field set) has been returned for the given Active Location and Radius, or the request was rejected as malformed. |
| **Validation** | Radius accepted only as exactly 1 or 2 (km) — no other values, no default substitution. Latitude ∈ [-90, 90], longitude ∈ [-180, 180] — enforced, not merely documented. `parkingType` is required/non-nullable on every dataset row. `[confidence: medium — the parkingType non-nullability rule is itself the PRD's own §9 medium-confidence inference, not an explicitly stated hard data rule]` |

---

## B2. Epic B — Nearby Parking Discovery: List view with Free/Paid classification

| Field | Content |
|---|---|
| **Story Title** | `Epic B — Nearby Parking Discovery: List view with Free/Paid classification` |
| **User Story** | As Ritu, I want to see nearby Parking Spots as a list — each tagged Free or Paid with its distance, address, and operating hours — so that I can quickly notice a Free spot near my destination without opening each one individually. |
| **Precondition** | B1 has returned a non-empty set of Parking Spots for the current Active Location and Radius. |
| **Acceptance Criteria** | 1. Every returned Parking Spot appears as a list item showing at minimum: name, Free/Paid Classification, distance, address, and operating hours.<br>2. The Free/Paid Classification is distinguishable by more than color alone (a text label accompanies the color).<br>3. Distance is shown rounded to the nearest 0.1km. `[confidence: medium — PRD's own ASSUMPTION, §9]`<br>4. The list is sorted by distance ascending by default.<br>5. A radius indicator is always visible on this default (non-empty) results view, naming the currently active Radius. `[confidence: high — Gate 2 decision; extends slightly beyond FR-6's literal zero-result-only wording, a deliberate traced extension per backlog note]`<br>6. The data-provenance disclosure string is always visible on this view.<br>7. Selecting a list item opens the Spot Detail Panel (C1) for that spot. |
| **Edge Cases** | 1. A Parking Spot is missing an optional field (e.g., no operating hours listed) — the list item omits that field rather than showing a blank placeholder.<br>2. Two spots are equidistant from the Active Location — both appear, in a stable (not flickering/reordering) relative order. |
| **Post Condition** | The list view is rendered with all returned spots visible, sorted by distance, radius indicator and disclosure present. |
| **Validation** | Distance rounding to nearest 0.1km; sort key is distance ascending only (no other default). `[confidence: medium — both are the PRD's own §9 ASSUMPTION items, not hard-stated rules]` |
| **Flag** | `[carried-forward, still-open — not Gate-2-resolved]` Full keyboard operability of this list→detail→route path is a hard NFR per PRD §10, but the backlog's Feasibility Flags note this as "best-effort-within-Must-have, not guaranteed" for a one-day build. Not silently assumed as fully conformant here. |

---

## B3. Epic B — Nearby Parking Discovery: Map view with Free/Paid markers + list-only degradation

| Field | Content |
|---|---|
| **Story Title** | `Epic B — Nearby Parking Discovery: Map view with Free/Paid markers + list-only degradation` |
| **User Story** | As Ritu, I want to see nearby Parking Spots as markers on a map, differentiated Free/Paid, so that I can spot a nearby free option visually — and if the map fails to load, I want to still be able to complete my search via the list, so that a technical failure never blocks me from finding a spot. |
| **Precondition** | B1 has returned a non-empty set of Parking Spots for the current Active Location and Radius. The Leaflet + OpenStreetMap map provider is the selected provider (Gate 2 decision). |
| **Acceptance Criteria** | 1. Every returned Parking Spot appears as a marker on the map at its coordinates.<br>2. Markers are visually differentiated Free/Paid by more than color alone (shape/icon differentiation, not color-only).<br>3. Selecting a marker opens the same Spot Detail Panel (C1) that selecting the equivalent list item (B2) would open.<br>4. If the map provider fails to load, the view degrades to the list-only view (B2) with the same underlying result set — this degradation is a hard requirement, not best-effort.<br>5. The radius indicator and data-provenance disclosure are visible on the map view exactly as on the list view.<br>6. The list/map presentation is a toggle (not a split-screen) — selecting the map toggles away from, not alongside, the list. `[confidence: high — Gate 2 decision]` |
| **Edge Cases** | 1. The map loads successfully but a marker's coordinates fall outside the visible map bounds at default zoom — the map either auto-fits bounds to show all markers, or the marker remains reachable via the list toggle.<br>2. The map fails to load partway through rendering (not immediately on load) — the degradation to list-only still occurs rather than leaving a partially-rendered, broken map visible.<br>3. Only one Parking Spot is returned — a single marker still renders correctly (not a degenerate/empty map). |
| **Post Condition** | Either a functioning map with differentiated markers is shown, or the view has degraded to list-only with the identical result set and no broken/partial map artifact left visible. |
| **Validation** | Degradation-to-list-only must trigger on any map provider load failure, not only a specific failure type — this is a hard requirement per PRD §10 Availability/degradation, not best-effort. `[confidence: high — directly derived from PRD FR-5 feature-specific NFR]` |
| **Flag** | `[RESOLVED]` Kept as one story per human decision — not split into B3a/B3b. This remains the backlog's designated highest-effort Must-have and descope-first pressure point (M/L); if the day runs behind, ship list-only and defer the map pane rather than the story being formally split now. |

---

## B4. Epic B — Nearby Parking Discovery: Zero-result radius messaging + one-tap 2km expansion

| Field | Content |
|---|---|
| **Story Title** | `Epic B — Nearby Parking Discovery: Zero-result radius messaging + one-tap 2km expansion` |
| **User Story** | As Arjun, I want to see a clear message and a one-tap option to expand my search to 2km when my initial 1km search returns no Parking Spots, so that I'm never left staring at a blank screen with no next step. |
| **Precondition** | B1 has returned zero Parking Spots for the current Active Location at the current Radius. |
| **Acceptance Criteria** | 1. When the 1km query returns zero spots, the system displays a message naming the searched Radius (1km) and offers a single one-tap action to expand the search to 2km.<br>2. Selecting the expansion action re-queries at 2km and, if spots are found, transitions to the Home Results view (B2/B3) for that Radius.<br>3. If the 2km query also returns zero spots, the message updates to suggest setting a different Active Location, rather than offering any further expansion.<br>4. The system never renders a bare empty list/map with no explanatory message, at either Radius. |
| **Edge Cases** | 1. The 1km query returns zero spots but the 2km query returns exactly one spot — the single-spot case renders correctly in Home Results, not as a degenerate/edge display.<br>2. User triggers the 2km expansion, then changes their Active Location (e.g., via Manual Area Entry) — the Radius resets to 1km for the new Active Location rather than silently carrying the expanded 2km forward. |
| **Post Condition** | The user sees either a successful (non-empty) result set at 1km or 2km, or a "try a different area" message with no further expansion offered, after both radii have returned zero. |
| **Validation** | Expansion is offered exactly once (1km → 2km), never a second time after 2km also returns zero. `[confidence: high — directly derived from PRD FR-6 consequences]` |

---

## C1. Epic C — Parking Detail & Route Handoff: Spot Detail Panel

| Field | Content |
|---|---|
| **Story Title** | `Epic C — Parking Detail & Route Handoff: Spot Detail Panel` |
| **User Story** | As Arjun, I want to open a Spot Detail Panel showing a Parking Spot's full name, address, price info, and operating hours, so that I can see exactly what a Paid spot will cost before deciding it's worth committing to. |
| **Precondition** | A Home Results view (B2 or B3) is showing at least one Parking Spot, reached via either a captured (A1) or manually-selected (A2) Active Location. |
| **Acceptance Criteria** | 1. Selecting a list item (B2) or a map marker (B3) opens the Spot Detail Panel for that spot in a single action.<br>2. The panel shows the spot's full name, address, Free/Paid Classification, price info (when the spot is Paid), and operating hours.<br>3. Any field absent from the Mock Dataset record for that spot (e.g., no listed price info on a Free spot) is omitted from the panel entirely — never shown as a blank placeholder.<br>4. Free/Paid Classification is always present on the panel — it is a required field and is never subject to the omission rule.<br>5. The same data-provenance disclosure string shown on Home Results is also always visible on this panel.<br>6. A dismiss action returns the user to whichever Home Results view (list or map) the panel was opened from. |
| **Edge Cases** | 1. A Free spot has no price info in the Mock Dataset — the price field is omitted entirely (not shown as "$0" or "Free" placeholder text) since price info doesn't apply to Free spots.<br>2. The panel is opened, then the underlying Active Location or Radius changes behind it (e.g., via a stray navigation) — the panel either closes or continues to reflect the spot it was opened for, without showing mismatched/stale data silently. |
| **Post Condition** | The Spot Detail Panel is visible with the selected spot's available fields, or has been dismissed back to the originating Home Results view. |
| **Validation** | Missing optional fields are omitted, never rendered blank; `parkingType` is exempt from omission since it's required/non-nullable at the data layer. `[confidence: medium — the non-nullability of parkingType is itself the PRD's own §9 medium-confidence inference]` |
| **Flag** | `[carried-forward, still-open — not Gate-2-resolved]` Keyboard operability of reaching this panel via the list path is a hard NFR per PRD §10; via the map path it's explicitly best-effort only per the same NFR. |

---

## C2. Epic C — Parking Detail & Route Handoff: Route handoff to external map provider

| Field | Content |
|---|---|
| **Story Title** | `Epic C — Parking Detail & Route Handoff: Route handoff to external map provider` |
| **User Story** | As Ritu, I want to select Route from the Spot Detail Panel and be handed off to an external map provider with turn-by-turn directions already pointed at my chosen spot, so that I don't have to search for the destination separately. |
| **Precondition** | The Spot Detail Panel (C1) is open for a selected Parking Spot. |
| **Acceptance Criteria** | 1. Selecting Route hands off to an external map provider with the selected Parking Spot's coordinates/address set as the destination only.<br>2. The user's Active Location is never passed to the external map provider as a routing origin — origin resolution, if any, is left entirely to the external provider.<br>3. Route Handoff is reachable in no more than 2 taps/selections from Home. `[confidence: medium — carried forward from PRD §11 Safety guardrail, itself tagged ASSUMPTION given driving-adjacent use context, and PRD-flagged as possibly infeasible to fully realize in a one-day build]`<br>4. If the primary map provider integration is unavailable, Route Handoff falls back to a plain external maps link rather than disappearing or erroring silently. |
| **Edge Cases** | 1. Selected spot has coordinates but no verified street address on record — the handoff still proceeds using coordinates as the destination.<br>2. Primary map provider integration is unavailable at the moment Route is selected (not just at app load) — the fallback link still triggers correctly rather than the action silently failing. |
| **Post Condition** | The user has been handed off to an external map provider (or its fallback link) with the selected spot set as the destination, and no Active Location data was transmitted as an origin. |
| **Validation** | Destination-only handoff — Active Location must never appear as an origin parameter in the handoff, under any provider/fallback path. `[confidence: high — directly derived from PRD FR-8 Privacy NFR]` |
| **Flag** | `[question back to Gate 2 / Architecture]` The backlog's own C2 note flags that Leaflet (the Gate 2-selected map display library for B3) is not itself a routing/directions service — it has no native "open turn-by-turn directions" deep link. The backlog proposes collapsing "primary" and "fallback" into a single `maps.google.com`-link target as the simplest option, but leaves this open for Architecture to confirm. AC 1 and AC 4 above are written at the behavior level precisely because the concrete route-link target is still undecided. |
| **Flag** | `[carried-forward, still-open — not Gate-2-resolved]` The ≤2-tap safety framing (AC 3) is itself flagged by the PRD as possibly infeasible to fully realize within a one-day build. |

---

## D1. Epic D — Search Refinement (Nice-to-Have): Free-text filter on visible results

| Field | Content |
|---|---|
| **Story Title** | `Epic D — Search Refinement (Nice-to-Have): Free-text filter on visible results` |
| **User Story** | As a driver browsing Parking Spot Finder results, I want to filter the visible Parking Spots by free-text, so that I can narrow an already-visible result set (e.g., to Free-only spots) without waiting on a new search. `[RESOLVED at Gate 2 — this capability is deliberately not tied to Ritu or Arjun specifically, matching PRD FR-9's own text ("not exercised by either named journey"); the backlog's Epic D header has been corrected to match]` |
| **Precondition** | Home Results (B2 and/or B3) is showing a non-empty set of Parking Spots for the current Active Location and Radius. |
| **Acceptance Criteria** | 1. Entering filter text narrows the visible results in both the list (B2) and map (B3) views consistently.<br>2. Clearing the filter restores the full within-Radius result set exactly as B1 originally returned it.<br>3. The filter operates only on the already-visible (already-queried) result set — it never triggers a new query to B1. |
| **Edge Cases** | 1. Filter text matches zero currently-visible spots — a "no matches" state is shown, distinct from B4's zero-radius-result state (this is a filter-narrowing empty state, not a query empty state).<br>2. Filter is applied, then the Active Location or Radius changes (e.g., via B4's expansion) — the filter is cleared/reset against the new result set rather than silently persisting against stale data. |
| **Post Condition** | The visible list/map reflects only spots matching the current filter text, or the full result set if no filter (or a cleared filter) is active. |
| **Validation** | Filter is client-side/visible-set-only — no interaction with B1's query parameters. `[confidence: medium — reasonable inference from backlog Dependencies §8 ("filters an already-rendered result set; no new query"), not separately restated as a hard rule in the PRD]` |

---

## D2. Epic D — Search Refinement (Nice-to-Have): Sort by Distance or Price

| Field | Content |
|---|---|
| **Story Title** | `Epic D — Search Refinement (Nice-to-Have): Sort by Distance or Price` |
| **User Story** | As Ritu, I want to re-sort the visible Parking Spots by Distance or Price, so that I can reorder results by whichever matters most to me at the moment, instead of only the default distance-ascending order. |
| **Precondition** | Home Results (B2 and/or B3) is showing a non-empty set of Parking Spots for the current Active Location and Radius. |
| **Acceptance Criteria** | 1. A sort control offers at minimum Distance and Price as sort keys.<br>2. Selecting a sort key re-orders the visible results (list, and map selection order where applicable) by that key, ascending. `[confidence: medium — PRD's own ASSUMPTION, §9: ascending-only for MVP]`<br>3. The selected sort choice persists for the remainder of the session.<br>4. On reload, the sort choice resets to the default (distance ascending) rather than persisting across reloads. |
| **Edge Cases** | 1. Sorting by Price is selected while a Free spot (no price info) is present in the result set — Free spots are placed consistently (e.g., grouped at one end) rather than causing an undefined/erratic order.<br>2. Two spots have identical values for the active sort key (e.g., same distance) — both appear, in a stable order, rather than flickering between renders. |
| **Post Condition** | The visible result set is ordered by the selected sort key (ascending) for the remainder of the session, or has reset to default distance-ascending after a reload. |
| **Validation** | Sort is ascending-only for both keys; session-only persistence (resets on reload). `[confidence: medium — both are the PRD's own §9 ASSUMPTION items]` |

---

## Dependency / sequencing notes (carried forward from backlog, not re-derived)

1. A1/A2 → B1: an Active Location must exist before the nearby query is callable.
2. A2's curated area list ↔ B1's Mock Dataset coverage must agree — sequence or pair these, don't build in isolation.
3. B1 → B2, B3, C1: all downstream rendering/detail consumes B1's single response shape; no separate detail-fetch endpoint is implied.
4. B1 → B4: zero-result messaging needs B1's result count at both 1km and 2km.
5. B2/B3 → C1: Spot Detail Panel is reached from either.
6. C1 → C2: Route action lives inside the detail panel.
7. Map provider (Leaflet + OpenStreetMap, Gate 2) unblocks B3 directly; C2's specific route-link target remains a small open detail for Architecture (see C2 flag above).
8. D1, D2 → B2/B3: refinement layers on top of already-rendered results; no independent path.

## Status

This artifact is **DRAFT**. Per protocol, work stops here — no downstream stage (Architecture/Development) is invoked. Awaiting explicit human review: approve / request changes / reject.
