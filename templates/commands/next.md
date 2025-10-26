---
description: Smart workflow navigator that recommends the next command based on feature state and validates prerequisites.
---

## Overview

`/speckit.next` analyzes current feature state and automatically recommends the next command in the Spec-Driven Development workflow. It validates prerequisites, checks for blockers, and provides clear actionable guidance.

## Execution Flow

### 1. Load Current State (Parallel)

Execute in parallel:

```yaml
PARALLEL Operations:
  1. Bash: git rev-parse --abbrev-ref HEAD 2>/dev/null
  2. Bash: ls .specify/memory/features/ 2>/dev/null
  3. Read: .specify/memory/next_actions.md (if exists)
```

### 2. Determine Active Feature

```yaml
Priority Order:
  1. IF git branch starts with "feature/":
     → Extract feature dir: feature/000001-auth → 000001-auth

  2. ELSE IF $SPECIFY_FEATURE env var set:
     → Use that value

  3. ELSE IF only one feature dir in .specify/memory/features/:
     → Use that feature

  4. ELSE IF multiple features exist:
     → List all and ask user to specify

  5. ELSE (no features found):
     → Report: "No active feature. Run /speckit.specify to create one."
     → EXIT
```

### 3. Load Feature State

```yaml
Read: .specify/memory/features/[FEATURE_DIR]/state.json

IF file not found:
  → Report: "State file missing. Run /speckit.pm to initialize or recreate state."
  → EXIT

Parse JSON and extract:
  - phase: Current workflow phase
  - progress: Percentage complete
  - gates_passed: Completed gates
  - gates_failed: Failed validation gates
  - blockers: Current blockers
```

### 4. Analyze Prerequisites

Based on phase, check required files and gates:

```yaml
Phase-Specific Prerequisites:

SPECIFYING:
  Required: specs/[FEATURE]/spec.md exists
  Optional: None
  Next: clarify

CLARIFYING:
  Required:
    - specs/[FEATURE]/spec.md exists
    - spec.md has >= 1 requirement section
  Check: Count [NEEDS CLARIFICATION] markers
  Next:
    IF markers > 0: Stay in clarify (incomplete)
    ELSE: Move to plan

PLANNING:
  Required:
    - specs/[FEATURE]/spec.md exists
    - No [NEEDS CLARIFICATION] markers remain
  Check: Constitution gates in state
  Next:
    IF gates_failed contains planning gates: Fix issues
    ELSE: Move to tasks

TASKING:
  Required:
    - specs/[FEATURE]/plan.md exists
    - plan.md has implementation sections
  Next: implement

IMPLEMENTING:
  Required:
    - specs/[FEATURE]/tasks.md exists
    - tasks.md has task list
  Check: Task completion status
  Next:
    IF tasks incomplete: Continue implement
    ELSE: validate/merge

VALIDATING:
  Check: Test results
  Next:
    IF tests passing: merge/complete
    ELSE: Fix failures

COMPLETED:
  Next: New feature or celebrate
```

### 5. Generate Recommendation

**Output Format** (Symbol-Based):

```
📍 [FEATURE_NAME]
🔹 Phase: [PHASE] ([PROGRESS]%)

Prerequisites:
  ✅ [COMPLETED_ITEM]
  ❌ [MISSING_ITEM]

🎯 Next: /speckit.[COMMAND]
💡 Reason: [WHY_THIS_COMMAND]

[OPTIONAL_NOTES]
```

### 6. Recommendation Logic

```yaml
IF phase == "SPECIFYING":
  IF spec.md exists:
    → Recommend: /speckit.clarify
    → Reason: "Spec created, resolve ambiguities"
  ELSE:
    → Error: "Spec missing - state inconsistent"

ELSE IF phase == "CLARIFYING":
  Count [NEEDS CLARIFICATION] markers in spec.md
  IF markers > 0:
    → Recommend: /speckit.clarify --continue
    → Reason: "[N] clarifications still needed"
  ELSE IF clarifications_count == 0:
    → Recommend: /speckit.clarify
    → Reason: "No clarifications recorded yet"
  ELSE:
    → Recommend: /speckit.plan
    → Reason: "Clarifications complete, ready for planning"

ELSE IF phase == "PLANNING":
  IF plan.md missing:
    → Recommend: /speckit.plan
    → Reason: "Create technical implementation plan"
  ELSE IF gates_failed not empty:
    → Report blockers
    → Recommend: Fix issues then re-run /speckit.plan
  ELSE IF plan.md exists AND gates passed:
    → Recommend: /speckit.tasks
    → Reason: "Plan validated, generate task breakdown"

ELSE IF phase == "TASKING":
  IF tasks.md missing:
    → Recommend: /speckit.tasks
    → Reason: "Generate executable task list"
  ELSE:
    → Recommend: /speckit.implement
    → Reason: "Tasks defined, begin implementation"

ELSE IF phase == "IMPLEMENTING":
  → Recommend: Continue implementation or validate
  → Note: "Check task progress in tasks.md"

ELSE IF phase == "VALIDATING":
  → Recommend: Fix test failures or create PR
  → Note: "Run tests to verify functionality"

ELSE IF phase == "COMPLETED":
  → Congratulate user
  → Recommend: /speckit.specify [new feature] or review accomplishments
```

### 7. Blocker Detection

If blockers exist in state:

```yaml
🚨 Blockers Detected:

1. [BLOCKER_1]
   Fix: [SUGGESTED_FIX]

2. [BLOCKER_2]
   Fix: [SUGGESTED_FIX]

⚠️ Resolve blockers before proceeding to next phase.
```

### 8. Auto-Execute Option (Future Enhancement)

```yaml
Prompt user:
  "Ready to proceed? (yes/auto/no)"

  IF "yes":
    → Show command, wait for manual execution

  IF "auto":
    → Execute recommended command automatically
    → Show results

  IF "no":
    → Exit, user can decide later
```

## Examples

### Example 1: Mid-Clarification

```
User: /speckit.next

📍 000001-auth-system
🔹 Phase: CLARIFYING (25%)

Prerequisites:
  ✅ Spec created
  ⚠️ 3 [NEEDS CLARIFICATION] markers remain

🎯 Next: /speckit.clarify
💡 Reason: Resolve 3 remaining ambiguities before planning

Run: /speckit.clarify
```

### Example 2: Ready for Planning

```
User: /speckit.next

📍 000042-payment-gateway
🔹 Phase: CLARIFYING (30%)

Prerequisites:
  ✅ Spec created
  ✅ All clarifications resolved (5 added)
  ✅ No ambiguities remain

🎯 Next: /speckit.plan
💡 Reason: Spec is clear, ready for technical planning

Run: /speckit.plan [tech stack details]
```

### Example 3: Blocker Detected

```
User: /speckit.next

📍 000007-real-time-dashboard
🔹 Phase: PLANNING (40%)

🚨 Blockers Detected:

1. Anti-abstraction gate failed
   Fix: Review constitution Article VIII - avoid custom wrappers

2. Simplicity gate failed
   Fix: Reduce from 5 projects to ≤3 projects

⚠️ Resolve blockers before proceeding.

Fix issues, then re-run: /speckit.plan
```

### Example 4: No Active Feature

```
User: /speckit.next

⚫ No active feature detected

Create a new feature with:
/speckit.specify [feature description]

Or switch to existing feature branch:
git checkout feature/000001-auth-system
```

### Example 5: Multiple Features

```
User: /speckit.next

🔍 Multiple features found:

1. 000001-auth-system (PLANNING - 45%)
2. 000003-payment-flow (IMPLEMENTING - 75%)
3. 000007-dashboard (CLARIFYING - 20%)

Which feature? (type number or feature name):
```

## Integration with PM Agent

`/speckit.next` complements `/speckit.pm`:

- **`/speckit.pm`**: Session start, broad context restoration
- **`/speckit.next`**: Action-focused, specific next step

Use together:

```bash
# Start session
/speckit.pm
# Output: 🟢 feature/000001-auth | 📋 PLANNING | 45%

# Get next action
/speckit.next
# Output: 🎯 Next: /speckit.tasks
```

## Edge Cases

### Spec File Missing But State Says PLANNING

```yaml
⚠️ State Inconsistency Detected

State says: PLANNING (50%)
Reality: specs/000001-auth/spec.md not found

This may indicate:
- File was deleted
- Wrong feature directory
- State file corruption

Recommended actions:
1. Check if spec file was moved
2. Or recreate spec with /speckit.specify
3. Or fix state file manually
```

### Git Branch Doesn't Match State

```yaml
⚠️ Branch Mismatch

Current branch: feature/000002-payments
State file: 000001-auth-system

Did you switch branches? Update state:
1. git checkout feature/000001-auth-system
   OR
2. /speckit.pm (will detect and update)
```

## Token Economy

**Target**: 200-400 tokens per invocation

**Optimizations**:
- Symbol-based output (10x compression)
- Parallel state loading
- Skip verbose explanations
- Direct actionable recommendations

## Success Criteria

- ✅ Correct next command 100% of time
- ✅ Prerequisites accurately validated
- ✅ Blockers clearly identified with fixes
- ✅ <400 tokens per invocation
- ✅ Graceful handling of edge cases
- ✅ Clear, actionable output

---

**Implementation**: Phase 2.3
**Dependencies**: Phase 1.2 (State Management)
**Token Budget**: 200-400 tokens
**Pattern**: Fast recommendation with minimal cognitive load
