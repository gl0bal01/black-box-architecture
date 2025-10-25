# Prompt Versions - Quick Reference

This repository contains **9 prompt files** across **2 versions** (Compact and Enhanced) to suit different needs.

## Quick Selection Guide

| Your Need | Recommended Version |
|-----------|-------------------|
| **Creating Claude Code slash commands** | ⭐ **Compact** (`commands/` or `skill/`) |
| **Frequent usage, token efficiency matters** | ⭐ **Compact** (`commands/` or `skill/`) |
| **Reference documentation** | **Enhanced** (`enhanced/`) |
| **Learning the methodology** | **Enhanced** (`enhanced/`) |
| **Deep examples and explanations** | **Enhanced** (`enhanced/`) |
| **Daily production work** | ⭐ **Compact** (`commands/` or `skill/`) |

## All Versions

### 📘 Original Prompts (~900 words each)

**Purpose**: The foundational prompts with Eskil's black box principles
**Note**: These are the source inspiration, not included in this repository

| Original Source | Words | Description |
|------|-------|----------|
| `claude-code-prompt.md` | 669 | Development/refactoring basics |
| `claude-prompt.md` | 938 | Architecture planning basics |
| `cursor-prompt.md` | 1,115 | Debugging/testing basics |

**Strengths**: Concise, educational, easy to understand
**Limitations**: Less structured, freeform outputs, fewer examples

**This repository evolved from these originals**, creating structured compact and comprehensive enhanced versions.

---

### 📗 Enhanced Prompts (~2,900 words each)

**Purpose**: Comprehensive guides with extensive examples and workflows

| File | Words | Use Case |
|------|-------|----------|
| `refactor.md` | 2,388 | Complete refactoring guide |
| `plan.md` | 2,930 | Complete architecture guide |
| `debug.md` | 3,385 | Complete debugging guide |

**Strengths**:
- Detailed step-by-step protocols
- Multi-language examples (Python, TS, Go, Rust, Java)
- Comprehensive checklists
- Risk assessment frameworks
- Extensive anti-pattern lists

**Limitations**: Too long for slash commands (high token usage)

**Best For**:
- Reference documentation
- Understanding principles deeply
- One-time major refactoring
- Training material

---

### 📕 Compact Prompts (~835 words each) ⭐ RECOMMENDED

**Purpose**: Command-optimized prompts with structured workflows

| File | Words | Tokens (est.) | Use Case |
|------|-------|---------------|----------|
| `commands/arch.md` or `skill/refactor.md` | 565 | ~750 | Development command |
| `commands/arch-plan.md` or `skill/plan.md` | 854 | ~1,140 | Planning command |
| `commands/arch-debug.md` or `skill/debug.md` | 1,086 | ~1,450 | Debugging command |

**Strengths**:
- ✅ Structured 4-phase protocol (same as Enhanced)
- ✅ Mandatory output templates (same as Enhanced)
- ✅ Quality checklists (same as Enhanced)
- ✅ Constraint handling (same as Enhanced)
- ✅ **71% smaller than Enhanced**
- ✅ **Similar token usage to Original**

**What's Different from Enhanced**:
- Condensed examples (patterns only, not extensive multi-language)
- Removed verbose explanations
- Streamlined educational content
- Combined redundant sections

**Best For**:
- ⭐ Claude Code slash commands
- Frequent usage
- Token-efficient structured workflow
- Production development work

---

## Token Comparison

```
Compact:   ████████████ (~3,300 tokens)  Baseline ⭐
Enhanced:  ████████████████████████████████████ (~12,000 tokens)  +264%

           Best for:
Compact:   Daily work, commands, production ⭐
Enhanced:  Reference, training, deep learning
```

## Feature Comparison Matrix

| Feature | Compact | Enhanced |
|---------|---------|----------|
| **Black box principles** | ✅ | ✅ |
| **Structured workflow** | ✅ | ✅ |
| **Mandatory output format** | ✅ | ✅ |
| **Quality checklists** | ✅ | ✅ |
| **Multi-language examples** | ✅ Brief | ✅ Extensive |
| **Constraint handling** | ✅ | ✅ |
| **Anti-pattern detection** | ✅ Concise | ✅ Extensive |
| **Risk assessment** | ✅ | ✅ |
| **Tool integration (Claude Code)** | ✅ | ✅ |
| **Token efficiency** | ✅ | ❌ |
| **Suitable for commands** | ✅ | ❌ |
| **Educational depth** | ⚠️ Focused | ✅ Comprehensive |
| **Code examples per language** | 1-2 | 10+ |
| **File size** | ~6 KB each | ~21 KB each |

Legend: ✅ Yes | ❌ No | ⚠️ Partial

## Recommended Workflow

### For Individual Developers

1. **Create commands** using Compact versions (`commands/` or `skill/`)
2. **Reference** Enhanced versions (`enhanced/`) when needed
3. **Daily work**: Use Compact for production development

### For Teams

1. **Standardize** on Compact for slash commands
2. **Share** Enhanced versions as team documentation
3. **Onboarding**: New members read Enhanced, then use Compact daily

### For Learning

1. **Start with** Enhanced (`enhanced/`) to understand principles deeply
2. **Practice with** Compact (`commands/` or `skill/`) in real projects
3. **Reference back** to Enhanced when encountering complex scenarios

## Installation Examples

### Claude Code Slash Commands (Recommended)

```bash
# Navigate to your project
cd your-project/

# Create commands directory
mkdir -p .claude/commands

# Copy compact prompts as commands
cp path/to/black-box-architecture/commands/arch.md .claude/commands/
cp path/to/black-box-architecture/commands/arch-plan.md .claude/commands/
cp path/to/black-box-architecture/commands/arch-debug.md .claude/commands/

# Now use with:
# /arch - Refactor code
# /arch-plan - Design architecture
# /arch-debug - Debug systematically
```

### Global Reference (Enhanced Versions)

```bash
# Bookmark these locations for reference
# Enhanced prompts are in:
black-box-architecture/enhanced/refactor.md
black-box-architecture/enhanced/plan.md
black-box-architecture/enhanced/debug.md

# Open when you need:
# - Detailed examples
# - Comprehensive explanations
# - Training material
```

## File Locations

```
black-box-architecture/
├── commands/                           (Compact - for slash commands) ⭐
│   ├── arch.md                         (Refactoring command)
│   ├── arch-plan.md                    (Planning command)
│   └── arch-debug.md                   (Debugging command)
├── skill/                              (Compact - for Claude skill) ⭐
│   ├── refactor.md                     (Refactoring)
│   ├── plan.md                         (Planning)
│   ├── debug.md                        (Debugging)
│   └── skill.json                      (Skill metadata)
├── enhanced/                           (Comprehensive reference)
│   ├── refactor.md                     (Enhanced refactoring guide)
│   ├── plan.md                         (Enhanced planning guide)
│   ├── debug.md                        (Enhanced debugging guide)
│   ├── VERSIONS.md                     (This file)
│   └── README.md                       (Enhanced directory guide)
├── docs/                               (Documentation)
│   ├── INSTALLATION.md
│   ├── USAGE.md
│   ├── PRINCIPLES.md
│   ├── EXAMPLES.md
│   └── CONTRIBUTING.md
├── examples/                           (Code examples)
│   ├── python/
│   ├── typescript/
│   ├── go/
│   ├── rust/
│   └── c/
├── README.md                           (Main repository readme)
├── LICENSE                             (MIT License)
└── .gitignore
```

## When to Use Each Prompt

### Refactoring Prompts
**Files**: `commands/arch.md`, `skill/refactor.md` (compact), `enhanced/refactor.md` (detailed)
**For**: Hands-on development, refactoring, code improvement
**Use when**:
- Breaking apart monolithic code
- Creating module boundaries
- Improving maintainability
- Working in Claude Code environment

### Planning Prompts
**Files**: `commands/arch-plan.md`, `skill/plan.md` (compact), `enhanced/plan.md` (detailed)
**For**: Strategic architecture, system design, planning
**Use when**:
- Designing new systems
- Major architectural decisions
- Planning module breakdowns
- Creating technical specifications

### Debugging Prompts
**Files**: `commands/arch-debug.md`, `skill/debug.md` (compact), `enhanced/debug.md` (detailed)
**For**: Development, debugging, testing
**Use when**:
- Building new features
- Debugging complex issues
- Writing test suites
- Code review and quality checks

## Getting Started

```
1. New to black box architecture?
   → Read one Enhanced prompt (enhanced/), then use Compact daily

2. Want to create slash commands?
   → Use Compact versions (commands/ or skill/)

3. Need comprehensive examples?
   → Reference Enhanced versions (enhanced/)

4. Daily development work?
   → Always use Compact versions (commands/ or skill/) ⭐
```

## Support & Updates

- **Issues**: Report in the repository
- **Suggestions**: Propose improvements via PR
- **Updates**: Check repository releases for changelog


