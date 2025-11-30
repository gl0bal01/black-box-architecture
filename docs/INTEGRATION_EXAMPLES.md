# Agent Integration Examples

Real-world examples showing how the orchestrator coordinates multiple agents to accomplish complex tasks.

## Example 1: Refactoring a God Object

### User Request
```
"The UserService class has grown to 500 lines and handles too many things.
Break it up into proper black box modules."
```

### Orchestrator Analysis
```markdown
Request Type: Refactoring
Complexity: Medium
Requires: Analysis → Design → Implementation

Workflow Decision:
arch-analyzer → arch-planner → [USER APPROVAL] → arch-implementer
```

### Agent Coordination Flow

#### Step 1: Orchestrator → Analyzer

**Delegation**:
```markdown
## Delegating to arch-analyzer

**Task**: Analyze UserService class for architectural violations

**Input**:
- File: src/services/user_service.ts
- Focus: Identify responsibilities, coupling issues, extraction opportunities

**Expected Output**:
- Current responsibilities breakdown
- Coupling analysis
- Recommended module extractions

**Context**: User reports class is too large (500 LOC) and handles too much
```

**Analyzer Response**:
```markdown
# Analysis: UserService

## Current Responsibilities
1. Authentication (login, logout, sessions)
2. Profile management (update, avatar upload)
3. Notifications (email preferences, send notifications)
4. Billing (subscription management, invoices)
5. Permissions (role checks, access control)

## Violations

### ❌ God Object
**Location**: src/services/user_service.ts:1-500
**Responsibilities**: 5 distinct concerns
**Public methods**: 18
**Impact**: Cannot assign single owner, hard to test

### ❌ Tight Coupling
Direct usage of:
- Stripe API (billing)
- SendGrid API (notifications)
- Database queries scattered throughout

## Recommended Extractions

1. **UserAuthenticator** - Authentication concern (100 LOC)
2. **ProfileManager** - Profile management (80 LOC)
3. **NotificationService** - User notifications (90 LOC)
4. **BillingManager** - Subscription handling (120 LOC)
5. **PermissionChecker** - Access control (60 LOC)
```

#### Step 2: Orchestrator → Planner

**Delegation**:
```markdown
## Delegating to arch-planner

**Task**: Design modular architecture for UserService refactoring

**Input**: Analysis showing 5 responsibilities to extract

**Expected Output**:
- Module interfaces for each extraction
- Dependency structure
- Phased implementation plan

**Context**: Need to break 500 LOC god object into 5 focused modules
```

**Planner Response**:
```markdown
# Architecture Design: UserService Modularization

## Module Interfaces

### UserAuthenticator
**Responsibility**: Handle user authentication and session management

**Interface**:
\`\`\`typescript
interface UserAuthenticator {
  login(email: string, password: string): Promise<AuthResult>;
  logout(sessionId: string): Promise<void>;
  validateSession(sessionId: string): Promise<boolean>;
}
\`\`\`

**Dependencies**: UserRepository (for user data)
**Replaceability**: Can swap with OAuth, SAML, etc.

### ProfileManager
**Responsibility**: Manage user profile data

**Interface**:
\`\`\`typescript
interface ProfileManager {
  updateProfile(userId: string, updates: ProfileData): Promise<Result>;
  uploadAvatar(userId: string, file: File): Promise<string>;
  getProfile(userId: string): Promise<UserProfile>;
}
\`\`\`

**Dependencies**: UserRepository, FileStorage (wrapped)
**Replaceability**: Can reimplement storage independently

[... additional modules ...]

## Dependency Structure

**Layer 1 (Application)**:
- Original UserService (now thin orchestrator)

**Layer 2 (Business Logic)**:
- UserAuthenticator
- ProfileManager
- NotificationService
- BillingManager
- PermissionChecker

**Layer 3 (Data Access)**:
- UserRepository

**Layer 4 (Platform)**:
- StripeWrapper
- SendGridWrapper
- FileStorageWrapper

## Implementation Roadmap

### Phase 1: Foundation (Extract UserAuthenticator)
**Effort**: 2 hours
**Why First**: Most isolated, used by all other modules
**Modules**: UserAuthenticator, UserRepository
**Success**: Can authenticate users through new interface

### Phase 2: Data Operations (Extract ProfileManager)
**Effort**: 2 hours
**Dependencies**: Uses UserRepository from Phase 1
**Modules**: ProfileManager, FileStorageWrapper
**Success**: Can manage profiles independently

### Phase 3: External Services (Extract NotificationService, BillingManager)
**Effort**: 3 hours
**Modules**: NotificationService, BillingManager, wrappers for SendGrid/Stripe
**Success**: External services properly wrapped

### Phase 4: Finalize (Extract PermissionChecker, slim UserService)
**Effort**: 2 hours
**Modules**: PermissionChecker
**Success**: UserService < 100 LOC, delegates to modules
```

#### Step 3: Orchestrator → User (Approval Checkpoint)

**Orchestrator Presents**:
```markdown
## Architecture Plan Complete

Analysis found 5 distinct responsibilities in UserService.

Proposed design creates 5 focused modules with clean interfaces.

Implementation will happen in 4 phases over ~9 hours:
1. UserAuthenticator (2h)
2. ProfileManager (2h)
3. NotificationService + BillingManager (3h)
4. PermissionChecker + slim UserService (2h)

Each phase is independently testable and reversible.

**Approve to proceed with implementation?**
- YES: Start with Phase 1
- NO: Modify plan first
- PARTIAL: Implement only specific phases
```

**User Approves**:
```
Yes, proceed with the plan. Start with Phase 1.
```

#### Step 4: Orchestrator → Implementer

**Delegation**:
```markdown
## Delegating to arch-implementer

**Task**: Implement Phase 1 of approved refactoring plan

**Input**: Approved architecture design
**Phase**: Phase 1 - Extract UserAuthenticator
**Modules to Create**:
1. UserAuthenticator interface and implementation
2. UserRepository interface and implementation

**Expected Output**:
- Working modules following black box principles
- Tests for all modules
- Integration with existing UserService
- Commit per module

**Context**: First phase of 4-phase UserService refactoring
```

**Implementer Response**:
```markdown
# Implementation Complete: Phase 1

## Modules Created

### UserAuthenticator
**Location**: src/auth/authenticator.ts:1-85
**Interface**: src/auth/authenticator.interface.ts:1-20
**Tests**: src/auth/authenticator.test.ts (12 tests, all passing)

**Black Box Compliance**:
- ✅ Interface clear (3 methods only)
- ✅ Implementation hidden
- ✅ Dependencies injected
- ✅ Fully testable with mocks

**Commit**: `feat: extract UserAuthenticator module`

### UserRepository
**Location**: src/data/user-repository.ts:1-60
**Interface**: src/data/user-repository.interface.ts:1-15
**Tests**: src/data/user-repository.test.ts (8 tests, all passing)

**Black Box Compliance**:
- ✅ Interface minimal (5 methods)
- ✅ Database details hidden
- ✅ Swappable implementations
- ✅ Tests use in-memory version

**Commit**: `feat: add UserRepository abstraction`

### Integration
**Updated**: src/services/user_service.ts
**Change**: Login/logout/session methods now delegate to UserAuthenticator
**Tests**: All existing UserService tests still passing (25/25)
**Commit**: `refactor: UserService uses UserAuthenticator`

## Verification
- ✅ All tests passing (45/45 total)
- ✅ No regressions
- ✅ UserService reduced by 100 lines
- ✅ Authentication logic isolated and testable

## Next Steps
Ready for Phase 2: Extract ProfileManager
```

#### Step 5: Orchestrator → User (Progress Report)

**Orchestrator Summarizes**:
```markdown
## Phase 1 Complete ✅

**What Was Done**:
- Extracted UserAuthenticator (85 LOC)
- Created UserRepository abstraction (60 LOC)
- Updated UserService to use new modules
- All tests passing (45/45)

**Impact**:
- UserService now 400 LOC (was 500)
- Authentication logic isolated
- Can be tested independently
- One responsibility extracted

**Next**:
- Phase 2: Extract ProfileManager (2 hours)
- Phase 3: Extract NotificationService + BillingManager (3 hours)
- Phase 4: Extract PermissionChecker (2 hours)

**Proceed with Phase 2?**
- YES: Continue automatically
- NO: Stop here, review first
- LATER: Pause, resume later
```

### Complete Agent Interaction Diagram

```
User Request
    ↓
┌────────────────────────────────────────┐
│       arch-orchestrator                │
│  "Analyzes: Needs analysis, design,    │
│   and implementation"                   │
└───────────┬────────────────────────────┘
            │
            ├──→ arch-analyzer ──→ Analysis Report
            │         ↓
            │    (Orchestrator receives)
            │         ↓
            ├──→ arch-planner ──→ Architecture Design
            │         ↓
            │    (Orchestrator receives)
            │         ↓
            ├──→ [USER APPROVAL] ──→ Approved
            │         ↓
            │    (Orchestrator proceeds)
            │         ↓
            └──→ arch-implementer ──→ Refactored Code
                      ↓
                 (Final report)
                      ↓
                  ┌────────┐
                  │  User  │
                  └────────┘
```

---

## Example 2: Debugging with Architecture Preservation

### User Request
```
"The checkout process fails when applying multiple discount coupons.
Find the bug and fix it without breaking our modular architecture."
```

### Orchestrator Analysis
```markdown
Request Type: Debugging
Complexity: Low-Medium
Requires: Debugging → (possibly) Implementation

Workflow Decision:
arch-debugger → [if fix needed] → arch-implementer
```

### Agent Coordination Flow

#### Step 1: Orchestrator → Debugger

**Delegation**:
```markdown
## Delegating to arch-debugger

**Task**: Debug checkout failure with multiple coupons

**Input**:
- Symptom: Checkout fails when applying multiple coupons
- Expected: Should handle multiple coupons correctly
- Module: Likely PricingCalculator or CheckoutService

**Expected Output**:
- Root cause identification
- Proposed fix maintaining black box boundaries

**Context**: System recently refactored to modular architecture,
must maintain clean module boundaries in fix
```

**Debugger Response**:
```markdown
# Debug Report: Multiple Coupon Failure

## Bug Analysis

**Expected**: Apply multiple coupons, get discounted total
**Actual**: Checkout throws error "Total cannot be negative"
**Location**: src/pricing/calculator.ts:52

## Module Boundary Isolation

**5-Step Black Box Analysis**:

1. **Module Identified**: PricingCalculator
   - Responsibility: Calculate order totals with discounts
   - Interface: calculate(items, coupons) → Total

2. **Input Contract**: ✅ Valid
   - Items: [{price: 100}]
   - Coupons: [{percent: 0.5}, {percent: 0.5}]

3. **Output Contract**: ❌ Violated
   - Should return non-negative total
   - Actually returns -50

4. **Assumptions**: ❌ Violated
   - Assumed: Coupons apply sequentially
   - Reality: Both apply to base price

5. **Dependencies**: ✅ Isolated
   - Works standalone
   - Bug is in this module

## Root Cause

**Location**: src/pricing/calculator.ts:45-52

**Issue**: Cumulative discount calculation error

\`\`\`typescript
// CURRENT (WRONG):
let discount = 0;
coupons.forEach(coupon => {
  discount += basePrice * coupon.percent; // Both apply to basePrice!
});
// Two 50% coupons = 100% discount = negative total
\`\`\`

**Why Bad**: Each coupon applies to original price, not cumulative
**Violation**: Business logic constraint (total >= 0)

## Fix Options

### Option 1: Sequential Application (RECOMMENDED)
\`\`\`typescript
let currentPrice = basePrice;
coupons.forEach(coupon => {
  currentPrice = currentPrice * (1 - coupon.percent);
});
// Two 50% coupons = 75% discount (correct)
\`\`\`

**Pros**:
- Mathematically correct
- Prevents negative totals
- Standard discount stacking

**Cons**: Changes discount calculation (but fixes bug)

**Black Box Impact**: ✅ Maintains boundaries
- Fix is internal to PricingCalculator
- Interface unchanged
- No other modules affected

### Option 2: Limit One Coupon
\`\`\`typescript
const coupon = coupons[0]; // Use only first
if (coupon) {
  discount = basePrice * coupon.percent;
}
\`\`\`

**Pros**: Simple, prevents issue
**Cons**: Changes feature behavior significantly

## Recommendation

**Option 1**: Sequential coupon application

**Implementation**:
1. Update PricingCalculator.calculate() method
2. Add test: multiple_coupons_sequential.test.ts
3. Verify: Total always >= 0

**No architecture changes needed** - fix is internal to module.
```

#### Step 2: Orchestrator → User (Fix Approval)

**Orchestrator Presents**:
```markdown
## Bug Found and Fix Proposed

**Root Cause**: Coupons apply to base price instead of sequentially

**Recommended Fix**: Apply coupons sequentially (standard approach)

**Impact**:
- Changes: src/pricing/calculator.ts:45-52 only
- Architecture: No changes, fix is internal
- Tests: Add new test for multiple coupons
- Breaking: May change discount amounts (but fixes bug)

**Approve fix?**
- YES: Implement Option 1 (sequential)
- NO: Propose different approach
- CLARIFY: Need more information
```

**User Approves**:
```
Yes, sequential application is correct. Implement it.
```

#### Step 3: Orchestrator → Implementer (or Debugger implements directly)

For simple fixes, debugger can implement directly:

**Debugger Implements**:
```markdown
# Fix Implemented

## Changes

**File**: src/pricing/calculator.ts
**Lines**: 45-52
**Change**: Sequential coupon application

\`\`\`diff
- let discount = 0;
- coupons.forEach(coupon => {
-   discount += basePrice * coupon.percent;
- });

+ let currentPrice = basePrice;
+ coupons.forEach(coupon => {
+   currentPrice = currentPrice * (1 - coupon.percent);
+ });
+ const discount = basePrice - currentPrice;
\`\`\`

## Tests

**Added**: src/pricing/calculator.test.ts:85-95
- Test: multiple_coupons_sequential
- Test: multiple_coupons_no_negative_total
- All tests passing (18/18)

## Verification

✅ Bug fixed (multiple coupons work)
✅ Total never negative
✅ All existing tests pass
✅ Black box boundaries maintained
✅ No other modules affected

**Commit**: `fix: apply coupons sequentially to prevent negative totals`
```

#### Step 4: Orchestrator → User (Final Report)

**Orchestrator Summarizes**:
```markdown
## Bug Fix Complete ✅

**Bug**: Multiple coupons caused negative totals
**Root Cause**: Coupons applied to base price, not sequentially
**Fix**: Sequential coupon application
**Impact**: One file changed (src/pricing/calculator.ts)

**Verification**:
- ✅ Bug no longer reproduces
- ✅ All tests passing
- ✅ Architecture boundaries maintained
- ✅ No regressions

**Commit**: fix: apply coupons sequentially to prevent negative totals
```

### Complete Agent Interaction Diagram

```
User Request
    ↓
┌────────────────────────────────────────┐
│       arch-orchestrator                │
│  "Analyzes: Debugging task"            │
└───────────┬────────────────────────────┘
            │
            └──→ arch-debugger
                      ↓
                 (Isolates bug)
                      ↓
                 (Proposes fix)
                      ↓
                 [USER APPROVAL]
                      ↓
                 (Implements fix)
                      ↓
                 (Verifies)
                      ↓
                  ┌────────┐
                  │  User  │
                  └────────┘
```

---

## Key Integration Patterns

### Pattern 1: Analysis → Design → Implementation
**Use When**: Refactoring or building new features
**Flow**: Analyzer provides input to Planner, Planner output goes to Implementer
**Checkpoints**: After analysis, after planning (mandatory)

### Pattern 2: Debug → Fix
**Use When**: Bug fixing
**Flow**: Debugger identifies issue, implements or delegates to Implementer
**Checkpoints**: After root cause, before implementation

### Pattern 3: Complete Validation
**Use When**: Major transformations
**Flow**: Analyzer → Planner → Implementer → Debugger (validates)
**Checkpoints**: After each phase

### Pattern 4: Iterative Refinement
**Use When**: Complex, multi-phase work
**Flow**: Analyzer → Planner → Implementer (Phase 1) → Review → Implementer (Phase 2)...
**Checkpoints**: After each phase completion

## Orchestrator Coordination Principles

### 1. **Smart Delegation**
Orchestrator chooses appropriate agents based on request type

### 2. **Information Flow**
Results from one agent inform the next agent's task

### 3. **User Approval Checkpoints**
Orchestrator requests approval before irreversible changes

### 4. **Context Passing**
Each agent receives necessary context from previous agents

### 5. **Result Assembly**
Orchestrator combines outputs into coherent final report

## Benefits of Agent Coordination

### Separation of Concerns
- Each agent focuses on its specialty
- No agent needs to know how others work
- Clear interfaces between agents

### Context Isolation
- Agents don't pollute each other's context
- Main conversation stays clean
- Can handle larger tasks without context overflow

### Composability
- Agents can be combined in different ways
- New workflows possible by recombining agents
- Extensible architecture

### Maintainability
- Each agent can be improved independently
- Agent behavior changes don't affect coordination
- Clear responsibilities make debugging easier

---

## Creating Custom Workflows

You can create your own agent coordination by invoking agents in sequence:

```
1. Call arch-analyzer for specific module
2. Review analysis
3. Call arch-planner with analysis results
4. Review plan
5. Call arch-implementer with approved plan
6. Call arch-debugger to validate

Or let orchestrator handle it automatically!
```

---

**The orchestrator itself follows black box principles: clean interface (user requests), hidden implementation (workflow logic), replaceable (different coordination strategies possible).**
