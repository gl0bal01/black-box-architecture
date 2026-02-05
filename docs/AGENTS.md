# Black Box Architecture - Agent System Guide

Complete guide to using the autonomous agent system for complex architectural work.

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Agent Architecture](#agent-architecture)
- [Individual Agents](#individual-agents)
- [Workflows](#workflows)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Overview

The Black Box Architecture agent system provides **autonomous specialists** that handle complex, multi-step architectural tasks without constant guidance.

Agents follow a shared contract: `agents/AGENTS_CONTRACT.md`.
Legacy prompts (`docs/legacy/skills/` and `docs/legacy/commands/`) are kept for compatibility but are not recommended for daily use.

### When to Use Agents vs Commands vs Skills (Legacy)

| Feature | Commands | Skills | Agents |
|---------|----------|--------|--------|
| **Trigger** | Manual (/command) | Auto-discovered | Delegated by orchestrator |
| **Autonomy** | Template execution | Knowledge reference | Multi-step reasoning |
| **Context** | Shared with main conversation | Shared with main conversation | Separate context window |
| **Best For** | Quick workflows | Packaged expertise | Complex tasks requiring decision-making |
| **Complexity** | Low | Medium | High |

**Use agents when:**
- Task requires multiple steps with autonomous decision-making
- Need to coordinate across multiple specializations (analysis → design → implementation)
- Want to prevent context pollution in main conversation
- Complex workflow with checkpoints and approvals

**Use commands when (legacy):**
- Simple, straightforward workflow
- User wants direct control over each step
- Quick template-based responses

**Use skills when (legacy):**
- Providing reusable expertise that should be auto-discovered
- Building knowledge modules agents can leverage
- Creating packaged capabilities

## Installation

### Project-Specific (Recommended)

```bash
# Clone the repository
git clone https://github.com/gl0bal01/black-box-architecture.git

# Copy agents to your project
cp -r black-box-architecture/agents .claude/agents/

# Commit to share with team
git add .claude/agents/
git commit -m "Add black box architecture agents"
```

### Personal (All Projects)

```bash
# Copy to home directory
cp -r black-box-architecture/agents ~/.claude/agents/

# Available in all your projects
```

### Verify Installation

Check that agents are available:

```bash
ls -la .claude/agents/
# Should show: arch-orchestrator.md, arch-analyzer.md, etc.
```

## Agent Architecture

The agent system follows black box principles itself:

```
┌─────────────────────────────────────────────────────────┐
│                    User Request                          │
└───────────────────┬─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────┐
│              arch-orchestrator (Meta-Agent)              │
│  • Analyzes request type                                 │
│  • Plans workflow                                        │
│  • Delegates to specialists                              │
│  • Assembles results                                     │
└───────┬─────────┬────────────┬───────────┬──────────────┘
        │         │            │           │
        ▼         ▼            ▼           ▼
    ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
    │Analyzer│ │Planner │ │Impl-   │ │Debug-  │
    │        │ │        │ │ementer │ │ger     │
    └────┬───┘ └───┬────┘ └───┬────┘ └───┬────┘
         │         │           │           │
         ▼         ▼           ▼           ▼
    ┌────────────────────────────────────────┐
    │        Shared Contract (AGENTS_CONTRACT.md)       │
    │  • scope • evidence • approvals • verification    │
    └────────────────────────────────────────┘
```

### Design Principles

**1. Separation of Concerns**
- Each agent has ONE responsibility
- No overlap between agent domains
- Clear boundaries

**2. Composability**
- Agents work together through orchestrator
- Well-defined interfaces between agents
- Results pass cleanly between agents

**3. Context Isolation**
- Each agent has separate context window
- Prevents context pollution
- Maintains focus on specific task

**4. Legacy Prompts (Optional)**
- Skills/commands can be used for compatibility
- Agents do not require them for daily use
- Keep legacy prompts aligned with the shared contract if used

## Individual Agents

### arch-orchestrator

**Role**: Coordination specialist

**Responsibility**: Receive requests, determine workflow, delegate to specialists, assemble results

**Input**: User request for architectural work

**Output**: Coordinated result from appropriate specialists

**Delegates To**: All other agents

**Legacy Prompts**: None (coordinates others)

**When to Use Directly**: When you have a complex request requiring multiple agents

**Example Request**:
```
"Analyze the user authentication system, design improvements,
and implement them following black box principles"
```

**Workflow**:
1. Analyzes request (identifies: analysis + design + implementation)
2. Delegates to arch-analyzer
3. Receives analysis, delegates to arch-planner
4. Receives plan, requests user approval
5. After approval, delegates to arch-implementer
6. Assembles final report

---

### arch-analyzer

**Role**: Analysis specialist

**Responsibility**: Explore codebase, identify architectural issues, map dependencies

**Input**: Codebase path/scope, optional focus area

**Output**: Comprehensive analysis with violations and recommendations

**Delegates To**: None (leaf agent)

**Legacy Prompts**: Optional (refactor/debug)

**When to Use Directly**: When you only want analysis, no changes

**Example Request**:
```
"Analyze the src/services/ directory and identify coupling issues"
```

**Workflow**:
1. Discovery: Maps structure, finds primitives
2. Dependency mapping: Analyzes imports, builds graph
3. Violation detection: Finds leaky abstractions, god objects, etc.
4. Opportunity identification: Suggests refactorings
5. Returns detailed report with file:line references

**Output Format**:
- Codebase overview
- Current architecture
- Dependency analysis
- Black box violations
- Recommendations with priorities
- Metrics (modularity score, coupling level, etc.)

---

### arch-planner

**Role**: Design specialist

**Responsibility**: Design black box architectures, create implementation roadmaps

**Input**: Problem statement OR analysis from arch-analyzer

**Output**: Complete architecture design with roadmap and risks

**Delegates To**: None (leaf agent)

**Legacy Prompts**: Optional (plan)

**When to Use Directly**: When designing new features or systems from scratch

**Example Request**:
```
"Design the architecture for a real-time notification system
that needs to scale to 1M users"
```

**Workflow**:
1. Problem understanding: Clarifies requirements and constraints
2. Primitive discovery: Identifies core data types
3. Architecture design: Defines modules and interfaces
4. Roadmap creation: Creates phased implementation plan
5. Risk assessment: Identifies and mitigates risks

**Output Format**:
- Executive summary
- Problem analysis
- System primitives
- Module architecture (with interfaces)
- Dependency structure
- Implementation roadmap (phased)
- Risk assessment
- Success metrics
- Architecture decision records (ADRs)

---

### arch-implementer

**Role**: Implementation specialist

**Responsibility**: Execute approved plans, refactor code following black box principles

**Input**: Approved architecture plan from arch-planner

**Output**: Refactored code with verification

**Delegates To**: None (does the implementation work)

**Legacy Prompts**: Optional (refactor)

**When to Use Directly**: When you have an approved plan ready to implement

**Example Request**:
```
"Implement the approved plan for extracting the UserRepository module"
```

**Workflow**:
1. Plan review: Understands design and build order
2. Incremental refactoring: ONE module at a time
   - Create interface
   - Implement behind interface
   - Write/update tests
   - Integrate
   - Verify & commit
3. Verification: Checks black box compliance
4. Documentation: Reports progress and changes

**Output Format**:
- Implementation summary
- Modules implemented (with locations)
- Refactoring details
- Testing results
- Verification checklist
- Commits made
- Next steps

---

### arch-debugger

**Role**: Debug specialist

**Responsibility**: Debug modular systems, isolate bugs, maintain boundaries

**Input**: Bug report with symptoms and affected modules

**Output**: Root cause analysis, proposed fix, verification plan

**Delegates To**: May suggest arch-implementer for complex fixes

**Legacy Prompts**: Optional (debug)

**When to Use Directly**: When debugging issues in modular systems

**Example Request**:
```
"The UserAuthenticator module returns null when it should return an error.
Find the bug and propose a fix that maintains black box boundaries."
```

**Workflow**:
1. Bug understanding: Gathers information, classifies bug
2. Module boundary isolation: 5-step black box isolation
   - Identify boundary
   - Verify input contract
   - Check output contract
   - Verify assumptions
   - Isolate dependency chain
3. Root cause analysis: Investigates module internals
4. Fix proposal: Designs options maintaining architecture

**Output Format**:
- Bug analysis (expected vs actual)
- Module boundary identification
- Isolation results
- Root cause (with violation type)
- Fix options (with pros/cons)
- Recommendation
- Verification plan

## Workflows

### Workflow 1: Analysis Only

**Goal**: Understand current architecture without making changes

**Agents**: arch-analyzer

**Steps**:
1. User requests analysis
2. Analyzer explores codebase
3. Returns detailed report

**Example**:
```
"Analyze the codebase and identify architectural violations"
```

**Output**: Analysis report with recommendations

---

### Workflow 2: Planning Only

**Goal**: Design architecture for new feature or system

**Agents**: arch-planner

**Steps**:
1. User describes what to build
2. Planner asks clarifying questions
3. Designs architecture
4. Returns plan with roadmap

**Example**:
```
"Design the architecture for adding multi-tenancy support"
```

**Output**: Architecture proposal with phased roadmap

---

### Workflow 3: Full Refactoring

**Goal**: Complete transformation from analysis to implementation

**Agents**: arch-analyzer → arch-planner → arch-implementer

**Steps**:
1. Orchestrator receives request
2. Delegates to analyzer (explores code)
3. Delegates to planner (designs improvements)
4. **USER APPROVAL CHECKPOINT**
5. Delegates to implementer (refactors code)
6. Returns complete report

**Example**:
```
"Refactor the payment processing module to follow black box principles"
```

**Output**: Refactored code with analysis and design documentation

**Critical**: User must approve plan before implementation!

---

### Workflow 4: Debug & Fix

**Goal**: Find and fix bugs while maintaining architecture

**Agents**: arch-debugger → arch-implementer (optional)

**Steps**:
1. User reports bug
2. Debugger isolates to specific module
3. Identifies root cause
4. Proposes fix options
5. If approved, implementer applies fix

**Example**:
```
"The cart checkout fails when applying coupons. Debug and fix."
```

**Output**: Bug fix with root cause analysis

---

### Workflow 5: Complete Transformation

**Goal**: Major architectural overhaul with validation

**Agents**: arch-analyzer → arch-planner → arch-implementer → arch-debugger

**Steps**:
1. Analyzer examines current state
2. Planner designs target architecture
3. **USER APPROVAL**
4. Implementer refactors code
5. Debugger validates no regressions
6. Returns comprehensive report

**Example**:
```
"Transform this monolith into a modular black box architecture"
```

**Output**: Completely refactored system with validation

## Best Practices

### 1. Start Small

Don't try to refactor entire codebase at once:

```
❌ "Refactor the entire application"
✅ "Refactor the user authentication module"
```

### 2. Use Checkpoints

Always review plans before implementation:

```
Workflow: Analyzer → Planner → [YOU REVIEW] → Implementer
```

### 3. Trust the Process

Agents follow structured protocols - let them complete phases:

```
✅ Let analyzer finish all 4 phases
❌ Don't interrupt mid-analysis
```

### 4. Provide Context

Help agents by providing scope:

```
❌ "Analyze the code"
✅ "Analyze src/services/*.ts for coupling issues"
```

### 5. One Module at a Time

For implementation, focus incrementally:

```
✅ "Implement the UserRepository module from the approved plan"
❌ "Implement all 15 modules at once"
```

### 6. Leverage Legacy Prompts (Optional)

Legacy prompts can be used directly if you need them:

```
Use refactor.md / plan.md / debug.md directly if you prefer
```

## Troubleshooting

### Agent Not Found

**Problem**: "Agent arch-analyzer not found"

**Solution**:
```bash
# Verify installation
ls .claude/agents/

# Re-copy if needed
cp -r black-box-architecture/agents .claude/agents/
```

### Unexpected Behavior

**Problem**: Agent doing something unexpected

**Solution**:
- Read the agent's markdown file to understand its protocol
- Provide clearer scope in your request
- Use a more specific workflow

### Context Overflow

**Problem**: Agent running out of context

**Solution**:
- Narrow the scope (fewer files/modules)
- Use analyzer with specific directory
- Break task into smaller pieces

### Need More Control

**Problem**: Want to control each step

**Solution**: Use legacy commands instead of agents
```bash
/arch          # More control than orchestrator
/arch-plan     # More control than planner
/arch-debug    # More control than debugger
```

## Advanced Usage

### Calling Agents Programmatically

Agents can be invoked by the orchestrator or directly:

**Direct invocation**:
```
[Request specific agent by name with clear task]
```

**Through orchestrator**:
```
[Complex request that needs multiple agents - orchestrator decides workflow]
```

### Custom Workflows

Create your own agent coordination:

1. Call analyzer for specific module
2. Review analysis
3. Call planner with analysis results
4. Review plan
5. Call implementer with approved plan
6. Call debugger to validate

### Integration with CI/CD

Agents can be part of automated workflows:

```bash
# Pre-commit hook: analyze changes
# CI: validate architecture
# CD: verify no violations introduced
```

## FAQ

**Q: Can I use agents without skills?**
A: Yes. Agents do not require skills; skills are legacy prompts.

**Q: Do agents modify code automatically?**
A: Only arch-implementer modifies code, and only after explicit approval.

**Q: Can agents work on any language?**
A: Yes - Python, TypeScript, Go, Rust, C, PHP, Java, and more.

**Q: How do I know which agent to use?**
A: Use arch-orchestrator for complex tasks - it will delegate appropriately.

**Q: Can I modify agent behavior?**
A: Yes - edit the agent .md files in .claude/agents/

**Q: Do agents require internet?**
A: No - they work offline with your local codebase.

## Next Steps

1. **Try Analysis**: Start with arch-analyzer on a small module
2. **Review Output**: See how agents structure responses
3. **Progress to Planning**: Use arch-planner for new features
4. **Full Workflow**: Try complete refactoring with orchestrator
5. **Customize**: Modify agents for your team's needs

## Resources

- [Agent Specifications](../agents/agent.json) - Technical details
- [Individual Agent Files](../agents/) - Full agent implementations
- [Skills Documentation](../docs/legacy/skills/) - Legacy knowledge modules (optional)
- [Examples](../examples/) - Real-world refactorings

---

**The agent system embodies black box architecture - each agent is replaceable, focused, and coordinates through clear interfaces.**
