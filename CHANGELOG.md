# CHANGELOG — Black Box Architecture Agents

## v2.0.2 — 2026-03-20

### Contract fixes and repo cleanup

| What | Why |
|---|---|
| Fixed implementer claiming "only agent that modifies code" | Debugger also has write access — claim was false |
| Clarified debugger autonomous mode scope | Approval gates (Rule 5) still apply during autonomous fixes — tension was unresolved |
| Added `Agent` field to lessons.md format (Rule 10) | Needed for filtering lessons by agent at session start |
| Bumped agent.json version to 2.0.2 | Was stuck at 2.0.0 despite v2.0.1 changes |
| Deleted `enhanced/` directory | Orphaned pre-agent prompts, no agent referenced them |
| Deleted `docs/legacy/` directory | Commands/skills marked "not recommended" — removed rather than maintaining dead code |
| Consolidated docs: removed WORKFLOWS.md, USAGE.md, INSTALLATION.md, INTEGRATION_EXAMPLES.md | Same 5 workflows described in 4+ files. AGENTS.md is the single source now |
| Restored README.md | Entry point for GitHub visitors was missing |
| Stripped legacy references from AGENTS.md, CONTRIBUTING.md | Removed references to deleted files/directories |

---

## v2.0.1 — 2026-02-25

### Post-review fixes

| What | Why |
|---|---|
| `git add -A` → `git add -u` everywhere | `-A` stages secrets, build artifacts, untracked junk — contradicted Rule 6 (Security) |
| Elegance check → Simplicity check | "Is there a more elegant way?" amplifies LLM over-engineering bias. Flipped to "is this the simplest solution that works?" |
| Stripped fake seed lessons from `tasks/lessons.md` | Pre-filled lessons disguised as real corrections were misleading. Template-only now — real lessons accumulate naturally |
| Added `SUCCESS CRITERIA MET WHEN` to response skeleton | Borrowed from GSD goal-backward verification. HOW TO VERIFY says what to run; this says what done looks like |
| Added atomic commits after each changeset (implementer) | Borrowed from GSD. Each task gets its own revertable commit — enables `git bisect` |
| Added deterministic quality gate verdict (PASS/CONCERNS/FAIL/BLOCKED) | Borrowed from BMAD. Self-assessed checkboxes are silent; one-word verdict makes the decision explicit |

---

## v2.0.0 — 2026-02-25

### What changed and why

---

### 🆕 New Files

**`tasks/lessons.md`** — Self-improvement loop
- Agents review this at session start
- Updated after every user correction
- Prevents repeating the same mistakes across sessions
- Inspired by the "Self-Improvement Loop" pattern from production agent workflows

**`tasks/todo.md`** — Task tracking
- Persistent progress across sessions
- Structured format: Plan → Track → Review
- Enables resuming work after interruption

---

### 📝 BLACKBOX_CONTRACT.md

| What changed | Why |
|---|---|
| Added Rule 9: Commit Before Non-Trivial Changes | Rollback safety — agents could make sweeping changes with no checkpoint |
| Added Rule 10: Self-Improvement Loop | Agents were repeating same mistakes across sessions |
| Added Rule 13: Quality Gates checklist | Formalizes the "would a staff engineer approve this?" check |
| Hardened Rule 5: Approval Gates with ⛔ symbol | Makes the hard stop visually explicit — easy to miss in wall of text |
| Added `tasks/todo.md` format spec | Prevents format drift across sessions |
| Added `LESSONS RELEVANT` to HANDOFF packet | Sub-agents now inherit session learnings from orchestrator |

---

### 📝 arch-orchestrator.md

| What changed | Why |
|---|---|
| Added Session Start Checklist | Ensures lessons.md and todo.md are loaded before work begins |
| Added plan mode trigger rule | Was implicit — now explicit: 5+ steps = plan first |
| Added ⛔ Approval Gates section | Consolidates all hard stops in one visible place |
| Added `tasks/todo.md` write instruction | Formalizes task tracking responsibility |

---

### 📝 arch-analyzer.md

| What changed | Why |
|---|---|
| Added Session Start (review lessons.md) | Applies past learnings to current analysis |
| Made ≥3 file:line minimum explicit in protocol | Was in contract but not in agent's own micro-protocol |
| Added POTENTIAL FOLLOW-UPS section to output | Explicit place for "nearby issues I noticed but won't touch" |

---

### 📝 arch-planner.md

| What changed | Why |
|---|---|
| Added Session Start | Consistency with other agents |
| Added Elegance Check | Planners were sometimes proposing hacky shortcuts under time pressure |
| Added `tasks/todo.md` write instruction | Plan output should be persisted, not just printed |
| Added commit checkpoint to roadmap phases | Safety net per phase, not just per session |

---

### 📝 arch-implementer.md

| What changed | Why |
|---|---|
| Added Session Start with lessons.md review | Most critical agent to have learnings — it's the one touching code |
| Added commit checkpoint as mandatory step 3 | Was in contract but not in agent's own protocol — easy to skip |
| Added Elegance Check | Implementers were shipping hacky fixes without pausing |
| Added ⛔ Approval Gates section | Hard stops visually consolidated |
| Added `tasks/lessons.md` update after corrections | Closes the self-improvement loop at the implementation level |

---

### 📝 arch-debugger.md

| What changed | Why |
|---|---|
| Added Session Start with lessons check for similar past bugs | Most impactful: past bugs are the best predictor of current bugs |
| Added Autonomous Mode section | Was implicit in the original — made explicit: just fix it, don't ask for hand-holding |
| Added LESSON CAPTURED to output skeleton | Formalizes lesson extraction as a required output, not optional |
| Added commit checkpoint before fix | Safety net for non-trivial fixes |

---

### 📝 agent.json

| What changed | Why |
|---|---|
| Version bumped to 2.0.0 | Breaking change: new required files (tasks/) |
| Added `changes_v2` field | Documents what's new for consumers of the pack |
| Added `autonomous_mode: true` to debugger | Documents the behavioral change |
| Added `writes_to` fields | Makes file ownership explicit per agent |
| Added `files` section | Central registry of shared files |

---

### What did NOT change

- Black-box principles (unchanged — they're correct)
- Core response skeleton (ASSUMPTIONS / PLAN / RESULT / VERIFY / NEXT)
- Sub-agent delegation policy (HANDOFF packet structure preserved, one field added)
- Verbosity defaults (5-12 lines)
- Agent model assignments (opus for orchestrator/planner, sonnet for analyzer/implementer/debugger)
- Scope discipline rules (untouched — already strong)
- Confusion management rules (untouched)
