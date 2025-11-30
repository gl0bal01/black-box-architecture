# Agent Workflows - Step-by-Step Examples

Practical examples showing how to use the agent system for common architectural tasks.

## Table of Contents

- [Workflow 1: Code Analysis](#workflow-1-code-analysis)
- [Workflow 2: Architecture Planning](#workflow-2-architecture-planning)
- [Workflow 3: Complete Refactoring](#workflow-3-complete-refactoring)
- [Workflow 4: Bug Debugging](#workflow-4-bug-debugging)
- [Workflow 5: Major Transformation](#workflow-5-major-transformation)

---

## Workflow 1: Code Analysis

**Goal**: Understand current architecture and identify issues

**Agents Used**: `arch-analyzer`

**Best For**: Initial exploration, identifying technical debt, planning improvements

### Step-by-Step

#### 1. Invoke the Analyzer

**Request**:
```
Use arch-analyzer to analyze the src/services directory and identify
architectural violations and coupling issues.
```

#### 2. Analyzer Executes (Autonomous)

**Phase 1 - Discovery (25%)**:
- Maps file structure using Glob
- Finds primitives (classes, interfaces, structs)
- Reads critical files
- **Outputs progress summary**

**Phase 2 - Dependency Mapping (25%)**:
- Analyzes imports/dependencies
- Builds dependency graph
- Identifies circular dependencies
- **Outputs dependency map**

**Phase 3 - Violation Detection (30%)**:
- Finds leaky abstractions
- Identifies god objects
- Detects tight coupling
- Finds unwrapped external dependencies
- **Outputs violations with file:line references**

**Phase 4 - Opportunities (20%)**:
- Suggests module extractions
- Identifies wrapper needs
- Proposes simplifications
- **Outputs prioritized recommendations**

#### 3. Review Analysis Report

You receive:

```markdown
# Architecture Analysis: Services Directory

## Executive Summary
Found 12 violations across 8 files. Main issues:
- UserService is a god object (450 LOC, 15 methods)
- Direct Stripe API usage in 5 files
- Circular dependency: OrderService ↔ PaymentService

## Violations

### ❌ God Object: UserService
**Location**: `src/services/user_service.ts:1`
**Size**: 450 lines, 15 public methods
**Responsibilities**: Authentication, profile management, notifications, billing
**Impact**: Hard to maintain, test, or replace

### ❌ Unwrapped External Dependency: Stripe
**Location**: `src/services/payment_service.ts:45`
**Used In**: 5 files
**Impact**: Tightly coupled to Stripe, can't swap payment providers

## Recommendations

### Priority 1: Extract UserAuthenticator Module
**Effort**: Medium
**Benefit**: High
**Impact**: Enables independent testing and development
```

#### 4. Next Actions

Based on analysis, you can:
- Request planning: "Based on this analysis, design an improved architecture"
- Focus on specific issue: "Analyze just the UserService god object in detail"
- Proceed to refactoring: "Use the full refactoring workflow"

### Expected Duration
- Small codebase (<50 files): 2-3 minutes
- Medium codebase (50-500 files): 5-10 minutes
- Large codebase (>500 files): Request narrower scope

---

## Workflow 2: Architecture Planning

**Goal**: Design architecture for new feature or refactoring

**Agents Used**: `arch-planner`

**Best For**: New features, system design, strategic planning

### Step-by-Step

#### 1. Invoke the Planner

**From Scratch**:
```
Use arch-planner to design the architecture for adding a real-time
notification system. Requirements: support email, SMS, and push
notifications, handle 10K notifications/minute, allow pluggable providers.
```

**From Analysis**:
```
Use arch-planner to design improvements based on the UserService
analysis. Focus on extracting authentication into a separate module.
```

#### 2. Planner Executes (Autonomous)

**Phase 1 - Problem Understanding (20%)**:
- Asks clarifying questions if needed
- Extracts requirements and constraints
- Identifies assumptions
- **Requests confirmation**

**Phase 2 - Primitive Discovery (25%)**:
- Identifies core data types
- Defines operations on primitives
- Validates primitives are fundamental
- **Outputs primitive definitions**

**Phase 3 - Architecture Design (35%)**:
- Draws module boundaries
- Designs interfaces (hiding implementation)
- Maps dependency structure (layers)
- **Outputs module architecture**

**Phase 4 - Roadmap & Risks (20%)**:
- Creates phased implementation plan
- Assesses risks with mitigation
- Defines success metrics
- **Outputs roadmap and risk assessment**

#### 3. Review Architecture Proposal

You receive:

```markdown
# Architecture Proposal: Notification System

## Executive Summary
Modular notification system with pluggable providers, supporting
multiple channels (email, SMS, push). Clean abstraction allows
swapping providers without changing business logic.

## System Primitives

### Notification
**Definition**: A message to be delivered to a user through a channel
**Properties**: recipient, content, channel, priority, metadata
**Operations**: send(), retry(), cancel(), get_status()
**Why Primitive**: Cannot be derived; fundamental unit

## Module Architecture

### NotificationDispatcher
**Responsibility**: Route notifications to appropriate channel provider
**Interface**:
\`\`\`typescript
interface NotificationDispatcher {
  dispatch(notification: Notification): Promise<DispatchResult>;
  get_status(notification_id: string): Promise<Status>;
}
\`\`\`
**Dependencies**: ChannelProvider (interface)
**Replaceable**: Yes - can reimplement routing logic independently

### ChannelProvider (Interface)
**Responsibility**: Send notification through specific channel
**Interface**:
\`\`\`typescript
interface ChannelProvider {
  send(notification: Notification): Promise<SendResult>;
  supports(channel: Channel): boolean;
}
\`\`\`
**Implementations**: EmailProvider, SMSProvider, PushProvider

### EmailProvider
**Responsibility**: Send email notifications
**Dependencies**: SMTP wrapper (platform layer)
**Replaceable**: Can swap with SendGrid, SES, etc.

## Implementation Roadmap

### Phase 1: Foundation (Week 1)
**Goal**: Core interfaces and one provider working
**Modules**:
1. Notification primitive
2. ChannelProvider interface
3. NotificationDispatcher
4. EmailProvider (using SMTP wrapper)

**Success Criteria**:
- [ ] Can send email notification
- [ ] All tests pass
- [ ] Interface documented

### Phase 2: Multiple Channels (Week 2)
**Goal**: Add SMS and Push providers
**Modules**:
1. SMSProvider (Twilio wrapper)
2. PushProvider (FCM wrapper)

**Success Criteria**:
- [ ] All three channels working
- [ ] Providers are swappable
- [ ] No shared state between providers

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Provider API changes | M | H | Wrap all providers, isolate changes |
| Rate limiting | H | M | Implement queue with backpressure |
| Channel failures | H | H | Retry logic, fallback channels |
```

#### 4. Get Approval & Next Steps

**Approve and proceed**:
```
This design looks good. Proceed with implementation of Phase 1.
```

**Request changes**:
```
Can you add support for notification templates and a/b testing?
```

**Just planning**:
```
Thanks, I'll review this with the team before implementing.
```

### Expected Duration
- Simple feature: 3-5 minutes
- Complex system: 10-15 minutes
- May include clarifying questions

---

## Workflow 3: Complete Refactoring

**Goal**: Transform existing code to follow black box principles

**Agents Used**: `arch-orchestrator` → `arch-analyzer` → `arch-planner` → `arch-implementer`

**Best For**: Improving existing code, breaking up monoliths, reducing coupling

### Step-by-Step

#### 1. Invoke the Orchestrator

**Request**:
```
Use the orchestrator to refactor the payment processing system.
Analyze the current implementation, design black box modules,
and implement the improvements.
```

#### 2. Orchestrator Coordinates (Phases)

**Orchestrator Decision**:
```
Request requires: analysis → design → implementation
Workflow: arch-analyzer → arch-planner → [APPROVAL] → arch-implementer
```

**Phase 1: Analysis** (Delegated to arch-analyzer)
```
Orchestrator → arch-analyzer:
"Analyze src/payments/ directory focusing on module boundaries
and coupling issues"
```

Analyzer explores and returns analysis.

**Phase 2: Design** (Delegated to arch-planner)
```
Orchestrator → arch-planner:
"Based on the analysis showing tight coupling to Stripe and
god object PaymentService, design a modular architecture"
```

Planner designs architecture and returns plan.

**Phase 3: User Approval Checkpoint**
```
Orchestrator → User:
"Review this architectural plan before implementation:
[Shows complete plan]

Approve to proceed with implementation?"
```

**Phase 4: Implementation** (After approval, delegated to arch-implementer)
```
Orchestrator → arch-implementer:
"Implement Phase 1 of the approved plan:
1. PaymentProvider interface
2. PaymentProcessor module
3. StripeProvider wrapper"
```

Implementer refactors code incrementally.

#### 3. Receive Complete Report

You receive:

```markdown
# Refactoring Complete: Payment Processing System

## Summary
Successfully transformed payment processing from monolithic
PaymentService (500 LOC) into 4 modular black boxes. Stripe
dependency now wrapped, system is testable and provider-agnostic.

## Work Completed

### Analysis Phase (arch-analyzer)
- Identified god object (PaymentService: 500 LOC, 20 methods)
- Found direct Stripe usage in 8 files
- Detected missing abstractions for payment providers

### Design Phase (arch-planner)
- Designed PaymentProvider interface
- Created PaymentProcessor module
- Planned provider wrappers (Stripe, PayPal ready)

### Implementation Phase (arch-implementer)

**Module 1: PaymentProvider Interface**
- Location: `src/payments/provider.interface.ts:1-25`
- Defines clean API for payment providers
- ✅ Tests: 5/5 passing

**Module 2: PaymentProcessor**
- Location: `src/payments/processor.ts:1-120`
- Orchestrates payment flow
- ✅ Tests: 12/12 passing
- ✅ No implementation details leaked

**Module 3: StripeProvider**
- Location: `src/payments/providers/stripe.ts:1-80`
- Wraps Stripe API behind interface
- ✅ Tests: 8/8 passing (with mock Stripe)
- ✅ Stripe now isolated to this file only

## Black Box Compliance

- [x] Interfaces clear and minimal
- [x] Implementations hidden
- [x] Single responsibilities
- [x] External deps wrapped (Stripe)
- [x] No circular dependencies
- [x] One-person ownership possible

## Metrics Improvement

**Before**:
- Files: 1 (monolithic)
- Lines: 500
- Public methods: 20
- Direct Stripe usage: 8 files
- Testability: Low (requires real Stripe)

**After**:
- Files: 4 (modular)
- Lines: 300 (less duplication)
- Public methods: 5 (provider interface)
- Direct Stripe usage: 1 file (wrapper)
- Testability: High (mockable interfaces)
```

#### 4. Verify Changes

```bash
# Run tests
npm test

# Review commits
git log --oneline

# See what changed
git diff HEAD~4
```

### Expected Duration
- Small refactoring (1-2 modules): 10-15 minutes
- Medium refactoring (3-5 modules): 20-30 minutes
- Large refactoring: Break into multiple sessions

### Critical Checkpoints

1. **After Analysis**: Review findings, adjust scope if needed
2. **After Planning**: **MUST APPROVE** before implementation starts
3. **During Implementation**: Can pause between modules
4. **After Implementation**: Verify tests pass, review changes

---

## Workflow 4: Bug Debugging

**Goal**: Find and fix bugs while maintaining modular boundaries

**Agents Used**: `arch-debugger` → (optionally) `arch-implementer`

**Best For**: Debugging in modular systems, maintaining architecture during fixes

### Step-by-Step

#### 1. Invoke the Debugger

**Request**:
```
Use arch-debugger to debug this issue:

When a user applies a discount coupon, the cart total becomes negative.
Expected: Total should never go below zero.
Affected module: ShoppingCart

Steps to reproduce:
1. Add item ($10)
2. Apply 50% coupon
3. Apply another 50% coupon
4. Total shows -$5
```

#### 2. Debugger Executes (Autonomous)

**Phase 1 - Bug Understanding (20%)**:
- Classifies bug type
- Identifies affected modules
- Confirms reproduction steps
- **Outputs bug classification**

**Phase 2 - Module Boundary Isolation (30%)**:
```
5-Step Black Box Isolation:

1. Identify Module Boundary
   Module: PricingCalculator
   Responsibility: Calculate totals with discounts

2. Verify Interface Contract
   Input: items[], coupons[]
   Expected: total >= 0
   Actual: total can be negative
   Contract violation: YES

3. Check Output Contract
   Should return non-negative total
   Currently returns negative
   Violation confirmed

4. Verify Assumptions
   Assumption: Coupons applied sequentially
   Reality: Both applied to same base
   Issue found

5. Isolate Dependency Chain
   Works in isolation: NO
   Problem in this module
```

**Phase 3 - Root Cause Analysis (30%)**:
```
Root Cause Found:

Location: src/pricing/calculator.ts:45-52
Issue: Each coupon applies to original price, not cumulative
Code:
  coupons.forEach(coupon => {
    discount += basePrice * coupon.percent  // ❌ Always basePrice
  })

Why Bad: Two 50% coupons = 100% discount, total becomes negative
Violates: Business logic constraint (total >= 0)
```

**Phase 4 - Fix Proposal (20%)**:
```
Option 1: Sequential Application
Files: src/pricing/calculator.ts:45-52
Code:
  let currentPrice = basePrice
  coupons.forEach(coupon => {
    currentPrice = currentPrice * (1 - coupon.percent)
  })

Pros: Correct math, prevents negative
Cons: Changes discount calculation behavior
Effort: Small

Option 2: Cap Total at Zero
Files: src/pricing/calculator.ts:58
Code:
  return Math.max(0, total)

Pros: Quick fix, prevents negative
Cons: Doesn't fix root cause
Effort: Trivial

RECOMMENDATION: Option 1
Reasoning: Fixes root cause, proper coupon stacking logic
```

#### 3. Review and Approve Fix

**Approve**:
```
Option 1 looks correct. Implement it.
```

**Request Different Approach**:
```
Actually, we want coupons to be mutually exclusive.
Only one coupon can be applied at a time.
```

#### 4. Implementation (If Approved)

Debugger can implement simple fixes, or delegate to arch-implementer:

```
Implementing Option 1...

✅ Updated: src/pricing/calculator.ts:45-52
✅ Added test: discount_stacking_prevents_negative.test.ts
✅ All tests passing (15/15)
✅ Verified: Total is always >= 0

Commits:
- feat: fix coupon stacking to prevent negative totals
```

### Expected Duration
- Simple bugs: 3-5 minutes
- Complex bugs: 10-15 minutes
- May need clarification on expected behavior

---

## Workflow 5: Major Transformation

**Goal**: Complete architectural overhaul with validation

**Agents Used**: All agents orchestrated together

**Best For**: Legacy system modernization, major refactoring, architecture migration

### Step-by-Step

#### 1. Invoke Full Transformation

**Request**:
```
Use all agents to transform the legacy monolithic application
into a modular black box architecture. The current system is
one large file (app.py, 2000 lines) handling authentication,
payments, notifications, and data access.
```

#### 2. Orchestrator Plans Full Workflow

```
Orchestrator Analysis:
Task: Complete transformation
Complexity: High
Workflow: analyzer → planner → [APPROVAL] → implementer → debugger

Estimated phases:
1. Analysis (10 min)
2. Planning (15 min)
3. USER APPROVAL CHECKPOINT
4. Implementation Phase 1 (20 min)
5. Implementation Phase 2 (20 min)
6. Validation & Debug (10 min)
```

#### 3. Phase-by-Phase Execution

**Phase 1: Deep Analysis**
```
arch-analyzer:
"Analyzing app.py (2000 lines)...

Found:
- 15 responsibilities in one file
- No module boundaries
- Direct database queries (50+ locations)
- Direct Stripe API usage (12 locations)
- SMTP code scattered throughout
- No testable interfaces

Recommended extraction:
1. UserAuthenticator (150 LOC)
2. PaymentProcessor (200 LOC)
3. NotificationSender (100 LOC)
4. UserRepository (80 LOC)
5. Core app logic (remaining)
```

**Phase 2: Strategic Design**
```
arch-planner:
"Designing modular architecture...

Module Architecture:
1. Platform Layer:
   - DatabaseWrapper (wraps SQLAlchemy)
   - StripeWrapper (wraps Stripe)
   - SMTPWrapper (wraps smtplib)

2. Data Layer:
   - UserRepository
   - PaymentRepository
   - NotificationRepository

3. Business Logic:
   - UserAuthenticator
   - PaymentProcessor
   - NotificationSender

4. Application Layer:
   - app.py (orchestrates above modules)

Roadmap:
Phase 1: Platform wrappers + repositories
Phase 2: Business logic modules
Phase 3: Slim down app.py to orchestration
```

**Phase 3: USER APPROVAL**
```
Orchestrator:
"Architecture plan complete. Review before proceeding:
[Shows complete plan with 15 modules, phased approach]

This is a major transformation (2000 LOC → 15 modules).
Approve to proceed with Phase 1 implementation?"

[Wait for user approval]
```

**Phase 4: Incremental Implementation**
```
arch-implementer:
"Implementing Phase 1: Platform Layer...

Module 1: DatabaseWrapper
✅ Created: src/platform/database.ts:1-60
✅ Tests: 8/8 passing
✅ Commit: feat: add database wrapper

Module 2: StripeWrapper
✅ Created: src/platform/stripe.ts:1-80
✅ Tests: 10/10 passing
✅ Commit: feat: add Stripe wrapper

Module 3: SMTPWrapper
✅ Created: src/platform/smtp.ts:1-45
✅ Tests: 5/5 passing
✅ Commit: feat: add SMTP wrapper

Phase 1 Complete. Proceed to Phase 2?"

[Continues through all phases]
```

**Phase 5: Validation**
```
arch-debugger:
"Validating refactored system...

✅ All module boundaries clean
✅ No circular dependencies
✅ All tests passing (85/85)
✅ No regressions in functionality
✅ External deps properly wrapped
✅ Black box principles maintained

System health: EXCELLENT
```

#### 4. Final Report

```markdown
# Transformation Complete: Monolith → Modular Architecture

## Before
- Files: 1 (app.py)
- Lines: 2000
- Modules: 0 (everything mixed)
- Testability: None (no mocks possible)
- Maintainability: Very low

## After
- Files: 18 (organized by layer)
- Lines: 1800 (removed duplication)
- Modules: 15 (clear responsibilities)
- Testability: High (all interfaces mockable)
- Maintainability: High

## Modules Created
[Complete list with descriptions]

## All Tests Passing
- Unit: 60/60
- Integration: 20/20
- E2E: 5/5

## Ready for Production
✅ All functionality preserved
✅ Performance unchanged
✅ Team can now work in parallel on modules
```

### Expected Duration
- Large transformation: 1-2 hours
- Expect multiple approval checkpoints
- Can pause between phases

---

## Tips for All Workflows

### 1. Provide Clear Scope
```
❌ "Refactor the code"
✅ "Refactor src/services/payment.ts to extract provider interface"
```

### 2. Use Checkpoints
Always review plans before implementation starts

### 3. Start Small
Begin with one module, expand as confidence grows

### 4. Trust the Process
Agents follow structured protocols - let them complete phases

### 5. Iterate
Analysis → Plan → Implement → Validate → Repeat

---

## Next Steps

1. **Try Analysis First**: Get comfortable with arch-analyzer
2. **Practice Planning**: Design a small new feature
3. **Small Refactoring**: Refactor one module end-to-end
4. **Build Confidence**: Gradually tackle larger transformations

See [AGENTS.md](AGENTS.md) for detailed agent documentation.
