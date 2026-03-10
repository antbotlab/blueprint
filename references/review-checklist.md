# Review Checklist

Phase 4 review checklist for Opus sub-agent. Execute each item against the plan.

1. **Completeness** — does the plan cover the full objective? Are there gaps between the last step and the goal?
2. **Step granularity** — is each step one-PR sized? Any step that's too large to review in one sitting?
3. **Dependency correctness** — are parallel steps truly independent? Would they conflict on shared files?
4. **Cold-start viability** — can an agent execute Step N by reading only: Step N's section + Design Decisions + Invariants + Branch Workflow? Or does it implicitly depend on knowledge from earlier steps?
5. **Rollback coverage** — does every step have a rollback strategy? Is "discard worktree" actually sufficient or would data be lost?
6. **Invariant sufficiency** — do the invariants catch real regressions? Are any missing?
7. **Exit criteria testability** — can each exit criterion be verified by running a command? Vague criteria like "code is clean" are rejected.
8. **Branch workflow compliance** — does every step follow the branch naming, Opus review gate, CI gate, and gh CLI merge procedure?
9. **Risk assessment** — are the riskiest steps identified? Do they have worktree isolation and manual verification?
10. **CLAUDE.md compliance** — does anything in the plan contradict project or workspace rules?
