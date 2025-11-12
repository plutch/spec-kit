# /speckit.epic.status - Epic Progress Dashboard

**Version**: 2.8.2
**Category**: Epic Management
**Purpose**: Monitor epic progress, wave completion, integration point status, and quality metrics across all features

---

## Usage

```bash
/speckit.epic.status <epic-name> [flags]
```

**Flags**:
- `--features`: Show detailed feature status table
- `--integration`: Show detailed integration point status
- `--quality`: Show detailed quality metrics per feature
- `--timeline`: Show timeline visualization with actual vs planned dates
- `--all`: Show all sections (equivalent to --features --integration --quality --timeline)

**Examples**:
```bash
/speckit.epic.status multi-tenant-saas
/speckit.epic.status payment-processing --features
/speckit.epic.status analytics-dashboard --all
```

---

## What This Command Does

This command provides a comprehensive dashboard of epic progress:

1. **Epic Overview**: Name, risk, timeline, overall progress
2. **Wave Progress**: Visual progress bars for each parallelization wave
3. **Feature Status**: Phase progression for all features (specification → complete)
4. **Integration Point Status**: Upstream/downstream dependency health (🟢/🟡/🔴)
5. **Quality Metrics**: Aggregated quality scores, critical issues, test coverage
6. **Critical Path Status**: Progress on bottleneck features
7. **Recommendations**: Smart next steps based on current state

**When to use this command**:
- Daily standups: Quick epic health check
- Sprint planning: Understand what can be started based on dependencies
- Stakeholder updates: Progress reporting
- Risk assessment: Identify blocked or at-risk features

**When NOT to use this command**:
- For individual feature status → Use `/speckit.status` instead
- For epic creation → Use `/speckit.epic.create`

---

## Prerequisites

- [ ] Epic created via `/speckit.epic.create` (epic-status.json exists)
- [ ] Optional: Features decomposed via `/speckit.epic.decompose` (for detailed status)

---

## Workflow Overview

```
Step 1: Context Loading (v2.7)
    ↓
Step 2: Epic Discovery & Data Collection
    ↓
Step 3: Epic Overview Dashboard
    ↓
Step 4: Wave Progress Visualization
    ↓
Step 5: Feature Status Table (--features)
    ↓
Step 6: Integration Point Status (--integration)
    ↓
Step 7: Quality Metrics Aggregation (--quality)
    ↓
Step 8: Timeline Visualization (--timeline)
    ↓
Step 9: Smart Recommendations
```

---

## Step 1: Context Loading (v2.7 - Context Optimization)

**Current Phase**: `epic_monitoring`

**Load Memory Files**:
- **Strategic**: constitution.md (FULL) - for constitutional compliance checking
- **Strategic**: epic-planning-principles.md (if exists)
- **Tactical**: SKIP (not needed during monitoring)
- **Reference**: SKIP (not needed during monitoring)

**Token Budget**: 10-15KB (strategic only, minimal for read-only monitoring)

**Action**:
1. Read `.specify/config.yml` (if exists)
2. Load memory files with `inclusion_mode: always` and `load_phases: epic_monitoring`
3. Filter to strategic sections only

**Expected Token Savings**: ~70% (25-35KB → 5-8KB)

---

## Step 2: Epic Discovery & Data Collection

**Purpose**: Gather all epic and feature data for dashboard

**Actions**:

1. **Discover Epic Directory**:
   - Check `.specify/epics/<epic-name>/` exists
   - **If not found**: Error "Epic '<epic-name>' not found. Run /speckit.epic.create first."

2. **Load Epic Files**:
   - Read `epic-status.json` (required) - PRIMARY source of truth
   - Read `epic-spec.md` (required) - Business goals, success criteria
   - Read `dependency-map.md` (required) - Critical path, integration points
   - Read `epic-architecture.md` (optional) - ADRs, constitutional compliance

3. **Load Feature Metadata** (for each feature in epic-status.json):
   - Read `.specify/features/[feature-id]-[slug]/spec-metadata.json` (if exists)
   - Read `.specify/features/[feature-id]-[slug]/quality.json` (if exists, from /speckit.analyze)
   - Extract: phase, approvals, overall_quality, critical_issues_count, risk_level

4. **Calculate Derived Metrics**:
   - Overall epic progress (% features complete)
   - Wave completion (% features in each wave complete)
   - Integration point health (% on track)
   - Quality score (average across features)
   - Critical path progress (% critical path features complete)

**Output**: Comprehensive epic state for dashboard rendering

---

## Step 3: Epic Overview Dashboard

**Purpose**: High-level epic health summary

**Output Format**:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   EPIC STATUS: EPIC-001 Multi-Tenant SaaS Platform
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 OVERVIEW
├─ Status: 🟢 ACTIVE
├─ Risk Level: 🔴 HIGH (Score: 10/12)
├─ Progress: ████████░░░░░░░░░░░░ 40% (2/5 features complete)
├─ Created: 2025-01-15
├─ Target: Q2 2025
└─ Days Remaining: 45 days

⏱️  TIMELINE
├─ Sequential Estimate: 10 weeks
├─ Parallelized Estimate: 6 weeks
├─ Time Savings: 40% (4 weeks saved)
├─ Elapsed: 3 weeks
└─ Estimated Completion: 2025-03-21

🎯 CRITICAL PATH
├─ Path: 001 → 003 → 005
├─ Duration: 7 weeks (3 + 3 + 1)
├─ Progress: ██████░░░░░░ 43% (001 complete, 003 in progress)
└─ Bottleneck: Feature 003 (Tenant Billing) - Week 4-6

📋 APPROVALS
├─ Planning: ✅ Approved (2025-01-18 by Alice)
├─ Architecture: ✅ Approved (2025-01-19 by Bob)
├─ Security: ⏳ In Progress
└─ Constitutional: ✅ Approved (2025-01-19 by Bob)

🔗 Quick Links
├─ Epic Spec: .specify/epics/multi-tenant-saas/epic-spec.md
├─ Architecture: .specify/epics/multi-tenant-saas/epic-architecture.md
└─ Dependencies: .specify/epics/multi-tenant-saas/dependency-map.md
```

**Data Sources**:
- epic-status.json: status, risk_level, risk_score, features count, approvals
- epic-spec.md: target_date, business goals
- dependency-map.md: critical_path, timeline estimates

---

## Step 4: Wave Progress Visualization

**Purpose**: Show parallelization wave completion

**Output Format**:

```
🌊 WAVE PROGRESS

Wave 1: Foundation (Week 1-3)
├─ Status: ✅ COMPLETE
├─ Duration: 3 weeks
├─ Features: 001 (Tenant Isolation)
├─ Progress: ████████████████████ 100% (1/1 complete)
└─ Actual Completion: 2025-02-10 (3 days late)

Wave 2: Core Features (Week 4-5) - ACTIVE WAVE
├─ Status: 🟡 IN PROGRESS
├─ Duration: 3 weeks (longest: Feature 003)
├─ Features: 002 (Dashboard), 003 (Billing), 004 (Admin)
├─ Progress: █████░░░░░░░░░░░░░░░ 33% (1/3 complete)
├─ Completion Estimate: 2025-03-14
└─ Dependencies: Blocked until Wave 1 complete ✅

    002 - Tenant Dashboard       [██████████░░░░░░░░░░] 50% (Implementation)
    003 - Tenant Billing          [████░░░░░░░░░░░░░░░░] 20% (Planning)
    004 - Tenant Admin            [████████████████████] 100% (Complete) ✅

Wave 3: Integration (Week 6)
├─ Status: ⬜ NOT STARTED
├─ Duration: 1 week
├─ Features: 005 (Tenant Onboarding)
├─ Progress: ░░░░░░░░░░░░░░░░░░░░ 0% (0/1 complete)
└─ Dependencies: Waiting for Wave 2 (002, 003, 004) ⏳

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OVERALL WAVE COMPLETION: ████████░░░░░░░░░░░░░░░░ 47% (1.33/3 waves)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Data Sources**:
- epic-status.json → timeline.waves[]
- Calculate progress per wave based on feature status

**Progress Calculation**:
- Wave progress = (completed features in wave) / (total features in wave) × 100%
- Overall wave completion = (completed waves + partial wave progress) / (total waves)

---

## Step 5: Feature Status Table (--features flag)

**Purpose**: Detailed feature-by-feature workflow progression

**Output Format**:

```
📋 FEATURE STATUS (5 features)

| ID  | Feature Name       | Risk | Phase           | Progress | Quality   | Issues | Status      |
|-----|-----------------------|------|-----------------|----------|-----------|--------|-------------|
| 001 | Tenant Isolation      | 🔴   | complete        | 100%     | EXCELLENT | 0      | ✅ COMPLETE |
| 002 | Tenant Dashboard      | 🟠   | implementation  | 50%      | GOOD      | 2      | 🟢 ON TRACK |
| 003 | Tenant Billing        | 🔴   | planning        | 20%      | -         | -      | 🟡 AT RISK  |
| 004 | Tenant Admin          | 🟠   | complete        | 100%     | GOOD      | 1      | ✅ COMPLETE |
| 005 | Tenant Onboarding     | 🟡   | specification   | 10%      | -         | -      | ⬜ BLOCKED  |

Legend:
├─ Phase: specification → gap_analysis → planning → tasks → implementation → reconciliation → complete
├─ Progress: % through current phase (auto-calculated from approvals)
├─ Quality: From /speckit.analyze (EXCELLENT/GOOD/ACCEPTABLE/POOR)
├─ Issues: Critical issues count from quality.json
└─ Status: ✅ COMPLETE | 🟢 ON TRACK | 🟡 AT RISK | 🔴 BLOCKED | ⬜ NOT STARTED

Detailed Phase Progression (Feature 002 - Tenant Dashboard):
├─ ✅ Specification (approved 2025-02-11)
├─ ✅ Gap Analysis (approved 2025-02-12)
├─ ✅ Planning (approved 2025-02-13)
├─ ⏳ Implementation (in progress since 2025-02-14)
├─ ⬜ Reconciliation (not started)
└─ ⬜ Complete (not started)
```

**Data Sources**:
- epic-status.json → features[]
- spec-metadata.json (per feature) → phase, approvals
- quality.json (per feature) → overall_quality, critical_issues_count

**Progress Calculation**:
- specification: 10%
- gap_analysis: 20%
- planning: 30%
- tasks: 40%
- implementation: 70% (if in progress), 80% (if reconciliation pending)
- reconciliation: 90%
- complete: 100%

**Status Logic**:
- ✅ COMPLETE: phase === "complete"
- 🟢 ON TRACK: phase progressing, no blockers, dependencies met
- 🟡 AT RISK: Behind schedule OR critical_issues > 3 OR dependencies delayed
- 🔴 BLOCKED: Dependencies not met OR approval blocked
- ⬜ NOT STARTED: phase === "specification" AND not generated

---

## Step 6: Integration Point Status (--integration flag)

**Purpose**: Monitor upstream/downstream dependency health

**Output Format**:

```
🤝 INTEGRATION POINTS (6 contracts)

Upstream Dependencies (What features are waiting for):
┌────────────────────────────────────────────────────────────────────────────┐
│ To Feature: 002 (Tenant Dashboard)                                         │
├────────────────────────────────────────────────────────────────────────────┤
│ From: 001 (Tenant Isolation) → ITenantService interface                   │
│ Status: 🟢 DELIVERED (2025-02-10, on time)                                │
│ Contract: getTenantContext(req) helper, tenant CRUD operations            │
└────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────┐
│ To Feature: 003 (Tenant Billing)                                          │
├────────────────────────────────────────────────────────────────────────────┤
│ From: 001 (Tenant Isolation) → tenant_id schema + RLS policies            │
│ Status: 🟢 DELIVERED (2025-02-10, on time)                                │
│ Contract: All tables have tenant_id column, RLS policies active           │
└────────────────────────────────────────────────────────────────────────────┘

Downstream Deliverables (What features must deliver):
┌────────────────────────────────────────────────────────────────────────────┐
│ From Feature: 002 (Tenant Dashboard)                                      │
├────────────────────────────────────────────────────────────────────────────┤
│ To: 005 (Tenant Onboarding) → Tenant dashboard UI component               │
│ Status: 🟡 AT RISK (Due: 2025-03-07, 5 days away, implementation 50%)    │
│ Risk: Feature 002 in implementation, may miss deadline                     │
└────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────┐
│ From Feature: 003 (Tenant Billing)                                        │
├────────────────────────────────────────────────────────────────────────────┤
│ To: 005 (Tenant Onboarding) → Subscription creation endpoint              │
│ Status: 🔴 BLOCKED (Due: 2025-03-14, Feature 003 only 20% complete)      │
│ Risk: HIGH - Feature 003 in planning, unlikely to meet deadline           │
└────────────────────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
INTEGRATION HEALTH: ████████████░░░░░░░░ 67% (4/6 on track)
  🟢 ON TRACK: 4 contracts  |  🟡 AT RISK: 1 contract  |  🔴 BLOCKED: 1 contract
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Data Sources**:
- dependency-map.md → Integration Points & Handoffs table
- epic-status.json → dependencies.integration_points[] (updated status)
- Calculate status based on:
  - Delivery date vs current date
  - Provider feature progress
  - Consumer feature needs

**Status Logic**:
- 🟢 ON TRACK: Delivered OR (due >7 days away AND provider >70% complete)
- 🟡 AT RISK: Due <7 days away AND provider <70% complete
- 🔴 BLOCKED: Due date passed AND not delivered OR provider <30% complete

---

## Step 7: Quality Metrics Aggregation (--quality flag)

**Purpose**: Aggregate quality scores across all features

**Output Format**:

```
📊 QUALITY METRICS (Aggregated across 5 features)

Overall Epic Quality: ████████████████░░░░ GOOD (8.2/10)
├─ Features Analyzed: 2/5 (40%)
├─ Excellent: 1 feature (001)
├─ Good: 1 feature (002, 004)
├─ Acceptable: 0 features
└─ Poor: 0 features

Critical Issues: 🟡 3 total across all features
├─ Feature 002: 2 critical issues (🟠 Security: Input validation missing)
├─ Feature 004: 1 critical issue (🟡 Performance: N+1 query detected)
└─ Recommendation: Address before epic completion

Test Coverage: ████████████████░░░░ 87.5% average
├─ Feature 001: 92.3% ✅
├─ Feature 002: 85.1% ✅
├─ Feature 004: 85.0% ✅
├─ Target: ≥85% (epic-wide)
└─ Status: ✅ MEETS TARGET

Security Review:
├─ Epic-Level Security Review: 🟡 IN PROGRESS
├─ Feature-Level Reviews: 2/5 complete
├─ OWASP Top 10 Validation: ✅ PASSED (Features 001, 004)
└─ Blocked Features: Feature 002 (security issues pending)

Per-Feature Quality Breakdown:

Feature 001 - Tenant Isolation (✅ Complete):
├─ Overall Quality: EXCELLENT (9.5/10)
├─ Critical Issues: 0
├─ Test Coverage: 92.3%
├─ Security: ✅ PASSED
└─ Status: Production-ready

Feature 002 - Tenant Dashboard (🟡 Implementation):
├─ Overall Quality: GOOD (8.0/10)
├─ Critical Issues: 2 (🟠 Input validation, 🟡 Error handling)
├─ Test Coverage: 85.1%
├─ Security: ⚠️ NEEDS REVIEW
└─ Status: Address critical issues before completion

Feature 004 - Tenant Admin (✅ Complete):
├─ Overall Quality: GOOD (7.1/10)
├─ Critical Issues: 1 (🟡 N+1 query performance)
├─ Test Coverage: 85.0%
├─ Security: ✅ PASSED
└─ Status: Production-ready (performance optimization recommended)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
QUALITY GATE: ⚠️ NEEDS REVIEW
  ├─ 3 critical issues pending resolution
  ├─ 3 features not yet analyzed (run /speckit.analyze for 003, 005)
  └─ Recommendation: Address issues in 002 before Wave 2 completion
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Data Sources**:
- quality.json (per feature) → overall_quality, critical_issues, test_coverage
- epic-status.json → quality.overall_quality, quality.critical_issues_count, quality.test_coverage_average

**Calculation**:
- Average quality: SUM(feature quality scores) / COUNT(analyzed features)
- Critical issues: SUM(critical_issues_count across all features)
- Test coverage: AVG(coverage_percentage across all features)

---

## Step 8: Timeline Visualization (--timeline flag)

**Purpose**: Show actual vs planned timeline with milestone tracking

**Output Format**:

```
📅 TIMELINE (Actual vs Planned)

Milestones:
├─ Foundation Complete (Feature 001)
│   ├─ Target: 2025-02-07 (Week 3)
│   ├─ Actual: 2025-02-10 (Week 3)
│   └─ Status: ✅ COMPLETE (3 days late)
│
├─ Core Features Complete (Features 002-004)
│   ├─ Target: 2025-03-14 (Week 5)
│   ├─ Estimate: 2025-03-18 (Week 6)
│   └─ Status: 🟡 AT RISK (4 days delay estimated, Feature 003 behind)
│
└─ Epic Complete (Feature 005)
    ├─ Target: 2025-03-21 (Week 6)
    ├─ Estimate: 2025-03-28 (Week 7)
    └─ Status: 🟡 AT RISK (7 days delay estimated, dependencies at risk)

Gantt Chart (Actual):
Week 1-3 [████████████████████] Wave 1: Foundation (001) ✅
Week 4-5 [██████████░░░░░░░░░░] Wave 2: Core (002, 003, 004) 🟡 (50% complete)
Week 6   [░░░░░░░░░░░░░░░░░░░░] Wave 3: Integration (005) ⬜ (blocked)

Timeline Health:
├─ Original Estimate: 6 weeks (parallelized)
├─ Current Estimate: 7 weeks (+1 week delay)
├─ Elapsed: 3 weeks
├─ Remaining: 4 weeks
└─ Delay Factors: Feature 003 planning slower than expected, Feature 002 security issues

Recommendations:
├─ 1. Expedite Feature 003 planning (currently in planning, needs to start implementation)
├─ 2. Resolve Feature 002 security issues to unblock Wave 2 completion
└─ 3. Consider adding resources to Feature 003 (critical path bottleneck)
```

**Data Sources**:
- epic-status.json → timeline.milestones[], timeline.waves[]
- Calculate delays based on actual completion dates vs target dates

---

## Step 9: Smart Recommendations

**Purpose**: Provide actionable next steps based on current epic state

**Output Format**:

```
💡 SMART RECOMMENDATIONS

Priority Actions:
├─ 🔴 CRITICAL (Do Now):
│   ├─ 1. Resolve 2 critical security issues in Feature 002 (Tenant Dashboard)
│   │   └─ Run /speckit.analyze 002-tenant-dashboard to generate remediation plan
│   └─ 2. Complete Feature 003 planning phase (blocking Wave 2 completion)
│       └─ Run /speckit.plan 003-tenant-billing to generate implementation plan
│
├─ 🟠 HIGH (This Week):
│   ├─ 3. Conduct epic-level security review (currently in progress)
│   │   └─ Review epic-architecture.md for OWASP Top 10 compliance
│   └─ 4. Monitor Feature 002 implementation progress (at risk of missing deadline)
│       └─ Daily standup check-in on dashboard UI component delivery
│
└─ 🟡 MEDIUM (Next Week):
    ├─ 5. Run /speckit.analyze for Features 003, 005 (not yet analyzed)
    ├─ 6. Optimize Feature 004 N+1 query performance issue (post-launch OK)
    └─ 7. Prepare Feature 005 specification (dependencies resolving soon)

Ready to Start:
├─ Feature 002: Continue implementation (50% complete, dependencies met)
└─ Feature 004: ✅ Complete (can deploy to staging)

Blocked Features:
├─ Feature 003: Waiting for planning completion (in progress)
├─ Feature 005: Blocked by Features 002, 003, 004 (Wave 2 not complete)
└─ Estimated Unblock: 2025-03-18 (when Wave 2 completes)

Next Commands:
├─ /speckit.analyze 002-tenant-dashboard --deep  # Address critical issues
├─ /speckit.plan 003-tenant-billing              # Complete planning phase
├─ /speckit.implement 002-tenant-dashboard       # Continue implementation
└─ /speckit.epic.status multi-tenant-saas --all  # Re-check status tomorrow
```

**Recommendation Logic**:

1. **Critical (🔴)**: Blockers, critical issues, missed deadlines
   - Unresolved critical issues in any feature
   - Features blocking critical path
   - Missed integration point deadlines

2. **High (🟠)**: At-risk features, upcoming deadlines
   - Integration points due within 7 days
   - Features >70% complete but not finished
   - Epic-level reviews pending

3. **Medium (🟡)**: Optimizations, future prep
   - Non-blocking quality improvements
   - Features not yet analyzed
   - Preparation for next wave

**Ready to Start**: Features with dependencies met, approval gates passed

**Blocked Features**: Features waiting on dependencies or approvals

---

## Step 10: Epic Status Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate epic status data was actually loaded and analyzed to prevent reporting "status shown" without reading epic files.

**Hallucination Prevention (7 Red Flags for Epic Status)**:

```yaml
Detect and BLOCK these patterns:

🚨 "Epic status shown" WITHOUT reading epic-status.json
   → Self-correction: "Wait, I need to actually load and parse epic-status.json file"

🚨 "Wave progress displayed" WITHOUT showing actual completion percentages
   → Self-correction: "Let me calculate wave completion from real feature statuses"

🚨 "Integration points analyzed" WITHOUT showing contract details
   → Self-correction: "I need to show actual integration contract statuses from epic data"

🚨 "Quality metrics aggregated" WITHOUT showing actual issue counts
   → Self-correction: "Let me count critical/major issues from quality.json files"

🚨 "Timeline shown" WITHOUT actual dates from epic-status.json
   → Self-correction: "I must read timeline.milestones[] and show real target vs actual dates"

🚨 "Recommendations generated" WITHOUT analyzing epic state
   → Self-correction: "I need to analyze blockers, delays, quality issues before recommending"

🚨 "Everything on track" statements WITHOUT evidence
   → Self-correction: "Need to verify with actual epic data, not assume"

IF detected: STOP → Load epic files → Analyze data → Report honestly
```

**Required Evidence (ALL must be provided)**:

1. **Epic Status Data Loaded**:
   - Proof: Quote epic ID, epic name, total features from epic-status.json
   - Example: "EPIC-001: Multi-Tenant SaaS Platform, 5 features"
   - Verify file was actually read (not hallucinated)

2. **Wave Progress Calculated**:
   - Proof: Show completion percentages per wave
   - Example: "Wave 1: 100% (1/1 complete), Wave 2: 50% (2/4 in progress)"
   - Must show actual numbers, not generic statements

3. **Feature Status Aggregated**:
   - Proof: Feature count by phase
   - Example: "COMPLETED: 1, IMPLEMENTING: 2, PLANNING: 1, SPECIFYING: 1"
   - Verify counts match epic data

4. **Smart Recommendations Based on Data**:
   - Proof: Recommendations tied to specific features or blockers
   - Example: "Resolve critical issues in Feature 002" (references actual feature ID + issue type)
   - NOT generic "review epic" suggestions

**Evidence-Based Validation Questions**:

1. **"Did I READ epic-status.json successfully?"**
   - Required Evidence: Quoted epic metadata (ID, name, feature count)
   - ✅ PASS: Epic data quoted from file
   - ❌ FAIL: Generic epic description without file data

2. **"Can I show wave completion percentages?"**
   - Required Evidence: Wave progress numbers (e.g., "Wave 1: 100%, Wave 2: 50%")
   - ✅ PASS: Completion percentages with actual feature counts
   - ❌ FAIL: No wave data or generic "in progress" statements

3. **"Can I count features by phase?"**
   - Required Evidence: Feature distribution (COMPLETED, IMPLEMENTING, etc.)
   - ✅ PASS: Feature counts by phase from epic data
   - ❌ FAIL: No phase distribution or hallucinated numbers

4. **"Are recommendations tied to specific features/issues?"**
   - Required Evidence: Recommendations reference actual feature IDs or quality issues
   - ✅ PASS: Specific recommendations (e.g., "Fix Feature 002 security issues")
   - ❌ FAIL: Generic recommendations ("review epic", "continue work")

**Output Format** (Present to User - ONLY if ALL evidence provided):

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Epic Status Analysis Complete - Review Gate Passed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Evidence Summary

### 1. Epic Data Loaded

Epic: [EPIC-ID] - [Epic Name]
Total Features: [N]
Created: [Date]
Target Completion: [Date]

Status: ✅ epic-status.json read successfully

### 2. Wave Progress

[Wave progress data with actual percentages]

Example:
- Wave 1 (Foundation): 100% complete (1/1 features)
- Wave 2 (Core): 50% complete (2/4 features)
- Wave 3 (Integration): 0% complete (0/1 features)

### 3. Feature Distribution by Phase

[Feature counts by phase from epic data]

Example:
- COMPLETED: 1 feature
- IMPLEMENTING: 2 features
- PLANNING: 1 feature
- SPECIFYING: 1 feature

Total: 5 features

### 4. Smart Recommendations

[Specific recommendations tied to actual features/issues]

Example:
- 🔴 CRITICAL: Resolve 2 security issues in Feature 002 (Tenant Dashboard)
- 🟠 MAJOR: Complete Feature 003 planning (blocking Wave 2)
- 🟡 MEDIUM: Review Feature 004 dependencies

## Review Gate Status: ✅ ANALYSIS COMPLETE

Epic status data successfully loaded and analyzed. Recommendations are evidence-based.

## Next Steps

[Continue to display full epic status dashboard]
```

**Review Gate Logic**:

```yaml
IF all 4 evidence questions = ✅ PASS:
  → Overall Status: ✅ ANALYSIS COMPLETE
  → Proceed to display full epic status (Steps 3-9 outputs)

ELSE IF 1-2 questions = ❌ FAIL:
  → Overall Status: ⚠️ INCOMPLETE DATA
  → Re-read epic-status.json and feature files
  → Recalculate missing metrics

ELSE IF 3+ questions = ❌ FAIL:
  → Overall Status: ❌ DATA NOT LOADED
  → Epic status command failed to load epic files
  → Verify epic exists and epic-status.json is valid JSON
```

**Note**: This review gate validates DATA LOADING, not epic health. A failing epic can still pass the review gate if status data was properly analyzed.

---

## Error Handling

**Common Errors**:

1. **Epic Not Found**:
   - **Error**: "Epic '<epic-name>' not found in .specify/epics/"
   - **Fix**: Run `/speckit.epic.create <epic-name>` first

2. **Missing epic-status.json**:
   - **Error**: "epic-status.json not found - epic may be corrupted"
   - **Fix**: Regenerate epic via `/speckit.epic.create --force`

3. **Feature Metadata Not Found**:
   - **Warning**: "Feature 002 metadata not found, using epic-status.json data only"
   - **Fix**: Run `/speckit.epic.decompose` to generate feature specs

4. **No Quality Data**:
   - **Info**: "No quality data found for features. Run /speckit.analyze per feature."
   - **Fix**: Run `/speckit.analyze <feature-id>` for each feature

---

## Token Economy (v2.8.2)

**Cost Breakdown**:

| Step | Tokens | Notes |
|------|--------|-------|
| Context Loading | 5-8K | Strategic memory only (v2.7 savings) |
| Epic Discovery & Data Collection | 10-15K | Read epic files + feature metadata (5 features) |
| Epic Overview | 2-3K | Basic calculations and rendering |
| Wave Progress | 3-5K | Wave-based calculations |
| Feature Status (--features) | 5-8K | Per-feature phase progression |
| Integration Points (--integration) | 5-8K | Dependency health calculations |
| Quality Metrics (--quality) | 5-8K | Aggregate quality scores |
| Timeline (--timeline) | 3-5K | Milestone tracking, Gantt visualization |
| Smart Recommendations | 3-5K | Logic-based prioritization |
| **Total (default)** | **25-38K** | ~$0.45-$0.70 |
| **Total (--all)** | **40-63K** | ~$0.75-$1.15 |

**Cost Comparison**:
- Manual epic status report: 1-2 hours gathering data from multiple files
- Epic status command: <2 minutes, comprehensive, always up-to-date
- **ROI**: 30-60× time savings

**Frequency**: Expected to run 1-2× per day (daily standup, progress checks)

---

## Best Practices

1. **Daily Monitoring**: Run `/speckit.epic.status` daily during active development
2. **Use Flags for Deep Dives**: Default view for quick health check, flags for detailed investigation
3. **Address Critical Issues First**: Prioritize 🔴 CRITICAL recommendations
4. **Update Integration Point Status**: Manually update epic-status.json when integration contracts delivered
5. **Track Milestone Delays**: Document reasons for delays in epic-spec.md or commit messages
6. **Quality Analysis**: Run `/speckit.analyze` for each feature after implementation
7. **Stakeholder Reports**: Use `--all` flag for comprehensive stakeholder updates

---

## Integration with Workflow

**Epic Status fits into monitoring phase**:

```
/speckit.epic.create           # Create epic
    ↓
/speckit.epic.decompose        # Generate feature specs
    ↓
[Feature Development Loop:]
    /speckit.plan (per feature)
    /speckit.implement (per feature)
    /speckit.epic.status       # ← Monitor progress daily
    /speckit.analyze (per feature)
    /speckit.reconcile (per feature)
    ↓
/speckit.epic.status --all     # Final epic status report
```

---

## Updating Integration Point Status

**Manual Update** (until auto-sync implemented):

Edit `epic-status.json` → `dependencies.integration_points[]`:

```json
{
  "from_feature": "001",
  "to_feature": "002",
  "contract": "ITenantService interface",
  "delivery_date": "2025-02-10",
  "actual_delivery_date": "2025-02-10",
  "status": "ON_TRACK"
}
```

**Status Values**:
- `"ON_TRACK"`: Delivered or on schedule
- `"AT_RISK"`: Behind schedule but recoverable
- `"BLOCKED"`: Missed deadline or major blocker

---

## Troubleshooting

### Issue: Epic Not Found

**Problem**: Command fails with "Epic not found: [EPIC_NAME]"

**Solution**: Verify epic exists and name is correct

```bash
# Check available epics
ls .specify/epics/

# If epic exists with different name:
/speckit.epic.status [CORRECT_EPIC_NAME]

# If epic doesn't exist, create it first:
/speckit.epic.create [EPIC_NAME]
```

### Issue: epic-status.json Corrupted or Missing

**Problem**: Error reading epic-status.json or malformed JSON

**Solution**: Validate and fix JSON syntax

```bash
# Option A: Validate JSON syntax
cat .specify/epics/[EPIC_NAME]/epic-status.json | jq .
# If error: "parse error at line X" → fix JSON syntax

# Option B: Regenerate from template (DESTRUCTIVE - loses current status)
cp .specify/templates/epics/epic-status-schema.json .specify/epics/[EPIC_NAME]/epic-status.json
# Manually populate epic details and feature statuses

# Option C: Restore from git if available
git checkout HEAD -- .specify/epics/[EPIC_NAME]/epic-status.json
```

### Issue: Feature Status Files Missing

**Problem**: Status dashboard shows "Feature 002: ❌ NO DATA"

**Solution**: Create missing spec-metadata.json files

```bash
# Check which features are missing metadata
ls specs/*/spec-metadata.json

# Option A: Run decomposition to create metadata for all features
/speckit.epic.decompose [EPIC_NAME]

# Option B: Create metadata manually for single feature
cat > specs/002-feature-name/spec-metadata.json <<EOF
{
  "version": "2.3.0",
  "feature_id": "002",
  "feature_name": "Feature Name",
  "epic_id": "EPIC-001",
  "epic_name": "[EPIC_NAME]",
  "phase": "specification",
  "wave": 2,
  "approvals": {},
  "metadata": {}
}
EOF
```

### Issue: Wave Calculation Errors

**Problem**: Wave completion percentages incorrect or waves shown as 0%

**Solution**: Verify wave assignments in feature metadata

```bash
# 1. Check wave assignments for all features
for f in specs/*/spec-metadata.json; do
  echo "$f:"
  cat "$f" | jq -r '.wave'
done

# 2. Fix incorrect wave assignments
nano specs/[FEATURE_ID]/spec-metadata.json
# Update: "wave": [CORRECT_WAVE_NUMBER]

# 3. Verify wave assignments match dependency-map.md
cat .specify/epics/[EPIC_NAME]/dependency-map.md

# 4. Re-check status
/speckit.epic.status [EPIC_NAME]
```

### Issue: Integration Point Status Mismatches

**Problem**: Integration contracts show wrong status or missing from dashboard

**Solution**: Sync dependency-map.md with actual feature progress

```bash
# 1. Check current integration contracts
cat .specify/epics/[EPIC_NAME]/dependency-map.md | grep -A 10 "Integration Points"

# 2. Update contract status based on actual delivery
nano .specify/epics/[EPIC_NAME]/dependency-map.md

# Example update:
# | 001 | 002 | ITenantService | 2025-02-10 | ON TRACK |
# →
# | 001 | 002 | ITenantService | 2025-02-12 | AT RISK | (delayed 2 days)

# 3. Re-run status to see updated integration health
/speckit.epic.status [EPIC_NAME]
```

### Issue: Quality Metrics Aggregation Failures

**Problem**: Epic quality score shows 0.0/10 or "NO DATA"

**Solution**: Ensure quality.json files exist for features

```bash
# 1. Check which features have quality analysis
ls specs/*/quality.json

# 2. Run quality analysis for missing features
cd specs/001-feature-name
/speckit.analyze

# 3. Repeat for all features in epic

# 4. Re-run epic status
/speckit.epic.status [EPIC_NAME]

# Note: Quality aggregation requires quality.json in feature directories
```

### Issue: Stale Status Data

**Problem**: Epic status shows outdated information (old phase, wrong percentages)

**Solution**: Update spec-metadata.json files with current phase

```bash
# 1. Check which features are complete but showing as "planning"
for f in specs/*/spec-metadata.json; do
  echo "$f: $(cat "$f" | jq -r '.phase')"
done

# 2. Update stale phases manually
nano specs/[FEATURE_ID]/spec-metadata.json
# Change: "phase": "planning" → "phase": "complete"

# 3. Alternative: Use /speckit.status in each feature directory
cd specs/001-feature-name
/speckit.status  # Will auto-update phase if implementation complete

# 4. Refresh epic status
/speckit.epic.status [EPIC_NAME]
```

### Issue: Incorrect Progress Percentages

**Problem**: Epic shows 80% complete but only 2/5 features done

**Solution**: Recalculate progress based on actual completion

```bash
# 1. Manually count completed features
grep -l '"phase": "complete"' specs/*/spec-metadata.json | wc -l

# 2. Check if epic-status.json has wrong feature count
cat .specify/epics/[EPIC_NAME]/epic-status.json | jq '.features | length'

# 3. If mismatch, update epic-status.json
nano .specify/epics/[EPIC_NAME]/epic-status.json
# Ensure feature count matches actual features

# 4. Re-run status
/speckit.epic.status [EPIC_NAME]
```

### Issue: Missing Milestone Data

**Problem**: Milestones section empty or shows "NO MILESTONES DEFINED"

**Solution**: Add milestones to epic-status.json

```bash
# 1. Edit epic-status.json to add milestones
nano .specify/epics/[EPIC_NAME]/epic-status.json

# 2. Add milestones array:
"timeline": {
  "milestones": [
    {
      "name": "Foundation Complete",
      "features": ["001", "002"],
      "target_date": "2025-02-15",
      "actual_date": null,
      "status": "IN_PROGRESS"
    }
  ]
}

# 3. Re-run status
/speckit.epic.status [EPIC_NAME]
```

### Issue: Critical Path Calculation Incorrect

**Problem**: Critical path shows wrong duration or missing features

**Solution**: Verify dependency graph and effort estimates

```bash
# 1. Review dependency graph
cat .specify/epics/[EPIC_NAME]/dependency-map.md

# 2. Check effort estimates for each feature
cat .specify/epics/[EPIC_NAME]/epic-status.json | jq '.features[] | {id: .feature_id, effort: .effort_weeks}'

# 3. Manually trace critical path:
# - Start with features in Wave 1
# - Follow longest dependency chain
# - Sum effort_weeks along the path

# 4. Update critical path in dependency-map.md if incorrect
nano .specify/epics/[EPIC_NAME]/dependency-map.md
```

### Issue: Smart Recommendations Not Showing

**Problem**: Recommendations section empty or unhelpful

**Solution**: Ensure sufficient epic data for analysis

```bash
# 1. Verify epic has features in various states (not all complete or all pending)
/speckit.epic.status [EPIC_NAME] | grep "Feature Status"

# 2. Check for blocked features (triggers recommendations)
cat .specify/epics/[EPIC_NAME]/epic-status.json | jq '.features[] | select(.status == "BLOCKED")'

# 3. Check for at-risk integration points
cat .specify/epics/[EPIC_NAME]/dependency-map.md | grep "AT RISK\|BLOCKED"

# 4. If all data looks good, recommendations will generate based on:
#    - Blocked features
#    - At-risk waves
#    - Quality issues (critical_issues_count > 0)
#    - Timeline delays
```

### Issue: Epic Status Command Hangs or Times Out

**Problem**: Command runs indefinitely without output

**Solution**: Check for circular dependencies or missing files

```bash
# 1. Interrupt command (Ctrl+C)

# 2. Validate all required files exist
ls .specify/epics/[EPIC_NAME]/{epic-spec.md,epic-architecture.md,dependency-map.md,epic-status.json}

# 3. Check for circular dependencies in dependency graph
cat .specify/epics/[EPIC_NAME]/dependency-map.md
# Look for cycles: F001 → F002 → F003 → F001

# 4. Fix circular dependencies in dependency-map.md
nano .specify/epics/[EPIC_NAME]/dependency-map.md

# 5. Re-run status
/speckit.epic.status [EPIC_NAME]
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.8.2 | 2025-01-15 | Initial release - Epic status monitoring |

---

**Command Status**: ✅ Ready for Use (v2.8.2)
**Last Updated**: 2025-01-15
**Maintainer**: Spec-Kit Framework
