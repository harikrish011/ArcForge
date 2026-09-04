---
title: Parking Spot Finder
created: 2026-09-04
updated: 2026-09-04
status: final
---

# PRD: Parking Spot Finder
*Working title — confirm.*

## 0. Document Purpose

This PRD defines the **Parking Spot Finder** product — a web app that helps a driver find free vs. paid parking within 1km of their location — for a Product Manager, downstream architecture/epics workflows, and reviewers assessing product requirements on their own terms.

This PRD is distilled from `ARC_Forge_Proposal_Parking_Spot_Finder.md`, a hackathon submission proposal that frames Parking Spot Finder as the vehicle for demonstrating a separate Agentic SDLC process (a coordinating agent, five human-approval gates, specialist agent roles, guardrails, and cost/token discipline). **By scope decision, that process content is not part of this PRD's requirements** — it describes how the team builds, not what the product does for its users. It is captured in `addendum.md` as build methodology and rationale, and is the intended input to a separate architecture/process artifact.

Features are grouped with FRs nested and globally numbered; assumptions and open ambiguities are tagged inline and captured with explicit confidence levels in §9.

## 1. Vision

Parking Spot Finder removes one small, recurrent friction from driving to a destination: not knowing whether nearby parking will cost money before you commit to a spot. A driver opens the app, grants location access (or sets an area manually), and immediately sees known parking locations within 1km — on a map and in a list — each honestly tagged Free/Paid, with distance, address, and operating hours. They pick one and get routed there.

The product deliberately does not attempt live occupancy, reservations, or payments — those require sensor and API infrastructure the MVP does not have. What it delivers instead is narrow and dependable: a fast, honest answer to "is there free parking near here, and how do I get there."

Success is a driver who trusts the Free/Paid badge enough to act on it without double-checking, and who reaches for the app again the next time they're headed somewhere unfamiliar.

## 2. Target User

### 2.1 Jobs To Be Done

- **Functional:** Quickly determine whether free parking exists near a destination before arriving, so I don't default to (or circle looking for) a paid option unnecessarily.
- **Functional:** Get turn-by-turn routing to a chosen spot without leaving the app to search separately.
- **Emotional:** Reduce the low-grade anxiety of "will I find somewhere to park" in the last few minutes of a drive.
- **Contextual:** Needs an answer fast, often glanced at while approaching a destination — not a research session.

### 2.2 Non-Users (v1)

- Facility operators or lot owners wanting to manage/list their own spots (no admin surface exists).
- Users needing a guaranteed, reservable spot (product surfaces known locations, not availability guarantees).
- Fleet or commercial parking managers.
- Users without a modern browser or without location capability who also decline manual area entry — the app cannot help them.

### 2.3 Key User Journeys

- **UJ-1. Ritu finds a free spot two blocks from her meeting.**
  - **Persona + context:** Ritu drives into a dense urban core for a client meeting three times a week and is never sure whether the street outside the office building is metered.
  - **Entry state:** Opens the app for the first time this session, not authenticated (no auth in this product), location services enabled on her phone/laptop browser.
  - **Path:** Opens Parking Spot Finder → grants the browser's location permission prompt → sees a combined list + map of spots within 1km → notices a Free-tagged spot two blocks away with hours covering her meeting window → taps it for the detail panel → confirms address and hours → taps Route.
  - **Climax:** The map provider opens with turn-by-turn directions already pointed at the free spot — she didn't have to search separately or guess.
  - **Resolution:** She parks for free, arrives on time, and the app becomes her default check before any drive into that part of the city.
  - **Edge case:** If the map provider SDK fails to load, she still completes the flow via the list view and a plain external maps link (ties to FR-5, FR-8).

- **UJ-2. Arjun still finds parking after declining the location prompt.**
  - **Persona + context:** Arjun has location services off by habit and doesn't want to turn them on for a one-off trip to an unfamiliar neighborhood.
  - **Entry state:** Opens the app, denies the browser's location permission prompt.
  - **Path:** Sees a manual area-entry prompt instead of a dead end → sets his search area → sees the same list + map for that area → the first search returns zero results within 1km → the app tells him so and offers to expand the radius → he expands it → sees a Paid spot with clear pricing → decides it's worth it and taps Route.
  - **Climax:** Even having opted out of location sharing, he still gets to a usable answer rather than a blank screen.
  - **Resolution:** He reaches a spot he can commit to, with no ambiguity about cost going in.
  - **Edge case:** Handled inline — the zero-result state itself is the edge case this journey exists to prove out.

## 3. Glossary

- **Parking Spot** — A single known parking location in the dataset: id, name, coordinates, address, parking type, price info, operating hours.
- **Free/Paid Classification (`parkingType`)** — The spot's cost category: `FREE` or `PAID`. The core trust signal the product exists to surface.
- **Radius** — The search distance from the Active Location within which spots are returned. Fixed at 1km for the initial query; not arbitrarily user-adjustable. The one exception is a single system-offered step to 2km when the 1km search returns zero results (FR-6) — a narrow, system-triggered escape hatch, not general radius control (see FR-3, FR-6, Non-Goals).
- **Mock Dataset** — The static, pre-seeded set of Parking Spots the MVP queries against. Not live or sensor-fed.
- **Active Location** — The coordinate pair the app currently searches from: either the browser-captured current location or a manually set area. Both feed the same downstream flow.
- **Manual Area Entry** — The fallback path a user takes to set an Active Location without granting browser location permission.
- **Spot Detail Panel** — The single-tap view showing a Parking Spot's full name, address, price info, and operating hours.
- **Route Handoff** — Handing the user off to an external map provider for turn-by-turn navigation to a chosen spot.

## 4. Information Architecture

- **Home** — combined list + map view of nearby results, Free/Paid badges visible without opening detail.
- **Manual Area Entry** — fallback state shown in place of Home when location permission is denied/unavailable (FR-2).
- **Spot Detail Panel** — overlay/modal reachable from Home; hosts the Route action (FR-7, FR-8).

No other screens exist in v1 — no accounts, no settings, no admin surface.

## 5. Features

FR-1–FR-8 are Must-Have; FR-9–FR-10 (Search Refinement, below) are Nice-to-Have, cut first under time pressure (see MVP Scope).

### 5.1 Location & Area Selection
**Description:** Establishes the Active Location the rest of the app searches from, either via browser geolocation or a manual fallback so a declined permission never dead-ends the user. Realizes UJ-1, UJ-2.

#### FR-1: Capture current location

The user can grant browser location permission so the app captures their current coordinates as the Active Location. Realizes UJ-1.

**Consequences (testable):**
- Browser geolocation is requested on app load or via an explicit "Use my location" action.
- Captured coordinates are held client-side for the session only — never persisted, never transmitted beyond the FR-3 query.
- A denied or unavailable permission falls through to FR-2 without a blocking error state.

#### FR-2: Manual area entry fallback

When location permission is denied or unavailable, the user can manually set an Active Location. Realizes UJ-2.

**Consequences (testable):**
- The manual-entry prompt appears immediately on denial/failure — no dead-end screen.
- Manual entry is a selection from a curated preset list of named areas/neighborhoods covered by the Mock Dataset — not free-text geocoding. `[ASSUMPTION: resolves former Open Question on input mechanism; revisit if free-text address entry is needed post-MVP]`
- Once set, a manual Active Location behaves identically to a captured one for FR-3 onward.

### 5.2 Nearby Parking Discovery
**Description:** Turns the Active Location into a classified, browsable set of nearby results in both list and map form. Realizes UJ-1, UJ-2.

#### FR-3: Nearby parking query (1km radius)

The system returns Parking Spots within a Radius of the Active Location, where distance to each spot is computed via the haversine formula over the Mock Dataset. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- `GET /parking/nearby?lat&lng&radius` returns spots where haversine distance ≤ the given radius; the server accepts only `radius=1` or `radius=2` (km) — see Radius, §3.
- `lat` must be in [-90, 90] and `lng` in [-180, 180]; values outside these bounds are malformed.
- Response includes id, name, coordinates, address, `parkingType` (`FREE`/`PAID`, required/non-nullable — every dataset row has a classification, no "unclassified" state in v1), price info, and operating hours per spot.
- Malformed or out-of-bounds `lat`/`lng`, or a `radius` value other than `1`/`2`, is rejected with a 4xx response rather than silently defaulting.

**Out of Scope:** Arbitrary user-driven radius control — e.g. a slider or free-text distance (Non-Goal, v1).

#### FR-4: List view with classification

The user can see nearby results as a list, each tagged Free/Paid with distance, address, and operating hours. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Every list item shows at minimum: name, Free/Paid badge, distance `[ASSUMPTION: rounded to nearest 0.1km]`, address, operating hours.
- List sorts by distance ascending by default.
- The Free/Paid badge is distinguishable by more than color alone (text label plus color), per the accessibility NFR in §10 Cross-Cutting NFRs.

#### FR-5: Map view with markers

The user can see the same nearby results as markers on a map, visually differentiated by Free/Paid. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Markers are visually distinguishable for Free/Paid without relying on color alone.
- Selecting a marker opens the same Spot Detail Panel (FR-7) as selecting the equivalent list item.
- If the map provider SDK fails to load, the app degrades to list-only view (FR-4) rather than blocking the user.

**Feature-specific NFRs:**
- Graceful degradation to list-only on map SDK failure is a hard requirement, not best-effort (ties to §10 Cross-Cutting NFRs → Availability/degradation).

#### FR-6: Zero-result radius messaging

When the initial 1km query returns no spots, the user sees an explanatory message rather than a blank state. Realizes UJ-2.

**Consequences (testable):**
- The zero-result state names the searched radius (1km) and offers a single one-tap expansion to 2km rather than leaving the user to guess a next step.
- If the 2km expansion also returns zero results, the message updates to suggest setting a different Active Location rather than offering further expansion.
- The app never renders a bare empty list/map with no explanatory copy.

### 5.3 Parking Detail & Route Handoff
**Description:** Lets the user confirm a spot's specifics and get moving toward it. Realizes UJ-1, UJ-2.

#### FR-7: Spot detail panel

The user can open a Spot Detail Panel to see a spot's full name, address, price info, and operating hours before deciding. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- Reachable in one tap/click from either a list item or a map marker.
- All fields present in the dataset for that spot are shown; missing fields are omitted rather than rendered as blank placeholders. `parkingType` is required/non-nullable (see FR-3), so it is never among the fields this omission rule could drop.
- The panel always shows the data-provenance disclosure defined in Cross-Cutting NFRs.

#### FR-8: Route handoff

From the Spot Detail Panel, the user can hand off to an external map provider for turn-by-turn navigation to the selected spot. Realizes UJ-1, UJ-2.

**Consequences (testable):**
- The Route action opens the map provider pre-populated with the spot's coordinates/address as the destination only — it never passes the user's Active Location as an origin parameter. Origin resolution, if any, is left entirely to the external map app and its own permission/location model.
- If the primary map provider integration is unavailable, the Route action falls back to a plain external maps link (`maps.google.com`) rather than disappearing.

### 5.4 Search Refinement *(Nice-to-Have — not required for MVP acceptance)*
**Description:** Lets a user narrow results without waiting on backend changes. Realizes UJ-1.

#### FR-9: Free-text filter

The user can filter list/map results by free-text (e.g., narrowing to "Free only"). Not exercised by either named journey in §2.3 — a general refinement capability layered on top of FR-3/FR-4/FR-5 once results are already visible.

**Consequences (testable):**
- Filtering updates both list and map views consistently.
- Clearing the filter restores the full within-radius result set.

**Notes:** `[NON-GOAL for MVP if time-constrained]` — first item cut under schedule pressure per the source proposal's Hackathon Survival Plan.

#### FR-10: Sort by distance or price

The user can re-sort results by Distance or Price instead of the default. Realizes UJ-1.

**Consequences (testable):**
- Sort control offers at minimum Distance and Price, `[ASSUMPTION: ascending only for MVP]`.
- Sort choice persists for the session but resets on reload.

**Notes:** `[NON-GOAL for MVP if time-constrained]` — second item cut under schedule pressure, after FR-9.

## 6. Non-Goals (Explicit)

- No real-time occupancy or live sensor data — the dataset is static/mock, not a live feed.
- No reservations or booking.
- No payments processing.
- No user accounts, authentication, or saved history.
- No admin dashboard or spot-management UI.
- No predictive/ML availability modeling.
- No reviews or ratings.
- No native mobile app in v1 — web only.
- No user-driven arbitrary radius control (slider, input, or free choice of distance) in v1 — see Radius, §3.
- No mechanism to report or correct an incorrect/stale listing (e.g., a "report wrong info" affordance) — a deliberate v1 exclusion, not a silent gap. The data-provenance disclosure (Cross-Cutting NFRs) is the v1 mitigation instead.

## 7. MVP Scope

### 7.1 In Scope
FR-1 through FR-8 (Must-Have) — see §5 Features for detail.

### 7.2 Out of Scope for MVP
- FR-9 and FR-10 (Nice-to-Have) — cut first under time pressure. `[NOTE FOR PM: revisit if build stays ahead of schedule.]`
- Polished UI styling beyond basic usability — explicitly deferred under time pressure, per the source proposal's Hackathon Survival Plan ("Can be skipped"). Must-Have FRs must be visually functional, not polished.
- Everything in §6 Non-Goals — deferred to a post-MVP decision, no timeline committed.

## 8. Success Metrics

*Product usage metrics below are `[ASSUMPTION]` — the source proposal defines hackathon-judging metrics (agent calls, gates exercised, defects caught) instead, which belong to the Agentic SDLC process and are captured in `addendum.md`, not here. They are also explicitly **not instrumented in the MVP**: the system has no accounts, no persistence, and no analytics/telemetry mechanism (per the Privacy NFR), so none of the metrics below can actually be measured by the system as built. They are directional, post-MVP targets — not MVP acceptance criteria — until a telemetry approach is scoped (§9 Assumptions & Ambiguities).*

**Primary**
- **SM-1**: `[ASSUMPTION: target 70%]` of sessions reach Route Handoff (FR-8) within 60 seconds of Active Location being set. Post-MVP target validating FR-3, FR-4, FR-5, FR-7, FR-8 once instrumented.

**Secondary**
- **SM-2**: `[ASSUMPTION: target <20%]` of sessions return zero results within 1km; of those, `[ASSUMPTION: target 50%]` engage the radius-expansion offer. Post-MVP target validating FR-6 once instrumented.

**Counter-metrics (do not optimize)**
- **SM-C1**: Free/Paid classification accuracy must not be sacrificed in pursuit of a faster SM-1 (i.e., faster taps must not come from misleadingly confident badges on uncertain data). Counterbalances SM-1.

## 9. Assumptions & Ambiguities

*Every inference made without explicit confirmation, and every question still genuinely open — one flat list of discrete, individually tagged items, not narrative prose. Canonical record: inline `[ASSUMPTION: ...]` tags elsewhere in this PRD are reading aids only, not a substitute for an entry here. Ordered by confidence ascending (low first) so the items most needing attention surface first.*

**Low confidence**
- **[ASSUMPTION · confidence: low]** (§5.1 FR-2) — Manual Area Entry uses a curated preset list of named areas, not free-text geocoding. This resolves what was previously an open question about the input mechanism, but it's a real scope decision made without user confirmation, not a low-stakes default.
- **[ASSUMPTION · confidence: low]** (§8 SM-1, SM-2) — Success Metric target percentages (70%, <20%, 50%) are invented placeholders with no basis in the source proposal or any usage data.
- **[ASSUMPTION · confidence: low]** (§10 Cross-Cutting NFRs → Performance) — The 500ms response-time bound assumes a Mock Dataset of up to ~500 rows; this row count is a pure order-of-magnitude guess pending the Mock Dataset sizing ambiguity below.

**Medium confidence**
- **[ASSUMPTION · confidence: medium]** (§3 Glossary / §5.2 FR-3) — `parkingType` is required/non-nullable at the data layer (no "unclassified" state in v1); inferred to reconcile FR-4's badge guarantee with FR-7's missing-field-omission rule, not stated by the source.
- **[ASSUMPTION · confidence: medium]** (§5.2 FR-4) — Distance display rounded to nearest 0.1km.
- **[ASSUMPTION · confidence: medium]** (§5.4 FR-10) — Sort is ascending-only for MVP.
- **[ASSUMPTION · confidence: medium]** (§10 Cross-Cutting NFRs → Browser support) — Latest evergreen browsers only (Chrome, Edge, Safari, Firefox); no legacy support.
- **[ASSUMPTION · confidence: medium]** (§11 Constraints and Guardrails → Safety) — The driving-safety framing (Route Handoff ≤2 taps) is a reasonable inference from the product's driving-adjacent use context, not an explicit request; may be infeasible to fully realize within a one-day build beyond its quantified form (see inline `[NOTE FOR PM]`).
- **[AMBIGUITY · confidence: medium]** — Whether the Mock Dataset will be hand-curated or seeded from a real source, and how large/representative it needs to be for the demo to read as credible. Also determines the Performance NFR row-count assumption above.
- **[AMBIGUITY · confidence: medium]** (§8 SM-1, SM-2) — Success Metrics require an anonymous event-logging mechanism not yet scoped, one that must reconcile with the no-persistence Privacy NFR. Treated as post-MVP until resolved.

**High confidence**
- **[ASSUMPTION · confidence: high]** (§6.2) — UI polish beyond basic usability is explicitly out of scope under time pressure, directly stated in the source proposal's Hackathon Survival Plan ("Can be skipped").
- **[ASSUMPTION · confidence: high]** (§11 Constraints and Guardrails → Cost) — No paid infrastructure spend for the MVP; directly implied by the hackathon framing and the source proposal's explicit free-tier requirement.
- **[AMBIGUITY · confidence: high]** (§5.3 FR-8, addendum) — Primary map provider/SDK is unselected; well-scoped (pick one meeting the free-tier criterion) and to be confirmed at build time. The fallback is already decided: a plain `maps.google.com` link.
- **[AMBIGUITY · confidence: high]** — Whether this product continues past the hackathon; determines whether "web only, fixed radius, no accounts" are permanent decisions or one-day cuts to revisit.

## 10. Cross-Cutting NFRs

- **Performance:** `[ASSUMPTION]` `/parking/nearby` responds within 500ms for a Mock Dataset of up to ~500 rows (order-of-magnitude bound; exact dataset size remains an open ambiguity, see §9).
- **Accessibility:** Free/Paid distinguishable by more than color in both list and map views (ties FR-4, FR-5). The list-view path (results → detail → route) must be fully keyboard-operable. Map-marker keyboard access is best-effort for MVP — the equivalent action is always reachable via the list, so it is not a launch gate. `[NOTE FOR PM: accessibility conformance beyond this split is unlikely to be fully realized within a one-day MVP.]`
- **Availability/degradation:** Map SDK unavailability degrades to list-only (FR-5) as a hard requirement, not best-effort.
- **Browser support:** `[ASSUMPTION]` Latest evergreen browsers (Chrome, Edge, Safari, Firefox); no legacy browser support.
- **Privacy:** Location data is used client-side only to compute distance to Mock Dataset points; it is not persisted server-side, not associated with a user (there are no accounts), and not transmitted to any third party beyond the necessary map-provider SDK call for rendering routes. Route Handoff (FR-8) never forwards Active Location to the map provider as a routing origin. The browser's native location-permission prompt is the sole consent mechanism — appropriate for a hackathon prototype, explicitly not a production-grade privacy design.
- **Security:** `/parking/nearby` is served over HTTPS; no credentials, API keys, or secrets are present in client-side code; malformed query params are rejected rather than processed (ties FR-3). No auth and no PII keep the risk surface small by design.
- **Data provenance disclosure:** The Home view and Spot Detail Panel (FR-7) always show a persistent, unobtrusive disclosure that classifications come from a static/demo dataset and may not reflect current signage or pricing — a fixed UI string, no backend work. This is the direct protection for the Vision's core trust claim, and the counterpart to SM-C1.

## 11. Constraints and Guardrails

**Cost:** The map provider must have a free tier sufficient for demo-day traffic; no paid infrastructure spend is assumed for the MVP. `[ASSUMPTION]`

**Safety:** The app should not demand sustained visual attention while a vehicle is in motion — Route Handoff must be reachable in ≤2 taps from Home (ties FR-7, FR-8). `[ASSUMPTION, given driving-adjacent use context]` `[NOTE FOR PM: a full "do not use while driving" interaction pattern beyond this is likely out of scope for a one-day MVP — revisit if this becomes a real product.]`

## 12. Platform

Web application, single surface (list + map), responsive for desktop and mobile browsers. No native mobile app in v1 (see Non-Goals). Backend/frontend technology choices (framework, database, hosting) are implementation details and live in `addendum.md`, not here — with one exception: the `/parking/nearby` query contract (§5.2 FR-3) is specified in this PRD because its shape (fixed lat/lng/radius semantics) is a product decision, not an arbitrary implementation choice.
