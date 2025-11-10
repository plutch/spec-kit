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

### 3. Load Feature State (v2.3 Unified State Management)

```yaml
Read state with priority order:
  1. FIRST: specs/[FEATURE]/spec-metadata.json (v2.3 primary)
  2. FALLBACK: .specify/memory/features/[FEATURE_DIR]/state.json (v2.1 compatibility)

IF spec-metadata.json found:
  → Use as primary state source
  → Parse JSON and extract:
    - phase: Current workflow phase (lowercase: specification, planning, etc.)
    - progress: Calculate from phase
    - approvals: Approval status for each phase
    - metadata: risk_level, overall_quality, etc.
  → Convert phase to uppercase for display (specification → SPECIFYING)

ELSE IF state.json found:
  → Use as fallback (v2.1 compatibility)
  → Parse JSON and extract:
    - phase: Current workflow phase (uppercase: SPECIFYING, PLANNING, etc.)
    - progress: Percentage complete
    - gates_passed: Completed gates
    - gates_failed: Failed validation gates
    - blockers: Current blockers

ELSE (neither found):
  → Report: "State files missing. Run /speckit.pm to initialize or /speckit.specify to create new feature."
  → EXIT

Phase Mapping (spec-metadata.json → Display):
  specification → SPECIFYING
  gap_analysis → GAP_ANALYSIS
  planning → PLANNING
  tasks → TASKING
  implementation → IMPLEMENTING
  reconciliation → RECONCILING
  complete → COMPLETED
```

### 3.5 Check Quality Scores (v2.3 Enhanced)

```yaml
Check quality from multiple sources (priority order):

1. FIRST: spec-metadata.json (v2.3)
   IF spec-metadata.json exists AND contains metadata.overall_quality:
     → Use overall_quality from metadata
     → Check metadata.critical_issues count

2. FALLBACK: .specify/memory/features/[FEATURE_DIR]/quality.json (v2.1/v2.2)
   IF quality.json exists:
     Parse quality scores:
       - overall_quality: Overall score (0-10)
       - last_analysis_date: When analyze was last run
       - last_analysis_command: analyze or analyze-ux
       - critical_issues: Count of 🔴 CRITICAL issues
       - major_issues: Count of 🟠 MAJOR issues

Quality-Based Flags:
  IF overall_quality < 7:
    → Flag: Quality review recommended before proceeding

  IF critical_issues > 0:
    → Flag: CRITICAL issues block workflow progression

Store for recommendation logic
```

### 3.6 Check Risk Level (v2.3 Enhanced)

```yaml
Check risk level from multiple sources (priority order):

1. FIRST: spec-metadata.json (v2.3)
   IF spec-metadata.json exists AND contains metadata.risk_level:
     → Use risk_level directly (HIGH/MEDIUM/LOW)
     → No parsing needed (pre-calculated)

2. FALLBACK: specs/[FEATURE]/spec.md Risk Assessment section
   IF Risk Assessment section exists:
     Parse risk score (0-12 scale):
       - Calculate: Data Sensitivity + Access Control + External Integration +
                    Performance + Complexity + Business Impact

     Classification:
       - 🔴 HIGH (8-12): Extra scrutiny required
       - 🟠 MEDIUM (4-7): Standard workflow
       - 🟢 LOW (0-3): Fast-track eligible

3. ELSE (no risk information found):
   → Risk Assessment not found (may be older spec format)
   → Proceed with standard workflow (assume MEDIUM)

Store risk level for recommendation logic
```

### 3.7 Detect Feature Type (Context-Aware Detection)

```yaml
Detect feature characteristics:

UI-Heavy Feature:
  Check: specs/[FEATURE]/UI-SPEC.md exists
  → IF exists AND quality.json missing OR last_analysis_command != "analyze-ux":
      → Recommend analyze-ux before planning

Hierarchical Specs:
  Check: specs/[FEATURE]/ for supplementary specs (API-SPEC.md, UI-SPEC.md, etc.)
  → IF supplementary specs exist:
      → Check last modification time vs. last validation
      → IF modified recently AND not validated:
          → Recommend validate-hierarchy before proceeding

Gap Reports:
  Check: .specify/memory/features/[FEATURE_DIR]/gaps.json
  → IF exists AND gaps_count > 0:
      → Recommend reconcile to close gaps

Store feature type flags for recommendation logic
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
    ELSE: Move to reconcile

RECONCILING:
  Required:
    - Implementation complete
  Check:
    - Gap report provided
    - Supplementary specs exist
  Next:
    IF gaps remain: Continue reconcile
    ELSE IF supplementary specs modified: validate-hierarchy
    ELSE: Move to validating

VALIDATING:
  Required:
    - Reconciliation complete (if applicable)
  Check:
    - Test results
    - Hierarchical spec validation (if applicable)
  Next:
    IF validation fails: Fix errors
    ELSE IF tests passing: merge/complete
    ELSE: Fix test failures

COMPLETED:
  Next: New feature or celebrate
```

### 5. Generate Recommendation

**Output Format** (Symbol-Based with Quality/Risk Context):

```
📍 [FEATURE_NAME]
🔹 Phase: [PHASE] ([PROGRESS]%)
[IF quality.json exists] 📊 Quality: [X.X]/10 [⚠️ if <7] [[LAST_ANALYSIS_DATE]]
[IF Risk Assessment exists] 🎯 Risk: [🔴 HIGH | 🟠 MEDIUM | 🟢 LOW] ([SCORE]/12)

Prerequisites:
  ✅ [COMPLETED_ITEM]
  ❌ [MISSING_ITEM]

[IF critical_issues > 0]
🚨 **BLOCKED**: [N] CRITICAL issues must be resolved before proceeding
   Fix: Run /speckit.clarify or address issues in spec.md

[IF overall_quality < 7 AND phase approaching planning/implementing]
⚠️ **Quality Warning**: Overall quality [X.X]/10 - recommend quality review
   Suggestion: Run /speckit.analyze or /speckit.analyze-ux before proceeding

🎯 Next: /speckit.[COMMAND] [IF risk HIGH: + quality gate recommendations]
💡 Reason: [WHY_THIS_COMMAND]

[IF UI-SPEC.md exists AND analyze-ux not run]
💡 Suggestion: Run /speckit.analyze-ux for UI quality baseline (feature has UI-SPEC.md)

[IF supplementary specs modified AND not validated]
⚠️ Hierarchical specs modified - run /speckit.validate-hierarchy before proceeding

[OPTIONAL_NOTES]
```

### 5.5 Pre-Flight Quality Gate (Before Phase Transitions)

```yaml
Quality-Gated Phase Transitions:

CLARIFYING → PLANNING:
  Check quality baseline:
    IF analyze not run (no quality.json):
      → Recommend: "Run /speckit.analyze first for quality baseline"
      → Note: "Optional but recommended - identifies issues early"

  IF risk level HIGH (8-12):
    → Recommend: "Run /speckit.clarify --edge-cases (high-risk feature)"
    → Note: "High-risk features require extra scrutiny for edge cases"

  IF critical_issues > 0:
    → BLOCK: "Cannot proceed - [N] CRITICAL issues must be resolved"
    → Fix: "Address critical issues before planning"

PLANNING → TASKING:
  Check plan quality:
    IF UI-SPEC.md exists AND analyze-ux not run:
      → Recommend: "Run /speckit.analyze-ux for UI quality check"
      → Note: "UI-heavy feature detected - validate design quality"

  IF risk level HIGH (8-12):
    → Note: "High-risk feature - ensure test strategy covers all risks"

TASKING → IMPLEMENTING:
  Check implementation readiness:
    IF overall_quality < 7:
      → Warn: "Quality score below 7/10 - implementation may be challenging"
      → Suggest: "Consider running /speckit.clarify to resolve ambiguities"

IMPLEMENTING → RECONCILING:
  Check completeness:
    IF gaps.json exists with gaps_count > 0:
      → Automatic: Proceed to reconcile (expected workflow)

RECONCILING → VALIDATING:
  Check hierarchy integrity:
    IF supplementary specs exist AND modified AND not validated:
      → BLOCK: "Must run /speckit.validate-hierarchy before validation"
      → Fix: "Validate hierarchical spec integrity (strict mode)"
```

### 6. Next Action Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate recommendation accuracy before presenting to user.

### Evidence Collection (Mandatory)

❓ **"Was active feature identified?"**
Action Required:
  - Verify feature directory/ID determined
  - Show ACTUAL feature ID and branch
  - Report: Feature number, name, branch

Expected Evidence:
  ✓ Feature ID: 000001-feature-name
  ✓ Git branch: feature/000001-feature-name (or env var/directory)
  ✓ Feature directory exists: .specify/memory/features/000001-feature-name/

❓ **"Was current phase validated?"**
Action Required:
  - Read state.json phase field
  - Verify phase matches file evidence
  - Report: Phase and supporting files

Expected Evidence:
  ✓ Phase from state.json: [SPECIFYING|CLARIFYING|PLANNING|...]
  ✓ File evidence matches:
    - SPECIFYING: spec.md exists
    - CLARIFYING: spec.md + clarifications
    - PLANNING: plan.md exists
    - TASKING: tasks.md exists
    - IMPLEMENTING: tasks in progress
  ✓ No phase-file contradictions

❓ **"Is next action appropriate for phase?"**
Action Required:
  - Map phase → next command
  - Check entry criteria for next command
  - Report: Next command and reason

Expected Evidence:
  ✓ Next command: /speckit.[COMMAND]
  ✓ Entry criteria met (files exist, gates passed)
  ✓ Reason matches phase logic

❓ **"Are phase transition criteria met?"**
Action Required:
  - Check prerequisites for phase transition
  - Verify no blockers present
  - Report: Prerequisites status

Expected Evidence:
  ✓ Prerequisites checked:
    - Required files exist
    - Quality gates passed (if applicable)
    - No blockers in state.json
  ✓ Blockers: None OR [List blockers with fixes]

IF any evidence is MISSING:
  ❌ CANNOT recommend action
  → Gather missing evidence first
  → Re-run this step with complete evidence

### Hallucination Prevention (7 Red Flags for Next Action)

```yaml
Detect and BLOCK these patterns:

🚨 "Next: /speckit.tasks" WITHOUT verifying plan.md exists
   → Self-correction: "Wait, I need to check if plan.md actually exists"

🚨 "Ready for planning" WITH >3 [NEEDS CLARIFICATION] markers
   → Self-correction: "Clarifications not complete, cannot proceed"

🚨 "Proceed to implement" WITHOUT checking task completion
   → Self-correction: "I need to verify tasks.md status first"

🚨 Recommending action WITHOUT validating entry criteria
   → Self-correction: "Must check prerequisites for recommended command"

🚨 "No blockers" WITHOUT reading state.json gates_failed
   → Self-correction: "I need to check for blocked gates"

🚨 Generic recommendation NOT based on actual phase
   → Self-correction: "Must use phase-specific logic, not guessing"

🚨 "Next action clear" WITHOUT showing reason
   → Self-correction: "Need to explain WHY this is the next step"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

✅ **READY (Clear Next Action)**:
```yaml
Criteria (ALL must be met):
  - Active feature identified
  - Phase validated against file evidence
  - Next action determined from phase logic
  - Entry criteria for next command met
  - No blockers present

IF ALL criteria met:
  → Present next action recommendation
```

⚠️ **NEEDS REVIEW** (User Decision Required):
```yaml
Criteria:
  - Minor blockers present (non-critical)
  - Multiple valid next actions
  - Phase unclear (state-file mismatch)

IF criteria met:
  → Present options to user
  → Ask for clarification or decision
```

❌ **NOT READY** (Blockers Present):
```yaml
Criteria (ANY triggers NOT READY):
  - No active feature found
  - State file missing or corrupted
  - Critical blockers in state.json (gates_failed)
  - Phase-file contradiction (e.g., PLANNING but no plan.md)

IF NOT READY:
  → Present blockers with evidence
  → Recommend: "Fix issues before proceeding"
  → STOP workflow progression
```

### Output Format (Present to User - ONLY if evidence provided)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 Next Action Recommendation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**Feature**: [000001-feature-name]
**Branch**: [feature/000001-feature-name]
**Phase**: [PHASE] ([PROGRESS]%)

**Prerequisites**:
  [✅ | ❌] [Prerequisite 1]
  [✅ | ❌] [Prerequisite 2]
  [...]

🎯 **Next**: /speckit.[COMMAND]
💡 **Reason**: [Why this is the appropriate next step]

[IF blockers present]
🚨 **Blockers**:
  1. [Blocker description]
     Fix: [Suggested resolution]

[IF phase unclear]
⚠️ **State Mismatch**:
  State says: [PHASE]
  Reality: [Actual file state]
  Fix: [Resolution steps]

Next Action: [Run recommended command OR Fix blockers]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 7. Recommendation Logic (Enhanced with Quality & Risk)

```yaml
IF phase == "SPECIFYING":
  IF spec.md exists:
    # Quality Pre-Flight Check
    IF quality.json NOT exists:
      → Suggest: "Optional: Run /speckit.analyze for early quality check"
      → Primary: /speckit.clarify
      → Reason: "Spec created, resolve ambiguities"
    ELSE:
      → Recommend: /speckit.clarify
      → Reason: "Spec created, resolve ambiguities"
  ELSE:
    → Error: "Spec missing - state inconsistent"

ELSE IF phase == "CLARIFYING":
  Count [NEEDS CLARIFICATION] markers in spec.md

  # Check for quality blockers
  IF critical_issues > 0:
    → BLOCK: "Cannot proceed - [N] CRITICAL issues detected"
    → Fix: "Resolve critical issues before planning"
    → Next: /speckit.clarify (to address issues)

  ELSE IF markers > 0:
    → Recommend: /speckit.clarify --continue
    → Reason: "[N] clarifications still needed"

  ELSE IF clarifications_count == 0:
    → Recommend: /speckit.clarify
    → Reason: "No clarifications recorded yet"

  ELSE:
    # Quality-gated transition to PLANNING
    IF quality.json NOT exists:
      → Recommend: /speckit.analyze
      → Reason: "Run quality analysis before planning (establishes baseline)"
      → Note: "Optional but recommended - identifies issues early"

    ELSE IF overall_quality < 7:
      → Warn: "Quality score [X.X]/10 below threshold"
      → Recommend: /speckit.clarify --expert OR /speckit.plan (user choice)
      → Reason: "Low quality may cause planning difficulties"

    ELSE IF risk_level == HIGH:
      → Recommend: /speckit.clarify --deep
      → Reason: "High-risk feature ([SCORE]/12) - comprehensive expert review recommended"
      → Note: "Expert mode includes cross-lens conflict detection (v2.2.0)"
      → Alternative: "Or /speckit.clarify --edge-cases for focused edge case validation only"
      → After expert review: "Proceed to /speckit.plan"

    ELSE:
      → Recommend: /speckit.plan
      → Reason: "Clarifications complete, ready for planning"

ELSE IF phase == "PLANNING":
  IF plan.md missing:
    # Check if UI quality analysis needed
    IF UI-SPEC.md exists AND (quality.json missing OR last_analysis_command != "analyze-ux"):
      → Recommend: /speckit.analyze-ux
      → Reason: "UI-heavy feature detected - validate design quality before planning"
      → Note: "After UI analysis, run /speckit.plan"
    ELSE:
      → Recommend: /speckit.plan
      → Reason: "Create technical implementation plan"

  ELSE IF gates_failed not empty:
    → Report blockers
    → Recommend: Fix issues then re-run /speckit.plan

  ELSE IF plan.md exists AND gates passed:
    # Quality-gated transition to TASKING
    IF risk_level == HIGH AND overall_quality < 8:
      → Warn: "High-risk feature with quality score [X.X]/10"
      → Recommend: /speckit.clarify --expert (strengthen spec) OR /speckit.tasks (proceed)
      → Reason: "High-risk features benefit from comprehensive review"
    ELSE:
      → Recommend: /speckit.tasks
      → Reason: "Plan validated, generate task breakdown"

ELSE IF phase == "TASKING":
  IF tasks.md missing:
    → Recommend: /speckit.tasks
    → Reason: "Generate executable task list"
  ELSE:
    # Quality-gated transition to IMPLEMENTING
    IF overall_quality < 7:
      → Warn: "Quality score [X.X]/10 - implementation may be challenging"
      → Suggest: "Consider /speckit.clarify to resolve ambiguities first"
      → Primary: /speckit.implement
      → Reason: "Tasks defined, begin implementation (with caution)"
    ELSE:
      → Recommend: /speckit.implement
      → Reason: "Tasks defined, begin implementation"

ELSE IF phase == "IMPLEMENTING":
  → Check task completion
  → IF tasks complete:
      # Check for hierarchical specs before reconcile
      IF supplementary specs exist AND modified AND not validated:
        → Recommend: /speckit.validate-hierarchy
        → Reason: "Validate hierarchical specs before reconciliation"
      ELSE:
        → Recommend: /speckit.reconcile
        → Reason: "Implementation done - identify and close gaps"
  → ELSE:
      → Recommend: Continue implementation
      → Note: "Check task progress in tasks.md"

ELSE IF phase == "RECONCILING":
  → Check for supplementary specs
  → IF supplementary specs exist AND modified AND not validated:
      → Recommend: /speckit.validate-hierarchy
      → Reason: "Validate updated hierarchical specs (strict mode)"
  → ELSE IF gaps remain (gaps.json exists with gaps_count > 0):
      → Recommend: Continue /speckit.reconcile
      → Note: "Address remaining gaps from gap report"
  → ELSE:
      → Recommend: Move to validation
      → Reason: "Gaps closed, ready for testing"

ELSE IF phase == "VALIDATING":
  → Check for hierarchical specs
  → IF supplementary specs exist AND not validated:
      → BLOCK: "Must run /speckit.validate-hierarchy first"
      → Reason: "Validate hierarchical spec integrity (blocks workflow)"
  → ELSE:
      → Recommend: Fix test failures or create PR
      → Note: "Run tests to verify functionality"
      → IF risk_level == HIGH:
          → Note: "High-risk feature - ensure comprehensive test coverage"

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

### Example 2: Ready for Planning (with Quality Check Recommendation)

```
User: /speckit.next

📍 000042-payment-gateway
🔹 Phase: CLARIFYING (30%)
🎯 Risk: 🔴 HIGH (9/12)

Prerequisites:
  ✅ Spec created
  ✅ All clarifications resolved (5 added)
  ✅ No ambiguities remain
  ❌ Quality baseline not established

🎯 Next: /speckit.analyze
💡 Reason: Run quality analysis before planning (establishes baseline)

Note: High-risk feature detected. Quality analysis recommended to identify issues early.

After analysis, proceed to: /speckit.plan [tech stack details]
```

### Example 3: UI-Heavy Feature Detection

```
User: /speckit.next

📍 000015-dashboard-redesign
🔹 Phase: PLANNING (15%)
📊 Quality: No baseline (analyze not run)

Prerequisites:
  ✅ Spec created
  ✅ Clarifications complete
  ✅ UI-SPEC.md detected
  ❌ UI quality analysis not performed

🎯 Next: /speckit.analyze-ux
💡 Reason: UI-heavy feature detected - validate design quality before planning

Note: Feature contains UI-SPEC.md. Running analyze-ux will check:
  - UI Design Quality (5 dimensions)
  - Component System Audit (Angular/Kendo library usage, design tokens, reuse score)
  - UX scoring (usability, accessibility, efficiency)

After UI analysis, proceed to: /speckit.plan
```

### Example 4: Reconciling Phase with Hierarchical Specs

```
User: /speckit.next

📍 000003-billing-console
🔹 Phase: RECONCILING (92%)
📊 Quality: 8.2/10 (2025-01-15)

Prerequisites:
  ✅ Implementation complete
  ✅ Supplementary specs exist (UI-SPEC.md, API-SPEC.md)
  ⚠️ Gaps identified (missing navigation)
  ⚠️ Hierarchical specs modified (not validated)

🎯 Next: /speckit.validate-hierarchy
💡 Reason: Supplementary specs modified - validate hierarchy (strict mode)

After validation, continue gap closure with: /speckit.reconcile
```

### Example 5: Critical Issues Blocking Workflow

```
User: /speckit.next

📍 000008-user-authentication
🔹 Phase: CLARIFYING (35%)
📊 Quality: 5.8/10 (2025-01-14)
🎯 Risk: 🔴 HIGH (10/12)

🚨 **BLOCKED**: 3 CRITICAL issues must be resolved before proceeding
   Fix: Run /speckit.clarify or address issues in spec.md

Critical Issues:
1. 🔴 Missing password storage security spec (FR-003)
2. 🔴 No session timeout defined (FR-008)
3. 🔴 GDPR compliance gap - data retention not specified

⚠️ High-risk feature with critical security/compliance gaps.

Next: /speckit.clarify (address critical issues)

Cannot proceed to planning until critical issues resolved.
```

### Example 6: Constitution Gate Blocker

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

### Example 7: No Active Feature

```
User: /speckit.next

⚫ No active feature detected

Create a new feature with:
/speckit.specify [feature description]

Or switch to existing feature branch:
git checkout feature/000001-auth-system
```

### Example 8: Multiple Features

```
User: /speckit.next

🔍 Multiple features found:

1. 000001-auth-system (PLANNING - 45%) [Quality: 7.5/10 | Risk: 🟠 MEDIUM]
2. 000003-payment-flow (IMPLEMENTING - 75%) [Quality: 8.1/10 | Risk: 🔴 HIGH]
3. 000007-dashboard (CLARIFYING - 20%) [Quality: Not analyzed | Risk: 🟢 LOW]

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

**Target**: 400-800 tokens per invocation (increased from v2.1 due to quality/risk checks)

**v2.2 Enhancement Impact**:
- Quality score checks: +100-150 tokens
- Risk level detection: +50-100 tokens
- Feature type detection: +100-150 tokens
- Enhanced recommendation logic: +100-200 tokens
- Total increase: ~400-600 tokens (justified by value)

**Optimizations**:
- Symbol-based output (10x compression)
- Parallel state loading
- Skip verbose explanations
- Direct actionable recommendations
- Conditional quality checks (only when quality.json exists)
- Conditional risk checks (only when Risk Assessment present)

## Success Criteria

- ✅ Correct next command 100% of time
- ✅ Prerequisites accurately validated
- ✅ Blockers clearly identified with fixes
- ✅ Quality-driven recommendations (when quality.json exists)
- ✅ Risk-aware recommendations (when Risk Assessment present)
- ✅ Context-aware feature type detection (UI, hierarchical specs, gaps)
- ✅ <800 tokens per invocation (with quality/risk checks)
- ✅ Graceful handling of edge cases
- ✅ Clear, actionable output

---

**Command Version**: 2.3.0
**Last Updated**: 2025-11-05
**Compatibility**: SpecKit v2.3+
**Implementation**: Phase 2.3
**Dependencies**: Phase 1.2 (State Management), Phase 2.2 (Quality Integration), Phase 2.3 (Unified State)
**Token Budget**: 400-800 tokens (200-400 base + 400-600 for quality/risk)
**Pattern**: Smart recommendation with quality and risk awareness
**New in v2.3**: Unified state management (spec-metadata.json primary, state.json fallback), phase mapping support, quality/risk from metadata
