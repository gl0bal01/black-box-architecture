# Black Box Architecture

> Transform any codebase into modular, maintainable "black boxes" using Eskil Steenberg's architecture principles.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](docs/CONTRIBUTING.md)

AI prompts optimized for **Claude Code**, **Claude** that teach your AI assistant to think in terms of replaceable, modular components.

## 📑 Table of Contents

- [What This Is](#-what-this-is)
- [Quick Start](#-quick-start)
  - [Option 1: Agents (Recommended)](#option-1-agents-recommended)
  - [Option 2: Legacy Prompts (Skills/Commands)](#option-2-legacy-prompts-skillscommands)
- [Best Daily Workflow](#-best-daily-workflow)
- [Core Philosophy](#-core-philosophy)
- [What's Included](#-whats-included)
- [Agents](#-agents)
- [Legacy Prompts](#-legacy-prompts)
- [Example Usage](#-example-usage)
- [Documentation](#-documentation)
- [Real-World Examples](#-real-world-examples)
- [Learn More](#-learn-more)
- [How It Works](#-how-it-works)
- [Contributing](#-contributing)
- [Next Steps](#-next-steps)

## 🎯 What This Is

Three specialized AI prompts that apply [Eskil Steenberg's](https://www.youtube.com/watch?v=sSpULGNHyoI) battle-tested principles:

- **Black box interfaces** - Clean APIs between modules
- **Replaceable components** - If you can't understand it, rewrite it
- **Constant velocity** - Write 5 lines today vs. edit 1 line later
- **Single responsibility** - One module, one person

## 🚀 Quick Start

### Option 1: Agents (Recommended)

```bash
# Clone the repository
git clone https://github.com/gl0bal01/black-box-architecture.git

# Install agents for this project
mkdir -p .claude/agents
cp -r black-box-architecture/agents/* .claude/agents/
```

### Option 2: Legacy Prompts (Skills/Commands)

Legacy prompts are still available but no longer the recommended path.

```bash
# Skills (legacy)
mkdir -p .claude/skills
cp -r black-box-architecture/docs/legacy/skills .claude/skills/black-box-architecture

# Commands (legacy)
mkdir -p .claude/commands
cp -r black-box-architecture/docs/legacy/commands/ .claude/commands/
```

## ✅ Best Daily Workflow

1. Use `arch-orchestrator` for most tasks.
2. Ask for concise output by default.
3. Request a full report only for big changes.
4. Approve any dependency, public API, or schema changes explicitly.

## 💡 Core Philosophy

**"It's faster to write 5 lines of code today than to write 1 line today and then have to edit it in the future."**
— Eskil Steenberg

These prompts optimize for:
- **Human cognitive load** over algorithmic efficiency
- **Long-term maintainability** over short-term cleverness
- **Team scalability** (one person per module)
- **Constant developer velocity** regardless of project size

## 📦 What's Included

- **Agents pack** in `agents/` with orchestrator + specialists
- **Shared contract** in `agents/AGENTS_CONTRACT.md`
- **Legacy prompts** in `docs/legacy/skills/` and `docs/legacy/commands/` (deprecated)
- **Examples and docs** in `examples/` and `docs/`

## 🤖 Agents

Agents are the primary, recommended path. They enforce:
- scope discipline
- approval gates for risky changes
- evidence-backed findings
- concise, reviewable outputs

## 🧩 Legacy Prompts

`docs/legacy/skills/` and `docs/legacy/commands/` are legacy prompts kept for compatibility.
They may drift from the contract and are not the best daily path.

## 🤖 Autonomous Agents (Recommended)

For daily architectural work, use the specialized agent system:

### Installation

```bash
# Copy agents to your project
cp -r agents/* .claude/agents/

# The orchestrator coordinates all agents automatically
```

### Agent Architecture

The agent system follows black box principles itself - specialized agents with clear responsibilities:

| Agent | Role | Autonomous Actions |
|-------|------|-------------------|
| **arch-orchestrator** | Coordination | Analyzes requests, delegates to specialists, assembles results |
| **arch-analyzer** | Analysis | Explores codebases, identifies violations, maps dependencies |
| **arch-planner** | Design | Designs architectures, creates roadmaps, assesses risks |
| **arch-implementer** | Implementation | Refactors code, maintains boundaries, verifies changes |
| **arch-debugger** | Debugging | Isolates bugs to modules, proposes fixes, maintains integrity |

### Workflows

**Analysis Only**: arch-analyzer explores codebase and reports findings

**Planning Only**: arch-planner designs architecture from requirements

**Full Refactoring**: arch-analyzer → arch-planner → [USER APPROVAL] → arch-implementer

**Debug & Fix**: arch-debugger → arch-implementer (if fix needed)

**Complete Transformation**: All agents work together for major architectural overhaul

### Why Agents?

- They follow a shared contract for consistent, reviewable outputs.
- They enforce approval gates for dependencies, APIs, and schemas.
- They scale from quick analysis to full refactors without changing tools.

**Learn More:**
- 📖 [Complete Agent Documentation](docs/AGENTS.md) - Detailed guide for each agent
- 🔄 [Agent Workflows](docs/WORKFLOWS.md) - Step-by-step examples
- 🔗 [Integration Examples](docs/INTEGRATION_EXAMPLES.md) - How agents coordinate
- ⚙️ [Agent Specifications](agents/agent.json) - Technical specifications

## 🎬 Example Usage

### Refactoring Example

```bash
# Using the orchestrator
Ask arch-orchestrator: Analyze UserService and propose black-box boundaries
```

**What you get:**
- Current architecture analysis with `file:line` references
- Identified primitives and coupling issues
- Proposed black box module design
- Step-by-step refactoring plan
- Risk assessment and mitigation
- Quality validation checklist

### Planning Example

```bash
# Using the orchestrator
Ask arch-orchestrator: Design a module map for a real-time chat app
```

**What you get:**
- System primitives identification
- Module architecture with clear boundaries
- Interface specifications
- Implementation roadmap (phased)
- Risk assessment
- Team organization recommendations

## 📚 Documentation

### Getting Started
- **[Installation Guide](docs/INSTALLATION.md)** - Setup for agents and legacy prompts
- **[Principles Guide](docs/PRINCIPLES.md)** - Eskil Steenberg's methodology explained

### Using the Tools
- **[Usage Guide](docs/USAGE.md)** - Agent workflow and legacy prompts
- **[Agent System Guide](docs/AGENTS.md)** - Complete agent documentation
- **[Workflows Guide](docs/WORKFLOWS.md)** - Step-by-step examples for each agent
- **[Integration Examples](docs/INTEGRATION_EXAMPLES.md)** - How agents coordinate together

### Reference & Examples
- **[Code Examples](docs/EXAMPLES.md)** - Before/after transformations in multiple languages
- **[Real-World Examples](examples/)** - Complete refactoring examples

### Contributing
- **[Contributing Guide](docs/CONTRIBUTING.md)** - How to contribute, report issues, add examples

## 🔍 Real-World Examples

See the [`examples/`](examples/) directory for complete before/after refactoring examples in:
- **Python** - Repository pattern, service abstractions
- **TypeScript** - Interface-driven design, dependency injection
- **Go** - Interface composition, struct patterns
- **Rust** - Trait-based black boxes, generic implementations
- **C** - Opaque types, function pointers (Eskil's approach!)
- **PHP** - Service layer, strategy pattern, Laravel integration

## 🎓 Learn More

### Core Resources

**Original Source:**

Watch Eskil Steenberg's complete lecture: [Architecting LARGE Software Projects](https://www.youtube.com/watch?v=sSpULGNHyoI)

This legend has built 3D engines, networked games, and complex systems all in C using these exact principles.

**Complete Documentation:**
- 📖 [Full Documentation Index](#-documentation)
- 🎨 [Skill System Guide](docs/legacy/skills/SKILL.md)
- 🤖 [Agent System Guide](docs/AGENTS.md)
- 📝 [Workflow Examples](docs/WORKFLOWS.md)

### Core Principles

1. **Primitive-First Design** - Identify core data types that flow through your system
2. **Black Box Boundaries** - Modules communicate only through documented interfaces
3. **Replaceable Components** - Any module can be rewritten using only its interface
4. **Single Responsibility** - One module = one person can own it
5. **Wrap Dependencies** - Never depend directly on code you don't control

## 🛠️ How It Works

- A shared contract governs agent behavior and output structure.
- Agents default to concise outputs and switch to full reports when needed.
- Approval gates prevent risky changes without explicit sign-off.
- Evidence and verification are required for non-trivial claims.

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](docs/CONTRIBUTING.md) for:
- How to report issues
- Suggesting improvements
- Sharing successful patterns
- Adding language examples

## 🔗 Related Resources

- [Original ai-architecture-prompts](https://github.com/Alexanderdunlop/ai-architecture-prompts) - Inspiration and foundation
- [Eskil's Video: Architecting LARGE Software Projects](https://www.youtube.com/watch?v=sSpULGNHyoI)
- [Enhanced Versions](enhanced/) - Legacy reference documentation

## 📄 License

MIT License - see [LICENSE](LICENSE) for details.

---

## 🌟 Why This Works

Traditional software grows complex over time. Developer velocity slows. Features take longer. Bugs multiply.

**Black box architecture** maintains constant velocity:
- Modules are small enough for one person
- Changes are isolated to single modules
- Components can be completely rewritten
- New developers can contribute immediately

**These AI prompts** teach your assistant to think this way automatically.

## 🚀 Next Steps

1. Install agents and run a small analysis with `arch-orchestrator`.
2. Use concise output for daily work, request full reports only when needed.
3. Approve dependencies, public API changes, and schemas explicitly.
4. If you must use legacy prompts, keep them aligned with the contract.

Watch Eskil Steenberg's [complete lecture](https://www.youtube.com/watch?v=sSpULGNHyoI) - the foundation of everything here.

---

**Not affiliated with Anthropic, Eskil Steenberg, or any tools mentioned. Battle-tested principles from real development work.**
