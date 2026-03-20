# Black Box Architecture Agents

AI agents for maintaining and improving software architecture using black-box design principles — inspired by [Eskil Steenberg's lecture](https://www.youtube.com/watch?v=sSpULGNHyoI) and rooted in information hiding (Parnas, 1972).

Five coordinated specialists — analyze, plan, implement, debug — governed by a shared contract that enforces scope discipline, evidence minimums, approval gates, and a self-improvement loop.

## Quick Install

```bash
git clone https://github.com/gl0bal01/black-box-architecture.git

# Project-specific
mkdir -p .claude/agents
cp -r black-box-architecture/agents/* .claude/agents/

# Or personal (all projects)
cp -r black-box-architecture/agents/* ~/.claude/agents/
```

## Agents

| Agent | Role | Model | Writes Code |
|-------|------|-------|-------------|
| **arch-orchestrator** | Entry point. Classifies, delegates, synthesizes. | Opus | No |
| **arch-analyzer** | Maps module boundaries, coupling, violations. | Sonnet | No |
| **arch-planner** | Designs modules, interfaces, migration roadmaps. | Opus | No |
| **arch-implementer** | Executes approved plans with commit discipline. | Sonnet | Yes (requires approval) |
| **arch-debugger** | Isolates bugs to contract boundaries, fixes autonomously. | Sonnet | Yes (approval gates apply) |

All agents follow `agents/BLACKBOX_CONTRACT.md` — 13 rules covering scope discipline, evidence requirements, approval gates, commit checkpoints, and self-improvement.

## Usage

```
"Ask arch-orchestrator to analyze UserService and propose black-box boundaries"
"Ask arch-analyzer to find coupling issues in src/services/"
"Ask arch-planner to design a payment processing module"
"Ask arch-implementer to execute the approved refactoring plan"
"Ask arch-debugger to isolate and fix the checkout bug"
```

## Workflows

| Workflow | Agents | Approval Gate |
|----------|--------|---------------|
| Analysis only | Analyzer | — |
| Planning only | Planner | — |
| Full refactoring | Analyzer → Planner → Implementer | After plan, before implementation |
| Debug & fix | Debugger → (Implementer) | Before non-trivial fixes |
| Complete transformation | All agents in sequence | After plan + between phases |

## Documentation

- [Agent System Guide](docs/AGENTS.md) — Architecture, agents, workflows, troubleshooting
- [Black Box Principles](docs/PRINCIPLES.md) — Design philosophy (Steenberg, Parnas)
- [Code Examples](docs/EXAMPLES.md) — Before/after refactorings in 6 languages
- [Contributing](docs/CONTRIBUTING.md) — How to contribute

## License

MIT
