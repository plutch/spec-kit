---
description: Project Manager Agent - Session context restoration and state management with intelligent next-action recommendations.
---

## Overview

PM Agent provides **cross-session memory** and **state management** for Spec-Driven Development workflows. It automatically restores context at session start and tracks feature progress through the specification lifecycle.

## Triggers

- **Session Start (Auto-Recommended)**: Restore context from previous session
- **Manual Invocation**: `/speckit.pm` for status check
- **State Queries**: "What's my progress?", "Where did I leave off?", "What's next?"

## Session Start Protocol (Parallel-with-Reflection)

**Pattern**: Wave → Checkpoint → Wave

### Wave 1 - PARALLEL Context Load (Silent)

Execute these reads in parallel:

```yaml
PARALLEL Operations:
  1. Bash: git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "no-git"
  2. Bash: git status --short | wc -l
  3. Read: .specify/memory/pm_context.md (if exists)
  4. Read: .specify/memory/last_session.md (if exists)
  5. Read: .specify/memory/next_actions.md (if exists)
  6. Bash: find specs/ -name "spec-metadata.json" -type f 2>/dev/null | head -5
  7. Bash: find .specify/memory/features -name "state.json" -type f 2>/dev/null | head -5
```

**Note (v2.3 State Management)**:
- **Primary**: spec-metadata.json (v2.3.0 standard, located in specs/{FEATURE}/)
- **Fallback**: state.json (v2.1 compatibility, located in .specify/memory/features/{FEATURE}/)
- **Phase Mapping**: spec-metadata.json uses lowercase (specification, planning, implementation) vs state.json uses uppercase (SPECIFYING, PLANNING, IMPLEMENTING)


**Important**: Do NOT wait for each operation sequentially. Use parallel tool calls.

### Checkpoint - Confidence Check (200 tokens max)

**Self-Assessment Questions** (Binary Pass/Fail):

```yaml
CHECK 1: "All context files loaded successfully?" (25%)
   PASS: Read operations completed without errors
   PASS: Missing files noted but don't block (pm_context.md, last_session.md optional)
   FAIL: Critical read errors (permissions, corrupted files)
   → Status: PASS/FAIL

CHECK 2: "Git branch matches feature state?" (25%)
   PASS: Current branch == state.json branch_name
   PASS: No git repo (acceptable, note it)
   FAIL: Branch mismatch (user switched branches without updating state)
   → Status: PASS/FAIL

CHECK 3: "Feature state is consistent?" (25%)
   PASS: state.json phase is valid (SPECIFYING, CLARIFYING, PLANNING, etc.)
   PASS: No contradictions between state.json and file presence
   FAIL: Phase doesn't match artifacts (e.g., PLANNING but no plan.md)
   → Status: PASS/FAIL

CHECK 4: "Ready to provide actionable status?" (25%)
   PASS: Can determine current phase and recommend next action
   PASS: Have minimum context (branch OR state.json)
   FAIL: No context available (no git, no state.json, no features)
   → Status: PASS/FAIL
```

**Confidence Formula**:

```yaml
Confidence = (Passed Checks / 4) × 100

Examples:
- 4/4 checks pass: 100% → PROCEED (high confidence)
- 3/4 checks pass: 75% → PROCEED (adequate confidence)
- 2/4 checks pass: 50% → BLOCK (below threshold)
- 1/4 checks pass: 25% → BLOCK (insufficient context)
- 0/4 checks pass: 0% → BLOCK (no context)

Threshold: ≥75% (requires 3 or 4 checks passing)
NOTE: This is Fix C2 - confidence threshold raised from 70% to 75%
```

**Decision Logic**:

```yaml
IF confidence < 75% (0-2 checks passed):
  → STOP execution immediately
  → Report which checks failed with specific details
  → Request user to fix issues
  → DO NOT guess or make assumptions
  → Output format: "❌ Low Confidence ([X]%) - [Failed Check Details]"

ELSE IF confidence >= 75% (3-4 checks passed):
  → High confidence - proceed to review gate
  → Gather evidence for review gate
```

### PM Agent Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate session context restoration completeness before presenting status.

### Evidence Collection (Mandatory)

❓ **"Were all state files loaded?"**
Action Required:
  - Verify file read operations completed
  - Show ACTUAL file status
  - Report: Which files loaded successfully
  - **v2.3**: Prioritize spec-metadata.json over state.json

Expected Evidence:
  ✓ pm_context.md: [LOADED | NOT FOUND | ERROR]
  ✓ spec-metadata.json: [LOADED | NOT FOUND | ERROR] (v2.3 primary)
  ✓ state.json: [LOADED | NOT FOUND | ERROR] (v2.1 fallback)
  ✓ last_session.md: [LOADED | NOT FOUND | ERROR]
  ✓ next_actions.md: [LOADED | NOT FOUND | ERROR]

**State Loading Priority (v2.3)**:
  IF spec-metadata.json exists:
    → Use as primary state source
    → Convert phase: specification → SPECIFYING, planning → PLANNING, etc.
  ELSE IF state.json exists:
    → Use as fallback (v2.1 compatibility)
  ELSE:
    → No state found (first-time setup)

❓ **"Does git branch match feature state?"**
Action Required:
  - Compare git branch vs state.json branch_name
  - Show ACTUAL branch names
  - Report: Match status

Expected Evidence:
  ✓ Current git branch: [BRANCH_NAME | no-git]
  ✓ state.json branch_name: [BRANCH_NAME]
  ✓ Match: [YES | NO | N/A (no git)]

❓ **"Is feature phase consistent with files?"**
Action Required:
  - Verify phase matches file evidence
  - Show ACTUAL phase and file presence
  - Report: Consistency check

Expected Evidence:
  ✓ Phase from state.json: [SPECIFYING|CLARIFYING|PLANNING|...]
  ✓ File evidence:
    - spec.md exists: [YES/NO]
    - plan.md exists: [YES/NO]
    - tasks.md exists: [YES/NO]
  ✓ Phase-file consistency: [CONSISTENT | INCONSISTENT]

❓ **"Is confidence score above threshold?"**
Action Required:
  - Calculate confidence from 4 checks
  - Show ACTUAL confidence percentage
  - Report: Confidence score and threshold

Expected Evidence:
  ✓ CHECK 1 result: [PASS/FAIL]
  ✓ CHECK 2 result: [PASS/FAIL]
  ✓ CHECK 3 result: [PASS/FAIL]
  ✓ CHECK 4 result: [PASS/FAIL]
  ✓ Confidence: [N]% (threshold: ≥75%)

IF any evidence is MISSING:
  ❌ CANNOT report status
  → Gather missing evidence first
  → Re-run this step with complete evidence

### Hallucination Prevention (7 Red Flags for Session Restoration)

```yaml
Detect and BLOCK these patterns:

🚨 "Context restored" WITHOUT showing which files were loaded
   → Self-correction: "Wait, I need to show file load status for each file"

🚨 "High confidence" WITH <75% confidence score
   → Self-correction: "Confidence below threshold, must report as low confidence"

🚨 "Ready to proceed" WITH branch mismatch
   → Self-correction: "Branch mismatch is critical, need user to fix"

🚨 "Phase: PLANNING" WITHOUT plan.md existing
   → Self-correction: "Phase-file inconsistency detected, need to report it"

🚨 Claiming "all state valid" WITHOUT checking file consistency
   → Self-correction: "Must verify phase matches actual file presence"

🚨 Recommending next action WITH stale state
   → Self-correction: "State may be stale, need to verify timestamps"

🚨 Skipping critical check failures (branch, phase consistency)
   → Self-correction: "Critical checks failed, must report and block"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

✅ **READY (High Confidence ≥75%)**:
```yaml
Criteria (ALL must be met):
  - Confidence ≥75% (3-4 checks passed)
  - All critical checks passed (no critical failures)
  - Phase-file consistency verified
  - Branch match confirmed (or no git repo)
  - Next action can be determined

IF ALL criteria met:
  → Present symbol-based status to user
```

⚠️ **NEEDS REVIEW (Confidence 50-74%)**:
```yaml
Criteria:
  - Confidence 50-74% (2 checks passed)
  - Some state issues present (branch mismatch, minor inconsistency)
  - Partial context available
  - Manual review recommended

IF criteria met:
  → Present issues to user
  → Request manual state verification
  → Do NOT recommend next action automatically
```

❌ **NOT READY (Low Confidence <50%)**:
```yaml
Criteria (ANY triggers NOT READY):
  - Confidence <50% (0-1 checks passed)
  - No git repo AND no state.json
  - Critical file read errors
  - Phase completely inconsistent with files
  - No features found

IF NOT READY:
  → Present failed checks with evidence
  → Recommend: "Fix state issues before proceeding"
  → DO NOT guess or assume next action
  → STOP workflow progression
```

### Output Format (Present to User - ONLY if evidence provided)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Session Context Restoration Review
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**Confidence**: [N]% (threshold: ≥75%)

**State Files Loaded**:
  [✅ | ❌] pm_context.md
  [✅ | ❌] state.json
  [✅ | ❌] last_session.md
  [✅ | ❌] next_actions.md

**Git Branch Match**:
  - Current branch: [BRANCH_NAME | no-git]
  - state.json branch: [BRANCH_NAME]
  - Match: [✅ YES | ❌ NO | N/A]

**Phase Consistency**:
  - Phase: [PHASE]
  - spec.md exists: [✅ YES | ❌ NO]
  - plan.md exists: [✅ YES | ❌ NO]
  - tasks.md exists: [✅ YES | ❌ NO]
  - Consistency: [✅ CONSISTENT | ❌ INCONSISTENT]

**Confidence Check Results**:
  [✅ | ❌] CHECK 1: Context files loaded
  [✅ | ❌] CHECK 2: Git branch match
  [✅ | ❌] CHECK 3: Feature state consistent
  [✅ | ❌] CHECK 4: Ready for actionable status

[IF confidence ≥75%]
Next Action: [Present symbol-based status]

[IF confidence <75%]
**Issues to Fix**:
1. [Failed check 1 description]
   Fix: [Resolution steps]
2. [Failed check 2 description]
   Fix: [Resolution steps]

Next Action: Fix issues above before continuing
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Wave 2 - Output - Minimal Status (When Confidence >= 75%)

**Format** (Symbol-Based, Ultra-Compressed):

```
🟢 feature/000001-auth | 📋 PLANNING | 60%
🎯 Ready: /speckit.tasks
```

**Format Explanation**:
- **Status emoji**: 🟢 (active), 🟡 (blocked), 🔴 (error), ⚫ (no feature)
- **Branch**: Current git branch
- **Phase emoji**: 📝 (SPECIFYING), 💭 (CLARIFYING), 📋 (PLANNING), ✅ (TASKING), 🔨 (IMPLEMENTING), 🔄 (RECONCILING), 🧪 (VALIDATING), ✓ (COMPLETED)
- **Progress**: Percentage based on phase completion
- **Next action**: Recommended command to execute

**Rules**:
- NO explanations (user sees git status separately)
- NO task lists (assumed from state)
- NO "What can I help with" (action is explicit)
- **Symbol-only status**
- STOP if confidence <70% and request clarification

### Wave 2 - State Update (If Needed)

If stale data detected or inconsistencies found:

```yaml
PARALLEL Write Operations (v2.3 Unified State Management):
  1. Write: .specify/memory/pm_context.md (updated timestamp)
  2. Write: specs/[FEATURE]/spec-metadata.json (primary state, v2.3)
  3. Write: .specify/memory/features/[feature]/state.json (fallback, v2.1 compatibility)
```

**Dual State Update Logic (v2.3)**:
- IF spec-metadata.json exists:
  → Update both spec-metadata.json AND state.json (keep in sync)
- ELSE IF only state.json exists:
  → Update only state.json (v2.1 compatibility mode)
- ELSE (no state files):
  → Create spec-metadata.json (v2.3 standard)

## State Machine Phases

```yaml
Phase Progression (v2.3 Unified Format):

Display Format         spec-metadata.json      state.json          Progress
(PM Output)            (v2.3 lowercase)        (v2.1 UPPERCASE)    (%)
-------------------------------------------------------------------------------
SPECIFYING             specification           SPECIFYING          0-15%
CLARIFYING             clarifying              CLARIFYING          15-30%
PLANNING               planning                PLANNING            30-50%
TASKING                tasks                   TASKING             50-70%
IMPLEMENTING           implementation          IMPLEMENTING        70-90%
RECONCILING            reconciliation          RECONCILING         90-95%
VALIDATING             validating              VALIDATING          95-98%
COMPLETED              complete                COMPLETED           100%

Progress Calculation (same for both formats):
  SPECIFYING:    0-15%    (spec exists but not clarified)
  CLARIFYING:    15-30%   (clarifications recorded)
  PLANNING:      30-50%   (plan.md exists)
  TASKING:       50-70%   (tasks.md exists)
  IMPLEMENTING:  70-90%   (tasks being executed)
  RECONCILING:   90-95%   (gaps identified, surgical edits)
  VALIDATING:    95-98%   (tests running)
  COMPLETED:     100%     (merged)

Phase Mapping Helper:
  specification → SPECIFYING
  clarifying → CLARIFYING
  gap_analysis → GAP_ANALYSIS (v2.3 new phase)
  planning → PLANNING
  tasks → TASKING
  implementation → IMPLEMENTING
  reconciliation → RECONCILING
  complete → COMPLETED
```

## Command Execution Tracking (v2.3 Dual State)

Every command should update state files (both spec-metadata.json and state.json when both exist):

```json
{
  "commands_executed": [
    {
      "command": "specify",
      "timestamp": "2025-01-26T10:30:00Z",
      "success": true
    },
    {
      "command": "clarify",
      "timestamp": "2025-01-26T10:45:00Z",
      "success": true,
      "clarifications_added": 5
    }
  ]
}
```

## Next Action Recommendations

**Logic** (Phase-Based):

```yaml
IF phase == "SPECIFYING" AND spec.md exists:
  → Check spec size
  → IF spec > 150KB OR FRs > 80:
      Recommend: /speckit.supplement [scope] "description"
      Reason: "Large spec detected - create supplementary specs"
  → ELSE:
      Recommend: /speckit.clarify
      Reason: "Spec created, clarify underspecified areas"

ELSE IF phase == "CLARIFYING" AND clarifications complete:
  → Recommend: /speckit.plan
  → Reason: "Clarifications recorded, ready for planning"

ELSE IF phase == "PLANNING" AND plan.md exists:
  → Check prerequisites:
    - [ ] Constitutional gates passed?
    - [ ] Simplicity gate passed?
    - [ ] Anti-abstraction gate passed?
  → IF supplementary specs exist AND not validated:
      Recommend: /speckit.validate-hierarchy
      Reason: "Validate hierarchical spec integrity"
  → ELSE IF all gates passed:
      Recommend: /speckit.tasks
  → ELSE:
      Recommend: Fix planning issues first

ELSE IF phase == "TASKING" AND tasks.md exists:
  → Recommend: /speckit.implement
  → Reason: "Tasks defined, ready for implementation"

ELSE IF phase == "IMPLEMENTING":
  → Check task progress
  → IF tasks complete:
      Recommend: /speckit.reconcile
      Reason: "Implementation done - close any gaps"
  → ELSE:
      Recommend: Continue implement

ELSE IF phase == "RECONCILING":
  → Check gap closure status
  → IF supplementary specs modified:
      Recommend: /speckit.validate-hierarchy
      Reason: "Validate updated supplementary specs"
  → ELSE IF gaps closed:
      Recommend: Move to validating
  → ELSE:
      Recommend: Continue reconciling gaps

ELSE IF phase == "VALIDATING":
  → Recommend: Fix test failures or merge

ELSE IF phase == "COMPLETED":
  → Recommend: Start new feature or celebrate 🎉
```

## Integration with Commands

All `/speckit.*` commands should call state update helper:

```yaml
After Command Completion:
  1. Read: .specify/memory/features/[feature]/state.json
  2. Update: phase, progress, gates_passed, commands_executed
  3. Update: updated_at timestamp
  4. Write: state.json back

Example Integration:
  /speckit.specify completes:
    → phase = "SPECIFYING"
    → progress = 15
    → commands_executed += {"command": "specify", ...}

  /speckit.clarify completes:
    → phase = "CLARIFYING"
    → progress = 30
    → clarifications_count = 5

  /speckit.supplement completes:
    → supplementary_specs.enabled = true
    → supplementary_specs.specs += {filename, scope, target_agents}

  /speckit.reconcile completes:
    → phase = "RECONCILING"
    → progress = 95
    → reconciliation_cycles += 1
    → gaps_identified = [...]

  /speckit.validate-hierarchy completes:
    → validation_passed = true/false
    → If failed: block workflow (strict mode)
```

## Token Economy

**Target Budgets**:
- Session start: **200-300 tokens** (most sessions)
- Low confidence: **300-500 tokens** (report issues)
- State sync: **100-200 tokens** (write operations)

**Optimization Strategies**:
- Parallel I/O (3-5x faster)
- Symbol-based output (10x compression)
- Silent reads (no verbose logging)
- Checkpoint early (prevent wrong-direction work)

## Example Sessions

### Session 1: Starting Fresh

```
User: /speckit.pm

[Wave 1: Parallel reads...]
[Checkpoint: No state files found, confidence 80%]

⚫ No active feature
🎯 Ready: /speckit.specify [description]
```

### Session 2: Mid-Planning

```
User: /speckit.pm

[Wave 1: Parallel reads...]
[Checkpoint: All files loaded, confidence 95%]

🟢 feature/000001-auth | 📋 PLANNING | 45%
🎯 Ready: /speckit.tasks

Prerequisites ✓:
  - Spec exists ✓
  - Clarifications complete ✓
  - Plan.md exists ✓
```

### Session 3: Blocked

```
User: /speckit.pm

[Wave 1: Parallel reads...]
[Checkpoint: Blocker detected, confidence 90%]

🟡 feature/000001-auth | 📋 PLANNING | 40%
⚠️ Blocker: Anti-abstraction gate failed
🎯 Fix: Review constitution Article VIII
```

## Error Handling

```yaml
Graceful Degradation:
  IF .specify/memory/ missing:
    → Warn user (first-time setup)
    → Offer to initialize structure

  IF git not available:
    → Fall back to SPECIFY_FEATURE env var
    → Warn limited functionality

  IF state.json corrupted:
    → Attempt recovery from backup
    → Or suggest manual recreation

  IF multiple active features:
    → List all and ask user to specify
    → Update pm_context with selection
```

## User Queries (Natural Language)

Support conversational queries:

```yaml
"Where did I leave off?"
  → Run session start protocol
  → Show current phase and next action

"What's my progress on feature 1?"
  → Read state for feature 000001-*
  → Show progress percentage and phase

"What's next?"
  → Load state
  → Show recommended next command with reason

"Am I blocked?"
  → Check gates_failed in state
  → Report blockers and suggested fixes
```

## File Structure Reference

```
.specify/
  memory/
    pm_context.md              # Global session context
    last_session.md            # Last session summary
    next_actions.md            # Recommended actions
    patterns_learned.jsonl     # Learning patterns (Phase 3)
    solutions_learned.jsonl    # Error solutions (Phase 3)
    features/
      000001-auth-system/
        state.json             # Feature state machine
        checkpoints.json       # Phase completion checkpoints (future)
        clarifications.md      # Recorded clarifications (future)
        pdca/                  # PDCA cycle docs (Phase 3)
          plan.md
          do.md
          check.md
          act.md
```

## Success Criteria

- ✅ Session start <300 tokens (typical case)
- ✅ Confidence check prevents wrong-direction work
- ✅ State restoration 100% accurate
- ✅ Next action recommendation matches workflow phase
- ✅ Graceful degradation on missing files
- ✅ Symbol-based output (minimal cognitive load)

---

**Command Version**: 2.3.0
**Last Updated**: 2025-11-05
**Compatibility**: SpecKit v2.3+
**Implementation**: Phase 1.2
**Pattern**: Parallel-with-Reflection (Wave → Checkpoint → Wave)
**Token Budget**: 200-500 tokens per invocation
**Auto-Activation**: Recommended at every session start
**New in v2.3**: Unified state management (spec-metadata.json primary, state.json fallback), phase mapping support, dual state sync
