---
stage: design
version: v2
status: DRAFT
agent: web-design-agent
approver: pending
timestamp: 2026-09-07T00:00:00
supersedes: none
---

# Design: Favorite Spots & Cross-Device Sync (Parking Spot Finder)

## Source

- **PRD:** `artifacts/requirements/prd_v1.md` (`status: APPROVED`, approved 2026-09-07 with the Gate 1 Approval Note's four live gaps knowingly unresolved — see "Flagged decisions" below).
- **Relationship to prior design:** This artifact is **additive** to `artifacts/design/design_v1.md` (`status: APPROVED`) — it does not replace or restate v1's six base-product screens, which remain valid and unchanged for a signed-out/non-favoriting user. It adds new screens (sign-in, favorites) and amends two existing v1 screens (Home Results, Spot Detail Panel) to add a favorite affordance.
- **Design system:** No new system defined here, per constraint. This artifact continues v1's approach: it targets the **same live Claude Design workspace/canvas that already contains the v1 "Parking Spot Finder" artboards**, and instructs Claude Design to inherit whatever system is already active there — component conventions, color/typography tokens, nav shell — rather than pin or invent one. `[confidence: high — directly per coordinator instruction]`

## Flagged decisions carried from the PRD (not silently resolved)

The PRD's Gate 1 Approval Note lists four items as live gaps, not settled requirements. Screen work below unavoidably has to render *something* concrete for three of them. Each concrete choice below is marked low/medium confidence and called out again inline at the screen it affects — treat these as proposals awaiting your sign-off, not as decided UX:

1. **FR-1 vs FR-3 (signed-in-but-unverified user + favorite control).** Screens 9 (Email Verification) and the amendments to Screens 2A/5A/12 render the favorite toggle as **visible but disabled**, with a tooltip/toast explaining verification is required. `[confidence: low — resolves an explicitly unresolved PRD conflict for the sake of having a drawable state; an alternative (hide the control entirely for unverified users) is equally consistent with the PRD text and should be confirmed, not assumed]`
2. **FR-6 vs Availability NFR (local cached copy on deletion).** Screen 14 (Delete Account) does **not** claim in its UI copy that favorites are erased from every device's local cache — only that the account's server-side credentials and favorites record are deleted. It additionally proposes clearing the *current* device's local cache and signing that device out as a client-side action taken at deletion time. `[confidence: low — the PRD specifies no purge step for other already-signed-in devices; this UI cannot promise something the requirement doesn't guarantee, so it says less than a user might expect. Needs a product decision, not a design one, on whether other devices' caches should be invalidated on next open.]`
3. **FR-6 erasure timeframe (three inconsistent PRD statements).** No screen in this artifact states a specific number of days anywhere in user-facing copy. Screen 14 uses non-committal language ("you'll get a confirmation email once this is complete"). `[confidence: high — deliberately avoids hardcoding an unsettled number, per explicit instruction]`. **This is a UI-specific gap worth flagging back to you**: if you want the deletion-confirmation screen to state a concrete SLA to the user (common pattern: "your data will be deleted within N days"), that requires picking one of the three PRD numbers first — that's a product decision, not mine to make by drafting copy.
4. **Ten Assumptions-section defaults.** Treated as final per the Gate 1 note (sort order, sync-on-load, session length, verification-gates-favorites, etc.) — used directly below without re-flagging each individually, except where a screen also touches one of the three items above.

**Not rendered anywhere, and flagged rather than invented:** the PRD's own review (T-3/C-2) notes no rule is stated for what happens when a stale, already-open second device acts on a favorite that was already changed elsewhere, nor whether a favorite *write* attempted during a sync-backend outage is blocked, queued, or rejected. Screens 2A/5A/12 (offline state) show the write control as **disabled with an inline "can't update favorites right now — you're offline" message** rather than silently failing or queuing. `[confidence: low — default proposed to satisfy T-4/C-2's gap; queuing-for-replay is an equally plausible reading of the PRD and would change this screen's behavior; needs confirmation]`

## Screens

### Amendments to v1 screens

#### 2A. Home Results — Favorite toggle (amends v1 Screen #2)
**Purpose:** Let a signed-in, verified user favorite/unfavorite a spot directly from the list, without opening the detail panel.
**Key UI Elements added:** A favorite icon (use the design system's existing saved/bookmark icon convention if one exists in the active workspace; otherwise a simple outline/filled toggle icon) on each list row, positioned so it doesn't collide with the existing Free/Paid badge. Filled = favorited, outline = not favorited.
**User Actions:** Tap icon → toggles favorite state in place, no confirmation (symmetric, non-destructive per FR-1). Map markers are unchanged from v1 (open Spot Detail Panel; no separate on-marker toggle, per PRD Assumption 1).
**Navigation:** No navigation change — stays on Home Results.
**Realizes:** FR-1, FR-5, Assumption 1; UJ-1 (Ritu).
**States (new, beyond v1's default/loading/map-unavailable):**
- *Signed-out:* tapping the icon opens Screen 11 (Sign-In Prompt) instead of toggling. `[confidence: medium — PRD Assumption 5]`
- *Signed-in, unverified:* icon shown disabled/dimmed; tap shows a toast/tooltip "Verify your email to save favorites." See Flagged Decision 1.
- *Sync-backend outage:* icon shown disabled; tap shows inline "Can't update favorites right now — you're offline." See the unrendered-gap note above.

#### 5A. Spot Detail Panel — Favorite toggle (amends v1 Screen #5)
**Purpose:** Same favorite/unfavorite capability as 2A, from the detail panel.
**Key UI Elements added:** Same favorite icon, placed near the panel's existing Route action — a secondary action, not competing with Route as the primary CTA.
**User Actions / States:** Identical behavior and identical signed-out/unverified/outage states to 2A.
**Navigation:** No change to v1's dismiss/Route behavior.
**Realizes:** FR-1, FR-5; UJ-1 (Ritu).

---

### New screens

#### 7. Sign In
**Purpose:** Let a returning user establish their identity to enable favorites/sync.
**Key UI Elements:** App shell consistent with Home (same header/logo treatment as v1 Screen #1), email field, password field, "Sign in" primary action, "Forgot password?" link, "Create an account" link, an explanatory line that signing in is optional and only needed for favorites (reinforces FR-3's "opt-in, not a gate on the core product").
**User Actions:** Submit valid credentials → signed in, returns to whichever screen triggered sign-in (Home Results, Favorites List, or Screen 11's prompt). Submit invalid credentials → inline error, no navigation. Tap "Forgot password?" → Screen 10. Tap "Create an account" → Screen 8.
**Navigation:** Entered from the account entry point in the Home nav shell (new persistent header icon, `[confidence: medium — reasonable placement, not specified by the PRD]`), or from Screen 11 (Sign-In Prompt), or from a "Sign in" link on the Favorites List's signed-out state.
**Realizes:** FR-3; UJ-1 (Ritu).
**States:** default, submitting (loading), invalid-credentials (error), account-not-found — treated as the same generic invalid-credentials error, not a distinct enumeration message, to avoid confirming which emails are registered. `[confidence: medium — standard security practice, not stated in PRD]`

#### 8. Register / Create Account
**Purpose:** Let a new user establish an email/password identity.
**Key UI Elements:** Email field, password field, password-confirmation field, "Create account" primary action, link back to Sign In, brief note that a verification email will be sent.
**User Actions:** Submit → account created, user is signed in immediately (per FR-3, unverified accounts can sign in), routes to Screen 9 (Email Verification pending state). Validation errors (weak password, email already registered, mismatched confirmation) shown inline.
**Navigation:** Entered from Screen 7 or Screen 11. → Screen 9 on success.
**Realizes:** FR-3, Assumption 9; UJ-1 (Ritu).
**States:** default, submitting, validation-error.

#### 9. Email Verification (pending state)
**Purpose:** Communicate that the account exists and works, but favorites/sync stay off until the emailed link is confirmed — directly surfaces the FR-1/FR-3 conflict as a visible, explained state rather than a silent gate.
**Key UI Elements:** Confirmation that an email was sent to the entered address, "Resend verification email" action, explicit copy: "You're signed in, but favoriting spots is disabled until you verify your email." A persistent, dismissable-but-reappearing banner carrying the same message should also appear across Home/Favorites while the account remains unverified — not just on this one screen.
**User Actions:** Tap "Resend" → re-sends, rate-limited (`[confidence: low — no PRD detail on resend limits]`). Clicking the emailed link (external, opens this app) → transitions to a brief "Email verified" confirmation, then into Home Results with the favorite toggle now enabled.
**Navigation:** Entered automatically after Screen 8. Dismiss/continue → Home Results (favorites remain disabled until verified).
**Realizes:** FR-3, Assumption 9; the FR-1/FR-3 conflict (Flagged Decision 1) — this screen exists specifically because that conflict has to be shown, not hidden.
**States:** pending (default), resent (confirmation toast), verified (transitional confirmation before redirect).

#### 10. Forgot / Reset Password
**Purpose:** Let a user regain account access without support intervention.
**Key UI Elements:** Step 1 (request): email field, "Send reset link" action. Step 2 (reached via emailed link, shown as a distinct state of the same screen): new-password field, confirm-password field, "Reset password" action.
**User Actions:** Step 1 submit → generic "If that email is registered, we've sent a reset link" confirmation (doesn't reveal whether the address exists, same reasoning as Screen 7's error handling). Step 2 submit → password updated, redirects to Screen 7 with a success message.
**Navigation:** Entered from Screen 7. → Screen 7 after successful reset.
**Realizes:** FR-3, Assumption 8; UJ-1 (Ritu).
**States:** request-default, request-submitted (confirmation), reset-default, reset-error (e.g., expired link), reset-success.

#### 11. Sign-In Prompt (modal, overlay on Home Results / Spot Detail Panel)
**Purpose:** Handle a signed-out user tapping the favorite toggle, per PRD Assumption 5 (prompt at point-of-action, no local-then-migrate favorite).
**Key UI Elements:** Short modal — "Sign in to save favorites" headline, brief one-line reason, "Sign in" and "Create account" actions, dismiss control. Consistent modal/overlay treatment with v1's Spot Detail Panel (same overlay pattern, not a new interaction paradigm).
**User Actions:** Tap Sign in → Screen 7 (returns to originating spot after success, and the favorite is *not* auto-applied retroactively — the user must tap the toggle again after signing in, since the PRD's assumption explicitly rejects local-hold-and-migrate). Tap Create account → Screen 8. Dismiss → returns to underlying screen, no favorite recorded.
**Navigation:** Triggered from 2A/5A only when signed-out. Not a standalone entry point.
**Realizes:** FR-1, Assumption 5; UJ-1 (Ritu), UJ-2 (Arjun — confirms he's never forced through this, since he never taps favorite).
**States:** default only.

#### 12. Favorites List
**Purpose:** Dedicated view of every spot the signed-in user has favorited, per FR-2.
**Key UI Elements:** Same shell/nav as Home. List rows matching the base list format (name, Free/Paid badge, address, operating hours — same fields as v1 Screen #2, per FR-2's consequence) plus the same favorite toggle as 2A (filled, since everything here is favorited by definition) so removal is one tap. Sort order is fixed (recency-favorited, most recent first, per Assumption 2) with no user-facing sort control, since the PRD doesn't grant one.
**User Actions:** Tap a row → opens Spot Detail Panel (v1 Screen #5/5A), same as Home Results (FR-2's consequence: no separate detail UI). Tap the favorite icon on a row → removes it from the list immediately (FR-5), no confirmation.
**Navigation:** Reachable from a persistent nav entry on Home (new "Favorites" nav item, `[confidence: medium — PRD says "reachable from Home," exact placement e.g. tab/header link is not specified]`). → Spot Detail Panel on row tap; → Screen 7 (Sign In) if reached while signed out.
**Realizes:** FR-2, FR-4, FR-5, Assumption 2, Assumption 3; UJ-1 (Ritu) — this is Ritu's primary new screen.
**States (the PRD explicitly requires the working prototype to cover degraded modes, not just happy path):**
- *Signed-out:* the nav entry is still visible (so a curious signed-out user can discover the feature) but tapping it routes to Screen 7 rather than showing an empty list, with a one-line explainer ("Sign in to see your favorite spots").
- *Loading:* skeleton/spinner while the sync-on-load fetch is in flight (Assumption 3) — target under 3 seconds per the NFR, though that NFR is explicitly a non-binding placeholder target, not something this screen enforces or displays a countdown against.
- *Empty (signed in, verified, zero favorites):* "You haven't favorited any spots yet" message, no spots to show — distinct from the signed-out empty message above.
- *Offline / sync-backend unreachable (NFR Availability, Assumption 4):* shows the **last-known-synced local copy**, read-only — favorite-removal icons on this screen are disabled in this state, with the same inline "you're offline" messaging as 2A/5A, plus a persistent banner at the top of the list itself: "Showing your last synced favorites. Some info may be out of date." This state is explicitly required by the NFR — not optional polish.
- *Error (a genuine failure distinct from "unreachable," e.g. an authenticated request that 4xx/5xx's rather than times out):* `[confidence: low — the PRD's Availability NFR only describes the offline/unreachable case, not a distinct authenticated-error case; this state is a reasonable prototype-completeness addition, not a stated requirement — flagged for confirmation, not silently assumed to be identical to the offline state]` a simple retry-affordance error message, separate from the offline read-only view, since an error is not the same guarantee as "we have a valid last-synced copy."

#### 13. Account / Settings
**Purpose:** Home for account-level actions the PRD requires but doesn't attach to any other screen: sign out, sign out everywhere, verification status, delete account.
**Key UI Elements:** Signed-in email address (read-only display), verification status indicator (verified / "resend verification" if not), "Sign out" action, "Sign out everywhere" action (NFR Security, Assumption 10), "Delete account" action (routes to Screen 14, styled as a destructive/lower-emphasis action, separated from the others).
**User Actions:** Sign out → clears this device's session, returns to signed-out Home. Sign out everywhere → confirms, then invalidates all sessions (per NFR Security) and signs this device out too, with a brief explanatory message ("You've been signed out of all devices"). Delete account → Screen 14.
**Navigation:** Reachable from the same account entry point in the Home nav shell as Screen 7 (shown here instead of Sign In once the user is signed in).
**Realizes:** FR-3 (sign out), NFR Security (session persistence / sign-out-everywhere, Assumption 10); UJ-1 (Ritu).
**States:** default only — no PRD-specified empty/loading/error variant beyond standard action-confirmation feedback.

#### 14. Delete Account
**Purpose:** Let a user request account and favorites-data erasure, per FR-6.
**Key UI Elements:** Clear warning that this is permanent ("This can't be undone" — matching FR-6's "irrecoverably deleted, not merely hidden" framing, without attaching a specific day-count), a re-authentication step (password re-entry, `[confidence: medium — reasonable friction for an irreversible action, not specified by the PRD]`), a final "Delete my account" destructive action, a cancel/back option.
**User Actions:** Confirm with password → request submitted; user is immediately signed out of this device and this device's local favorites cache is cleared (see Flagged Decision 2 — this is as far as this screen's copy commits to, since the PRD specifies no cross-device purge). Cancel → returns to Screen 13, no change.
**Navigation:** Entered only from Screen 13. → signed-out Home on completion, with a confirmation message.
**Realizes:** FR-6, NFR Privacy; UJ-1 (Ritu).
**States:** default (warning + re-auth), submitting, confirmed (post-delete message — deliberately non-committal on timing, per Flagged Decision 3), error (re-auth failed, shown inline, no navigation).

## Claude Design Prompt

*Paste this into the same live Claude Design canvas/workspace that already contains the v1 "Parking Spot Finder" artboards (Location Permission, Home Results, Zero Results, Manual Area Entry, Spot Detail, and optionally Search Refinement). This prompt adds new artboards to that existing flow and references the design system already active there — do not introduce new colors, typography, or components.*

```
Continue building on the existing "Parking Spot Finder" canvas in this workspace. Use exactly the design system, color/typography tokens, nav shell, and component conventions already established by the existing artboards (Home — Location Permission, Home — Results, Home — Zero Results, Manual Area Entry, Spot Detail Panel) — do not introduce new tokens, new components, or a different visual style.

This adds a "Favorite Spots & Cross-Device Sync" feature on top of that existing product. Add these new artboards, connected by arrows to each other and to the existing artboards where noted:

1. AMEND: HOME — RESULTS (the existing artboard)
   Add a favorite icon to each list row (filled = favorited, outline = not favorited), positioned so it doesn't collide with the existing Free/Paid badge. Use the design system's existing bookmark/save icon convention if one exists; otherwise a simple heart or star outline/fill toggle consistent with the existing icon style. Do not add a favorite control to map pins — tapping a pin still opens the Spot Detail Panel as today.

2. AMEND: SPOT DETAIL PANEL (the existing artboard)
   Add the same favorite icon near the existing "Route" button, as a secondary action (Route remains the primary CTA).

3. SIGN IN
   Same header/logo treatment as the existing Home artboards. Email field, password field, primary "Sign in" button, "Forgot password?" link, "Create an account" link, and a one-line note that signing in is optional and only needed to save favorites.

4. CREATE ACCOUNT
   Email field, password field, confirm-password field, primary "Create account" button, link back to Sign In, and a note that a verification email will be sent.

5. EMAIL VERIFICATION (PENDING)
   Confirmation that a verification email was sent, a "Resend email" link/button, and explicit copy: "You're signed in, but favoriting spots is disabled until you verify your email." Show this same message as a dismissible banner overlaid at the top of a Home Results-style background, to indicate it persists across the app while unverified.

6. FORGOT PASSWORD
   Two states of one artboard (show both side by side, labeled "Step 1" and "Step 2"): Step 1 has an email field and "Send reset link" button; Step 2 (reached after clicking an emailed link) has a new-password field, confirm-password field, and "Reset password" button.

7. SIGN-IN PROMPT (modal/overlay, shown on top of the Home Results artboard)
   A small modal: "Sign in to save favorites" headline, one supporting line, "Sign in" and "Create account" buttons, and a dismiss control. Use the same overlay/modal treatment as the existing Spot Detail Panel.

8. FAVORITES LIST — default state
   Same shell/nav as Home Results, with a new persistent "Favorites" nav entry visible from Home. List rows in the same format as Home Results (name, Free/Paid badge, address, operating hours) plus the same favorite icon (shown filled, since everything here is already favorited) for one-tap removal. No sort/filter controls on this screen.

9. FAVORITES LIST — empty state
   Same shell, with a centered message: "You haven't favorited any spots yet."

10. FAVORITES LIST — offline / read-only state
    Same shell, with a banner at the top of the list: "Showing your last synced favorites. Some info may be out of date." Favorite-removal icons on this screen appear visibly disabled/dimmed in this state.

11. FAVORITES LIST — signed-out state
    Same shell, with the list area replaced by a short message: "Sign in to see your favorite spots" and a "Sign in" button.

12. ACCOUNT / SETTINGS
    Signed-in user's email address, a verification status line, "Sign out" button, "Sign out everywhere" button, and a visually separated, lower-emphasis "Delete account" button (destructive action, styled distinctly from the others).

13. DELETE ACCOUNT
    A clear warning that this action is permanent and cannot be undone, a password re-entry field to confirm identity, a destructive "Delete my account" button, and a "Cancel" option. Do not include any specific number of days in this artboard's copy — keep the confirmation message generic (e.g., "You'll receive a confirmation email once this is complete").

Interaction notes to reflect visually (labels/annotations, not functional code):
- Tapping the favorite icon on Home Results or Spot Detail Panel while signed out opens artboard 7 (Sign-In Prompt) instead of toggling.
- Tapping the favorite icon while signed in but unverified shows the icon in a disabled/dimmed style with a small tooltip: "Verify your email to save favorites."
- Tapping "Sign in" on artboard 3 with valid credentials leads back to whichever screen was in use, or to Home Results by default.
- Tapping "Create account" on artboard 4 leads to artboard 5 (Email Verification Pending).
- The new "Favorites" nav entry on Home Results leads to artboard 8 (or 9/10/11 depending on state, shown here as separate labeled artboards rather than a single interactive one).
- The account entry point in the Home nav shell leads to artboard 3 (Sign In) when signed out, or artboard 12 (Account/Settings) when signed in.

Do not add: social sharing of favorites, reviews/ratings, drag-to-reorder of favorites, a data-export/download screen, or any admin/management surface — all explicitly out of scope for this feature.
```

## Open items for the human before running this prompt

- `[NOTE FOR PM]` Confirm or override **Flagged Decision 1** (favorite control shown disabled-with-tooltip vs. hidden entirely for signed-in-unverified users) — this directly resolves the PRD's live FR-1/FR-3 conflict at the UI layer, and should be a deliberate choice, not an artifact of how I happened to draw it.
- `[NOTE FOR PM]` Confirm or override **Flagged Decision 2** (Delete Account clears only this device's local cache; other signed-in devices' cached favorites are left unaddressed by both the PRD and this design) — this is the FR-6/Availability-NFR conflict surfacing as an actual UX gap, not just a spec inconsistency.
- `[NOTE FOR PM]` **Flagged Decision 3**: if you want the Delete Account confirmation screen to state a specific erasure timeframe to the user, tell me which of the PRD's three numbers (≤30 days / 45 days / other) to use, or confirm you're fine with the generic "confirmation email once complete" copy as drafted.
- `[NOTE FOR PM]` The offline-write behavior on Screens 2A/5A/12 (disabled with inline "you're offline" message) is one plausible reading of an unstated PRD gap (T-4/C-2) — an equally valid alternative is to let the tap succeed locally and queue for replay once reconnected. Confirm which you want before treating this as final.
- `[NOTE FOR PM]` Nav placement for the account entry point and the "Favorites" nav item (both `[confidence: medium]`) — adjust freely in Claude Design if you have a stronger preference than what's described above.
