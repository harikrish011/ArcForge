---
stage: requirements
version: v1
status: DRAFT
agent: prd-review-agent
approver: pending
timestamp: 2026-09-07T00:00:00
supersedes: none
---

# PRD Review: prd_v1.md — Favorite Spots & Cross-Device Sync

| Field | Value |
|---|---|
| PRD Reviewed | prd_v1.md |
| Reviewer | prd-review-agent (opus) |
| Date | 2026-09-07 |
| Checks Run | All three (testability, unresolved-assumption leakage, cross-requirement conflicts) — no subset requested |

Scope note: this review checks only testability, assumption leakage, and cross-requirement conflicts. Prose, structure, and requirement coverage/"missing NFRs" are deliberately not assessed. Sections of the PRD not named below had no finding in these three categories.

---

## Testability findings

### T-1 — FR-6 states two different erasure timeframes and calls both "the" target
**As written:** FR-6 body — "erased within a compliant timeframe." FR-6 consequence — "GDPR's 'without undue delay' (≤30 days) and CCPA's 45-day window are not legally binding here, but are adopted as the voluntary target timeframe anyway." A third consequence says a deletion request "results in" irrecoverable deletion, with no timing at all.

**Why it fails:** Three different timing statements for one requirement, and the phrase "the voluntary target timeframe" (singular) is attached to two different numbers. A QA or user-story agent cannot derive a single acceptance criterion — 30 days and 45 days produce different pass/fail results, and "results in" reads as immediate.

**What would resolve it:** One stated number for the erasure SLA (e.g., "credentials and favorites are irrecoverably deleted within N days of a confirmed deletion request"), and whether the user-visible confirmation is immediate even if backend erasure is asynchronous.

### T-2 — NFR Performance has no pass/fail condition and an unquantified precondition
**As written:** "Favorites appear within 3 seconds of sign-in completing, under normal network conditions. Treated as a target, not a hard launch gate." `[confidence: low — a placeholder number]`

**Why it fails:** Two independent problems. "Normal network conditions" is unquantified, so the measurement setup is unspecified. More fundamentally, "a target, not a hard launch gate" means the requirement cannot fail — a QA agent handed this has no criterion to assert against, only a number to report. The PRD's own confidence flag concedes the number is a placeholder.

**What would resolve it:** A stated network/device baseline for measurement, and an explicit decision on whether this is an asserted threshold or an observability metric. If it is genuinely not a gate, saying so as a non-binding metric (rather than in the NFR list alongside binding items) would stop a downstream agent from writing a test that enforces it.

### T-3 — FR-4/FR-5 specify a sync outcome but not resolution behavior for the divergence the PRD itself creates
**As written:** FR-4 — sync is "bidirectional," and a change on one device "is reflected on another the next time that other device is opened while signed in"; and explicitly "not a real-time push while multiple devices are simultaneously open." FR-5 — "Removal syncs across devices the same way additions do."

**Why it fails:** The PRD deliberately chooses sync-on-load and deliberately excludes real-time push, which guarantees a window where a second, already-open device holds a stale list. If the user acts on that stale list (re-favorites a spot removed on device A, or removes one already removed), the resulting state is not stated anywhere. A QA agent would have to invent last-write-wins, union-merge, or a refresh-on-action rule to test FR-4/FR-5 at all.

**What would resolve it:** A one-line conflict rule for divergent favorite/unfavorite actions across sessions (e.g., last action wins by timestamp; or the list is re-fetched before any write). This is a product-visible outcome, not solely an architecture detail — the user sees whether their removal "came back."

### T-4 — Behavior of the favorite control is unspecified in two states the PRD explicitly creates
**As written:** FR-1 — "a signed-in user can mark or unmark a Parking Spot as a favorite with a single action," with the affordance "directly on each list item." FR-3 — an unverified account "can sign in but favorites/sync stay disabled." NFR Availability — during a sync-backend outage the Favorites view is a "last-known-synced local copy (read-only)."

**Why it fails:** FR-1 describes only the success path. The PRD creates three other states — signed-out, signed-in-but-unverified, and backend-unreachable — and states no behavior for the favorite control in any of them (hidden? disabled? tapped and prompts? tapped and queues? tapped and errors?). Each is an objectively checkable behavior that a downstream agent must currently guess. See also C-1, C-2 (conflicts) and L-4 (the signed-out case exists only as an unconfirmed assumption).

**What would resolve it:** One consequence line per state under FR-1 stating what the control does.

### T-5 — Applicability of base accessibility/browser NFRs to the new surfaces is ambiguous by explicit wording
**As written:** "This feature must not change the base PRD's Performance, Accessibility, Availability, or Browser-support NFRs (§10) **for any flow that doesn't touch favorites/sign-in**."

**Why it fails:** The scoping clause is explicit, and by exclusion it leaves undefined whether those base NFRs apply to the new surfaces this feature introduces (Favorites view, registration, sign-in, password reset, verification). A downstream agent cannot tell from this sentence whether the base accessibility and browser-support bars are inherited by the new screens or intentionally not applied to them. This is a question of which existing requirement governs, not a request for a new NFR.

**What would resolve it:** State whether base §10 Accessibility/Browser-support carry over unchanged to favorites and sign-in surfaces.

---

## Assumption leakage findings

### L-1 — Assumptions preamble and Open Ambiguities directly contradict each other on whether the two Security/PII follow-ons are still open
**Where:** Assumptions preamble — "Two items that were flagged Security/PII (data-access rights, GDPR/CCPA jurisdiction) were deliberately excluded from this default-pass and **remain in Open Ambiguities for explicit human resolution instead**." Open Ambiguities — "No `[Needs Clarification]` items remain — the two Security/PII flags ... and the follow-on jurisdiction and data-access-right questions they raised are all resolved above."

**What's inconsistent:** The document asserts both that these two items remain open pending human resolution and that they are resolved. Neither appears in the Open Ambiguities list, so on the preamble's reading, two Security/PII items were routed to a section that does not contain them — the exact "dropped marker" pattern this check exists to catch. A Gate 1 reviewer reading only one of the two sections gets the opposite impression from a reviewer reading the other.

**What needs to go back to the human:** Confirmation that both items were in fact resolved by the human (out-of-band context suggests they were), and correction of the stale preamble sentence so the document's own record of its decision state is internally consistent. Until that is fixed, the PRD cannot be relied on as ground truth about which Security/PII questions are settled.

### L-2 — Jurisdiction claim is marked `human-confirmed` with no record of the human's input, and it is load-bearing for two downgrades
**Where:** Privacy NFR — "The product is not specifically targeting or serving EU or California users, so GDPR/CCPA are not statutorily triggered here — erasure is adopted as a voluntary privacy baseline/best practice rather than a legal compliance requirement." `[confidence: high — human-confirmed]` Also FR-6 and the Security NFR carry `human-confirmed` tags.

**What's inconsistent:** Three things. (a) No `human-confirmed` tag anywhere in the document is accompanied by any record of what the human actually said, and the Source section states the opposite — "No human product context was supplied for this specific feature ... every requirement below is my own proposal rather than a stated ask." A reader cannot distinguish confirmed decisions from agent proposals except by trusting an untraceable tag. (b) This particular claim is a factual assertion about the product's market, not a product preference, and two consequential decisions rest on it: erasure downgraded from legal obligation to voluntary baseline, and the right-to-access/export capability excluded from scope. If the claim is wrong or changes, both decisions need revisiting. (c) The same claim carries inconsistent confidence in the two places it appears: FR-6 flags it `[confidence: medium — ... the architecture/legal review downstream should confirm this framing is acceptable]`, while the Privacy NFR presents it as `high — human-confirmed` and settled. A downstream agent reading only the NFR section would never learn that a legal confirmation was requested.

**What needs to go back to the human:** Explicit re-confirmation of the jurisdiction statement, with the human's own words recorded in the document; and reconciliation of the two conflicting confidence tags. Note also that FR-6 parks a legal determination on "the architecture/legal review downstream" — under `docs/agent-protocol.md` §5, the Architecture Agent is read-only + generate and has no authority to make that call, so as written this question has no owner.

### L-3 — Ten assumptions are documented as unconfirmed defaults but are stated as settled requirements in the body
**Where:** Assumptions preamble — "every resolution below is a requirements-agent default, not a human-authored decision, and is flagged as pending sanity-check rather than confirmed." The same items then appear in the requirement body as flat obligations: FR-3 — "Email verification **is required** at registration"; NFR Security — "Sessions persist across browser restarts for up to 30 days of inactivity ... and a 'sign out everywhere' capability **is included** as a product requirement"; NFR Availability — the read-only degradation behavior; FR-2 — sort order; FR-4 — sync-on-load.

**What's inconsistent:** The inline confidence flags are present and honest, which mitigates this considerably. The residual problem is that these unconfirmed defaults are not equal in weight: Assumption 9 (email verification) is stated in FR-3 as a hard gate on the entire feature — an unverified user has no favorites and no sync — and Assumption 3 (sync-on-load) determines the whole sync model that FR-4, FR-5, and T-3 above depend on. Downstream agents will build a schema, an auth flow, and a verification-gated capability on top of items the document itself labels "pending sanity-check."

Separately, out-of-band context indicates the human has since said they are comfortable treating these defaults as final. **The document does not record that.** The preamble still says "pending sanity-check," all ten items still carry pending flags, and the version history has a single "Initial draft" row with no entry for the confirmation. As it stands, the artifact presents ten open items to Gate 1 that are, in reality, closed — which is the same traceability failure as L-2 in the opposite direction.

**What needs to go back to the human:** Record the confirmation in the document (preamble text, confidence flags, and a version-history row), so the artifact's stated decision state matches the actual one. If the human wants the two highest-leverage defaults — email verification as a feature gate (Assumption 9) and sync-on-load (Assumption 3) — looked at individually before being locked, that should happen before Gate 1 rather than after, since both shape FR-3 and FR-4 rather than merely detailing them.

### L-4 — Assumption 5 carries behavior that no functional requirement states
**Where:** Assumption 5 `[confidence: medium]` — "A user who favorites a spot without being signed in is prompted to sign in at that point, rather than the favorite being held locally and migrated after a later sign-in." Retained from the initial draft and explicitly not part of the batch default pass.

**What's inconsistent:** FR-1 grants the capability only to "a signed-in user," while FR-1's consequence places the affordance "directly on each list item" — surfaces a signed-out user sees. The only description of what happens when a signed-out user activates that control lives in a medium-confidence assumption, referenced by no requirement. This assumption also silently decides a real product question — whether pre-sign-in favorites are preserved through registration — which affects the first-run experience and the data model.

**What needs to go back to the human:** Confirm the prompt-at-point-of-action behavior and the no-local-migration decision, and promote whichever is chosen into FR-1 or FR-3 as a stated consequence rather than leaving it in Assumptions.

### L-5 — An open ambiguity is stated to change the Security NFR, which is presented as confirmed
**Where:** Open Ambiguities — "Whether an existing identity provider is available to build on ... versus needing to stand up sign-in from scratch — **materially changes both scope and the Security NFR answer above**, and isn't something this PRD can infer." Security NFR / FR-3 — presented as settled, `[confidence: high — human-confirmed]`.

**What's inconsistent:** The PRD states on its own authority that an unresolved question materially changes the Security NFR, and then presents that NFR as confirmed and final. Both cannot be true for a Gate 1 reviewer. The likely reality is narrower than the wording implies — the human confirmed *email/password* as the mechanism, leaving only build-vs-buy open — but as written the document says the security answer itself is provisional.

**What needs to go back to the human:** Either narrow the ambiguity to what is genuinely open (build-vs-buy, which is an architecture input) or downgrade the Security NFR's confidence to match. As written, the two statements cannot both be relied on.

---

## Cross-requirement conflicts

### C-1 — FR-1 vs FR-3: does a signed-in but unverified user have favorites?
- FR-1: "a **signed-in user** can mark or unmark a Parking Spot as a favorite with a single action."
- FR-3: "an unverified account **can sign in** but favorites/sync **stay disabled** until the email is confirmed."

Both are stated as unconditional consequences. FR-1's precondition is "signed in"; FR-3 creates a class of users who are signed in and have no favorites capability. As written, FR-1 grants what FR-3 withholds, for the same user, in the same state. A downstream agent implementing FR-1 literally would ship an ungated toggle; one implementing FR-3 literally would gate it. Note that the gating side of this conflict originates in Assumption 9, a `pending sanity-check` default (see L-3), so this conflict should be resolved by deciding the assumption, not by patching FR-1.

### C-2 — FR-1/FR-5 vs NFR Availability: writes during a sync-backend outage
- FR-1 / FR-5: a signed-in user can mark, unmark, and remove favorites — stated with no availability precondition.
- NFR Availability: on an unreachable sync backend, the Favorites view "falls back to a last-known-synced **local copy (read-only)**."

"Read-only" negates FR-1's and FR-5's capability for the duration of the outage, and no requirement states which governs or what the user sees when they try. The NFR's stated intent — "never leave the user at a dead end" — is also in tension with a control that silently stops working. This is the same gap as T-4 viewed as a conflict rather than a testability hole; resolving it needs one sentence stating whether favorite writes are blocked, queued for replay, or rejected with a message during degraded mode.

### C-3 — FR-6 vs NFR Availability: erasure cannot reach the local cached copy the NFR mandates
- FR-6: a deletion request results in the account's credentials **and its favorites list** being "**irrecoverably deleted, not merely hidden or soft-deleted**."
- NFR Availability: every signed-in device retains "a last-known-synced local copy" of the favorites list.

The availability NFR requires the favorites list to exist on each device outside the sync backend; FR-6 asserts the favorites list is irrecoverably deleted on request. As written the two cannot both hold — a deleted account's favorites would survive in local copies on every device the user signed in on, and FR-6 specifies no client-side purge or invalidation. This is not merely an implementation detail: FR-6's whole value is the "not merely hidden" guarantee, and the erasure posture is what the PRD offers in place of the data-access right it excluded from scope (L-2). Resolution needs FR-6 to state explicitly whether erasure includes local caches, and what happens to the local copy on sign-out or account deletion.

### C-4 — FR-4 vs FR-2: FR-4 syncs a reordering capability that no requirement grants
- FR-4: "a favorite added, removed, **or reordered** on one device is reflected on another."
- FR-2: "Default sort order is recency-favorited (most recently favorited first)" — a system-determined order.

No requirement anywhere in the PRD lets a user reorder their favorites, and FR-2 defines ordering as derived from favorite recency. FR-4's "reordered" therefore either (a) is a stray word, or (b) implies a manual-ordering capability that FR-2's system sort contradicts and that no FR specifies. Because it is untestable as written, a downstream agent may reasonably infer (b) and build drag-to-reorder plus an order field into the sync model — scope this PRD never approved. Needs a one-word deletion or an explicit FR, and that is a human call, not mine to make.

---

## Verdict

`Blocking issues found`

The blocking core is not the individual product decisions — most are reasonable — but that **the document's record of its own decision state is self-contradictory**, and every downstream agent treats this file as ground truth. Specifically:

1. **L-1** — the PRD says, in two places, both that the two Security/PII follow-ons remain open for human resolution and that they are resolved. A Gate 1 approval cannot mean anything definite while that is true.
2. **L-3** — the human's confirmation of the ten defaults exists outside the artifact; inside it, ten items including a hard feature gate (FR-3 email verification) and the entire sync model (FR-4 sync-on-load) are still labeled "pending sanity-check."
3. **C-1, C-3** — two genuine contradictions where a downstream agent implementing one requirement literally would violate another, one of them (C-3) undercutting the erasure guarantee that the PRD offers in place of the excluded data-access right.
4. **T-1** — the erasure SLA, the PRD's most compliance-adjacent number, has three inconsistent statements and no single derivable criterion.

None of these require new requirements or re-scoping — they are corrections and one-line clarifications to text already present. C-2, C-4, T-2 through T-5, and L-2, L-4, L-5 are all real but individually approvable-with-eyes-open; they are listed so the human can decide which to fix now and which to carry.

Per `docs/agent-protocol.md` §3 and my own constraints, this verdict informs the Gate 1 decision and is not the decision. The PRD is unmodified; `status` remains `DRAFT` and `approver` remains `pending`. Whether to revise to `prd_v2.md` before Gate 1, or approve with these findings on the record, is the human's call.
