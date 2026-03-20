# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

Black Box Architecture Agents — a system of five coordinated AI agents that help developers maintain and improve software architecture following black-box design principles (Eskil Steenberg's philosophy). This is a **documentation-only project** (no executables, no build system).

## Architecture

```
User Request → arch-orchestrator (Opus, entry point)
                ├→ arch-analyzer   (Sonnet, read-only, maps boundaries/violations)
                ├→ arch-planner    (Opus, read-only, designs modules/roadmaps)
                ├→ arch-implementer (Sonnet, write access, executes plans)
                └→ arch-debugger   (Sonnet, write access, isolates/fixes bugs)
```

All agents are governed by `agents/BLACKBOX_CONTRACT.md` (13 global rules). The orchestrator classifies requests and delegates via HANDOFF packets.

**Workflows**: Analysis only | Planning only | Full refactoring (analyze → plan → approve → implement) | Debug & fix | Complete transformation (all agents)

## Key Directories

- `agents/` — Agent specs + `agent.json` registry + `BLACKBOX_CONTRACT.md` shared rules
- `docs/` — AGENTS (system guide), PRINCIPLES (philosophy), EXAMPLES (code), CONTRIBUTING
- `examples/` — Multi-language before/after examples (Python, TypeScript, Go, Rust, PHP, C)
- `tasks/` — Persistent session state: `lessons.md` (self-improvement) + `todo.md` (tracking)

## Critical Conventions

- **Scope discipline**: Touch only what's required. List nearby issues as POTENTIAL FOLLOW-UPS.
- **Approval gates (⛔)**: Hard stops before adding deps, changing APIs/schemas, deleting code, widening permissions.
- **Evidence minimums**: ≥3 file:line references per architectural claim.
- **Commit discipline**: `git add -u` (never `-A`), checkpoint before non-trivial work, atomic commits per changeset.
- **Self-improvement loop**: After corrections, update `tasks/lessons.md` with mistake → rule → agent → date.
- **Session start**: Agents read `tasks/lessons.md` first to apply past learnings.

## Installation (into target projects)

```bash
mkdir -p .claude/agents
cp -r agents/* .claude/agents/
```

## No Build/Test/Lint

Pure documentation project. No commands to run. Validate by reading agent specs and ensuring contract compliance.
