# Reconcile Guide - Post-Implementation Gap Closure

**Command**: `/speckit.reconcile`
**Version**: SpecKit v2.1
**Purpose**: Close specification gaps discovered after implementation

---

## Table of Contents

1. [Overview](#overview)
2. [When to Use](#when-to-use)
3. [Quick Start](#quick-start)
4. [Gap Types](#gap-types)
5. [Workflow](#workflow)
6. [Best Practices](#best-practices)
7. [Examples](#examples)
8. [Integration](#integration)

---

## Overview

The reconcile command fills the gap between implementation and specification by applying surgical, append-only edits to close gaps discovered during testing, QA, or production deployment.

**Problem Solved**: Post-implementation gaps like:
- Missing navigation routes
- Outdated acceptance criteria
- Incomplete integration tasks
- Missing documentation

**Solution**: Structured reconciliation process with strict constraints to maintain spec integrity.

---

## When to Use

### ✅ Use Reconcile When:

- Implementation complete (`/speckit.implement` finished)
- Gaps discovered during testing/QA/review
- Changes are **additive** (new tasks, updated criteria)
- Core functional requirements unchanged

### ❌ Do NOT Use Reconcile When:

- Core FRs need major changes → Use `/speckit.specify` + new branch
- Architecture decisions changed → Use `/speckit.amend-technical`
- Starting new feature → Use `/speckit.specify`

---

## Quick Start

```bash
# 1. Identify gap (e.g., missing navigation)
Gap: Users cannot access billing console from menu

# 2. Run reconcile with gap report
/speckit.reconcile

# 3. Provide structured gap report when prompted:
## Gap Report
### Gaps Identified
1. **Missing Navigation** (HIGH)
   - Users cannot access billing console from main menu
   - No route configured in app routing
   - Impact: Feature inaccessible to end users

# 4. Answer clarifying questions (≤5)
Q: Menu placement?
A: Main menu (top-level)

# 5. Review sync impact report
Files modified:
- spec.md: Updated AC-015
- UI-SPEC.md: Added navigation section
- plan.md: Added routing configuration
- tasks.md: Appended 2 new tasks

# 6. Implement new tasks
/speckit.implement
```

---

## Gap Types

### 1. Missing Navigation

**Symptoms**: Feature implemented but no user access route

**Reconcile Actions**:
- Edit spec.md: Add/update acceptance criteria for navigation
- Edit UI-SPEC.md: Add navigation section (if hierarchical)
- Edit plan.md: Add routing configuration
- Edit tasks.md: Append navigation implementation tasks

**Integration Tests Required**: YES (navigation access control)

### 2. Outdated Acceptance Criteria

**Symptoms**: Implementation diverged from spec (features added/removed)

**Reconcile Actions**:
- Edit spec.md: Update affected acceptance criteria
- Mark changes with timestamp: `✅ UPDATED (2025-11-01)`
- Add revision note explaining drift

**Integration Tests Required**: NO (unless behavior changed)

### 3. Incomplete Integration

**Symptoms**: Missing webhooks, API wiring, service connections

**Reconcile Actions**:
- Edit spec.md: Add FR for integration (if missing)
- Edit API-SPEC.md: Document integration contracts (if hierarchical)
- Edit plan.md: Add integration architecture section
- Edit tasks.md: Append integration tasks + tests

**Integration Tests Required**: YES (always for integrations)

### 4. Missing Documentation

**Symptoms**: Undocumented APIs, components, workflows

**Reconcile Actions**:
- Edit API-SPEC.md: Document endpoints (if hierarchical)
- Edit tasks.md: Append documentation tasks
- Optional: Create API reference docs

**Integration Tests Required**: NO (documentation only)

### 5. Incomplete Tasks

**Symptoms**: Tasks marked done but gaps remain

**Reconcile Actions**:
- Edit tasks.md: Append missing subtasks
- Edit spec.md: Clarify requirements if ambiguous
- Edit plan.md: Update implementation strategy if needed

**Integration Tests Required**: Depends on gap type

---

## Workflow

### Step 1: Prepare Gap Report

Create a structured gap report:

```markdown
## Gap Report

**Feature**: 000003-billing-console
**Discovered By**: QA Testing | Code Review | Production
**Date**: 2025-11-01

### Gaps Identified

1. **[Gap Type]** ([HIGH | MEDIUM | LOW])
   - [Description of what's missing/wrong]
   - [Impact on users/system]
   - [Why this is a problem]

2. **[Another Gap]** ([Severity])
   - [Description]
   - [Impact]
```

### Step 2: Run Reconcile

```bash
/speckit.reconcile
```

### Step 3: Answer Clarifications

Reconcile will ask ≤5 clarifying questions:

```
Q1: Navigation placement? (A) Main menu (B) Sidebar (C) Dashboard widget
Q2: Route path? (A) /billing (B) /console/billing (C) /dashboard/billing
Q3: Priority? (A) Fix all now (B) Defer LOW severity
```

### Step 4: Review Sync Impact Report

**Generated**: `specs/{feature-id}/SYNC-IMPACT-REPORT-{cycle}.md`

**Contents**:
- Files modified (with line numbers)
- Outstanding decisions
- Integration tests required
- Reconciliation metadata

### Step 5: Validate Changes

**Strict Mode** (v2.1 default):

```bash
/speckit.validate-hierarchy
# Errors block workflow if validation fails
```

### Step 6: Implement New Tasks

```bash
# Tasks appended to tasks.md
/speckit.implement
```

---

## Best Practices

### 1. One Reconciliation Per Gap Discovery

**Good**: Find gap → Reconcile → Fix → Find next gap → Reconcile

**Bad**: Wait 2 weeks, collect all gaps → One massive reconcile

**Why**: Focused reconciliations are easier to review and less error-prone.

### 2. Append-Only Edits

**Preserve existing structure**:

```markdown
## Acceptance Criteria

### AC-015: Export Data ✅ UPDATED
- Original criteria (preserved)
- **NEW**: Additional capability (appended, marked with date)
```

**Don't rewrite**:

```markdown
### AC-015: Export Data (COMPLETELY REWRITTEN)
- All new, original deleted ❌
```

### 3. Reference Supplementary Specs

If hierarchical specs exist, reconcile should update domain-specific specs:

```
Gap: Missing API documentation
→ Edit API-SPEC.md (not spec.md)

Gap: Missing UI navigation
→ Edit UI-SPEC.md (if exists), else spec.md
```

### 4. Integration Tests Are Mandatory

For wiring/integration gaps, ALWAYS add integration test task:

```markdown
#### Task N: Integration Test - Stripe Webhook Handling
- [ ] Test payment.succeeded triggers notification
- [ ] Test payment.failed triggers retry
- [ ] Test invalid signature rejected
- [ ] **File**: src/services/payment/payment.integration.spec.ts
- [ ] **Priority**: HIGH
```

### 5. Track Reconciliation Cycles

Multiple reconciliations indicate:
- Incomplete initial spec (learn for next feature)
- Complex domain (acceptable for large features)
- Process issue (>3 cycles → investigate)

**Learning**:

```jsonl
// .specify/memory/patterns_learned.jsonl
{"pattern_id":"nav_gap_001","trigger":"new UI but no route","questions":["Menu placement?","Route path?"],"success_rate":1.0,"times_used":3}
```

---

## Examples

### Example 1: Simple Navigation Gap

**Gap Report**:
```
Users cannot access billing console. No menu entry. Route not configured.
```

**Questions** (2):
1. Menu placement? → Main menu
2. Route path? → /console/billing

**Edits** (3 files):
- UI-SPEC.md: Added navigation section
- plan.md: Added routing configuration
- tasks.md: Appended navigation task

**Result**: Sync Impact Report shows 1 new task, ready to implement.

### Example 2: Complex Integration Gap

**Gap Report**:
```
## Gaps
1. Stripe webhook handling missing
2. Notification service not wired
3. Error retry logic not implemented
4. Integration tests missing
```

**Questions** (5):
1. Which webhooks? → payment.succeeded, payment.failed
2. Notification targets? → Email + SMS
3. Retry strategy? → Exponential backoff
4. Test coverage? → Full error scenarios
5. Priority? → Fix all now

**Edits** (4 files):
- spec.md: Added FR-081 (Webhook handling)
- API-SPEC.md: Documented webhook endpoints
- plan.md: Added integration architecture
- tasks.md: Appended 6 tasks (implementation + tests)

**Result**: 6 new tasks, 3 integration tests required.

---

## Integration

### With Hierarchical Specs

**Reconcile automatically detects supplementary specs**:

```bash
if [ -f "UI-SPEC.md" ] && gap_type == "missing_navigation"; then
  # Edit UI-SPEC.md for UI gaps
  TARGET_SPEC="UI-SPEC.md"
elif [ -f "API-SPEC.md" ] && gap_type == "incomplete_integration"; then
  # Edit API-SPEC.md for API gaps
  TARGET_SPEC="API-SPEC.md"
else
  # Fallback to parent spec.md
  TARGET_SPEC="spec.md"
fi
```

### With State Machine

**Reconcile updates state.json**:

```json
{
  "phase": "RECONCILING",
  "progress": 96,
  "reconciliation_cycles": 1,
  "gaps_identified": [
    {"type": "missing_navigation", "status": "fixed", "severity": "HIGH"}
  ],
  "sync_reports": [
    {"cycle": 1, "path": "SYNC-IMPACT-REPORT-001.md"}
  ]
}
```

### With Validation

**Strict mode blocks workflow**:

```bash
/speckit.reconcile
# Edits made...

/speckit.validate-hierarchy
# Auto-runs after reconcile (if configured)

# If validation fails:
❌ ERROR: UI-SPEC.md missing parent reference
BLOCKED: Fix errors before continuing
```

---

## Troubleshooting

### Issue: Too Many Clarification Markers

```
❌ ERROR: Would exceed 3 [NEEDS CLARIFICATION] marker limit

RESOLUTION:
1. Resolve existing clarifications first, OR
2. Split gap report into multiple reconciliations
```

### Issue: Validation Fails (Strict Mode)

```
❌ ERROR: Validation failed after reconcile

RESOLUTION:
1. Check frontmatter in edited supplementary specs
2. Run /speckit.validate-hierarchy for details
3. Fix validation errors
4. Re-run reconcile if needed
```

### Issue: Gap Report Malformed

```
⚠️ WARNING: Gap report appears empty

RESOLUTION:
Provide structured gap report:
## Gap Report
### Gaps Identified
1. [Description]
```

---

## Next Steps

After successful reconciliation:

1. **Implement New Tasks**: `/speckit.implement`
2. **Run Tests**: Especially integration tests
3. **Update PM Context**: `/speckit.pm`
4. **Get Next Action**: `/speckit.next`

---

## Related Commands

- `/speckit.amend-technical` - Amend architecture decisions
- `/speckit.validate-hierarchy` - Validate supplementary specs
- `/speckit.pm` - Update session context
- `/speckit.next` - Get next action

---

**Version**: 2.1.0
**Last Updated**: 2025-11-01
**See Also**: [reconcile.md](../templates/commands/reconcile.md) (full command spec)
