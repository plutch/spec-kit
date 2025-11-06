---
description: Display current phase, approval status, and workflow progress for the active feature specification
allowed-tools: Bash, Read
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

This command provides visibility into the current state of your Spec-Driven Development workflow:
- **Phase tracking**: Where are you in the workflow (specification → gap analysis → planning → tasks → implementation → reconciliation)?
- **Approval gates**: Which phases are complete and approved?
- **Workflow readiness**: Can you proceed to the next phase, or are prerequisites missing?
- **Quality metrics**: Overall quality score and critical issues count (if available)

## Execution Flow

### Phase 1: Locate Specification Files

**Step 1.1: Detect Specification Directory**
```text
# Default search paths (in priority order):
1. .specify/specs/current/    # Current spec location
2. .specify/specs/            # Spec directory root
3. ./                          # Current directory (fallback)

# Look for spec-metadata.json in these locations
```

**Step 1.2: Handle User-Provided Path**
If user provides a path argument:
```
/speckit.status .specify/specs/feature-name/
/speckit.status oauth-integration
```
→ Search in specified path first, then fallback to defaults

### Phase 2: Read Specification Metadata

**Step 2.1: Load spec-metadata.json**
```json
{
  "version": "2.3.0",
  "feature_name": "oauth-integration",
  "phase": "planning",
  "approvals": {
    "specification": {
      "generated": true,
      "approved": true,
      "timestamp": "2025-11-05T10:30:00Z"
    },
    "gap_analysis": {
      "generated": true,
      "approved": true,
      "timestamp": "2025-11-05T11:00:00Z"
    },
    "planning": {
      "generated": true,
      "approved": false,
      "timestamp": "2025-11-05T12:00:00Z"
    },
    "tasks": {
      "generated": false,
      "approved": false,
      "timestamp": null
    },
    "implementation": {
      "generated": false,
      "approved": false,
      "timestamp": null
    },
    "reconciliation": {
      "generated": false,
      "approved": false,
      "timestamp": null
    }
  },
  "metadata": {
    "created_at": "2025-11-05T09:00:00Z",
    "updated_at": "2025-11-05T12:00:00Z",
    "created_by": "speckit",
    "spec_version": "1.0",
    "risk_level": "MEDIUM",
    "overall_quality": 82
  }
}
```

**If spec-metadata.json not found:**
→ Check for legacy spec files (spec.md, plan-*.md, tasks-*.md)
→ Report: "No metadata file found - run `/speckit.specify` to create a new specification"

### Phase 3: Read Specification Files (Optional Context)

**Step 3.1: Load spec.md (if exists)**
- Extract feature name from title
- Check for Risk Assessment section
- Extract overall risk level (HIGH/MEDIUM/LOW)

**Step 3.2: Load quality.json (if exists)**
```json
{
  "overall_quality": 82,
  "critical_issues": 2,
  "major_issues": 5,
  "medium_issues": 8
}
```

**Step 3.3: Load gap-analysis.md (if exists)**
- Check if gap analysis has been performed
- Extract implementation readiness status

**Step 3.4: Load plan-*.md (if exists)**
- Count number of implementation phases
- Extract security review status
- Check test strategy completeness

**Step 3.5: Load tasks-*.md (if exists)**
- Count total tasks
- Count completed tasks (marked with `[x]`)
- Calculate completion percentage

### Phase 4: Calculate Workflow Status

**Phase Progression Model:**
```
specification → gap_analysis → planning → tasks → implementation → reconciliation → complete
     ↓              ↓              ↓          ↓          ↓                ↓
  Must approve  Must approve  Must approve  Optional  Must complete  Feature done
```

**4.1: Determine Current Phase**
- **specification**: Initial spec creation
- **gap_analysis**: Feasibility and codebase analysis
- **planning**: Implementation plan with phases
- **tasks**: Task breakdown (optional - can implement directly from plan)
- **implementation**: Actual code changes
- **reconciliation**: Post-implementation gap closure
- **complete**: Feature fully implemented and reconciled

**4.2: Check Phase Readiness**

**Can proceed to next phase if:**
- Current phase is `generated: true` AND `approved: true`
- Previous phases all approved
- No blocking quality issues (if quality checks enabled)

**Cannot proceed if:**
- Current phase not yet generated
- Current phase generated but not approved
- Previous phase not approved
- Critical quality issues exist (quality score < 60 or critical_issues > 0)

### Phase 5: Display Status Report

**Format: Visual Dashboard with Status Indicators**

```markdown
# Spec-Kit Workflow Status

## Feature: [Feature Name]
**Spec Version**: [spec_version] | **Risk Level**: 🔴 HIGH / 🟠 MEDIUM / 🟢 LOW
**Created**: [created_at] | **Updated**: [updated_at]

---

## Phase Progression

✅ **Specification** (specification)
   - Generated: ✓ | Approved: ✓
   - Timestamp: 2025-11-05 10:30:00
   - Next: Run `/speckit.validate-gap` or `/speckit.clarify` if needed

✅ **Gap Analysis** (gap_analysis)
   - Generated: ✓ | Approved: ✓
   - Timestamp: 2025-11-05 11:00:00
   - Next: Run `/speckit.plan`

⏳ **Planning** (planning) ← **CURRENT PHASE**
   - Generated: ✓ | Approved: ✗
   - Timestamp: 2025-11-05 12:00:00
   - **Action Required**: Review plan-1.md and approve to proceed
   - Next: Run `/speckit.tasks` or `/speckit.implement`

⬜ **Tasks** (tasks)
   - Generated: ✗ | Approved: ✗
   - Timestamp: Not started
   - Blocked: Planning not approved yet

⬜ **Implementation** (implementation)
   - Generated: ✗ | Approved: ✗
   - Timestamp: Not started
   - Blocked: Planning not approved yet

⬜ **Reconciliation** (reconciliation)
   - Generated: ✗ | Approved: ✗
   - Timestamp: Not started
   - Blocked: Implementation not complete

---

## Quality Metrics

**Overall Quality**: 82/100 🟢
- Critical Issues: 2 🔴
- Major Issues: 5 🟠
- Medium Issues: 8 🟡

**Recommendation**: Address 2 critical issues before proceeding to implementation.

---

## Workflow Readiness

**Can Proceed to Next Phase?** 🔴 NO
**Reason**: Planning phase not approved yet

**Required Actions:**
1. Review [plan-1.md](.specify/specs/oauth-integration/plan-1.md)
2. Approve planning phase: `echo "planning_approved" > .specify/specs/oauth-integration/approved.txt` (manual) or use `-y` flag in next command
3. Run `/speckit.tasks` (optional) or `/speckit.implement` (direct)

**Suggested Next Command:**
```
/speckit.tasks
```

---

## Files Detected

| File | Status | Size | Last Modified |
|------|--------|------|---------------|
| [spec.md](spec.md) | ✓ Generated | 15.2 KB | 2025-11-05 10:30 |
| [gap-analysis.md](gap-analysis.md) | ✓ Generated | 8.5 KB | 2025-11-05 11:00 |
| [plan-1.md](plan-1.md) | ✓ Generated | 22.1 KB | 2025-11-05 12:00 |
| [tasks-1.md](tasks-1.md) | ⬜ Not created | - | - |
| [quality.json](quality.json) | ✓ Generated | 1.2 KB | 2025-11-05 12:00 |
| [spec-metadata.json](spec-metadata.json) | ✓ Generated | 0.8 KB | 2025-11-05 12:00 |

---

## Memory Files Active

**Always Loaded**:
- ✓ [constitution.md](../../memory/constitution.md) (24.5 KB)

**Conditionally Loaded**:
- ✓ [api-standards.md](../../memory/api-standards.md) (18.3 KB) - Loaded when in API context
- ✓ [testing-approach.md](../../memory/testing-approach.md) (20.1 KB) - Loaded when in test context

**Manual Loading**:
- ⬜ [deployment-runbook.md](../../memory/deployment-runbook.md) - Use `@deployment-runbook.md` to load

---

## Workflow Commands Reference

| Command | Purpose | Can Run Now? |
|---------|---------|--------------|
| `/speckit.specify` | Create or update specification | ✓ Yes (re-generate) |
| `/speckit.clarify` | Clarify ambiguities in spec | ✓ Yes |
| `/speckit.validate-gap` | Analyze implementation feasibility | ✓ Yes (re-run) |
| `/speckit.plan` | Generate implementation plan | ✓ Yes (re-generate) |
| `/speckit.tasks` | Break down plan into tasks | 🔴 No (planning not approved) |
| `/speckit.implement` | Execute implementation | 🔴 No (planning not approved) |
| `/speckit.reconcile` | Post-implementation gap closure | 🔴 No (implementation not complete) |
| `/speckit.analyze` | Quality analysis | ✓ Yes |
| `/speckit.next` | Smart workflow navigator | ✓ Yes |
```

### Phase 6: Additional Status Indicators

**If tasks.md exists, add Task Progress:**
```markdown
## Task Progress

**Total Tasks**: 25
**Completed**: 8 (32%)
**In Progress**: 3
**Blocked**: 1
**Pending**: 13

**Recent Completions**:
- [x] Phase 1.1: Create OAuth provider configuration
- [x] Phase 1.2: Set up OAuth endpoints
- [x] Phase 2.1: Implement token validation

**Current Tasks**:
- [ ] Phase 2.2: Add token refresh logic (IN PROGRESS)
- [ ] Phase 2.3: Implement session management (IN PROGRESS)
- [ ] Phase 3.1: Add OAuth UI components (IN PROGRESS)

**Blocked Tasks**:
- [ ] Phase 4.1: Security audit (BLOCKED: Waiting for implementation completion)
```

**If reconciliation.md exists, add Reconciliation Status:**
```markdown
## Reconciliation Status

**Gaps Identified**: 12
**Gaps Closed**: 7 (58%)
**Gaps Remaining**: 5

**Critical Gaps**:
1. 🔴 OAuth token refresh not implemented
2. 🔴 PKCE flow missing for mobile clients

**Recommendations**:
- Run `/speckit.reconcile` to create reconciliation plan
- Address critical gaps before marking feature complete
```

### Phase 7: Smart Recommendations

**Based on current state, provide contextual recommendations:**

**If specification not approved:**
→ "Run `/speckit.clarify` to address ambiguities before proceeding"

**If quality score < 60:**
→ "Quality score too low - run `/speckit.analyze` to identify and fix issues"

**If critical issues > 0:**
→ "Critical issues detected - address before implementation"

**If risk level is HIGH and no gap analysis:**
→ "HIGH risk feature requires gap analysis - run `/speckit.validate-gap`"

**If planning approved but no tasks:**
→ "You can skip tasks and run `/speckit.implement` directly, or run `/speckit.tasks` for detailed breakdown"

**If implementation complete but no reconciliation:**
→ "Implementation done - run `/speckit.reconcile` to verify all requirements met"

### Phase 8: Status Review Gate

**Evidence-Based Self-Check** (Mandatory):

❓ **"Was spec-metadata.json found and parsed?"**
- Show file path and parse status
- Report: Metadata version and feature name

❓ **"Were all phase statuses evaluated?"**
- Show phase progression with approval status
- Report: Current phase and blocking reasons (if any)

❓ **"Were workflow recommendations provided?"**
- Show suggested next command
- Report: Blocking issues (if any)

---

## Usage Examples

**Check current feature status:**
```
/speckit.status
```

**Check status for specific feature:**
```
/speckit.status oauth-integration
/speckit.status .specify/specs/user-dashboard/
```

**Check status with file listing:**
```
/speckit.status --verbose
```

---

**Last Updated**: 2025-11-05
**Version**: 1.0.0
