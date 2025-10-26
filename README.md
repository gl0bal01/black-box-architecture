# Black Box Architecture

> Transform any codebase into modular, maintainable "black boxes" using Eskil Steenberg's architecture principles.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](docs/CONTRIBUTING.md)

AI prompts optimized for **Claude Code**, **Claude** that teach your AI assistant to think in terms of replaceable, modular components.

## 🎯 What This Is

Three specialized AI prompts that apply [Eskil Steenberg's](https://www.youtube.com/watch?v=sSpULGNHyoI) battle-tested principles:

- **Black box interfaces** - Clean APIs between modules
- **Replaceable components** - If you can't understand it, rewrite it
- **Constant velocity** - Write 5 lines today vs. edit 1 line later
- **Single responsibility** - One module, one person

## 🚀 Quick Start

### Option 1: As a Claude Skill (Recommended)

```bash
# Install the skill (if Claude skill system is available)
claude skill install black-box-architecture

# Use in any project
# The AI will automatically apply black box principles
```

### Option 2: As Slash Commands

```bash
# Copy commands to your project
cp -r commands/ .claude/commands/

# Use with:
# /arch - Refactor code with black box principles
# /arch-plan - Design system architecture
# /arch-debug - Debug with modular isolation
```

## 💡 Core Philosophy

**"It's faster to write 5 lines of code today than to write 1 line today and then have to edit it in the future."**
— Eskil Steenberg

These prompts optimize for:
- **Human cognitive load** over algorithmic efficiency
- **Long-term maintainability** over short-term cleverness
- **Team scalability** (one person per module)
- **Constant developer velocity** regardless of project size

## 📦 What's Included

### Three Specialized Prompts

| Prompt | Use Case | Token Cost |
|--------|----------|------------|
| **refactor** | Break apart monoliths, create module boundaries | ~750 tokens |
| **plan** | Design new systems, strategic architecture | ~1,140 tokens |
| **debug** | Systematic debugging, testing strategies | ~1,450 tokens |

### Key Features

- ✅ **Structured 4-phase workflow** (Discovery → Analysis → Design → Implementation)
- ✅ **Mandatory output templates** (consistent, parseable responses)
- ✅ **Quality validation checklists** (explicit success criteria)
- ✅ **Multi-language support** (Python, TypeScript, Go, Rust, C, PHP)
- ✅ **Token-optimized** (compact but comprehensive)
- ✅ **Tool integration** (Glob, Grep, Read, Edit for Claude Code)

## 🎬 Example Usage

### Refactoring Example

```bash
# Using as a command
/arch Analyze the UserService class and break it into black box modules
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
# Using the skill or command
/arch-plan I'm building a real-time chat app with React and Node.js
```

**What you get:**
- System primitives identification
- Module architecture with clear boundaries
- Interface specifications
- Implementation roadmap (phased)
- Risk assessment
- Team organization recommendations

## 📚 Documentation

- [Installation Guide](docs/INSTALLATION.md) - Detailed setup for skills and commands
- [Usage Guide](docs/USAGE.md) - Workflows and examples
- [Principles Guide](docs/PRINCIPLES.md) - Eskil's methodology explained
- [Code Examples](docs/EXAMPLES.md) - Before/after transformations
- [Contributing](docs/CONTRIBUTING.md) - How to contribute

## 🔍 Real-World Examples

See the [`examples/`](examples/) directory for complete before/after refactoring examples in:
- **Python** - Repository pattern, service abstractions
- **TypeScript** - Interface-driven design, dependency injection
- **Go** - Interface composition, struct patterns
- **Rust** - Trait-based black boxes, generic implementations
- **C** - Opaque types, function pointers (Eskil's approach!)
- **PHP** - Service layer, strategy pattern, Laravel integration

## 🎓 Learn More

### Original Source

Watch Eskil Steenberg's complete lecture: [Architecting LARGE Software Projects](https://www.youtube.com/watch?v=sSpULGNHyoI)

This legend has built 3D engines, networked games, and complex systems all in C using these exact principles.

### Core Principles

1. **Primitive-First Design** - Identify core data types that flow through your system
2. **Black Box Boundaries** - Modules communicate only through documented interfaces
3. **Replaceable Components** - Any module can be rewritten using only its interface
4. **Single Responsibility** - One module = one person can own it
5. **Wrap Dependencies** - Never depend directly on code you don't control

## 🛠️ How It Works

### The 4-Phase Protocol

Each prompt follows a structured workflow:

**Phase 1: Discovery (15-20%)**
- Map codebase structure
- Find core primitives
- Read critical files
- STOP and confirm understanding

**Phase 2: Analysis (25-30%)**
- Identify black box boundaries
- Map dependencies
- Find coupling violations
- Document with file:line references

**Phase 3: Design (30-35%)**
- Design clean interfaces
- Show before/after examples
- Plan migration path
- Get user approval

**Phase 4: Implementation (30-35%)**
- Refactor one module at a time
- Surgical edits with tests
- Validate continuously
- Commit incrementally

### Structured Output Format

All responses follow a consistent template:

```markdown
## 🔍 Current Architecture
[Primitives, modules, coupling issues, violations]

## 🎯 Proposed Black Box Design
[Module designs with interfaces]

## 📝 Implementation Steps
[Specific, actionable steps]

## ⚠️ Risks & Mitigation
[What could go wrong + how to prevent]

## ✅ Quality Gates
[Validation checklist]
```

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](docs/CONTRIBUTING.md) for:
- How to report issues
- Suggesting improvements
- Sharing successful patterns
- Adding language examples

## 📊 Token Efficiency

These prompts are **token-optimized** for frequent use:

- Compact versions: ~3,300 tokens total
- Enhanced versions available (12,000 tokens) for reference
- 71% smaller while maintaining full structured workflow

## 🔗 Related Resources

- [Original ai-architecture-prompts](https://github.com/Alexanderdunlop/ai-architecture-prompts) - Inspiration and foundation
- [Eskil's Video: Architecting LARGE Software Projects](https://www.youtube.com/watch?v=sSpULGNHyoI)
- [Enhanced Versions](enhanced/) - Comprehensive reference documentation

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

---

**Not affiliated with Anthropic, Eskil Steenberg, or any tools mentioned. Battle-tested principles from real development work.**

