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
