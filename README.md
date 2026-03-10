![Blueprint — construction plans for AI coding agents](https://github.com/user-attachments/assets/23587ca9-1b58-4257-950d-0a1c144592ef)

# Blueprint

> Turn a one-line objective into a construction plan that any AI agent can execute cold — across sessions, branches, and sub-agents.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

AI coding agents handle small tasks well, but fall apart on complex projects. They forget context across sessions, re-litigate decisions that were already made, skip verification, and produce work that no other agent can pick up. Blueprint solves this: it generates a self-contained Markdown plan where every step is independently executable, every decision is locked with rationale, and every verification is a command you can run.

Built for developers who use AI coding agents on projects too large for a single session — multi-step refactors, plugin extractions, architecture migrations, and phased feature rollouts. The generated plans are plain Markdown — any AI agent can read and execute them.

## What You Get

A Markdown plan file designed so that a fresh AI agent in a new session can pick up any step and execute it without reading the rest of the plan:

- **Steps** — each one-PR sized, with task list, rollback strategy, verification commands, and exit criteria
- **Dependency graph** — which steps can run in parallel, which must be serial
- **Design decisions** — rationale for key choices, locked against re-litigation
- **Invariants** — properties verified after every step (build passes, no regressions)
- **Progress log** — single source of truth for execution state across sessions
- **Review log** — findings from an independent review that stress-tests the plan for gaps and flaws

<details>
<summary>Example: one step from a 9-step plugin extraction plan</summary>

```markdown
### Step 04: Redirect core scheduler calls through plugin registry [x]

**Branch**: `taskflow-step-04-redirect-calls`
**Size**: M
**Isolation**: worktree
**Agent**: Opus

**Context** (cold-start brief):
The core currently calls the scheduler directly via
`import { Scheduler } from './scheduler/index.js'` in 3 files.
The plugin registry now has a `scheduler` slot with `getScheduler()`.
This step replaces all direct imports with registry calls — the critical
bridge that decouples core from the scheduler implementation.

**Tasks**:
1. [guided] Find all files importing from `./scheduler/`
2. [guided] Replace direct imports with `getScheduler()` + descriptive error
3. [guided] Update bootstrap to warn if no scheduler is registered
4. [guided] Update tests to register a mock scheduler in beforeEach
5. [exact] Run: `pnpm -r build && pnpm -r test && pnpm -r typecheck`

**Rollback**: Discard worktree. Main tree is unaffected.

**Verification**:
- [ ] `pnpm -r build && pnpm -r test` — all tests pass
- [ ] `grep -r "from.*scheduler" packages/core/src/ | wc -l` — returns 0

**Exit criteria**: No file in core directly imports from `./scheduler/`.
All tests pass with mock scheduler registration.
```

The **Context** field is the key — it tells a cold-start agent exactly what state the codebase is in and what this step does, without reading prior steps. Task markers: `[guided]` = agent decides the approach; `[exact]` = run this command verbatim.

</details>

<details>
<summary>Example: dependency graph with parallel groups</summary>

```
Step 01 (interface)
  ├──→ Step 02 (registry slot)
  │      └──→ Step 04 (redirect calls) ──→ Step 05a (move files)
  │                                              └──→ Step 05b (tests)
  └──→ Step 03 (package scaffold) ──────────────→ Step 05a

Parallelizable:
  Group A (after Step 01): Step 02 and Step 03 — no shared files
```

</details>

Full examples: [`small-plan.md`](examples/small-plan.md) (3 steps, direct workflow) · [`large-plan.md`](examples/large-plan.md) (9 steps, branch workflow with step split)

## Installation

### Claude Code

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/antbotlab/blueprint.git ~/.claude/skills/blueprint
```

Claude Code discovers the skill automatically — `/blueprint` is ready to use.

### Other AI agents

The generated plans are standard Markdown files. Any AI agent that can read files can execute a Blueprint plan. Point the agent to `SKILL.md` as a system prompt or instruction file, and it will know how to generate and execute plans.

### Requirements

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) for the `/blueprint` slash command. Git and [GitHub CLI](https://cli.github.com/) unlock the full branch/PR/CI workflow, but Blueprint works without them — it detects their absence and automatically switches to direct mode.

## Quick Start

```
/blueprint myapp "migrate database to PostgreSQL"
```

This scans your codebase, designs a step-by-step plan, reviews it with an Opus sub-agent (falls back to Sonnet if unavailable), and saves it to `plans/myapp-migrate-to-postgresql.md`.

## Key Features

**Cold-start execution** — Every step includes a self-contained context brief. A fresh agent in a new session can execute any step without reading prior steps or conversation history.

**Cross-session resumption** — Plans track execution state in a Progress Log. A new session reads the log, checks branch/file state, and picks up exactly where the last session stopped.

**Living documents** — Plans mutate during execution. Steps can be split, inserted, skipped, reordered, or abandoned — all with formal protocols and an audit trail.

**Adversarial review** — Every plan is reviewed by an Opus sub-agent against a checklist covering completeness, dependency correctness, cold-start viability, anti-pattern detection, and more. Falls back to Sonnet if Opus is unavailable.

**Parallel step detection** — The dependency graph identifies steps with no shared files or output dependencies, so they can run concurrently.

**Graceful degradation** — No Opus? Falls back to Sonnet. No GitHub CLI? Switches to direct mode. No project memory? Relies on codebase scanning. Blueprint adapts instead of failing.

## How Blueprint Differs from `/plan`

Claude Code's built-in `/plan` generates a persisted Markdown plan and works well for straightforward tasks. Blueprint is designed for multi-session, multi-agent projects where each step must be independently executable:

| | `/plan` | `/blueprint` |
|---|---|---|
| Zero setup | Yes — built in | Requires install |
| Persisted plan file | Yes — `~/.claude/plans/` | Yes — project `plans/` directory |
| Cold-start steps | No — steps assume shared context | Yes — each step has a self-contained context brief |
| Adversarial review | No | Yes — Opus sub-agent review gate |
| Branch/PR/CI workflow | No | Yes — built into every step |
| Dependency graph | No | Yes — parallel groups and serial constraints |
| Plan mutation protocol | No | Yes — split, insert, skip, reorder, abandon |

## How It Works

1. **Research** — Scans the codebase, reads project memory if available, runs pre-flight checks (git repo, gh auth, default branch).
2. **Design** — Breaks the objective into one-PR-sized steps, identifies parallelism, assigns model tiers (Opus for architecture, Sonnet for implementation), marks risky steps for worktree isolation.
3. **Draft** — Generates the plan from a structured template. Includes branch workflow rules, CI policy, invariants, and rollback strategies — all inline, fully self-contained.
4. **Review** — Delegates adversarial review to an Opus sub-agent. Issues are fixed and re-reviewed until the plan passes.
5. **Register** — Saves the plan and updates project memory.

## Directory Structure

```
blueprint/
├── SKILL.md                     Main skill definition
├── references/
│   ├── plan-template.md         Full plan template (branch workflow)
│   ├── plan-template-light.md   Lightweight template (direct workflow)
│   ├── review-checklist.md      Phase 4 review checklist
│   └── anti-patterns.md         Planning anti-patterns & rebuttals
├── workflows/
│   ├── plan-mutation.md         Protocol for modifying plans mid-execution
│   └── resumption.md            How to resume a partially-executed plan
└── examples/
    ├── small-plan.md            3-step direct-workflow example
    └── large-plan.md            9-step branch-workflow example
```

## Contributing

Bug reports and feature requests: [GitHub Issues](https://github.com/antbotlab/blueprint/issues).

Pull requests welcome — please open an issue first to discuss non-trivial changes.

## License

[MIT](./LICENSE) &copy; 2026 antbotlab
