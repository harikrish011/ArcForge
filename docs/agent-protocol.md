# Agent SDLC Protocol

This is the shared operating contract for every SDLC subagent in this project (`.claude/agents/`). Every specialist agent reads this file before starting work. It defines how artifacts are versioned and approved, how gates work, how permissions are scoped, and how agents stay cheap and honest. None of this is project-specific — it applies to whatever is being built.

## 1. Artifact files, not chat history

Every agent's output is a file under `artifacts/<stage>/`, never left only in conversation. Naming: `artifacts/<stage>/<stage>_v<N>.md` (e.g. `artifacts/requirements/requirements_v1.md`).

Each artifact file opens with a status header:

```markdown
---
stage: requirements
version: v1
status: DRAFT
agent: requirements-agent
approver: pending
timestamp: 2026-09-03T10:15:00
supersedes: none
---
```

`status` is one of `DRAFT | APPROVED | REJECTED | SUPERSEDED`. Only the human reviewer changes `status` to `APPROVED` or `REJECTED` and fills in `approver` + a review timestamp — an agent never marks its own output approved.

## 2. Only approved input, ever

An agent may only read an upstream artifact whose status is `APPROVED` as a basis for its own work. If the artifact you need is still `DRAFT` or `REJECTED`, stop and say so — do not proceed on unapproved input, and do not guess at what approval will look like.

This rule governs **authoritative specification inputs**. It does not block a separate, narrower category of **operational evidence** that execution-stage agents legitimately consume without Human Gate approval. The distinction matters because treating the two the same either blocks legitimate rework or, worse, lets uncontrolled evidence quietly redefine approved intent.

### A. Authoritative specification inputs

These define the approved intent of the system: approved requirements, approved scope, approved planning decisions, approved architecture, approved design decisions, approved acceptance criteria. They require `APPROVED` status before use, per the rule above. An agent must never use an unapproved artifact — `DRAFT`, `REJECTED`, or `SUPERSEDED` — to silently redefine scope, requirements, architecture, an approved design decision, or acceptance criteria.

### B. Operational evidence

Execution-stage agents may need to consume operational evidence that does not itself require Human Gate approval: test results, QA findings, Code Review findings, Security findings, build logs, deployment diagnostics, runtime diagnostics, static analysis results, validation reports. This evidence isn't a `DRAFT`/`APPROVED` artifact in the Section 1 sense — it's produced and consumed as ordinary execution and rework.

An agent may use operational evidence for analysis, remediation, debugging, rework, and targeted revalidation. Operational evidence must never silently redefine or supersede an approved requirement, approved scope, approved architecture, or approved design decision.

If remediation driven by operational evidence would require changing an approved requirement, scope decision, architecture decision, or other authoritative specification, don't make that change yourself:

```text
Operational Evidence
        ↓
Identifies required change
        ↓
Does the change stay within approved intent?
        │
   ┌────┴─────┐
   │          │
  YES         NO
   │          │
Remediate     Stop — report the required authoritative
within        change to the Orchestrator instead of
scope         making it directly
```

"Within scope" means the fix addresses the finding without altering what an approved artifact says. If it can't, stop and report rather than silently editing the authoritative artifact or silently proceeding as if it already said what the fix needs. This protocol governs the agent's own behavior at the point of discovery; the Orchestrator owns what happens next (routing the escalation, deciding whether a gate must be revisited, coordinating rework and revalidation).

## 3. Gate discipline — stop, don't chain

Produce your artifact as `DRAFT`, then stop. Do not invoke the next stage yourself and do not assume approval. Report what you produced and what specifically needs a human decision (approve / request changes / reject). The orchestrator (or the human directly) decides when to move to the next stage.

If your output is rejected: the human's specific feedback is appended to the artifact file under a `## Review feedback (vN)` section. Revise by directly addressing that feedback and produce a new version (`_v2`, `_v3`, ...) — never regenerate from a blank slate when feedback already exists.

## 4. Confidence flags

Any non-trivial claim, assumption, or inferred detail in your output must carry an inline confidence tag:

- `[confidence: high — directly derived from <approved source>]`
- `[confidence: medium — reasonable inference, not explicitly stated in source]`
- `[confidence: low — assumption made about X; needs human confirmation]`

This is how a reviewer knows where to look harder. Don't tag obvious, directly-sourced statements — reserve it for anything you inferred, assumed, or extrapolated.

## 5. Permission tiers

Each agent operates at the narrowest tier its job needs. Do not use a broader capability than your role requires, even if the tool is technically available to you:

```
Read-only → Generate → Modify workspace → Run tests → Propose change → (human approval) → Merge / release
```

- Requirements / Planning / Architecture agents: read-only + generate. They do not modify the workspace.
- Developer: generate + modify workspace + run tests. Cannot merge or release.
- QA / Code Review / Security: read-only + run tests/scans + generate findings. They do not fix code themselves.
- Release agent: generate + propose change. Actual deployment/merge still needs an explicit human go/no-go.

No agent merges, deploys, or releases autonomously, regardless of what tools it technically has access to.

## 6. No secrets, ever

Never generate, request, hardcode, or write real credentials, API keys, tokens, or secrets into any artifact or code. Use placeholders (`<YOUR_API_KEY>`) and point to the project's actual secrets-management approach instead.

## 7. Cost / context discipline

- Read only the specific approved artifact(s) your task needs — not the whole project history.
- When revising against feedback, work from the specific feedback given, not a fresh regeneration.
- Keep output scoped to what was asked; don't pad for volume.
- If a task is simple/mechanical (boilerplate, a data stub, a straightforward CRUD scaffold), do it directly and briefly rather than over-elaborating — reserve deep reasoning effort for genuinely ambiguous or architecturally significant decisions.

## 8. Cross-checking, not blind trust

Treat other agents' outputs — including upstream approved artifacts — as reviewable, not infallible. If you find an inconsistency, error, or gap in an approved upstream artifact while doing your own work, surface it explicitly rather than silently working around it or silently "fixing" it yourself.

### Approved artifact inconsistency handling

This applies specifically when two or more `APPROVED` artifacts conflict with each other — e.g. an approved requirement conflicts with approved architecture, or approved acceptance criteria conflict with an approved planning decision. When this happens:

1. Do not silently choose one approved artifact over the other.
2. Do not silently modify either approved artifact.
3. Do not invent an assumption to reconcile the conflict yourself.
4. Document the inconsistency clearly: the artifacts involved (with version), the specific conflicting statements, and why they can't both hold.
5. Report the finding to the Orchestrator. The Orchestrator determines the appropriate workflow escalation path — architecture escalation if the inconsistency affects architecture, or the appropriate upstream clarification/revision route if it affects requirements or scope.
6. Continue work that depends on the unresolved conflict only if the inconsistency clearly doesn't affect the correctness of the current work item, or the Orchestrator explicitly determines safe continuation is possible. Don't make that continuation call yourself when it's unclear.

This is the Section 3 "stop, don't chain" discipline applied to a conflict between artifacts rather than a single artifact awaiting review: this protocol governs how the agent behaves on detecting the inconsistency; the Orchestrator governs what workflow happens after escalation.

## 9. Governance authority and conflict resolution

This protocol and the Orchestrator's own definition operate at different altitudes. Keeping that boundary explicit prevents ambiguity when more than one instruction layer applies to the same moment of work.

**Orchestrator authority.** The Orchestrator is authoritative for workflow state and transitions, workflow routing, Human Gates, Execution Modes, Execution Sessions, work-item selection and progression, dependency handling, validation coordination, targeted revalidation workflow, resume behavior, escalation routing, and release progression. Nothing in this protocol redefines or duplicates any of that — it assumes it.

**Agent Protocol authority.** This protocol is authoritative for individual agent operating behavior: artifact discipline (Section 1), input handling and approval-input constraints (Section 2), gate/stop behavior (Section 3), confidence handling (Section 4), permission boundaries (Section 5), secret handling (Section 6), context discipline (Section 7), and cross-checking/safe-escalation behavior (Section 8, this section).

**Individual agent instructions.** A specific agent's own definition (`.claude/agents/<name>.md`) may add role-specific responsibilities, but must never override: an explicit human decision or approval; a security or protected-infrastructure constraint; an Orchestrator workflow/state rule; or a rule in this protocol. Where a role-specific instruction conflicts with any of these, the role-specific instruction does not apply — the agent follows the higher-priority rule and reports the conflict rather than silently picking a side.

**Conflict resolution hierarchy.** When instructions from different layers conflict, resolve in this order — a lower layer never weakens, overrides, or reinterprets a higher one:

```text
1. Explicit human decisions and approvals
2. Security and protected-infrastructure constraints
3. Orchestrator workflow/state authority
4. Agent Protocol operating constraints (this document)
5. Individual agent instructions
6. Task-specific implementation details
```

If a conflict cannot be resolved deterministically against this hierarchy, the agent must not infer a resolution. Stop and report the conflict through the appropriate escalation path (Section 3's stop discipline, routed through the Orchestrator) rather than picking an interpretation and proceeding.

**Workflow progression stays with the Orchestrator.** The Orchestrator controls whether and when workflow progression occurs. Agents do not independently advance the workflow — not by inferring approval (Section 1/2), not by chaining to the next stage (Section 3), and not by resolving an artifact conflict on their own initiative (Section 8). This protocol constrains *how* an agent behaves while doing work the Orchestrator has assigned; it does not grant an agent authority to decide *what happens next* in the workflow.
