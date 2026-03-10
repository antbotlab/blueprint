# Blueprint

> Turn complex tasks into step-by-step construction plans that AI agents can execute autonomously.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Blueprint is a custom skill (slash command) for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Once installed, invoke it as `/blueprint <project> <objective>` inside any Claude Code session.

## Prerequisites

- **Required**: [git](https://git-scm.com/), [GitHub CLI (`gh`)](https://cli.github.com/) authenticated, [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- **Optional**: Opus model access (enhances review quality), project memory system

## Installation

```bash
git clone https://github.com/antbotlab/blueprint.git
mkdir -p ~/.claude/skills
ln -s "$(pwd)/blueprint" ~/.claude/skills/blueprint
```

Run from the directory where you want to keep the cloned repo. The symlink uses `$(pwd)` to resolve the absolute path.

## Quick Start

In Claude Code, type:

```
/blueprint myapp "migrate database to PostgreSQL"
```

- `myapp` — project name (used for file naming and locating code)
- `"migrate database to PostgreSQL"` — the objective

This creates a plan file at `plans/myapp-migrate-to-postgresql.md` (relative to your working directory) containing step-by-step execution instructions, a dependency graph, and a review log. See [`examples/`](examples/) for sample output.

## How It Works

The skill runs through 5 phases:

1. **Research** — Scans the codebase, reads project memory (if available), and maps out the current state: tech stack, architecture, relevant files.
2. **Design** — Determines the workflow mode (branching + PRs, or direct commits), sizes each step, and assigns the right model (Opus for architectural decisions, Sonnet for implementation).
3. **Draft** — Generates a Markdown plan file from a structured template. Each step includes a task list, verification checklist, exit criteria, and rollback strategy. The plan also includes a dependency graph showing which steps can run in parallel.
4. **Review** — Delegates an adversarial review to an Opus sub-agent (falls back to Sonnet if unavailable), which runs a 13-item checklist against the plan. Issues are fixed and re-reviewed until the plan passes.
5. **Register** — Saves the plan to `plans/` and updates project memory (if available).

## Design Principles

- **Plans are written for AI execution**, not human documentation. Every step is self-contained — a fresh agent in a new session can pick up from any step by reading the plan file alone.
- **Progressive disclosure** — The main skill file stays under 300 lines. Templates, review checklists, anti-pattern catalogs, and workflow protocols live in dedicated subdirectories.
- **Graceful degradation** — No Opus? Falls back to Sonnet. No GitHub CLI? Switches to direct mode (no PRs). No project memory? Relies on codebase scanning.
- **Guardrails** — A catalog of 10+ planning anti-patterns and 6+ rationalization rebuttals. The plan is self-checked by the drafting agent, then independently reviewed by a sub-agent.

## Skills

| Skill | Description | Status |
|-------|-------------|--------|
| `/blueprint` | Construction plan generator for multi-step engineering tasks | Stable |

## Directory Structure

```
blueprint/
├── SKILL.md                     Main skill definition (≤300 lines)
├── references/
│   ├── plan-template.md         Full plan template (branch workflow)
│   ├── plan-template-light.md   Lightweight template (direct workflow)
│   ├── review-checklist.md      13-item Phase 4 review checklist
│   └── anti-patterns.md         Planning anti-patterns & rationalization rebuttals
├── workflows/
│   ├── plan-mutation.md         Protocol for modifying plans mid-execution
│   └── resumption.md            How to resume a partially-executed plan
└── examples/
    ├── small-plan.md            3-step direct-workflow example
    └── large-plan.md            8–10 step full-branch-workflow example
```

## License

[MIT](LICENSE)
