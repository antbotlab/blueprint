---
name: blueprint
description: Create a construction plan for a multi-step engineering task. Researches codebase, drafts plan with dependency graph, delegates Opus review, saves to plans/.
disable-model-invocation: false
argument-hint: <project> <objective>
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent
---

# Blueprint — Construction Plan Generator

Create a construction plan for a multi-step engineering task. `$ARGUMENTS` format: `<project> <objective>` — project name (for locating code and naming the file) and a one-line goal.

**Core principle**: The plan's only reader is an AI agent with zero prior context. Every line must serve cold-start execution — not human reporting, not status theater.

## Phase 1 — Research

Gather the minimum context needed to make sound design decisions.

1. Read `memory/MEMORY.md` to find the project's path and current status.
2. Read the project's CLAUDE.md, README, and package.json (or equivalent entry point).
3. Read existing plans in `plans/` for the same project — avoid duplicating or contradicting active plans.
4. Scan source code: directory structure, key modules, dependency graph, test setup.
5. If the objective is ambiguous, **ask the user** targeted clarifying questions. Never guess intent.

Output a brief context summary (5-10 lines) before proceeding. Do not ask for confirmation — proceed to Phase 2.

## Phase 2 — Design

Make architectural and sequencing decisions.

1. Break the objective into steps. Each step = one PR's worth of change.
2. For each step, decide:
   - Can it run in parallel with other steps? → draw dependency edges.
   - Does it modify shared files? → must be serial with other steps touching those files.
   - Is it risky (large moves, entry point rewrites, breaking changes)? → mark for worktree isolation.
   - Does it need deep reasoning? → assign Opus. Otherwise default Sonnet.
3. Assign Size (S/M/L) per step based on scope, complexity, and risk. This is a judgment call — no formula.
4. Identify invariants — properties that must hold after every single step (e.g., build passes, no SDK leak into core).
5. Identify risks and decide rollback strategy per step.
6. Make key design decisions and record rationale ("chose A because B fails under X, C adds unnecessary complexity").
7. Determine workflow mode:
   - **Git projects with CI**: full branch workflow (branch → Opus review → push → CI green → `/pr` merge). This is the default.
   - **Non-git or docs-only tasks**: direct workflow (edit files in place, no branch/PR). Agent judges which mode applies.

## Phase 3 — Draft

Write the plan file at `plans/{project}-{objective-slug}.md` following the template below.

The plan must be **fully self-contained** — an executing agent reads only the plan file and the project's CLAUDE.md. All branch workflow rules, CI policy, and review gates must be written directly into the plan. Do not reference external documents for critical workflow rules.

### Plan Template

```markdown
# {Title} — Construction Plan

## Objective
{1-2 sentences: what + why}

## Constraints
- Sub-agents: read project CLAUDE.md before starting work
- {project-specific constraints}

## Prerequisites
- [ ] {conditions that must be true before Step 01}

## Design Decisions
{Key choices made during planning, with rationale.
Executing agents must not re-litigate these unless they find factual errors.}

## Invariants
{Properties that must hold after EVERY step completes. Verified before push.}
- All CI checks pass: `{build + test + lint + format commands}`
- {project-specific invariants}

## Branch & Merge Workflow

### Branch naming
- Format: `{project}-step-{NN}-{short-desc}` (kebab-case, e.g., `antbot-step-03-plugin-loader`)
- Always branch from latest `main`: `git pull origin main && git checkout -b <branch>`
- One branch per step. Never bundle multiple steps into one branch.

### Pre-push Opus review gate
Every branch must pass an Opus sub-agent review before pushing to remote. No exceptions.
1. Executing agent completes all tasks and runs verification locally.
2. Executing agent commits all changes on the feature branch.
3. **Before `git push`**: delegate review to an Opus sub-agent:
   - Prompt: "Review all commits on branch `{branch}` against `main`. Check: correctness, edge cases, consistency with existing code, CLAUDE.md compliance, security. Return a structured list of findings (critical / important / minor)."
   - If critical or important findings: fix, commit, re-review. Iterate until clean.
   - Minor findings: fix if trivial, otherwise log in Progress Log.
4. Only after review passes: `git push -u origin <branch>`.

### Post-push CI gate
After push, CI must pass before merge.
1. After push: `gh run list -b <branch> --limit 1` to find the CI run.
2. Wait for CI: `gh run watch <run-id>`. Never proceed while CI is pending.
3. If CI fails: diagnose locally → fix → commit → re-review (Opus gate) → push → wait for CI again. Never merge a red branch.
4. After CI is green: use `/pr` skill to create PR and merge (squash merge via GitHub).
5. After merge: `git checkout main && git pull origin main`. Verify merge commit.

### Zero-tolerance CI policy
- A CI failure that reaches `main` is a **P0 construction incident** — permanent, public, damages project credibility.
- If a step's CI fails 3 times consecutively, **stop execution**. Report to user with: what failed, what was tried, root cause hypothesis.
- When sequencing multiple PR merges, verify CI passes at each stage before proceeding to the next.

## Steps

### Step {NN}: {title} [ ]

**Branch**: `{project}-step-{NN}-{short-desc}`
**Size**: S / M / L
**Isolation**: worktree / main-tree
**Agent**: Sonnet / Opus

**Context** (cold-start brief):
{Minimum background for an agent that has NOT read previous steps.
Must be self-contained with Design Decisions + Invariants.}

**Tasks**:
1. ...
2. ...

**Rollback**: {recovery strategy on failure — usually "discard worktree" or "revert commit"}

**Verification**:
- [ ] Automated: `{exact commands}`
- [ ] Manual: `{action + expected result}` (only at E2E milestones)

**Exit criteria**: {specific, measurable, unambiguous}

---

## Dependency Graph

{ASCII diagram showing step dependencies}

**Parallelizable**: {list groups of steps that can run concurrently}

## Progress Log

| Step | Status | Branch | PR  | Notes |
|------|--------|--------|-----|-------|
| 01   | [ ]    | —      | —   | —     |

## Review Log

- **{date}**: {reviewer} — {summary of findings and fixes}
```

**Note on non-git / docs-only plans**: Omit the "Branch & Merge Workflow" section entirely. Steps use direct file edits without branch/PR/CI gates.

## Phase 4 — Review

Delegate adversarial review of the complete plan to an **Opus sub-agent**.

Review checklist for the sub-agent:
1. **Completeness** — does the plan cover the full objective? Are there gaps between the last step and the goal?
2. **Step granularity** — is each step one-PR sized? Any step that's too large to review in one sitting?
3. **Dependency correctness** — are parallel steps truly independent? Would they conflict on shared files?
4. **Cold-start viability** — can an agent execute Step N by reading only: Step N's section + Design Decisions + Invariants + Branch Workflow? Or does it implicitly depend on knowledge from earlier steps?
5. **Rollback coverage** — does every step have a rollback strategy? Is "discard worktree" actually sufficient or would data be lost?
6. **Invariant sufficiency** — do the invariants catch real regressions? Are any missing?
7. **Exit criteria testability** — can each exit criterion be verified by running a command? Vague criteria like "code is clean" are rejected.
8. **Branch workflow compliance** — does every step follow the branch naming, Opus review gate, CI gate, and `/pr` merge rules?
9. **Risk assessment** — are the riskiest steps identified? Do they have worktree isolation and manual verification?
10. **CLAUDE.md compliance** — does anything in the plan contradict project or workspace rules?

Fix all critical and important findings. Log everything in Review Log.

## Phase 5 — Register

1. Save the plan file (written in Phase 3, refined in Phase 4 via Edit tool).
2. Update `memory/MEMORY.md`:
   - Add plan entry under the relevant project with `created: {date}` and brief description.
3. Present to user:
   - Plan file path
   - Step count and estimated parallelism
   - Prerequisites checklist (user must confirm before execution begins)

## Rules

- Never write a plan for a task that can be done in under 3 tool calls. Just do it.
- Never fabricate code structure or file paths. Every reference in the plan must be verified against the actual codebase in Phase 1.
- Plans are written in English. Conversation stays in the user's language.
- One plan per file. Never append a new plan to an existing plan file.
- If an active plan already exists for the same project+objective, ask the user whether to supersede or create a parallel plan.
- Code snippets in plans are allowed for key configurations (package.json, tsconfig, etc.) but not for implementation code. Implementation is the executing agent's job.
