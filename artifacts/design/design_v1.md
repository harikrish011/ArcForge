---
stage: design
version: v1
status: APPROVED
agent: web-design-agent
approver: lekshmi.lelithambika@experionglobal.com
timestamp: 2026-09-04T00:00:00
supersedes: none
---

# Design: Parking Spot Finder

## Source

- **PRD:** `_bmad-output/planning-artifacts/prds/prd-BMAD-2026-09-04/prd.md` (`status: final`, updated 2026-09-04)
- **Design system:** none referenced by this artifact — the human selects the design system directly in the Claude Design window at generation time. The prompt below is written to defer to whatever system is active there rather than pin a specific one. `[confidence: high — directly follows the human's stated preference, not inferred]`

## Screens

### 1. Home — Location Permission
**Purpose:** Capture the user's current location, or route to manual entry if they decline.
**Key UI Elements:** App name/logo, one-line explainer of why location is requested, "Use my location" action, browser-native permission prompt (system UI, not custom-drawn).
**User Actions:** Grant → proceeds to Home Results. Deny/unavailable → routes to Manual Area Entry.
**Navigation:** Entry point (app load). → Home Results on grant; → Manual Area Entry on deny.
**Realizes:** FR-1; entry state for UJ-1, UJ-2.
**States:** loading (awaiting permission response).

### 2. Home — Results (List + Map)
**Purpose:** Show nearby Parking Spots within the Active Location's radius, each classified Free/Paid.
**Key UI Elements:** List/map view control `[confidence: medium — PRD specifies "combined list + map view," not whether it's a toggle or split layout; drawn here as a toggle, adjust freely]`; list items showing name, Free/Paid badge (never color-only), distance, address, operating hours; map with markers differentiated Free/Paid (not color-only); persistent one-line data-provenance disclosure ("classifications come from a static/demo dataset and may not reflect current signage or pricing"); radius indicator.
**User Actions:** Toggle list/map. Tap a list item or marker → Spot Detail Panel.
**Navigation:** From Home — Location Permission (grant) or Manual Area Entry. → Spot Detail Panel on selection; → Home — Zero Results when the query returns nothing.
**Realizes:** FR-3, FR-4, FR-5; UJ-1, UJ-2 (post-location).
**States:** default (results present); loading (query in flight); map-unavailable (degrades to list-only view, same content, no map pane).

### 3. Home — Zero Results
**Purpose:** Explain no spots were found and offer the one-tap radius expansion, rather than a blank screen.
**Key UI Elements:** Message naming the searched radius (1km), "Expand to 2km" action; on a second empty result, the message instead suggests setting a different area (no further expansion offered).
**User Actions:** Tap Expand → re-queries at 2km, returns to Home Results. If still empty → prompt routes to Manual Area Entry.
**Navigation:** Empty-state variant of Home Results. → Home Results (2km) on successful expansion; → Manual Area Entry if still empty after expansion.
**Realizes:** FR-6; UJ-2.
**States:** this screen *is* the empty state for Home Results — no separate loading/error variant needed.

### 4. Manual Area Entry
**Purpose:** Let the user set an Active Location without granting browser permission.
**Key UI Elements:** A curated, selectable list of named areas/neighborhoods — not a free-text field `[confidence: medium — this reflects a PRD assumption (FR-2) that itself carries only medium confidence; if the human overrides that assumption, this screen changes to a text-entry pattern instead]`.
**User Actions:** Select an area → proceeds to Home Results for that area.
**Navigation:** Entered from Home — Location Permission (denial) or Home — Zero Results (persistent empty). → Home Results.
**Realizes:** FR-2; UJ-2.
**States:** default only (fixed curated list, no empty/error variant specified).

### 5. Spot Detail Panel (overlay/modal)
**Purpose:** Show a spot's full information and offer Route Handoff before the user commits.
**Key UI Elements:** Spot name, address, Free/Paid badge, price info (Paid spots only), operating hours, the same data-provenance disclosure as Home Results, a "Route" action, a dismiss control.
**User Actions:** Tap Route → hands off to an external map provider (destination only — this app never sends the user's location as a routing origin). Dismiss → returns to whichever Home Results view it was opened from.
**Realizes:** FR-7, FR-8; UJ-1, UJ-2.
**States:** default. Missing optional fields (address/hours/price) are omitted, not shown blank; the Free/Paid badge is always present (required field, never missing).

### 6. Home Results — Search Refinement *(Nice-to-Have, not required for MVP)*
**Purpose:** Narrow the visible results without a new query.
**Key UI Elements:** Free-text filter control, sort control (Distance / Price, ascending only) — both layered onto Home Results, not a separate screen.
**User Actions:** Filter or sort; clearing restores the full result set.
**Navigation:** Overlay/inline addition to Home Results (#2).
**Realizes:** FR-9, FR-10.
**States:** N/A — cut first under time pressure per the PRD; include only if time allows.

## Claude Design Prompt

*Ready to paste into the Claude Design window. Use whatever design system you have selected there — this prompt intentionally specifies no colors, typography, or component library of its own.*

```
Create a multi-artboard canvas prototype for a web app called "Parking Spot Finder." Use the design system already active in this workspace for all colors, typography, spacing, and components — do not introduce new tokens or a new visual style.

The app helps a driver find free vs. paid parking within walking distance of their location, shown as a combined list + map, with a one-tap route handoff. Build these artboards, laid out left to right as a flow with arrows showing navigation between them:

1. HOME — LOCATION PERMISSION
   App name/logo, a one-line explainer of why location access is requested, and a primary "Use my location" action. Show the state as if a location-permission dialog is about to appear (a light system-style overlay is fine).

2. HOME — RESULTS (default state)
   A list/map toggle at the top. List view: each row shows a spot name, a Free/Paid badge (must NOT rely on color alone — pair the color with a text label like "FREE" or "PAID"), distance, address, and operating hours. Map view: pins differentiated Free/Paid the same non-color-only way. Include a persistent, unobtrusive one-line disclosure near the top or bottom: "Classifications are from a static demo dataset and may not reflect current signage or pricing." Show 4-6 example spots with realistic-looking names/addresses/hours, a mix of Free and Paid.

3. HOME — ZERO RESULTS (empty state variant of artboard 2)
   Same shell as Results, but the list/map area shows: "No parking spots found within 1km." and a clear "Expand search to 2km" button.

4. MANUAL AREA ENTRY
   A simple screen with a short prompt ("Choose your area") and a selectable list of 4-6 named neighborhoods/areas (not a text input field).

5. SPOT DETAIL (overlay/modal, shown on top of artboard 2)
   A card or modal showing: spot name, full address, Free/Paid badge, price info (only if Paid — e.g. "$3/hr"), operating hours, the same data-provenance disclosure line as Results, a primary "Route" button, and a close/dismiss control.

Interaction notes to reflect in the visual design (labels, affordances, or annotations — this is a static prototype, not functional code):
- Tapping a list row or map pin on artboard 2 opens artboard 5.
- Tapping "Route" implies handoff to an external maps app — show it as a button, not a live map.
- Tapping "Use my location" and granting access leads to artboard 2; declining leads to artboard 4.
- Tapping an area on artboard 4 leads to artboard 2.

Do not add: user accounts, login, payments, admin/management screens, or reviews/ratings — this product explicitly excludes all of them.
```

## Open items for the human before running this prompt

- `[NOTE FOR PM]` Confirm whether list/map is a toggle or a split-screen layout (§4.2 FR-4/FR-5 of the PRD don't specify) — the prompt currently leaves this to Claude Design's judgment; add a line to the prompt if you have a preference.
- `[NOTE FOR PM]` The Manual Area Entry screen assumes a curated preset list per the PRD's own `[ASSUMPTION · confidence: low]` on FR-2 — if that assumption gets overturned in favor of free-text entry, this screen (and prompt section 4) needs a follow-up revision.
