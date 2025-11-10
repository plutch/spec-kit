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

Unlike a full specification amendment, reconcile performs **surgical edits with version tracking** to maintain specification integrity while closing gaps. Edits can be append-only OR replaced with version history annotations to prevent strikethrough accumulation.

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

1. **Missing Navigation** (🟠 MAJOR)
   - Users cannot access billing console from main menu
   - No route configured in app routing
   - Impact: Feature inaccessible to end users (significant UX issue)

2. **Outdated Acceptance Criteria** (🟡 MEDIUM)
   - AC-015 says "Export to CSV", but we implemented "Export to CSV and PDF"
   - Spec doesn't reflect actual shipped behavior
   - Impact: Documentation mismatch

3. **Incomplete Integration** (🔴 CRITICAL)
   - No integration test for Stripe webhook handling
   - Wiring between payment service and notification service missing
   - Impact: Production risk - payment data could be lost or incorrect

4. **Missing Documentation** (🟡 MEDIUM)
   - API endpoints not documented in API-SPEC.md
   - Developer onboarding will be difficult
   - Impact: Maintainability and technical debt
```

---

## Execution Protocol

### Step 0: Gather Live Implementation Context (NEW v2.3)

**Purpose**: Gather real-world evidence of what was actually implemented vs. what was specified

**Use Bash Tool for Live Context**:

```bash
# 1. Show what was implemented (committed changes)
git diff --name-status origin/main...HEAD

# 2. Get actual file changes for the feature
git diff origin/main...HEAD -- src/

# 3. Check for uncommitted work
git status --short
```

**Why This Matters**:
- Gap reports may be incomplete or inaccurate
- Git diff shows ACTUAL implementation vs baseline
- Helps identify silent gaps not in user's gap report
- Provides evidence for surgical edits

**Extract from Git Diff**:
1. **New files added**: May need documentation, navigation, or integration
2. **Modified files**: Check if related specs were updated
3. **File patterns**: Frontend changes may need UI-SPEC.md updates
4. **Integration points**: New API calls suggest integration gaps

**Example Context Extraction**:
```bash
# Git shows these changes:
src/app/features/billing/billing.component.ts (new file)
src/app/app-routing.module.ts (modified)
src/app/services/payment.service.ts (modified)

# Inferred gaps to validate against user's gap report:
- New billing component → Need UI-SPEC.md update?
- Modified routing → Need route documentation?
- Modified payment service → Need API-SPEC.md update?
```

**Proceed to Step 1** with both user's gap report AND git diff context

---

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
  - 🔴 CRITICAL: Security vulnerabilities, data loss risks, compliance gaps, payment integrity issues, user-blocking defects
  - 🟠 MAJOR: Missing error handling for critical flows, significant UX issues, incomplete integration, production risks
  - 🟡 MEDIUM: Edge case gaps in non-critical flows, tech debt, documentation gaps
  - 🟢 LOW: Maintainability issues, optimization opportunities, minor polish
  - ℹ️ INFO: Observations, recommendations, best practices
```

**Output**: Categorized gap list with severity

---

### Step 2: Ask Clarifying Questions (≤5 Maximum)

**Constraint**: Maximum 5 questions total to keep reconciliation lightweight

**Question Types**:
1. **Scope Questions**: "Should navigation be in main menu or sidebar?"
2. **UX Questions**: "Should route be /billing or /console/billing?"
3. **Integration Questions**: "Which Stripe webhooks need handling (payment.succeeded, payment.failed)?"
4. **Priority Questions**: "Fix all gaps now or defer 🟡 MEDIUM severity to future sprint?"
5. **Impact Questions**: "Does this change require database migration?"

**Example**:
```markdown
## Clarifying Questions

1. **Navigation Placement** (🟠 MAJOR priority)
   - [ ] A) Main menu (top-level)
   - [ ] B) Settings submenu
   - [ ] C) Dashboard widget link

2. **API Documentation Scope** (🟡 MEDIUM priority)
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
4. **Quick Wins** (high-impact, low-effort remaining improvements)
5. **Reconciliation Metadata** (cycle number, timestamp, gap summary)

**Quick Wins Section**:

Purpose: Identify 3-5 remaining improvements that provide significant value with minimal effort after gap closure.

**Quick Win Criteria**:
- **High Impact**: Measurably improves quality, reduces risk, or improves maintainability
- **Low Effort**: Can be fixed in 1-2 days (simple additions, clarifications, refinements)
- **Actionable**: Specific change with clear location in spec or code
- **Independent**: Can be implemented without blocking on other changes

**Examples**:
- Adding missing error messages for edge cases (improves UX)
- Adding JSDoc comments to new API functions (improves maintainability)
- Adding validation for boundary conditions (prevents future bugs)
- Clarifying ambiguous acceptance criteria (improves testability)
- Adding performance monitoring for new endpoints (improves observability)

**Output Format**:
```markdown
## Quick Wins (High Impact, Low Effort)

### 1. **[Quick Win Title]**
- **Current State**: [What's missing/ambiguous/incomplete?]
- **Simple Fix**: [What's the specific change?]
- **Impact**: [Quantified benefit - prevents X, improves Y, reduces Z]
- **Effort**: [1-2 days] [Why it's quick]
- **Location**: [File path:line or Section: FR-XXX]

### 2. **[Quick Win Title]**
[Same format]

### 3. **[Quick Win Title]**
[Same format]

[Include up to 5 quick wins total if identified]
```

**Quick Wins Rationale**: After closing critical gaps, quick wins provide momentum-building improvements that prevent future reconciliation cycles. Completing quick wins reduces technical debt and improves long-term maintainability.

---

### Step 5: Reconciliation Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate surgical edits and gap closure integrity before marking reconciliation complete.

#### MANDATORY: The Four Reconciliation Questions

##### ❓ Question 1: Were all surgical edits append-only OR replaced with version history?

**Action Required**:
- Review git diff for modified files
- Verify edits use acceptable patterns:
  ✅ Append-only (new sections added)
  ✅ Replacement with version annotation (clean updates with version history)
  ✅ Strikethrough for minor inline edits (limited to <3 versions)
- Check [NEEDS CLARIFICATION] markers added, not removed

**Expected Evidence Format**:
```bash
Run: git diff specs/[FEATURE_ID]/
Output: Show additions (+) and modifications
Verification: Check for version annotations OR limited strikethrough
```

**Readiness Determination**:
```yaml
IF silent deletions found (no version annotation):
  → ❌ NOT ACCEPTABLE (audit trail lost)
IF excessive strikethrough (>3 versions shown):
  → ⚠️ NEEDS CONSOLIDATION (use version history instead)
IF append-only OR version history replacements:
  → ✅ ACCEPTABLE PATTERN
```

---

#### Version History Pattern (Recommended for Multi-Cycle Reconciliation)

**When to Use**:
- Updating existing sections/requirements across multiple reconciliation cycles
- Replacing outdated acceptance criteria or assumptions
- Consolidating accumulated strikethrough (>3 versions)

**Pattern**:
```markdown
### [Section Title] (v[N], Updated: YYYY-MM-DD)
[Current clean content without strikethrough]

**Version History**:
- v[N] (YYYY-MM-DD): [What changed in this version] (Reconcile Cycle [N])
- v[N-1] (YYYY-MM-DD): [Previous version changes] (Reconcile Cycle [N-1])
- v1 (YYYY-MM-DD): [Original content] (Original spec)
```

**Example**:
```markdown
### AC-015: Export Billing Data (v3, Updated: 2025-11-03)
Users can export billing data to CSV, PDF, Excel, and JSON formats.

**Version History**:
- v3 (2025-11-03): Added Excel and JSON export (Reconcile Cycle 3)
- v2 (2025-11-01): Added PDF export (Reconcile Cycle 1)
- v1 (2025-10-15): CSV export only (Original spec)
```

**Benefits**:
- ✅ Clean, readable specs (no accumulated strikethrough)
- ✅ Full audit trail (version history section)
- ✅ Clear current state (latest version at top)
- ✅ Maintainable over time (consolidates history)

---

##### ❓ Question 2: Are [NEEDS CLARIFICATION] markers ≤ 3?

**Action Required**:
- Count [NEEDS CLARIFICATION] markers in modified specs
- Verify constraint met (≤3 total per reconciliation cycle)

**Expected Evidence Format**:
```bash
Run: grep -c "\[NEEDS CLARIFICATION\]" specs/[FEATURE_ID]/*.md
Output: Count per file
Total: [X] markers (target: ≤3)
```

**Readiness Determination**:
```yaml
IF total_markers > 3:
  → ❌ TOO MANY CLARIFICATION MARKERS
IF total_markers ≤ 3:
  → ✅ MARKER CONSTRAINT MET
```

---

##### ❓ Question 3: Were integration tests added for critical gaps?

**Action Required**:
- Review Sync Impact Report for 🔴 CRITICAL gaps
- Verify integration tests added to tasks.md for wiring gaps
- Check test coverage for high-risk gap closures

**Expected Evidence Format**:
```yaml
Critical Gaps from Sync Impact Report:
- Gap 1 (🔴 CRITICAL): [Description]
  Integration Test: [Test ID added to tasks.md] ✅
- Gap 2 (🔴 CRITICAL): [Description]
  Integration Test: [Test ID added to tasks.md] ✅
- Gap 3 (🟠 MAJOR): [Description]
  Integration Test: [Optional] ⚠️
```

**Readiness Determination**:
```yaml
IF critical_gaps_without_tests > 0:
  → ❌ MISSING INTEGRATION TESTS
IF all_critical_gaps_have_tests:
  → ✅ TEST COVERAGE COMPLETE
```

---

##### ❓ Question 4: Are modified files syntactically valid?

**Required Evidence (ALL must be provided)**:
1. **File Existence Check (MANDATORY)**:
   ```bash
   Run: ls -la specs/[FEATURE_ID]/
   Output: Show modified files with timestamps
   ```

2. **Content Validation (MANDATORY)**:
   ```bash
   Run: grep -c "^---$" specs/[FEATURE_ID]/spec.md
   Output: 2 (YAML frontmatter delimiters)

   Run: grep -c "^# " specs/[FEATURE_ID]/spec.md
   Output: >0 (markdown headers present)
   ```

3. **Traceability (MANDATORY)**:
   ```yaml
   Modified Files:
   - spec.md: [X] surgical edits
   - UI-SPEC.md: [Y] surgical edits (if applicable)
   - plan.md: [Z] requirement additions
   - tasks.md: [W] new tasks appended

   Coverage: 100% of gaps from Gap Report addressed
   ```

**IF any evidence is MISSING**:
```
❌ CANNOT report reconciliation complete
→ Gather missing evidence first
→ Re-run this step with complete evidence
```

---

##### ❓ Question 5: Do surgical edits maintain constitutional compliance? (NEW v2.4)

**Purpose**: Prevent constitutional drift during reconciliation cycles. Verify surgical edits don't introduce violations.

**⚠️ SCOPE CLARIFICATION** (NEW - addresses Finding #2):
- **IF surgical edits modify SPECIFICATIONS** (spec.md, UI-SPEC.md, plan.md, tasks.md) → Apply **specification-level validation** (checks 1, 4 below)
- **IF surgical edits modify CODE** (src/\*, tests/\*) → Apply **code-level validation** (all checks 1-4 below)

Most reconciliations are **specs-only** (append-only edits to requirements). Code-level pattern scanning (check 3) only applies when implementation files are modified.

---

**Required Evidence (ALL must be provided)**:

1. **Complexity Analysis (MANDATORY - applies to BOTH specs and code)**:
   ```yaml
   Question: Do edits introduce new frameworks, projects, or abstractions?

   Evidence Format:
   - New Frameworks Added: [None / Framework name + justification]
   - New Projects/Services: [None / Project name + justification]
   - New Abstractions: [None / Abstraction type + problem solved]

   Example (specs-only):
   - New Frameworks Added: None (spec edits don't add frameworks)
   - New Projects/Services: None
   - New Abstractions: None (no architectural changes in specs)

   Example (code edits):
   - New Frameworks Added: None
   - New Projects/Services: None
   - New Abstractions: Repository pattern for data access (justified: improves testability)
   ```

2. **TDD Compliance Check (MANDATORY if new CODE functionality)**:
   ```yaml
   Question: If surgical edits add new functionality, were tests added BEFORE implementation?

   Evidence Format:
   IF specs-only edits (spec.md, plan.md, tasks.md):
     - Tests Added: N/A (specification updates only - no implementation changes)
     - TDD Compliance: N/A (applies to code, not specs)

   IF code edits (src/*, tests/*) with new functionality:
     - Tests Added: [Yes/No]
     - Test Location: [path/to/test.spec.ts]
     - Test Coverage: [N]% for new functionality
     - TDD Evidence: [Git commit showing test before impl]

   IF code edits (refactoring only, no new functionality):
     - Tests Added: N/A (refactoring - existing tests still pass)
     - TDD Compliance: ✅ PASS (no new functionality requiring tests)
   ```

3. **Prohibited Patterns Scan (MANDATORY if CODE edits OR tasks.md references patterns)**:
   ```yaml
   Question: Do surgical edits introduce prohibited patterns from constitution?

   Evidence Format:
   IF specs-only edits (spec.md, UI-SPEC.md, plan.md):
     - Pattern Scan: N/A (specs don't contain code patterns)
     - Task Description Check: [Scan tasks.md for pattern references]
       Example: "Task references fs.readFileSync" → ⚠️ WARNING

   IF code edits (src/*, tests/*):
     - Read `.specify/memory/constitution.md` (if exists)
     - Patterns Scanned: [N] patterns from constitution
     - Violations Found: [None / List violations with file:line]

     Example:
     - Patterns Scanned: 3 patterns (fs.readFileSync, db.query, global state)
     - Violations Found: None

     OR (if violation found):
     - Violations Found:
       - src/utils/file.ts:42 - Pattern "fs.readFileSync" detected
       - Remediation: Use fs.promises.readFile() instead

   IF constitution.md missing:
     - Pattern Scan: N/A (no constitution defined)
   ```

4. **Simplicity Preservation (MANDATORY - applies to BOTH specs and code)**:
   ```yaml
   Question: Do surgical edits maintain simplicity constraints?

   Evidence Format:
   - Append-Only Edits: [Yes/No]
   - Version History Used: [Yes/No/N/A]
   - Simplicity Maintained: [Yes - no new complexity / No - see justification]

   IF simplicity concerns:
     - Justification: [Why additional complexity necessary]
     - Mitigation: [How complexity is controlled]

   Example (specs-only):
   - Append-Only Edits: Yes (added acceptance criteria to spec.md)
   - Simplicity Maintained: Yes (no new complexity in specifications)

   Example (code edits):
   - Append-Only Edits: No (refactored existing function)
   - Simplicity Maintained: Yes (reduced cyclomatic complexity from 8 to 5)
   ```

**Constitutional Compliance Status** (determined after evidence gathered):
- ✅ **COMPLIANT**: No new complexity, TDD followed (if applicable), no prohibited patterns, simplicity maintained
- ⚠️ **CONDITIONAL**: New complexity justified, TDD evidence partial, simplicity near limits
- ❌ **NON-COMPLIANT**: Unjustified complexity, TDD skipped, prohibited patterns found, simplicity violated

**IF any evidence is MISSING**:
```
❌ CANNOT report constitutional compliance
→ Gather missing evidence first
→ Re-run this step with complete evidence
```

**IF ❌ NON-COMPLIANT**:
```
🔴 CONSTITUTIONAL VIOLATION in surgical edits
→ List violations found
→ STOP reconciliation
→ Remediate violations before proceeding
→ Re-run Step 5 with corrected edits
```

---

#### Hallucination Prevention (7 Red Flags for Reconciliation)

```yaml
Detect and BLOCK these patterns:

🚨 "Reconciliation complete" WITHOUT git diff evidence
   → Self-correction: "Wait, I need to show surgical edits made"

🚨 "All gaps closed" WITHOUT showing which specs were modified
   → Self-correction: "Let me verify files actually changed"

🚨 "Append-only edits" WITH silent deletions (no version annotation)
   → Self-correction: "Found deletions without version history - need to add version annotation or use strikethrough"

🚨 Claiming "gaps addressed" WITHOUT mapping to Gap Report
   → Self-correction: "Need to map each gap to specific edit"

🚨 Skipping [NEEDS CLARIFICATION] count
   → Self-correction: "I need to verify marker constraint (≤3)"

🚨 Hiding integration test gaps
   → Self-correction: "Must report missing tests for critical gaps"

🚨 "Ready for COMPLETED" statements WITHOUT validation
   → Self-correction: "Need to verify surgical edit integrity first"

IF detected: STOP → Gather evidence → Report honestly
```

---

#### Determine Status

##### ✅ **READY for VALIDATING or COMPLETED**

**Criteria (ALL must be met)**:
- [ ] All surgical edits follow acceptable patterns:
  - Append-only (new sections added), OR
  - Replacement with version history annotation, OR
  - Strikethrough for minor inline edits (<3 versions)
- [ ] [NEEDS CLARIFICATION] markers ≤ 3
- [ ] Integration tests added for all 🔴 CRITICAL gaps
- [ ] Modified files are syntactically valid (markdown, YAML frontmatter)

**IF ALL criteria met**:
→ Proceed to next step (present reconciliation completion report)

---

##### ⚠️ **NEEDS REVIEW** (with risks noted)

**Criteria**:
- Some 🟠 MAJOR gaps lack integration tests (acceptable if documented)
- [NEEDS CLARIFICATION] count = 3 (at limit, but acceptable)
- Minor syntax warnings (not blocking)

**IF criteria met**:
→ Present risks to user
→ Ask: "Minor gaps remain. Mark reconciliation complete or address gaps?"
→ Wait for user decision before proceeding

---

##### ❌ **NOT READY** (more reconciliation work needed)

**Criteria (ANY triggers NOT READY)**:
- Silent deletions (no version annotation or strikethrough)
- Excessive strikethrough (>3 versions without consolidation)
- [NEEDS CLARIFICATION] markers > 3 (constraint violated)
- 🔴 CRITICAL gaps lack integration tests (blocking)
- Modified files have syntax errors (invalid YAML, broken markdown)

**IF NOT READY**:
→ Present issues to user with evidence
→ Recommend specific actions:
  - Fix non-append-only edits
  - Reduce [NEEDS CLARIFICATION] markers
  - Add missing integration tests
  - Fix syntax errors
→ STOP (do not mark reconciliation complete)

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
- 1 🔴 CRITICAL (integration - payment wiring)
- 1 🟠 MAJOR (navigation - user access)
- 2 🟡 MEDIUM (acceptance criteria, documentation)

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

## Spec Metadata Integration (NEW v2.3)

After reconciliation, update `spec-metadata.json` in the spec directory:

```json
{
  "version": "2.3.0",
  "feature_name": "billing-console",
  "phase": "reconciliation",
  "approvals": {
    "reconciliation": {
      "generated": true,
      "approved": false,
      "timestamp": "2025-11-01T14:30:00Z"
    }
  },
  "metadata": {
    "reconciliation_cycles": 1,
    "gaps_addressed": 4,
    "files_modified": ["spec.md", "UI-SPEC.md", "plan.md", "tasks.md"],
    "sync_report_path": "SYNC-IMPACT-REPORT-001.md"
  }
}
```

**Dual State Tracking Note (v2.3 Transition)**:
- IF `state.json` exists: Update both `state.json` AND `spec-metadata.json`
- IF only `spec-metadata.json` exists: Update only `spec-metadata.json`
- This ensures compatibility during v2.3 migration period

**After Reconciliation Complete**:

1. **Present Sync Impact Report** to user
2. **Ask for approval**: "Review reconciliation. Approve to proceed? (yes/no)"
3. **IF approved**:
   ```json
   {
     "approvals": {
       "reconciliation": {
         "generated": true,
         "approved": true,
         "timestamp": "2025-11-01T14:35:00Z"
       }
     }
   }
   ```
4. **Recommend next command**:
   ```
   ✅ Reconciliation approved. Recommended next steps:

   1. Run `/speckit.status` to view updated workflow state
   2. Implement new tasks added during reconciliation
   3. Run integration tests identified in sync report
   4. Run `/speckit.validate-hierarchy` if using supplementary specs
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
1. `/speckit.status` → View updated workflow state (NEW v2.3)
2. `/speckit.tasks` → Re-run to update task list (optional)
3. `/speckit.implement` → Implement new tasks added during reconciliation
4. `/speckit.pm` → Update session context (if not using spec-metadata.json)

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

**Command Version**: 2.3.0
**Last Updated**: 2025-11-05
**Compatibility**: SpecKit v2.3+
**New in v2.3**: Live context gathering (git diff), spec-metadata.json integration, dual state tracking
