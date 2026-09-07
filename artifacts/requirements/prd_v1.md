---
stage: requirements
version: v1
status: APPROVED
agent: requirements-agent
approver: lekshmi.lelithambika@experionglobal.com
timestamp: 2026-09-07T00:00:00
supersedes: none
---

# PRD: Favorite Spots & Cross-Device Sync (Parking Spot Finder)

## Document Metadata

| Field | Value |
|---|---|
| Title | Favorite Spots & Cross-Device Sync |
| Author | Lekshmi Lelithambika |
| Created | 2026-09-07 |
| Status | APPROVED |

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| v1 | 2026-09-07 | Lekshmi Lelithambika | Initial draft |

## Gate 1 Approval Note

Approved as-is by lekshmi.lelithambika@experionglobal.com on 2026-09-07, with the review findings in `artifacts/requirements/prd_review_v1.md` (verdict: `Blocking issues found`) knowingly accepted rather than fixed in a v2. On record for downstream agents:

- The Assumptions-preamble/Open-Ambiguities contradiction described in that review is stale text, not a live open question — both flagged Security/PII items (data-access rights, GDPR/CCPA jurisdiction) are in fact resolved, per this document's Source/FR-6/NFR Privacy sections.
- All ten Assumptions-section defaults are treated as final, despite still being individually marked `pending sanity-check` in that section.
- Two unresolved logic conflicts remain **not fixed**, only acknowledged: (1) FR-1 vs. FR-3 on favorites access for signed-in-but-unverified accounts; (2) FR-6's "irrecoverably deleted" guarantee vs. the Availability NFR's last-known-synced local copy, which has no specified client-side purge on account deletion.
- FR-6's erasure timeframe remains stated three inconsistent ways (compliant timeframe / ≤30 days / 45 days / untimed consequence) — no single number should be treated as authoritative without going back to the human.

Downstream agents (design, planning, architecture) working from this PRD should treat the four items above as live gaps to design around or flag again, not as settled requirements.

## Source

This PRD scopes a new feature on top of the existing, shipped Parking Spot Finder product, whose requirements live in `_bmad-output/planning-artifacts/prds/prd-BMAD-2026-09-04/prd.md` (`status: final`, 2026-09-04) — treated here as the approved upstream baseline. `[confidence: high — directly read from that document]`

That baseline PRD explicitly lists "No user accounts, authentication, or saved history" as a v1 Non-Goal (§6), and frames it as a decision the team may revisit "if this becomes a real product" (§9, AMBIGUITY, high confidence: "Whether this product continues past the hackathon"). This PRD is that revisit — it proposes reversing that specific non-goal for one feature, not a general re-scoping of the base product. `[confidence: high — directly derived from base PRD §6, §9]`

No human product context was supplied for this specific feature beyond "draft a PRD for an idea you have in mind" — the feature concept itself, and every requirement below, is my own proposal rather than a stated ask. Confidence flags throughout reflect that: treat this document as a starting point for discussion, not a captured requirement.

## Goals and Objectives

- Let a **returning, repeat user** (not a one-off visitor) preserve a personal shortlist of parking spots they trust, so they stop re-running the full discovery flow (FR-1–FR-6 of the base PRD) every time they return to a place they've already parked successfully.
- Make that shortlist available regardless of which device the user opens the app on — e.g., checked on a phone before leaving, or on a desktop while planning a trip — since a purely local/session-only favorites list (no accounts needed) would not satisfy this.
- Reinforce the base product's own success framing: the base Vision names a user who "reaches for the app again the next time they're headed somewhere unfamiliar" as the win condition. Favorites directly serves the *familiar*-destination counterpart of that same loyalty goal. `[confidence: medium — reasonable extension of the base Vision statement, not itself stated as a goal there]`

This feature does **not** aim to: add social features (sharing favorites with other users), add reviews/ratings, or change the core discovery/routing flow (base FR-1–FR-8 are unaffected and remain fully usable without ever signing in).

## Problem Statement

The base product is deliberately session-only and account-free (base PRD §6, §10 Privacy NFR): no search state, favorite, or preference survives a reload or follows the user to a second device. For a one-off visitor to an unfamiliar area (the product's originally scoped use case — see base UJ-2), this is a non-issue. But for a **repeat user** who parks in the same handful of places regularly (the base PRD's own UJ-1 persona, Ritu, who "drives into a dense urban core for a client meeting three times a week"), it means re-running the same location grant → browse → detail → route sequence from scratch on every single visit, with the app retaining no memory that she's done this before.

This PRD scopes a feature to close that specific gap: let a user mark a spot as a favorite and have it persist and sync across the devices they use the app from. Doing so requires *some* form of user identity, which is a materially bigger change to the base product's architecture and privacy posture than any other feature it currently has — the base PRD's Non-Goals and Privacy NFR were both written on the explicit premise of "no accounts" (§6, §10). This document treats that trade-off as a deliberate, flagged decision to be made at Gate 1, not something to slip in quietly.

**Explicitly out of scope for this feature:**
- Sharing a favorites list with other users, or any other social/collaborative capability.
- Spot reviews, ratings, or user-submitted corrections to listing data (the base PRD already treats "report wrong info" as a deliberate v1 exclusion — §6 — and this feature does not revisit that).
- Any change to the core discovery, list/map, detail, or route-handoff flows (base FR-1–FR-8) for a user who never signs in — those must keep working exactly as today, sign-in-free.
- Building a general-purpose account/profile system (e.g., editable user profiles, notification preferences) beyond the minimum identity needed to sync a favorites list.
- A right-to-access/data-export capability (letting a user download/view what's stored about them) — only erasure (FR-6) is in scope; access/export is deferred.

## Personas / User Roles

Reuses the base PRD's existing persona rather than inventing a new one, since this feature is explicitly aimed at deepening her use case:

- **Ritu (base UJ-1)** — a repeat urban driver who visits some destinations regularly. Primary beneficiary: she should be able to favorite the free spot she found two blocks from her regular client's office once, and have it waiting for her on both her phone and her laptop from then on, without re-discovering it each visit.
- **Arjun (base UJ-2)**, the one-off/unfamiliar-area visitor, is a non-beneficiary of this feature by design — his flow is unaffected and requires no sign-in.

## Functional Requirements

### FR-1: Mark a spot as a favorite
From the Spot Detail Panel (base PRD §5.3), a signed-in user can mark or unmark a Parking Spot as a favorite with a single action.

**Consequences (testable):**
- The favorite affordance is reachable from both the Spot Detail Panel and directly on each list item; map markers keep today's behavior (open the Spot Detail Panel) rather than getting a separate on-marker toggle. `[confidence: medium — default proposed by requirements agent, pending human sanity-check]`
- Un-favoriting is symmetric — same control, one action, no confirmation step, since it's non-destructive (the spot itself isn't deleted, only removed from the personal list).

### FR-2: View a Favorites list
The user has a dedicated Favorites view, reachable from Home, listing every spot they've favorited.

**Consequences (testable):**
- Each entry shows at minimum the same core fields as the base list view (base FR-4): name, Free/Paid badge, address, operating hours.
- Selecting an entry opens the same Spot Detail Panel used elsewhere (base FR-7) — no separate detail UI is introduced.
- Default sort order is recency-favorited (most recently favorited first). `[confidence: medium — default proposed by requirements agent, pending human sanity-check]`

### FR-3: Sign in with email and password to enable favorites and sync
The user establishes an identity via an email address and password so their favorites list is tied to them rather than to a single device/session. `[confidence: high — human-confirmed]`

**Consequences (testable):**
- A user can register an account with an email and password, then sign in and sign out with those credentials.
- A user who never signs in retains full use of the base discovery/route flow (base FR-1–FR-8) with no favorites capability — sign-in is opt-in, not a gate on the core product.
- Password-reset ("forgot password") is in scope, as a near-mandatory companion to any password-based login. `[confidence: medium — default proposed by requirements agent, pending human sanity-check]`
- Email verification is required at registration — an unverified account can sign in but favorites/sync stay disabled until the email is confirmed (verification also gives a confirmed contact point for erasure/security requests under FR-6). `[confidence: medium — default proposed by requirements agent, pending human sanity-check]`

### FR-4: Cross-device sync
Once signed in on a second device, a user's previously saved favorites appear there without manual re-entry.

**Consequences (testable):**
- Sync direction is bidirectional: a favorite added, removed, or reordered on one device is reflected on another the next time that other device is opened while signed in. `[confidence: medium — reasonable default for "sync," not explicitly specified]`
- Sync is sync-on-load — favorites refresh when the app opens or reconnects, not a real-time push while multiple devices are simultaneously open. `[confidence: medium — default proposed by requirements agent, pending human sanity-check]`

### FR-5: Remove a favorite
The user can remove a spot from their Favorites list from either the Favorites view (FR-2) or the Spot Detail Panel (FR-1's un-favorite control).

**Consequences (testable):**
- Removal syncs across devices the same way additions do (FR-4).

### FR-6: Account and favorites data deletion (GDPR/CCPA-style erasure)
The user can request deletion of their account, and their identity data (email, password) and favorites list are erased within a compliant timeframe. `[confidence: high — human-confirmed]`

**Consequences (testable):**
- A deletion request results in the account's email/password credentials and its favorites list being irrecoverably deleted, not merely hidden or soft-deleted.
- Timeframe: since the product is not targeting EU/California users, GDPR's "without undue delay" (≤30 days) and CCPA's 45-day window are not legally binding here, but are adopted as the voluntary target timeframe anyway, for a consistent, defensible standard rather than an undefined one. `[confidence: medium — standard statutory reference points repurposed as a voluntary target, not a compliance obligation; the architecture/legal review downstream should confirm this framing is acceptable]`
- A right-to-access/data-export capability is explicitly out of scope for this feature — only erasure (deletion) is included. `[confidence: high — human-confirmed]`

## Non-Functional Requirements

- **Security:** Authentication is email/password-based. `[confidence: high — human-confirmed]` How passwords/session tokens are hashed, stored, and transmitted is implementation detail for the Architecture Agent, not this PRD — but at the product level, passwords must never be stored, logged, or exposed in plain text anywhere (UI, logs, artifacts). `[confidence: high — baseline expectation, not a discretionary product choice]` Sessions persist across browser restarts for up to 30 days of inactivity (a "remembered" session, not re-login on every visit), and a "sign out everywhere" capability is included as a product requirement — useful account-safety self-service, e.g. after a lost device. `[confidence: medium — default proposed by requirements agent, pending human sanity-check; exact session mechanics (token type, refresh model) remain Architecture Agent implementation detail]` The base PRD's existing Security NFR ("no auth and no PII keep the risk surface small by design," §10) is explicitly superseded by this feature.
- **Privacy:** GDPR/CCPA-style right-to-erasure applies (see FR-6). `[confidence: high — human-confirmed]` Identity data collected is limited to email address and password (never stored in plaintext) plus the favorites list itself — no other PII is implied by email/password login. The product is not specifically targeting or serving EU or California users, so GDPR/CCPA are not statutorily triggered here — erasure is adopted as a voluntary privacy baseline/best practice rather than a legal compliance requirement. `[confidence: high — human-confirmed]` The base PRD's Privacy NFR (§10) is written entirely on the "no accounts" premise and does not carry over.
- **Availability/degradation:** If the sync backend is unreachable, the Favorites view falls back to a last-known-synced local copy (read-only) rather than an error state, mirroring the base product's "never leave the user at a dead end" pattern (base FR-2, FR-6). `[confidence: medium — default proposed by requirements agent, pending human sanity-check]`
- **Performance:** Favorites appear within 3 seconds of sign-in completing, under normal network conditions. Treated as a target, not a hard launch gate. `[confidence: low — a placeholder number, not derived from any stated requirement; needs human confirmation or a better-informed target]`
- **Compatibility with base NFRs:** This feature must not change the base PRD's Performance, Accessibility, Availability, or Browser-support NFRs (§10) for any flow that doesn't touch favorites/sign-in. `[confidence: high — directly follows from Problem Statement's scope boundary]`

## Assumptions

*All items below were originally raised in draft-first review. The human asked for a batch pass at reasonable defaults rather than resolving each individually — every resolution below is a requirements-agent default, not a human-authored decision, and is flagged as pending sanity-check rather than confirmed. Two items that were flagged Security/PII (data-access rights, GDPR/CCPA jurisdiction) were deliberately excluded from this default-pass and remain in Open Ambiguities for explicit human resolution instead.*

1. **[confidence: medium — default, pending sanity-check]** (FR-1) — Favorite toggle appears on both list items and the Spot Detail Panel; map markers keep today's behavior (open detail panel), no separate on-marker toggle.
2. **[confidence: medium — default, pending sanity-check]** (FR-2) — Favorites list default sort order is recency-favorited (most recent first).
3. **[confidence: medium — default, pending sanity-check]** (FR-4) — Sync is sync-on-load, not real-time push across simultaneously open devices.
4. **[confidence: medium — default, pending sanity-check]** (NFR Availability) — On sync-backend outage, the Favorites view degrades to a last-known-synced local read-only copy rather than an error state.
5. **[confidence: medium]** — A user who favorites a spot without being signed in is prompted to sign in at that point, rather than the favorite being held locally and migrated after a later sign-in. *(Retained from initial draft — already medium confidence, not part of this batch pass.)*
6. **[confidence: medium]** — This feature is scoped for the same web-only, evergreen-browser platform as the base product (base PRD §10, §12) — no native app implied. *(Retained from initial draft.)*
7. **[confidence: low — default, pending sanity-check]** (NFR Performance) — Target: favorites appear within 3 seconds of sign-in completing, under normal network conditions; not a hard launch gate.
8. **[confidence: medium — default, pending sanity-check]** (FR-3) — Password-reset ("forgot password") is in scope as a companion to email/password login.
9. **[confidence: medium — default, pending sanity-check]** (FR-3) — Email verification is required at registration; sign-in works unverified, but favorites/sync stay disabled until confirmed.
10. **[confidence: medium — default, pending sanity-check]** (NFR Security) — Sessions persist up to 30 days of inactivity; a "sign out everywhere" capability is included.

## Open Ambiguities

No `[Needs Clarification]` items remain — the two Security/PII flags (sign-in mechanism, data-retention/erasure posture) and the follow-on jurisdiction and data-access-right questions they raised are all resolved above.

Two product-direction questions remain, neither of which this PRD can resolve on its own:

- Whether this feature is even in scope for the product's current stage — the base PRD frames "no accounts" as a hackathon-MVP simplification that may or may not be worth revisiting depending on whether the product continues past the hackathon (base PRD §9, high-confidence ambiguity). This PRD assumes "yes, worth scoping," but that framing decision sits above this document and belongs to whoever owns product direction now.
- Whether an existing identity provider is available to build on (e.g., an org SSO, a third-party auth service already in use elsewhere) versus needing to stand up sign-in from scratch — materially changes both scope and the Security NFR answer above, and isn't something this PRD can infer.
