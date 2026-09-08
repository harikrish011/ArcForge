---
name: git-operations
description: 'Perform git actions (status, pull, push, checkout, branch, merge) safely in this repository. Use when the user asks to sync/pull latest, check out or create a branch, commit, push, merge, or otherwise wants a git action performed directly rather than through a specific SDLC agent.'
---

# Git Operations

## Purpose

A single, safe entry point for git actions in this repository, invocable directly by the user rather than only through `developer` or `release-agent`. Behavior is identical to what those agents already follow — this skill does not define new rules, it exposes the existing protocol as a directly callable skill.

**Source of truth:** `docs/git-operations.md`. Read it before acting. If this skill's summary and that doc ever disagree, the doc wins — update this file to match rather than the other way around.

## Desired Outcomes

1. Routine, reversible git actions (checking status, syncing with the base branch) happen without unnecessary friction.
2. Any action that touches a remote or shared branch, or that could discard work, always gets an explicit, specific human confirmation first — no exceptions, no relying on an earlier approval.
3. The user always knows exactly what was done (branch, commit hash, what was pushed/merged) afterward.

## Step 1 — Classify the request

Read `docs/git-operations.md` §1 and classify what's being asked into one of three tiers before doing anything:

```text
Tier 1 — Read-only
  status, log, diff, branch -a, remote -v, fetch, show
  → just run it, report the result.

Tier 2 — Local, reversible
  pull --ff-only, checkout -b <new branch>, add/commit (local only), stash
  → run it, then state what was done (branch, commit synced to).
  If pull can't fast-forward: STOP, report the divergence, ask how to proceed.

Tier 3 — Remote-mutating / hard-to-reverse
  push (incl. --force), merge into a shared/base branch, checkout that
  discards local changes, reset --hard, branch -D / push --delete,
  rebase of shared commits, any action on a protected branch (main,
  master, release/*, or whatever this repo treats as protected)
  → never run without the confirmation in Step 3, every single time.
```

## Step 2 — Tier 1 / Tier 2: just do it, then report

For Tier 1, run the read-only command and summarize the result.

For Tier 2:
1. `git status` — if there are uncommitted changes that the requested action would affect, stop and ask whether to stash, commit, or discard them first (discarding is Tier 3 — never assume it).
2. `git fetch`.
3. Perform the requested Tier 2 action (`pull --ff-only`, new local branch, local commit, stash).
4. Report exactly what happened: base branch, resulting commit/branch, anything skipped.

If `git pull --ff-only` cannot fast-forward, stop — do not merge, rebase, or force it. Report how many commits each side has diverged and ask the human how to proceed.

## Step 3 — Tier 3: always confirm first, every time

Never run a Tier 3 action from an inferred "yes," a prior approval elsewhere in the conversation, or a Gate/release approval (content approval and git-action approval are different decisions — see `docs/git-operations.md` §3). Ask concretely:

```text
Target branch:      <branch>
Action:              push | merge | force-push | reset --hard | ...
What will change:    <n> commits to push, or <files/branches affected>
Remote state:        <n commits ahead/behind, or up to date>

Confirm: proceed with <action> on <branch>? (yes/no)
```

Only after an explicit "yes" to that specific prompt, execute the action, then report the resulting commit hash(es)/branch state.

## Step 4 — Hygiene (always applies)

- Never commit or push secrets, `.env` values, or credentials — use placeholders and point to the project's real secrets mechanism instead.
- Follow the repository's existing branch-naming and commit-message conventions if discoverable (`git branch -a`, `git log --oneline`); don't invent a competing convention.
- Never rewrite already-pushed history (`rebase`, `commit --amend`, `push --force`) without the Step 3 confirmation, regardless of how minor the change looks.

## Relationship to the SDLC agents

`developer` uses this same protocol automatically for pre-work sync (Tier 2 only, per `developer.md` §1A). `release-agent` uses it for the post-Gate-5 push/merge handoff (Tier 2 + Tier 3, per `release-agent.md`'s Git Release Handoff section). This skill is the same rules made directly callable by the user outside of those two flows — it does not grant any broader authority than either agent already has, and it does not bypass `orchestrator.md` §19 (Destructive Operations) or §19A (Agent Definition Protection).
