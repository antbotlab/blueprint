<!-- Plan template for branch-workflow projects. Used by /blueprint Phase 3. -->

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
- Format: `{project}-step-{NN}-{short-desc}` (kebab-case, e.g., `myapp-step-03-auth-module`)
- Always branch from latest default branch (detected in Phase 1 pre-flight): `git pull origin {default-branch} && git checkout -b <branch>`
- One branch per step. Never bundle multiple steps into one branch.

### Pre-push Opus review gate
Every branch must pass an Opus sub-agent review before pushing to remote. No exceptions.
1. Executing agent completes all tasks and runs verification locally.
2. Executing agent commits all changes on the feature branch.
3. **Before `git push`**: delegate review to an Opus sub-agent:
   - Prompt: "Review all commits on branch `{branch}` against `{default-branch}`. Check: correctness, edge cases, consistency with existing code, CLAUDE.md compliance, security. Return a structured list of findings (critical / important / minor)."
   - If critical or important findings: fix, commit, re-review. Iterate until clean.
   - Minor findings: fix if trivial, otherwise log in Progress Log.
4. Only after review passes: `git push -u origin <branch>`.

### Post-push CI gate
After push, CI must pass before merge.
1. After push: `gh run list -b <branch> --limit 1` to find the CI run.
2. Wait for CI: `gh run watch <run-id>`. Never proceed while CI is pending.
3. If CI fails: diagnose locally → fix → commit → re-review (Opus gate) → push → wait for CI again. Never merge a red branch.
4. After CI is green: `gh pr create --title '{step title}' --body '{summary}' && gh pr merge --squash --delete-branch`. If pre-flight found no `gh` auth, this step is omitted entirely (plan uses direct workflow).
5. After merge: `git checkout {default-branch} && git pull origin {default-branch}`. Verify merge commit.

### Zero-tolerance CI policy
- A CI failure that reaches `main` is a **P0 construction incident** — permanent, public, damages project credibility.
- If a step's CI fails 3 times consecutively, **stop execution**. Report to user with: what failed, what was tried, root cause hypothesis.
- When sequencing multiple PR merges, verify CI passes at each stage before proceeding to the next.

## Agent Autonomy Boundary

Operations are classified by reversibility. Executing agents must respect these boundaries.

**Autonomous** (execute without asking):
- branch, add, commit, build, test, lint — all local and reversible

**User-confirmed** (ask before executing):
- push, PR create, PR merge, force operations (push --force, reset --hard, rebase), remote configuration changes, tag creation + push — all remote-visible or hard to reverse
- Confirm with user before the first push of each branch. Subsequent pushes to the same branch (e.g., after fixing CI) may proceed without re-confirmation.

**Sequencing with review gate** (branch-workflow plans):
1. Complete work + commit locally
2. Opus sub-agent review + fix findings
3. Ask user permission to push
4. Push to remote

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
