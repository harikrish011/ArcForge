# Addendum: Parking Spot Finder PRD

Companion to `prd.md`. Holds content that informed the PRD but doesn't belong in a product requirements doc: implementation-level technical choices, and the Agentic SDLC process/methodology that the source proposal treats as the actual hackathon submission. The canonical, full-detail version of everything below is `ARC_Forge_Proposal_Parking_Spot_Finder.md` at the project root — this addendum is a condensed extract, not a replacement. The Technical-How section is `bmad-architecture` input; the Agentic SDLC Process section is traceability only (see its closing note for a possible future home).

## Technical-How (→ input to `bmad-architecture`)

- **Frontend:** React — list view and map view, location permission prompt, spot detail panel, route CTA.
- **Backend:** Node.js — single `GET /parking/nearby?lat&lng&radius` endpoint over the mock dataset, haversine-distance filter, no auth layer for MVP.
- **Database:** PostgreSQL — one `parking_spots` table seeded from the mock dataset (id, name, latitude, longitude, address, parking_type, price_info, operating_hours).
- **Maps — `[OPEN]` primary provider:** pick one with a generous free tier and a straightforward JS SDK for markers + route deep-link; final choice confirmed against free-tier terms at build time (source proposal §12; tracked as a high-confidence AMBIGUITY in PRD §9 Assumptions & Ambiguities). This is the one undecided item in this section — everything else here is settled.
- **Maps — fallback (decided):** `maps.google.com` (PRD FR-8); see the fallback ladder below for how it's used.
- **Geocoding:** none. Manual Area Entry (PRD FR-2) uses a curated preset list of named areas, not free-text-to-coordinates geocoding — avoids taking on a geocoding service's own cost and failure modes for the MVP.
- **Error handling:** location denied → manual area entry (PRD FR-2); zero results within radius → expand the search to 2km with a message, then suggest a different area if still zero (PRD FR-6).
- **Hackathon fallback ladder** (source proposal, Hackathon Survival Plan): the PRD's committed fallback is list-only view and the plain `maps.google.com` link decided above (PRD FR-5, FR-8 — hard requirements, not best-effort). The source proposal's alternate idea of a static embedded map image with pins was not adopted as a requirement; it remains a build-day option if the list-only fallback still isn't enough, not something this PRD specifies.

## Agentic SDLC Process (traceability only — not product requirements)

The source proposal's real point: the Parking Spot Finder app is a small, deliberately boring vehicle for demonstrating a governed, human-in-the-loop way of building software with AI agents. Summarized here for traceability; see the source doc for full detail, mermaid diagrams, and the evaluation-criteria mapping.

- **Orchestration model:** a coordinating/orchestrator agent delegates scoped sub-tasks to specialist agents (Requirements, Planning, Architecture, Development, QA/Test, Code Review, Security/Guardrail, Release/Documentation) and does not proceed past a stage until its human approval gate clears. Five human gates total: Requirements, Architecture, Code, Test/Quality, Release. These gates are coarse checkpoints between stages; the permission tiers below are the finer-grained capability ladder an agent climbs *within* and *between* those checkpoints — same oversight model, two levels of granularity, not two separate controls.
- **New agent roles not in existing Forge/BMAD templates:** a dedicated **QA/Test Agent** and a dedicated **Security/Guardrail Agent** — explicitly framed as the team's own addition, not a claimed existing capability.
- **Artifact discipline:** every agent output is written to a versioned file (`requirements_v1.md`, `architecture_v1.md`, etc.) carrying a status header (`DRAFT` / `APPROVED` / `REJECTED` / `SUPERSEDED`) plus approver name and timestamp. Only `APPROVED` artifacts are valid inputs to the next agent — enforced procedurally by the team, not automatically.
- **Guardrails:** no autonomous merges/releases; agents cannot silently alter an approved requirement; input validation, no hardcoded secrets, dependency scanning, least-privilege permission tiers (Read-only → Generate → Modify workspace → Run tests → Propose change → Human approval → Merge/release).
- **Confidence calibration:** every agent output carries a self-flagged confidence note (high/low, with the specific assumption named) so humans know where to look harder.
- **Cost/token discipline:**
  - Simple sub-tasks routed to a smaller/cheaper model.
  - Each agent scoped to only the approved artifacts it needs, not full conversation history.
  - Rejected outputs regenerated against specific feedback rather than from scratch.
  - Rough agent-call/token totals tracked and stated at demo time.
- **One-day execution plan and demo script:** a specific 9:00–18:00 schedule per SDLC stage, and a deliberately staged "human catches a real AI-introduced defect during QA, routes it back, shows the fix" moment as the rubric's named "human caught the AI" evidence.
- **Success/judging metrics** (distinct from the product usage metrics in PRD §8): time from idea → approved requirements, time from approved architecture → working code, number of SDLC stages with real agent involvement, number of human gates exercised, defects caught by QA/Test and Code Review agents pre-sign-off, approximate agent calls/token usage. These measure the *process*, not the parking app's usage.
- **Team/agent responsibility matrix and risk register** — see source proposal §15 and §18 for the full tables:
  - Per-role human owner (§15).
  - Risk register (§18) covering: hallucination, poor generated code, security vulnerabilities, excessive agent autonomy, map/API unavailability, time overrun, approval bottlenecks, conflicting agent outputs.

If this process is formalized as its own artifact later, it's a candidate for a `bmad-architecture` spine or a `bmad-project-context` doc describing team/agent working conventions — not a rework of this PRD.
