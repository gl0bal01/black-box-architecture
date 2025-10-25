# Installation Guide

This guide covers how to install and use the Black Box Architecture prompts in your projects.

## Table of Contents

- [As a Claude Skill](#as-a-claude-skill)
- [As Slash Commands](#as-slash-commands)
- [Manual Usage](#manual-usage)
- [Verification](#verification)

---

## As a Claude Skill

### Prerequisites

- Claude Code CLI installed
- Access to Claude skill system (if available)

### Installation

```bash
# Install the skill globally
claude skill install black-box-architecture

# Verify installation
claude skill list
```

### Usage

Once installed, the skill is available in all your projects:

```bash
# Start Claude Code in any project
cd your-project/

# Use natural language to invoke
# Claude will automatically use the black box architecture methodology
```

**Example prompts:**
- "Refactor this UserService using black box principles"
- "Design the architecture for a payment processing system"
- "Debug this integration issue using modular isolation"

The AI will automatically apply the appropriate variant (refactor, plan, or debug) based on your request.

---

## As Slash Commands

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
cp /path/to/black-box-architecture/commands/*.md .claude/commands/
```

#### Option 2: Direct Download

```bash
# Navigate to your project
cd /path/to/your/project

# Create commands directory
mkdir -p .claude/commands

# Download command files
curl -o .claude/commands/arch.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/commands/arch.md
curl -o .claude/commands/arch-plan.md https://raw.githubusercontent.com/gl0bal01//black-box-architecture/main/commands/arch-plan.md
curl -o .claude/commands/arch-debug.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/commands/arch-debug.md
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

## Manual Usage

If you prefer not to install as a skill or command, you can use the prompts manually:

### Prerequisites

- Access to Claude Code, Claude.ai, or Cursor
- The prompt files from this repository

### Usage

1. **Copy the appropriate prompt** from the `skill/` directory:
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

### Verify Skill Installation

```bash
# Check if skill is installed
claude skill list | grep black-box-architecture

# Test the skill
claude
> "Can you explain the black box architecture principles you know?"
```

The AI should demonstrate knowledge of:
- 4-phase execution protocol
- Black box principles
- Structured output format
- Quality validation checklists

### Verify Command Installation

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

### Skill Not Found

**Problem**: `claude skill install black-box-architecture` returns "Skill not found"

**Solutions**:
1. Check if Claude skill system is available in your version
2. Try manual installation via slash commands instead
3. Verify network connection for skill registry

### Commands Not Showing Up

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

### Update Skill

```bash
# Update to latest version
claude skill update black-box-architecture

# Or reinstall
claude skill uninstall black-box-architecture
claude skill install black-box-architecture
```

### Update Commands

```bash
# Re-download latest versions
cd /path/to/your/project
curl -o .claude/commands/arch.md https://raw.githubusercontent.com/gl0bal01//black-box-architecture/main/commands/arch.md
curl -o .claude/commands/arch-plan.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/commands/arch-plan.md
curl -o .claude/commands/arch-debug.md https://raw.githubusercontent.com/gl0bal01/black-box-architecture/main/commands/arch-debug.md
```

---

## Uninstallation

### Uninstall Skill

```bash
claude skill uninstall black-box-architecture
```

### Uninstall Commands

```bash
# Remove command files
rm .claude/commands/arch.md
rm .claude/commands/arch-plan.md
rm .claude/commands/arch-debug.md
```

---

## Next Steps

After installation, see:
- [Usage Guide](USAGE.md) - Learn how to use the prompts effectively
- [Examples](EXAMPLES.md) - See real-world refactoring examples
- [Principles](PRINCIPLES.md) - Understand the methodology


