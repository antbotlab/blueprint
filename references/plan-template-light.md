<!-- Lightweight plan template for direct-workflow projects (no git/CI). Used by /blueprint Phase 3. -->

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
{Properties that must hold after EVERY step completes.}
- {project-specific invariants}

## Agent Autonomy Boundary

Operations are classified by reversibility. Executing agents must respect these boundaries.

**Autonomous** (execute without asking):
- file edits, builds, tests, lint — all local and reversible

**User-confirmed** (ask before executing):
- deleting files or directories, overwriting critical config, any operation that is hard to reverse

## Steps

### Step {NN}: {title} [ ]

**Size**: S / M / L
**Agent**: Sonnet / Opus

**Context** (cold-start brief):
{Minimum background for an agent that has NOT read previous steps.
Must be self-contained with Design Decisions + Invariants.}

**Tasks**:
1. ...
2. ...

**Rollback**: {recovery strategy on failure}

**Verification**:
- [ ] Automated: `{exact commands}`
- [ ] Manual: `{action + expected result}` (only at E2E milestones)

**Exit criteria**: {specific, measurable, unambiguous}

---

## Dependency Graph

{ASCII diagram showing step dependencies}

**Parallelizable**: {list groups of steps that can run concurrently}

## Progress Log

| Step | Status | Notes |
|------|--------|-------|
| 01   | [ ]    | —     |

## Review Log

- **{date}**: {reviewer} — {summary of findings and fixes}
