# Enhanced Versions - Reference Documentation

This directory contains the **comprehensive, detailed** versions of the black box architecture prompts with extensive examples and explanations.

## 📁 Files in This Directory

### Enhanced Prompts (~63 KB total)

- **`refactor.md`** (~17 KB) - Comprehensive refactoring guide with multi-language examples
- **`plan.md`** (~20 KB) - Complete architecture planning methodology
- **`debug.md`** (~26 KB) - Detailed debugging and testing strategies

### Documentation

- **`VERSIONS.md`** - Quick reference guide comparing compact and enhanced versions

## 🎯 When to Use Enhanced vs. Compact

### Use Enhanced (these files) when:
- ✅ Learning the methodology for the first time
- ✅ Need detailed code examples in multiple languages (Python, TS, Go, Rust, Java)
- ✅ Want comprehensive explanations of principles
- ✅ Creating training material or documentation
- ✅ Doing one-time deep architectural analysis
- ✅ Need extensive before/after examples

### Use Compact (in `../commands/` and `../skills/`) when:
- ✅ Creating slash commands
- ✅ Frequent usage
- ✅ Token efficiency is critical
- ✅ You already understand the principles
- ✅ Production development work

## 📊 Size Comparison

| Version | Files | Total Size | Token Count | Best For |
|---------|-------|------------|-------------|----------|
| **Compact** | `../commands/`, `../skills/` | ~18 KB | ~3,300 tokens | Commands, daily use |
| **Enhanced** | This directory | ~63 KB | ~12,000 tokens | Reference, learning |

**Enhanced is ~3.5x larger** but provides:
- 10+ code examples per language
- Step-by-step tutorials
- Detailed pattern explanations
- Comprehensive testing strategies
- Educational deep-dives

## 🔍 What's Included in Enhanced

### 1. Multi-Language Code Examples

Each prompt includes extensive examples in:
- Python (class-based and functional)
- TypeScript (React components, APIs)
- Go (interface-driven design)
- Rust (trait-based patterns)
- Java (enterprise patterns)

### 2. Before/After Transformations

Detailed refactoring examples showing:
- Current state (coupled code)
- Analysis of problems
- Proposed black box design
- Final implementation
- Testing strategies

### 3. Educational Content

- Philosophy explanations
- Risk assessment frameworks
- Team organization advice
- Architecture decision records
- Success metrics definitions

### 4. Comprehensive Protocols

- Phase-by-phase workflows
- Mandatory output templates
- Quality validation checklists
- Debugging methodologies
- Testing patterns

## 📖 Recommended Usage

### For New Users

1. **Start here** - Read one enhanced prompt fully
2. **Understand principles** - See VERSIONS.md for version comparison
3. **Practice with compact** - Use `../commands/` for actual work
4. **Reference back** - Return to enhanced when you need detailed examples

### For Experienced Users

1. **Use compact** - For daily development work
2. **Reference enhanced** - When you encounter edge cases
3. **Share enhanced** - For onboarding new team members

## 🚀 Using Enhanced Prompts

### Manual Paste (Best Method)

```bash
# Copy the content of an enhanced prompt
cat enhanced/refactor.md

# Paste into Claude/Cursor when you need comprehensive guidance
```

### One-Time Analysis

When doing major refactoring or architecture work, paste an enhanced prompt for:
- Complete methodology
- Extensive examples
- Detailed checklists
- Risk assessment frameworks

## 📝 Files Overview

### refactor.md
**Purpose**: Hands-on development and code refactoring

**Includes**:
- 4-phase execution protocol (Discovery → Analysis → Design → Implementation)
- Tool integration (Glob, Grep, Read, Edit)
- Multi-language refactoring examples
- Quality gates and validation checklists
- Migration path planning

**Token cost**: ~5,500 tokens

---

### plan.md
**Purpose**: Strategic architecture planning and system design

**Includes**:
- Strategic consultation protocol
- Primitive discovery framework
- Module architecture templates
- Risk assessment tables
- Implementation roadmaps
- Team organization guidance

**Token cost**: ~4,800 tokens

---

### debug.md
**Purpose**: Development, testing, and systematic debugging

**Includes**:
- Development workflow protocol
- Systematic debugging methodology
- Bug analysis templates
- Comprehensive testing patterns
- Multi-language test examples
- Quality checklists

**Token cost**: ~6,200 tokens

---

## 🔗 See Also

- [Compact versions](../commands/) - Token-optimized for commands
- [Skill package](../skills/) - Claude skill installation
- [Main README](../README.md) - Repository overview
- [VERSIONS.md](VERSIONS.md) - Version comparison guide

## 💡 Pro Tip

**For maximum efficiency**:
- Use compact versions (skill or commands) for 95% of work
- Paste enhanced versions when:
  - Onboarding new developers
  - Major architectural decisions
  - Complex refactoring projects
  - Need detailed examples in specific language

---

**These enhanced prompts are your reference library. The compact versions are your daily tools.**
