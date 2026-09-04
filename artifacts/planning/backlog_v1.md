---
stage: planning
version: v1
status: APPROVED
agent: planning-agent
approver: lekshmi.lelithambika@experionglobal.com
timestamp: 2026-09-04T00:00:00
supersedes: none
---

# Backlog: Parking Spot Finder

## Sources

- **Requirements (primary, authoritative):** `_bmad-output/planning-artifacts/prds/prd-BMAD-2026-09-04/prd.md` (`status: final`)
- **Design (secondary, optional input):** `artifacts/design/design_v1.md` (`status: APPROVED`) — used to sharpen story granularity and cross-check screen-implied capabilities against PRD stories. It does not outrank the PRD.

## Constraint scoped against

**One-day hackathon build.** All Must-have / Nice-to-have / Out-of-scope calls below, and all feasibility flags, are made against a single working day for a small hackathon team, not against a normal sprint. This mirrors the PRD's own framing (§7 MVP Scope, §9, §11) — this backlog does not loosen or tighten that constraint, only makes its consequences explicit for planning.

---

## Epics

### Epic A — Location & Area Selection
Establishes the Active Location the rest of the app searches from. Traces to PRD §5.1 (FR-1, FR-2), realizes UJ-1, UJ-2.

### Epic B — Nearby Parking Discovery
Turns the Active Location into a classified, browsable result set (list + map), including the zero-result path. Traces to PRD §5.2 (FR-3, FR-4, FR-5, FR-6), realizes UJ-1, UJ-2.

### Epic C — Parking Detail & Route Handoff
Lets the user confirm a spot and get moving toward it. Traces to PRD §5.3 (FR-7, FR-8), realizes UJ-1, UJ-2.

### Epic D — Search Refinement (Nice-to-Have)
Client-side narrowing of already-visible results. Traces to PRD §5.4 (FR-9, FR-10). `[RESOLVED — corrected]` FR-10 realizes UJ-1; FR-9 does not realize either named journey per the PRD's own text ("not exercised by either named journey") — D1 is a general capability, not tied to a specific persona. Explicitly the first and second items cut under time pressure per PRD §7.2.

---

## Stories / Tasks

### Epic A — Location & Area Selection

**A1. Capture current location via browser geolocation**
- Traces to: FR-1; UJ-1 entry state.
- Notes: includes the session-only, never-persisted handling of captured coordinates, and the no-blocking-error fall-through to A2 on denial/failure (both are testable consequences of FR-1, not additional scope).
- Priority: **Must-have**
- Effort: **S**
- Dependencies: none (entry point).

**A2. Manual area entry fallback (curated preset list)**
- Traces to: FR-2; UJ-2 entry state.
- Notes: the preset-list-not-free-text mechanism is a PRD-level `[ASSUMPTION · confidence: low]` (§9) — this backlog does not relitigate it, just flags that it's a low-confidence scope decision worth a second look at Gate 2/3 if time allows.
- Priority: **Must-have**
- Effort: **S/M** — small UI, but needs a curated list of named areas that actually overlaps the Mock Dataset's coverage (content-curation time, not just code).
- Dependencies: needs the Mock Dataset's area/neighborhood coverage decided first (see B1 note) so the preset list isn't disconnected from what B1 can actually return.

---

### Epic B — Nearby Parking Discovery

**B1. `/parking/nearby` query endpoint + Mock Dataset**
- Traces to: FR-3.
- Notes: FR-3's consequences require the endpoint to return id, name, coordinates, address, `parkingType`, price info, and operating hours per spot — i.e., the full record, not a thin summary. This means B-epic's list/map rendering (B3/B4) and C1's detail panel can all read from this one response shape; no separate "get spot detail" endpoint is implied by the PRD. Building the Mock Dataset itself (seed data) is necessary supporting work for FR-3, not a new requirement — flagged here as a task, not invented scope.
- Dataset size: **decided at Gate 2 — ~12-15 hand-curated rows**, covering one demo area with a Free/Paid mix, plus at least one area with zero results to exercise B4.
- Priority: **Must-have**
- Effort: **S/M** — firmed up from M now that dataset size is decided; haversine calc, bounds validation, and radius∈{1,2}-only validation are all small; curating ~12-15 realistic rows is bounded, predictable work.
- Dependencies: needs an Active Location (Epic A) to be callable end-to-end; curate this dataset's areas together with A2's preset list (see Dependencies §2) so they agree.

**B2. List view with Free/Paid classification**
- Traces to: FR-4.
- Notes: distance rounding `[confidence: medium — PRD's own ASSUMPTION, §9]`, non-color-only badge (also an NFR, §10 Accessibility), default sort by distance ascending. Layout: **decided at Gate 2 — toggle** between list and map (not split-screen); matches the design artifact's draft, no change to this estimate. Acceptance criteria now also include: **decided at Gate 2 — an always-visible radius indicator** on the default (non-empty) view, per the design artifact. This extends slightly beyond FR-4/FR-6's literal wording (the PRD only required naming the radius in the zero-result state) — noted here for traceability, not silently folded in.
- Priority: **Must-have**
- Effort: **S**
- Dependencies: B1 (needs query results to render).

**B3. Map view with Free/Paid markers + list-only degradation**
- Traces to: FR-5; feature-specific NFR (§10 Availability/degradation — hard requirement, not best-effort).
- Notes: same-panel-on-select behavior must match B2/C1 (selecting a marker opens the same Spot Detail Panel as selecting a list item). Renders as one panel of the toggle (Gate 2 decision, see below), not a split layout.
- Map provider: **decided at Gate 2 — Leaflet + OpenStreetMap.** No API key/billing setup required, which removes the setup-time uncertainty this story previously carried.
- Priority: **Must-have** — **designated at Gate 2 as the pressure-point story: if the day runs behind, descope this one first** (e.g. ship list-only and defer the map pane) rather than discovering the tradeoff mid-day.
- Effort: **M/L** — the provider-selection risk is resolved, but genuine graceful degradation (not just a try/catch) alongside a working embedded map is still real effort for one day; see Feasibility Flags.
- Dependencies: B1.

**B4. Zero-result radius messaging + one-tap 2km expansion**
- Traces to: FR-6; UJ-2 climax/edge case.
- Notes: two-tier behavior — 1km empty → offer 2km; 2km also empty → switch to "try a different area" messaging, no further expansion offered. No bare empty state is ever allowed to render (also ties to the "never a blank screen" framing in UJ-2).
- Priority: **Must-have**
- Effort: **S/M** — UI is small; the two-tier state logic (first-empty vs. still-empty-after-expansion) needs a bit of care to get right.
- Dependencies: B1 (needs to know the query returned zero rows at each radius).

---

### Epic C — Parking Detail & Route Handoff

**C1. Spot Detail Panel**
- Traces to: FR-7; UJ-1, UJ-2.
- Notes: one-tap reachable from either a list item (B2) or a map marker (B3); missing optional fields omitted (never blank placeholders); `parkingType` is required so it's never affected by the omission rule; always shows the data-provenance disclosure (§10 NFR — same string as Home view).
- Priority: **Must-have**
- Effort: **S** — mostly presentational, reusing the B1 response shape (see B1 note).
- Dependencies: B1 (data), B2/B3 (entry points into the panel).

**C2. Route handoff to external map provider**
- Traces to: FR-8; UJ-1 climax; §11 Safety guardrail (Route Handoff ≤2 taps from Home).
- Notes: destination-only handoff — Active Location is never passed as an origin parameter (Privacy NFR, §10). Falls back to a plain `maps.google.com` link if the primary provider integration is unavailable, rather than disappearing.
- Map provider: **decided at Gate 2 — Leaflet + OpenStreetMap for B3's display.** `[flag · confidence: medium]` Leaflet is a display library, not a routing/directions service — it doesn't have its own "open turn-by-turn directions" deep link the way a provider like Google Maps does. This story's route target (a `maps.google.com` link, an OSM directions link, or a generic `geo:` URI) is therefore still an open technical detail. Since `maps.google.com` was already the PRD-decided fallback and needs no API key either, using it as the sole route target (collapsing "primary" and "fallback" into one) is the simplest option — flagged here for the Architecture Agent to confirm rather than decided by this backlog.
- Priority: **Must-have**
- Effort: **S/M** — likely trivial if the `maps.google.com`-for-everything simplification above is confirmed; slightly more if a distinct OSM-native routing link is wanted instead.
- Dependencies: C1.

---

### Epic D — Search Refinement (Nice-to-Have)

**D1. Free-text filter on visible results**
- Traces to: FR-9.
- Notes: PRD explicitly notes this is not exercised by either named user journey (UJ-1/UJ-2) — a general capability layered on top of B2/B3, not journey-critical. Filters both list and map consistently; clearing restores the full within-radius set.
- Priority: **Nice-to-have** — PRD-designated first cut under time pressure (§5.4, §7.2).
- Effort: **S**
- Dependencies: B2, B3 (filters an already-rendered result set; no new query).

**D2. Sort by Distance or Price**
- Traces to: FR-10; UJ-1.
- Notes: ascending-only `[confidence: medium — PRD's own ASSUMPTION, §9]`; sort choice persists for the session, resets on reload.
- Priority: **Nice-to-have** — PRD-designated second cut under time pressure (§5.4, §7.2).
- Effort: **S**
- Dependencies: B2, B3.

---

## Cross-cutting tasks (not separately FR-numbered, but required by specific FRs/NFRs — fold into the stories above, don't build as standalone epics)

- **Data-provenance disclosure string** (§10 NFR) — one fixed UI string, shown persistently on Home Results (B2/B3) and Spot Detail Panel (C1). No backend work. Fold into B2/B3/C1 acceptance criteria.
- **Non-color-only Free/Paid distinction** (§10 Accessibility, ties FR-4/FR-5) — fold into B2 (badge = color + text label) and B3 (marker shape/icon differentiation, not color-only).
- **Keyboard operability of the list→detail→route path** (§10 Accessibility) — explicit hard requirement for the list-view path; map-marker keyboard access is PRD-designated best-effort only, since the list path already covers the same action. Fold into B2/C1/C2 acceptance criteria; do not scope keyboard support into B3 (map).
- **HTTPS + malformed-input rejection on `/parking/nearby`** (§10 Security, ties FR-3) — fold into B1 acceptance criteria.

---

## Priority Split (against the one-day-hackathon constraint)

**Must-have (build first, in this rough order):** A1, A2, B1, B2, B3, B4, C1, C2 — i.e., all of FR-1 through FR-8, per PRD §7.1. This is the full MVP; nothing here is optional under the PRD's own scope decision.

**Nice-to-have (build only if Must-haves finish early):** D1, D2 — i.e., FR-9, FR-10, per PRD §5.4/§7.2, in that cut order (D1 before D2 if only one fits).

**Out of scope (do not build, per PRD §6/§7.2):** live/sensor occupancy, reservations/booking, payments, accounts/auth/saved history, admin/spot-management UI, predictive/ML availability, reviews/ratings, native mobile app, user-driven arbitrary radius control, "report incorrect listing" affordance, and polished UI styling beyond basic usability. None of these have stories above by design — do not add them even if time allows, since the PRD marks them as deferred product decisions, not schedule cuts.

---

## Dependencies

1. **A1/A2 → B1**: an Active Location (captured or manual) must exist before the nearby query is callable.
2. **A2 curated area list ↔ B1 Mock Dataset coverage**: these two need to agree on which named areas/neighborhoods actually exist in the ~12-15-row dataset (decided at Gate 2) — sequence or pair these, don't build in isolation.
3. **B1 → B2, B3, C1**: all downstream rendering and detail views consume B1's single response shape; per the B1 note, no separate detail-fetch endpoint is implied.
4. **B1 → B4**: zero-result messaging needs to observe B1's result count at both 1km and 2km.
5. **B2/B3 → C1**: Spot Detail Panel is reached from either.
6. **C1 → C2**: Route action lives inside the detail panel.
7. **Map provider — decided at Gate 2: Leaflet + OpenStreetMap.** Unblocks B3 directly. C2's specific route-link target is still an open technical detail for Architecture (see C2 notes) but is no longer schedule-blocking, since a simple `maps.google.com`-link fallback is already available either way.
8. **D1, D2 → B2/B3**: refinement layers on top of already-rendered results; no independent path.

---

## Feasibility Flags

- **`[RESOLVED at Gate 2]` Map provider: Leaflet + OpenStreetMap.** Unblocks B3 directly; no API key or billing setup needed, removing the setup-time uncertainty this flag originally carried. C2's route-link target remains a small open technical detail for Architecture (see C2 notes), but is no longer schedule-blocking.

- **`[flag · confidence: medium]` — carried forward, now with an explicit fallback plan.** B3 (map view + hard-requirement graceful degradation) is still the highest-effort single Must-Have story even with the provider question resolved — a working embedded map with differentiated markers plus a genuine, tested degradation path to list-only is real effort for one day. **`[RESOLVED at Gate 2]`**: designated as the story to descope-to-minimum first if the day runs behind (see B3 notes), rather than discovering the tradeoff mid-day.

- **`[flag · confidence: medium]` The PRD itself already flags accessibility conformance and the ≤2-tap safety framing as "unlikely to be fully realized within a one-day MVP" (§10, §11 `[NOTE FOR PM]`).** This backlog carries that forward rather than silently dropping it: B2/C1/C2's keyboard-operability and tap-count acceptance criteria should be treated as best-effort-within-Must-have, not guaranteed, given the constraint. Not one of the five Gate 2 items decided — still open; revisit if it becomes a concern during Architecture/Development.

- **`[flag · confidence: low]` A2's curated-area-list scope decision (PRD §9, itself tagged `[ASSUMPTION · confidence: low]`) was made without user confirmation.** If a reviewer wants to revisit free-text geocoding instead, that changes A2's effort from S/M to at least M (geocoding integration or a third-party lookup), and would also affect the Manual Area Entry screen in the design artifact. Not one of the five Gate 2 items decided — still open, though now bounded by the ~12-15-row dataset decision below.

- **`[RESOLVED at Gate 2]` Mock Dataset sizing: ~12-15 hand-curated rows.** Firms up B1's effort from M to S/M (see B1 notes).

- **`[RESOLVED at Gate 2]` Overall Must-have capacity: accepted as-is.** 8 FRs spanning a backend endpoint, a mock dataset, location capture + fallback, list + map + degradation, zero-result logic, a detail panel, and route handoff is a full day's work for a small team, but the PRD already fixed that scope. B3 is the designated pressure point (see above) rather than a blanket risk with no plan.

---

## Design Cross-Check

Design artifact used: `artifacts/design/design_v1.md` (`status: APPROVED`), 6 screens. Cross-checked against the 10 FRs above.

**FR/UJ → screen coverage:** every Must-have and Nice-to-have FR has a corresponding screen (FR-1→Screen 1, FR-2→Screen 4, FR-3/4/5→Screen 2, FR-6→Screen 3, FR-7/8→Screen 5, FR-9/10→Screen 6). No PRD story is missing a screen.

**Screen-implied capability with no corresponding PRD story — surfaced as a question, now resolved at Gate 2:**

- **`[RESOLVED at Gate 2]` Screen 2's persistent "radius indicator" (Key UI Element on the default, non-empty results view) is kept in scope**, extending slightly beyond FR-6's literal zero-result-only wording — a deliberate, traced decision (see B2 notes), not a silent scope creep.

- **`[RESOLVED at Gate 2]` Screen 2's list/map presentation is a toggle**, not split-screen — matches the design artifact's draft as originally assumed; no change to B2/B3 effort estimates.

**PRD story with no corresponding screen:** none found. FR-3 (the query endpoint itself) has no screen, which is expected and correct — it's a backend contract, not a UI surface.

**Conflicts between the two sources:** none found. Where the design artifact carries its own lower-confidence assumptions (e.g., the Manual Area Entry curated-list pattern), it correctly inherits the PRD's own equally-low-confidence assumption rather than contradicting it — flagged above under Feasibility Flags, not treated as a new conflict.

---

## Gate 2 decisions (resolved by lekshmi.lelithambika@experionglobal.com, 2026-09-04)

1. **Map provider:** Leaflet + OpenStreetMap. Unblocks B3; C2's specific route-link target left as a small open detail for Architecture.
2. **Mock Dataset size:** ~12-15 hand-curated rows, one demo area with a Free/Paid mix plus at least one zero-result area.
3. **Home Results layout:** toggle (not split-screen) — no change from the design's draft.
4. **Radius indicator on default Home Results:** kept in scope, extending slightly beyond FR-6's literal wording (noted, not silent).
5. **Overall one-day capacity:** accepted as-is (full FR-1–FR-8 Must-have scope); B3 designated as the descope-first pressure point if the day runs behind.

Not resolved at Gate 2 (carried forward, still open — see Feasibility Flags): the accessibility/safety best-effort framing, and A2's curated-list-vs-free-text assumption.

This artifact is **APPROVED** (see frontmatter). All five originally-open items were decided before approval.
