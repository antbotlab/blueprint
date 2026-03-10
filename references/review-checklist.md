# Review Checklist

Phase 4 review checklist for the strongest-model sub-agent. Execute each item against the plan.

1. **Completeness** — does the plan cover the full objective? Are there gaps between the last step and the goal?
2. **Step granularity** — is each step one-PR sized? A step is one-PR sized when: (a) a reviewer can understand the full diff in a single review session, (b) it changes ≤15 files, (c) it has a single logical intent describable in one sentence. Flag any step that violates any of these criteria.
3. **Dependency correctness** — are parallel steps truly independent? Would they conflict on shared files?
4. **Cold-start viability** — can an agent execute Step N by reading only: Step N's section + Design Decisions + Invariants + Branch Workflow? Or does it implicitly depend on knowledge from earlier steps?
5. **Rollback coverage** — does every step have a rollback strategy? Is "discard worktree" actually sufficient or would data be lost?
6. **Invariant sufficiency** — do the invariants catch real regressions? Are any missing?
7. **Exit criteria testability** — can each exit criterion be verified by running a command? Vague criteria like "code is clean" are rejected.
8. **Branch workflow compliance** (branch-workflow plans only; skip for direct-workflow plans) — does every step follow the branch naming, strongest-model review gate, CI gate, and gh CLI merge procedure?
9. **Risk assessment** — are the riskiest steps identified? Do they have worktree isolation and manual verification?
10. **CLAUDE.md compliance** — does anything in the plan contradict project or workspace rules?
11. **Anti-pattern scan** — does the plan contain any pattern from `references/anti-patterns.md`?
12. **Verification command validity** — are the commands in each step's Verification section real commands that exist in the project? Check against package.json scripts, Makefile targets, or CI workflow definitions.
13. **Self-consistency** — does the plan violate any of its own stated principles? Check: (a) do any steps violate the anti-patterns from `references/anti-patterns.md`? (b) do exit criteria contradict invariants? (c) do design decisions contradict each other?
