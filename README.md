# Black Box Architecture

> Transform any codebase into modular, maintainable "black boxes".

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](docs/CONTRIBUTING.md)

AI agents for **Claude Code** that enforce replaceable, modular architecture with built-in approval gates, commit discipline, and a self-improvement loop.

## What This Is

Five coordinated agents that apply [Eskil Steenberg's](https://www.youtube.com/watch?v=sSpULGNHyoI) battle-tested principles:

- **Black box interfaces** - Clean APIs between modules
- **Replaceable components** - If you can't understand it, rewrite it
- **Constant velocity** - Write 5 lines today vs. edit 1 line later
- **Single responsibility** - One module, one person

## Quick Start

```bash
# Clone the repository
git clone https://github.com/gl0bal01/black-box-architecture.git

# Install agents + task files for your project
mkdir -p .claude/agents
cp -r black-box-architecture/agents/* .claude/agents/
mkdir -p tasks
cp -r black-box-architecture/tasks/* tasks/
```

## Agents

The agent system follows black box principles itself — specialized agents with clear responsibilities:

| Agent | Role | Model | What it does |
|-------|------|-------|-------------|
| **arch-orchestrator** | Entry point | Opus | Classifies requests, delegates, enforces approval gates, emits PASS/CONCERNS/FAIL/BLOCKED verdict |
| **arch-analyzer** | Analysis | Sonnet | Maps module boundaries, coupling, violations. Read-only. Min 3 `file:line` per claim |
| **arch-planner** | Design | Opus | Designs boundaries, interfaces, phased roadmaps. Simplicity check on non-trivial designs |
| **arch-implementer** | Implementation | Sonnet | Executes approved plans. Atomic commits per changeset. Only agent that modifies code |
| **arch-debugger** | Debugging | Sonnet | Isolates bugs to contract boundaries. Autonomous mode — just fix it. Captures lessons |

### Workflows

| Request | Flow |
|---------|------|
| Analysis only | Analyzer → synthesize |
| Plan only | Planner → request approval |
| Implement approved plan | Implementer |
| Debug | Debugger → optionally Analyzer if architecture unclear |
| Refactor toward black-box | Analyzer → Planner → **APPROVAL** → Implementer |
| New feature | Planner → **APPROVAL** → Implementer |

## What's New in v2.0

- **Self-improvement loop** — agents review `tasks/lessons.md` at session start, update it after corrections. Mistakes don't repeat across sessions
- **Task tracking** — persistent progress in `tasks/todo.md`. Resume work after interruptions
- **Commit discipline** — `git add -u` before and after each changeset. Atomic, bisectable history. Never `git add -A`
- **Goal-backward verification** — SUCCESS CRITERIA MET WHEN defines what done looks like, separate from HOW TO VERIFY
- **Quality gate verdicts** — PASS / CONCERNS / FAIL / BLOCKED instead of silent self-assessed checklists
- **Approval gates** — hard stops for deps, public API, schema, deletions, permission widening
- **Session start checklists** — every agent loads lessons and context before working
- **Debugger autonomous mode** — given a bug report, just fix it. Escalate only when genuinely blocked

See [CHANGELOG.md](CHANGELOG.md) for full details.

## Best Daily Workflow

1. Use `arch-orchestrator` for most tasks — it picks the right workflow
2. Concise output by default (5-12 lines). Full reports only when needed
3. Approve any dependency, public API, or schema changes explicitly
4. Check `tasks/lessons.md` periodically — it captures what the agents learned

## How It Works

- A [shared contract](agents/AGENTS_CONTRACT.md) governs all agent behavior (13 rules)
- Agents default to concise outputs and switch to full reports when warranted
- Approval gates prevent risky changes without explicit sign-off
- Every architectural claim requires min 3 `file:line` evidence points
- Agents commit before and after each changeset for safe rollback
- Corrections are captured in `tasks/lessons.md` and reviewed next session

## Core Philosophy

**"It's faster to write 5 lines of code today than to write 1 line today and then have to edit it in the future."**
— Eskil Steenberg

1. **Primitive-First Design** - Identify core data types that flow through your system
2. **Black Box Boundaries** - Modules communicate only through documented interfaces
3. **Replaceable Components** - Any module can be rewritten using only its interface
4. **Single Responsibility** - One module = one person can own it
5. **Wrap Dependencies** - Never depend directly on code you don't control

## Documentation

- **[Agent System Guide](docs/AGENTS.md)** - Complete agent documentation
- **[Workflows Guide](docs/WORKFLOWS.md)** - Step-by-step examples for each agent
- **[Integration Examples](docs/INTEGRATION_EXAMPLES.md)** - How agents coordinate
- **[Principles Guide](docs/PRINCIPLES.md)** - Eskil Steenberg's methodology explained
- **[Code Examples](docs/EXAMPLES.md)** - Before/after transformations in multiple languages
- **[Agent Specifications](agents/agent.json)** - Technical specs and policies
- **[Contributing Guide](docs/CONTRIBUTING.md)** - How to contribute

## Real-World Examples

See [`examples/`](examples/) for complete before/after refactoring examples in Python, TypeScript, Go, Rust, C, and PHP.

## Legacy Prompts

Skills and commands in `docs/legacy/` are kept for compatibility but are no longer the recommended path. Use agents instead.

## Related Resources

- [Eskil Steenberg: Architecting LARGE Software Projects](https://www.youtube.com/watch?v=sSpULGNHyoI) - The foundation of everything here

## Contributing

Contributions welcome! See [CONTRIBUTING.md](docs/CONTRIBUTING.md).

## License

MIT License - see [LICENSE](LICENSE) for details.

---
