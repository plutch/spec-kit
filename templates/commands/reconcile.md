# Reconcile - Post-Implementation Gap Closure

**Command**: `/speckit.reconcile`
**Phase**: RECONCILING (95-98%)
**Purpose**: Surgical edits to specifications after implementation to close discovered gaps
**Pattern**: Gap Detection → Clarification → Surgical Edits → Sync Impact Report

---

## Overview

The reconcile command addresses post-implementation gaps that emerge after `/speckit.implement`:
- Missing navigation routes or menu entries
- Outdated acceptance criteria
- Incomplete integration tasks
- Missing wiring or documentation

Unlike a full specification amendment, reconcile performs **surgical, append-only edits** to maintain specification integrity while closing gaps.

---

## When to Use

Use `/speckit.reconcile` when:
- ✅ Implementation is complete (`/speckit.implement` finished)
- ✅ Gaps discovered during testing or review
- ✅ Changes are **additive** (new tasks, updated criteria, additional integration)
- ✅ Core functional requirements unchanged

Do NOT use reconcile when:
- ❌ Core FRs need major changes (use `/speckit.specify` + new branch)
- ❌ Architecture decisions changed (use `/speckit.amend-technical`)
- ❌ Starting new feature (use `/speckit.specify`)

---

## Input: Gap Report

Provide a structured gap report describing what was discovered:

```markdown
## Gap Report

**Feature**: 000003-billing-console
**Discovered By**: QA Testing
**Date**: 2025-11-01

### Gaps Identified

1. **Missing Navigation** (HIGH)
   - Users cannot access billing console from main menu
   - No route configured in app routing
   - Impact: Feature inaccessible to end users

2. **Outdated Acceptance Criteria** (MEDIUM)
   - AC-015 says "Export to CSV", but we implemented "Export to CSV and PDF"
   - Spec doesn't reflect actual shipped behavior
   - Impact: Documentation mismatch

3. **Incomplete Integration** (HIGH)
   - No integration test for Stripe webhook handling
   - Wiring between payment service and notification service missing
   - Impact: Production risk

4. **Missing Documentation** (LOW)
   - API endpoints not documented in API-SPEC.md
   - Developer onboarding will be difficult
   - Impact: Maintainability
```

---

## Execution Protocol

### Step 1: Parse Gap Report

**Task**: Analyze gaps and categorize by type and severity

```yaml
Gap Categories:
  - missing_navigation: Routes, menus, links
  - outdated_criteria: Acceptance criteria drift
  - incomplete_integration: Missing tests, wiring, webhooks
  - missing_documentation: Undocumented APIs, components
  - incomplete_tasks: Tasks marked done but gaps remain

Severity Levels:
  - HIGH: Blocks production deployment or user access
  - MEDIUM: Reduces quality but not blocking
  - LOW: Technical debt, maintainability issues
```

**Output**: Categorized gap list with severity

---

### Step 2: Ask Clarifying Questions (≤5 Maximum)

**Constraint**: Maximum 5 questions total to keep reconciliation lightweight

**Question Types**:
1. **Scope Questions**: "Should navigation be in main menu or sidebar?"
2. **UX Questions**: "Should route be /billing or /console/billing?"
3. **Integration Questions**: "Which Stripe webhooks need handling (payment.succeeded, payment.failed)?"
4. **Priority Questions**: "Fix all gaps now or defer LOW severity to future sprint?"
5. **Impact Questions**: "Does this change require database migration?"

**Example**:
```markdown
## Clarifying Questions

1. **Navigation Placement** (HIGH priority)
   - [ ] A) Main menu (top-level)
   - [ ] B) Settings submenu
   - [ ] C) Dashboard widget link

2. **API Documentation Scope** (LOW priority)
   - Should we document:
   - [ ] A) All endpoints (30+ endpoints)
   - [ ] B) Only public-facing endpoints (8 endpoints)
   - [ ] C) Defer to separate documentation sprint

3. **Integration Test Coverage**
   - [ ] A) Add tests for all Stripe webhooks (payment, subscription, refund)
   - [ ] B) Focus on critical path only (payment.succeeded)

4. **Acceptance Criteria Update**
   - [ ] A) Update AC-015 to match implementation (CSV + PDF)
   - [ ] B) Revert implementation to match spec (CSV only)

5. **Priority**
   - [ ] A) Fix all gaps in this reconciliation
   - [ ] B) Fix HIGH/MEDIUM now, defer LOW to backlog
```

---

### Step 3: Surgical Edits

**Constraints**:
- **Append-only** where possible (preserve existing structure)
- **Absolute paths** in all references
- **Maximum 3 [NEEDS CLARIFICATION]** markers per run
- **Always include integration tests** when wiring gaps exist

**Edit Targets**:

#### 3.1 spec.md (Parent Specification)
```markdown
# When to Edit:
- Outdated acceptance criteria (update to reflect implementation)
- Missing user stories (add story for navigation access)
- Functional requirements gaps (add FR for integration)

# Example Edit (Append to Acceptance Criteria):
## Acceptance Criteria

### AC-015: Export Billing Data ✅ UPDATED
- ~~Users can export billing data to CSV format~~ (ORIGINAL)
- Users can export billing data to **CSV or PDF format** (UPDATED: 2025-11-01)
- Export includes all transactions from selected date range
- **NEW**: PDF export includes company logo and branding (RECONCILE: 2025-11-01)
```

#### 3.2 *-SPEC.md (Supplementary Specifications)
```markdown
# When to Edit:
- UI-SPEC.md: Missing navigation sections, component details
- API-SPEC.md: Undocumented endpoints, missing schemas
- TECHNICAL-SPEC.md: Integration architecture, wiring diagrams

# Example Edit (Append to UI-SPEC.md Navigation Section):
## 5.3 Navigation

### Main Menu Structure (UPDATED: 2025-11-01 via Reconcile)

#### NEW: Billing Console Entry
- **Label**: "Billing"
- **Icon**: `billing-icon.svg`
- **Route**: `/console/billing`
- **Permission**: `billing.console.view`
- **Position**: After "Dashboard", before "Settings"
- **Badge**: Shows count of pending invoices
- **Implements**: FR-008 (User Access to Billing Console)
```

#### 3.3 plan.md (Implementation Plan)
```markdown
# When to Edit:
- Missing routing sections
- Incomplete integration contracts
- Testing strategy gaps

# Example Edit (Append to Routing Section):
## 6. Routing & Navigation (UPDATED: 2025-11-01 via Reconcile)

### NEW: Billing Console Route
- **Path**: `/console/billing`
- **Component**: `BillingConsoleComponent`
- **Guards**: `AuthGuard`, `BillingPermissionGuard`
- **Lazy Loading**: Yes (billing.module.ts)
- **Menu Integration**: Main menu (see UI-SPEC.md Section 5.3)
- **Reference**: [UI-SPEC.md](UI-SPEC.md#53-navigation)
```

#### 3.4 tasks.md (Task Checklist)
```markdown
# When to Edit:
- Always (append new tasks for gap closure)

# Example Edit (Append to Frontend Tasks):
## Frontend Tasks (Angular 20)

### NEW: Navigation Integration (RECONCILE: 2025-11-01)

#### Task 23: Add Billing Menu Entry
- [ ] Update main-menu.component.ts
- [ ] Add billing icon to assets/icons/
- [ ] Configure route in app-routing.module.ts
- [ ] Add permission check: `billing.console.view`
- [ ] **File**: `src/app/core/layout/main-menu/main-menu.component.ts`
- [ ] **File**: `src/app/app-routing.module.ts`
- [ ] **Reference**: [UI-SPEC.md Section 5.3](UI-SPEC.md#53-navigation)
- [ ] **Implements**: FR-008
- [ ] **Integration Test**: Navigate to /console/billing and verify access control

#### Task 24: Integration Test - Navigation Access
- [ ] Test authenticated user can access billing console
- [ ] Test unauthenticated user redirected to login
- [ ] Test user without `billing.console.view` sees 403 error
- [ ] **File**: `src/app/features/billing/billing.integration.spec.ts`
```

---

### Step 4: Generate Sync Impact Report

**Template**: Use `.specify/templates/reports/sync-impact-template.md`

**Report Contents**:
1. **Files Modified** (with absolute paths and line numbers)
2. **Outstanding Decisions** (unanswered questions or new clarification markers)
3. **Integration Tests Required** (derived from gaps)
4. **Reconciliation Metadata** (cycle number, timestamp, gap summary)

---

## Output: Sync Impact Report

```markdown
# Sync Impact Report

**Feature**: 000003-billing-console
**Reconciliation Cycle**: 1
**Generated**: 2025-11-01T14:30:00Z
**Command**: /speckit.reconcile

---

## Summary

**Gaps Addressed**: 4
- 2 HIGH severity (navigation, integration)
- 1 MEDIUM severity (acceptance criteria)
- 1 LOW severity (documentation)

**Files Modified**: 4
**New Tasks**: 2
**Integration Tests Required**: 2

---

## Files Modified

### 1. spec.md
**Path**: `/home/user/project/specs/000003-billing-console/spec.md`
**Lines Modified**: 145-152
**Change**: Updated AC-015 to reflect CSV + PDF export capability
**Type**: APPEND (acceptance criteria)

### 2. UI-SPEC.md
**Path**: `/home/user/project/specs/000003-billing-console/UI-SPEC.md`
**Lines Modified**: 287-298
**Change**: Added navigation section for billing console menu entry
**Type**: APPEND (new section)

### 3. plan.md
**Path**: `/home/user/project/specs/000003-billing-console/plan.md`
**Lines Modified**: 412-420
**Change**: Added routing configuration for /console/billing
**Type**: APPEND (routing section)

### 4. tasks.md
**Path**: `/home/user/project/specs/000003-billing-console/tasks.md`
**Lines Modified**: 189-210
**Change**: Appended 2 new tasks (navigation integration, integration test)
**Type**: APPEND (new tasks)

---

## Outstanding Decisions

### Decision 1: API Documentation Scope
**Status**: [NEEDS CLARIFICATION]
**Question**: Document all 30+ endpoints or only 8 public-facing endpoints?
**Impact**: Documentation effort (2 hours vs 8 hours)
**Assigned To**: Product Manager
**Due**: Before next sprint planning

---

## Integration Tests Required

### Test 1: Navigation Access Control
**File**: `src/app/features/billing/billing.integration.spec.ts`
**Coverage**:
- Authenticated user can access /console/billing
- Unauthenticated user redirected to /login
- User without permission sees 403 error
**Priority**: HIGH (blocks production deployment)

### Test 2: Stripe Webhook Handling
**File**: `src/app/services/payment/payment.integration.spec.ts`
**Coverage**:
- payment.succeeded event triggers notification
- payment.failed event triggers retry logic
- Invalid webhook signature rejected
**Priority**: HIGH (production risk)

---

## Reconciliation Metadata

**Cycle**: 1 (first reconciliation for this feature)
**Gaps by Type**:
- missing_navigation: 1
- outdated_criteria: 1
- incomplete_integration: 1
- missing_documentation: 1

**Severity Breakdown**:
- HIGH: 2 (50%)
- MEDIUM: 1 (25%)
- LOW: 1 (25%)

**Clarification Markers Added**: 1 (within limit of 3)

---

## Next Steps

1. **Resolve Outstanding Decisions**
   - Answer clarification questions
   - Update spec if needed

2. **Implement New Tasks**
   - Task 23: Add billing menu entry
   - Task 24: Integration test for navigation

3. **Run Integration Tests**
   - Validate navigation access control
   - Validate Stripe webhook handling

4. **Validate Hierarchy** (if using supplementary specs)
   ```bash
   /speckit.validate-hierarchy specs/000003-billing-console/
   ```

5. **Update PM Context**
   - Mark RECONCILING phase complete
   - Transition to VALIDATING phase (98-100%)

---

## Validation

Run strict validation to ensure all edits are properly integrated:

```bash
/speckit.validate-hierarchy specs/000003-billing-console/
```

Expected result: All checks pass (strict mode blocks workflow on errors)

---

**End of Sync Impact Report**
```

---

## State Management Integration

After reconciliation, update `.specify/memory/features/{featureId}/state.json`:

```json
{
  "feature_num": "000003",
  "phase": "RECONCILING",
  "progress": 96,
  "reconciliation_cycles": 1,
  "gaps_identified": [
    {"type": "missing_navigation", "status": "fixed", "severity": "HIGH"},
    {"type": "outdated_criteria", "status": "fixed", "severity": "MEDIUM"},
    {"type": "incomplete_integration", "status": "fixed", "severity": "HIGH"},
    {"type": "missing_documentation", "status": "pending", "severity": "LOW"}
  ],
  "sync_reports": [
    {
      "cycle": 1,
      "timestamp": "2025-11-01T14:30:00Z",
      "path": "specs/000003-billing-console/SYNC-IMPACT-REPORT-001.md",
      "gaps_addressed": 4,
      "files_modified": 4
    }
  ],
  "commands_executed": [
    {"cmd": "specify", "timestamp": "2025-10-15T10:00:00Z"},
    {"cmd": "clarify", "timestamp": "2025-10-15T11:00:00Z"},
    {"cmd": "plan", "timestamp": "2025-10-16T09:00:00Z"},
    {"cmd": "tasks", "timestamp": "2025-10-16T10:00:00Z"},
    {"cmd": "implement", "timestamp": "2025-10-20T16:00:00Z"},
    {"cmd": "reconcile", "timestamp": "2025-11-01T14:30:00Z"}
  ]
}
```

Update PM context in `.specify/memory/pm_context.md`:

```markdown
## Current Session

- **Branch**: feature/000003-billing-console
- **Feature**: billing-console
- **Phase**: RECONCILING
- **Progress**: 96%
- **Reconciliation Cycle**: 1

## Recent Activity

**2025-11-01**: Reconciliation cycle 1 complete
- Fixed 4 gaps (2 HIGH, 1 MEDIUM, 1 LOW)
- Modified: spec.md, UI-SPEC.md, plan.md, tasks.md
- Outstanding: 1 decision pending (API documentation scope)
- Next: Implement tasks 23-24, run integration tests
```

---

## Usage Examples

### Example 1: Simple Gap (Missing Navigation)

```bash
/speckit.reconcile

# Input gap report:
"Users cannot access billing console. No menu entry exists. Route /console/billing not configured."

# Questions asked (2):
1. Menu placement: main menu or sidebar?
2. Route path: /billing or /console/billing?

# Edits made (3 files):
- UI-SPEC.md: Added navigation section
- plan.md: Added routing configuration
- tasks.md: Appended navigation task

# Result:
Sync Impact Report generated at specs/000003-billing-console/SYNC-IMPACT-REPORT-001.md
```

### Example 2: Complex Gap (Multiple Integration Issues)

```bash
/speckit.reconcile

# Input gap report (formatted):
## Gaps
1. Stripe webhook handling missing
2. Notification service not wired
3. Error retry logic not implemented
4. Integration tests missing

# Questions asked (5 - maximum):
1. Which webhooks to handle? (payment.succeeded, payment.failed, etc.)
2. Notification targets? (email, SMS, push)
3. Retry strategy? (exponential backoff, circuit breaker)
4. Test coverage level? (happy path only, or full error scenarios)
5. Priority? (fix all now, or defer some)

# Edits made (4 files):
- spec.md: Added FR for webhook handling
- API-SPEC.md: Documented webhook endpoints
- plan.md: Added integration architecture section
- tasks.md: Appended 6 integration tasks

# Result:
Sync Impact Report with 6 new tasks, 3 integration tests required
```

---

## Best Practices

### 1. One Reconciliation Per Gap Discovery
Don't batch multiple testing cycles into one reconcile. Each reconciliation should be focused.

**Good**:
```
QA finds navigation gap → /speckit.reconcile → Fix
Next day, integration gap found → /speckit.reconcile → Fix
```

**Bad**:
```
Wait 2 weeks, collect all gaps → One massive reconcile
```

### 2. Append-Only Philosophy
Preserve existing spec structure. Add new sections, don't rewrite.

**Good**:
```markdown
## Acceptance Criteria

### AC-015: Export Data ✅ UPDATED
- Original criteria (preserved)
- **NEW**: Additional capability (appended)
```

**Bad**:
```markdown
## Acceptance Criteria

### AC-015: Export Data (COMPLETELY REWRITTEN)
- All new criteria, original deleted
```

### 3. Reference Supplementary Specs
If hierarchical specs exist, reconcile should update them.

**Check**:
```typescript
if (await fileExists(`${featureDir}/UI-SPEC.md`)) {
  // Gap is UI-related → Edit UI-SPEC.md
} else {
  // No supplementary spec → Edit spec.md
}
```

### 4. Integration Tests Are Mandatory
For any wiring or integration gap, ALWAYS add integration test task.

**Template**:
```markdown
#### Task N: Integration Test - {Gap Type}
- [ ] Test scenario: {scenario}
- [ ] Expected behavior: {behavior}
- [ ] Error handling: {error cases}
- [ ] **File**: {absolute path to test file}
- [ ] **Priority**: HIGH
```

### 5. Track Reconciliation Cycles
Multiple reconciliations indicate:
- Incomplete initial specification (learn for next feature)
- Complex domain (acceptable for large features)
- Process issue (if >3 cycles, investigate root cause)

**Learning**:
```jsonl
// .specify/memory/patterns_learned.jsonl
{"pattern_id":"nav_gap_001","pattern_name":"missing_navigation_route","trigger":"feature implements new UI but no route","questions":["Menu placement?","Route path?","Permissions required?"],"success_rate":1.0,"times_used":3,"effectiveness_score":9.0}
```

---

## Integration with Other Commands

### Before Reconcile
1. `/speckit.implement` → Implementation complete
2. Testing/QA → Gaps discovered

### During Reconcile
1. `/speckit.reconcile` → Parse gaps, ask questions, edit specs
2. `/speckit.validate-hierarchy` → Strict validation (auto-run after edits)

### After Reconcile
1. `/speckit.tasks` → Re-run to update task list (optional)
2. `/speckit.implement` → Implement new tasks
3. `/speckit.pm` → Update session context

---

## Error Handling

### Scenario 1: Too Many Clarification Markers
If reconcile would add >3 [NEEDS CLARIFICATION] markers:

```
❌ ERROR: Reconciliation would exceed clarification marker limit (3 max)

Current markers in spec: 2
New markers from reconcile: 2
Total would be: 4 (exceeds limit)

RESOLUTION:
1. Resolve existing clarifications first, OR
2. Split gap report into multiple reconciliations
```

### Scenario 2: Validation Failure (Strict Mode)
If edits fail validation:

```
❌ ERROR: Reconciliation edits failed validation

Validation errors:
- UI-SPEC.md: Missing parent reference in frontmatter
- plan.md: Does not reference UI-SPEC.md

BLOCKED: Workflow cannot continue until validation passes
Run: /speckit.validate-hierarchy --fix-auto (if available)
```

### Scenario 3: No Gaps in Report
```
⚠️ WARNING: Gap report appears empty or malformed

Expected format:
## Gap Report
### Gaps Identified
1. [Gap description]

Received: [empty or invalid format]

RESOLUTION: Provide structured gap report (see examples above)
```

---

## Exit Criteria

Reconciliation is complete when:
1. ✅ All gaps in report addressed (or deferred with decision)
2. ✅ Sync Impact Report generated
3. ✅ Validation passes (strict mode)
4. ✅ State updated (RECONCILING phase, cycle tracked)
5. ✅ PM context updated
6. ✅ New tasks appended to tasks.md
7. ✅ Integration tests identified

---

## Related Commands

- `/speckit.amend-technical` - Amend architecture decisions (not gap closure)
- `/speckit.validate-hierarchy` - Validate supplementary spec relationships
- `/speckit.pm` - Update session context after reconciliation
- `/speckit.next` - Get next recommended action post-reconcile

---

**Command Version**: 2.1.0
**Last Updated**: 2025-11-01
**Compatibility**: SpecKit v2.1+
