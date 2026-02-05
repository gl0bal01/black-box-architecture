# Installation Guide

This guide covers how to install and use the Black Box Architecture prompts in your projects.

## Table of Contents

- [As Agents (Recommended)](#as-agents-recommended)
- [Legacy: Claude Skills](#legacy-claude-skills)
- [Legacy: Slash Commands](#legacy-slash-commands)
- [Legacy: Manual Usage](#legacy-manual-usage)
- [Verification](#verification)

---

## As Agents (Recommended)

### Prerequisites

- Claude Code CLI installed

### Installation

```bash
# Clone the repository
git clone https://github.com/gl0bal01/black-box-architecture.git

# Install agents for this project
mkdir -p .claude/agents
cp -r black-box-architecture/agents .claude/agents/
```

### Verification

```bash
# Check that agents are present
ls -la .claude/agents/
# Should show: arch-orchestrator.md, arch-analyzer.md, etc.
```

### Usage

Use the orchestrator for most tasks:
- "Ask arch-orchestrator to analyze module boundaries in UserService"
- "Ask arch-orchestrator to design a module map for payments"
- "Ask arch-orchestrator to isolate and fix a failing integration"

---

## Legacy: Claude Skills

Skills are legacy prompts kept for compatibility. Agents are the preferred path.

### Installation

Skills in Claude Code are discovered automatically from the `~/.claude/skills/` directory (personal) or `.claude/skills/` directory (project-specific).

```bash
# Clone the repository
git clone https://github.com/gl0bal01/black-box-architecture.git

# For personal use (available in all your projects)
mkdir -p ~/.claude/skills
cp -r black-box-architecture/docs/legacy/skills ~/.claude/skills/black-box-architecture

# OR for project-specific use (shared with team via git)
mkdir -p .claude/skills
cp -r black-box-architecture/docs/legacy/skills .claude/skills/black-box-architecture
```

---

## Legacy: Slash Commands

### Prerequisites

- Claude Code CLI installed
- Project with `.claude/commands/` directory (or will create)

### Installation

#### Option 1: Clone and Copy

```bash
# Clone this repository
git clone https://github.com/gl0bal01/black-box-architecture.git

# Navigate to your project
cd /path/to/your/project

# Create commands directory if it doesn't exist
mkdir -p .claude/commands

# Copy command files
cp /path/to/black-box-architecture/docs/legacy/commands/*.md .claude/commands/
```

#### Option 2: Direct Download

```bash
# Navigate to your project
cd /path/to/your/project

# Create commands directory
mkdir -p .claude/commands

# Download command files
curl -o .claude/commands/arch.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/docs/legacy/commands/arch.md
curl -o .claude/commands/arch-plan.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/docs/legacy/commands/arch-plan.md
curl -o .claude/commands/arch-debug.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/docs/legacy/commands/arch-debug.md
```

### Usage

After installation, use the commands in Claude Code:

```bash
# Refactor code with black box principles
/arch Analyze the UserService class and suggest modular refactoring

# Design system architecture
/arch-plan Design a microservices architecture for an e-commerce platform

# Debug with modular isolation
/arch-debug This integration test is failing between PaymentService and OrderService
```

### Command Reference

| Command | Purpose | Best For |
|---------|---------|----------|
| `/arch` | Refactor and analyze code | Breaking apart monoliths, creating module boundaries |
| `/arch-plan` | Design architecture | New systems, strategic planning, module design |
| `/arch-debug` | Debug systematically | Integration issues, testing strategies, bug isolation |

---

## Legacy: Manual Usage

If you prefer not to install agents, you can use the legacy prompts manually.

### Prerequisites

- Access to Claude Code, Claude.ai, or Cursor
- The prompt files from this repository

### Usage

1. **Copy the appropriate prompt** from the `docs/legacy/skills/` directory:
   - `refactor.md` - For code refactoring
   - `plan.md` - For architecture planning
   - `debug.md` - For debugging

2. **Paste into your AI tool** at the start of your conversation

3. **Provide your code context** (using tools like [repomix](https://github.com/yamadashy/repomix))

4. **Ask your question**

**Example workflow:**

```bash
# Extract code context
npx repomix --include "src/**/*.ts" --output context.xml

# Then in Claude:
# 1. Paste the refactor.md prompt
# 2. Attach context.xml
# 3. Ask: "Refactor the UserService to use black box modules"
```

---

## Verification

### Verify Agent Installation

```bash
ls -la .claude/agents/
# Should show: arch-orchestrator.md, arch-analyzer.md, etc.
```

### Verify Legacy Skill Installation

```bash
# Check if skill directory exists
ls -la ~/.claude/skills/black-box-architecture
# OR for project-specific
ls -la .claude/skills/black-box-architecture

# Test the skill in Claude Code
claude
> "What skills are available?"
> "Can you explain the black box architecture principles you know?"
```

The AI should demonstrate knowledge of black box principles and the legacy structured output format.

### Verify Legacy Command Installation

```bash
# Check if commands exist
ls -la .claude/commands/arch*.md

# Should show:
# arch.md
# arch-plan.md
# arch-debug.md

# Test a command
claude
> /arch
```

The command should expand and show the black box refactoring prompt.

### Verify Prompt Content

```bash
# Check that prompts are not corrupted
head -n 5 .claude/commands/arch.md

# Should start with:
# # Black Box Architecture - Development & Refactoring (Compact)
```

---

## Troubleshooting

### Agent Not Available

**Problem**: Agents are not listed or not used

**Solutions**:
1. Verify `.claude/agents/` exists: `ls -la .claude/agents/`
2. Confirm files are present (arch-orchestrator.md, arch-analyzer.md, etc.)
3. Restart Claude Code CLI

### Legacy Skill Not Discovered

**Problem**: Claude doesn't seem to use the skill

**Solutions**:
1. Verify the skill directory exists: `ls ~/.claude/skills/black-box-architecture`
2. Check that the skill files are present (refactor.md, plan.md, debug.md, skill.json)
3. Restart Claude Code
4. Try asking explicitly: "What skills are available?"
5. Try manual installation via slash commands instead

### Legacy Commands Not Showing Up

**Problem**: `/arch` command doesn't work

**Solutions**:
1. Verify `.claude/commands/` directory exists
2. Check file permissions: `chmod 644 .claude/commands/*.md`
3. Restart Claude Code CLI
4. Try running from project root directory

### Prompts Too Long

**Problem**: Hitting token limits

**Solutions**:
1. These are already compact versions (~750-1,450 tokens each)
2. Use only one variant at a time
3. For reference, see enhanced versions separately
4. Consider using skill installation (more efficient)

### File Encoding Issues

**Problem**: Prompts display with garbled characters

**Solutions**:
1. Ensure files are UTF-8 encoded
2. Re-download from repository
3. Check for BOM (Byte Order Mark) issues

---

## Updating

### Update Agents

```bash
# Pull latest changes
cd /path/to/black-box-architecture
git pull

# Copy to project agents directory (overwrites existing)
cp -r agents .claude/agents/
```

### Update Legacy Skill

```bash
# Pull latest changes
cd /path/to/black-box-architecture
git pull

# Copy to skills directory (overwrites existing)
cp -r docs/legacy/skills ~/.claude/skills/black-box-architecture
# OR for project-specific
cp -r docs/legacy/skills .claude/skills/black-box-architecture
```

### Update Legacy Commands

```bash
# Re-download latest versions
cd /path/to/your/project
curl -o .claude/commands/arch.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/docs/legacy/commands/arch.md
curl -o .claude/commands/arch-plan.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/docs/legacy/commands/arch-plan.md
curl -o .claude/commands/arch-debug.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/docs/legacy/commands/arch-debug.md
```

---

## Uninstallation

### Uninstall Agents

```bash
rm -rf .claude/agents/
```

### Uninstall Legacy Skill

```bash
# Remove from personal skills
rm -rf ~/.claude/skills/black-box-architecture

# OR remove from project skills
rm -rf .claude/skills/black-box-architecture
```

### Uninstall Legacy Commands

```bash
# Remove command files
rm .claude/commands/arch.md
rm .claude/commands/arch-plan.md
rm .claude/commands/arch-debug.md
```

---

## Next Steps

After installation, see:
- [Usage Guide](USAGE.md) - Learn how to use agents and legacy prompts
- [Examples](EXAMPLES.md) - See real-world refactoring examples
- [Principles](PRINCIPLES.md) - Understand the methodology
