# Epic Management Guide

**Version**: 2.8.3
**Last Updated**: 2025-01-15
**Target Audience**: Product Managers, Technical Leads, Engineering Teams

---

## Table of Contents

1. [Introduction](#introduction)
2. [When to Use Epic Management](#when-to-use-epic-management)
3. [Epic Management Workflow](#epic-management-workflow)
4. [Epic Creation (v2.8.0)](#epic-creation-v280)
5. [Epic Decomposition (v2.8.1)](#epic-decomposition-v281)
6. [Epic Monitoring (v2.8.2)](#epic-monitoring-v282)
7. [Team Coordination](#team-coordination)
8. [Best Practices](#best-practices)
9. [Common Pitfalls](#common-pitfalls)
10. [Troubleshooting](#troubleshooting)
11. [Case Study: Multi-Tenant SaaS Platform](#case-study-multi-tenant-saas-platform)

---

## Introduction

Epic management in Spec-Kit enables coordinated development of multi-feature initiatives through:
- **Shared Architecture**: Architectural Decision Records (ADRs) inherited by all features
- **Dependency Tracking**: Critical path analysis and parallelization planning
- **Integration Contracts**: Explicit upstream/downstream dependencies
- **Constitutional Compliance**: Project-wide principles propagated to all features
- **Progress Monitoring**: Real-time epic health dashboard

**Epic Management vs Single-Feature Development**:

| Aspect | Single Feature | Epic (Multi-Feature) |
|--------|----------------|---------------------|
| Scope | 1 feature | 2+ related features |
| Architecture | Per-feature decisions | Shared ADRs across features |
| Dependencies | Minimal | Explicit dependency graph |
| Coordination | Self-contained | Team coordination required |
| Monitoring | `/speckit.status` | `/speckit.epic.status` |
| Timeline | Linear (1 feature) | Parallelized (waves) |
| Commands | `/speckit.specify` → `/speckit.plan` | `/speckit.epic.create` → `/speckit.epic.decompose` |

---

## When to Use Epic Management

### Use Epic Management When:

✅ **2+ features share architecture**
- Example: Multi-tenancy requires shared database schema, authentication, patterns
- Benefit: Consistent implementation across features, no architectural drift

✅ **Features have dependencies**
- Example: Feature B needs Feature A's API before it can proceed
- Benefit: Dependency tracking prevents integration failures

✅ **Multi-team coordination needed**
- Example: 2 teams developing features in parallel (waves)
- Benefit: Clear handoff expectations, integration contracts

✅ **Complex initiative (>4 weeks)**
- Example: Payment processing system with 5+ features
- Benefit: Timeline optimization (40% reduction via parallelization)

✅ **Shared constitutional requirements**
- Example: Epic-wide TDD, security standards, prohibited patterns
- Benefit: Consistent compliance across all features

### Don't Use Epic Management When:

❌ **Single independent feature** → Use `/speckit.specify` instead
❌ **Features with no shared architecture** → Develop independently
❌ **Simple CRUD features (<1 week each)** → Overhead not justified
❌ **Exploratory/prototype work** → Too much upfront planning

---

## Epic Management Workflow

### Complete Workflow (6-Week Example)

```
WEEK 1-2: Epic Planning & Architecture
┌─────────────────────────────────────────────────┐
│ /speckit.epic.create multi-tenant-saas         │
│                                                 │
│ Outputs:                                        │
│ ├─ epic-spec.md (vision, goals, features)      │
│ ├─ epic-architecture.md (ADRs, patterns)       │
│ ├─ dependency-map.md (critical path, waves)    │
│ └─ epic-status.json (progress tracking)        │
│                                                 │
│ Stakeholder Approval:                           │
│ ├─ Product: Vision & business goals ✅          │
│ ├─ Tech Lead: Architecture decisions ✅         │
│ └─ Security: Compliance validation ✅           │
└─────────────────────────────────────────────────┘
           ↓
WEEK 3-4: Feature Spec Generation
┌─────────────────────────────────────────────────┐
│ /speckit.epic.decompose multi-tenant-saas      │
│                                                 │
│ For Each Feature (001-005):                    │
│ ├─ spec.md (with ADR inheritance)              │
│ ├─ Integration requirements (contracts)        │
│ ├─ Constitutional requirements (TDD, etc.)     │
│ └─ spec-metadata.json (epic linkage)           │
│                                                 │
│ Feature Review:                                 │
│ ├─ Engineers: User stories & acceptance        │
│ ├─ Product: Business value alignment           │
│ └─ Tech Lead: ADR applicability                │
└─────────────────────────────────────────────────┘
           ↓
WEEK 5-10: Feature Development (Waves)
┌─────────────────────────────────────────────────┐
│ Wave 1 (Week 5-7): Foundation                   │
│ ├─ Feature 001: Tenant Isolation               │
│ │   /speckit.plan 001-tenant-isolation         │
│ │   /speckit.implement 001-tenant-isolation    │
│ │   /speckit.reconcile 001-tenant-isolation    │
│ └─ Daily: /speckit.epic.status (monitor)       │
│                                                 │
│ Wave 2 (Week 8-10): Core Features (Parallel)   │
│ ├─ Team A:                                      │
│ │   ├─ Feature 002: Tenant Dashboard           │
│ │   └─ Feature 004: Tenant Admin               │
│ ├─ Team B:                                      │
│ │   └─ Feature 003: Tenant Billing             │
│ └─ Daily: /speckit.epic.status --integration   │
│                                                 │
│ Wave 3 (Week 11): Integration                   │
│ ├─ Feature 005: Tenant Onboarding              │
│ └─ Final: /speckit.epic.status --all           │
└─────────────────────────────────────────────────┘
```

---

## Epic Creation (v2.8.0)

### Step-by-Step Guide

#### 1. Identify Epic Scope

**Questions to Answer**:
- What is the high-level vision? (1-2 paragraphs)
- What business goals does this achieve? (3-5 measurable goals)
- What features are included? (minimum 2, typically 3-7)
- What architectural decisions span multiple features?
- What are the dependencies between features?

**Example**:
```
Vision: Enable SaaS multi-tenancy to support 100+ customers on shared infrastructure

Business Goals:
1. Revenue: Support 10x customer growth without infrastructure rebuild
2. Cost: Reduce per-customer infrastructure cost by 70%
3. Speed: Onboard new customers in <1 hour (vs 3 days currently)

Features:
001 - Tenant Isolation (database schema, RLS policies)
002 - Tenant Dashboard (per-tenant analytics)
003 - Tenant Billing (subscription management)
004 - Tenant Admin (user/role management)
005 - Tenant Onboarding (automated provisioning)
```

#### 2. Run Epic Creation Command

```bash
/speckit.epic.create multi-tenant-saas
```

**Interactive Prompts**:
1. Epic Name: "Multi-Tenant SaaS Platform"
2. Epic ID: EPIC-001 (auto-generated)
3. Target Date: Q2 2025
4. Vision: [Paste vision statement]
5. Business Goals: [Enter 3-5 goals]
6. Feature List: [Enter feature IDs and names]
7. For Each Feature:
   - Name, risk level, complexity, effort estimate, dependencies
8. Architectural Decisions:
   - Database choice (PostgreSQL with RLS)
   - Multi-tenancy approach (subdomain-based)
   - Authentication strategy (JWT with tenant claim)

**Outputs**:
- `.specify/epics/multi-tenant-saas/epic-spec.md`
- `.specify/epics/multi-tenant-saas/epic-architecture.md`
- `.specify/epics/multi-tenant-saas/dependency-map.md`
- `.specify/epics/multi-tenant-saas/epic-status.json`

#### 3. Review & Approve Epic

**Review Checklist**:
- [ ] **epic-spec.md**: Vision clear, business goals measurable, feature breakdown complete
- [ ] **epic-architecture.md**: ADRs documented with context/decision/consequences
- [ ] **dependency-map.md**: Critical path identified, parallelization opportunities clear
- [ ] **epic-status.json**: Timeline feasible, risk assessment accurate

**Stakeholder Approvals**:
1. **Product Owner**: Approve vision, business goals, success criteria
2. **Tech Lead**: Approve architectural decisions, technology choices
3. **Security**: Validate compliance requirements (OWASP, GDPR, etc.)
4. **Engineering Manager**: Confirm timeline, resource allocation

**Update epic-status.json**:
```json
{
  "approvals": {
    "planning": {
      "approved": true,
      "timestamp": "2025-01-18T10:00:00Z",
      "approved_by": "Alice (Product Owner)"
    },
    "architecture": {
      "approved": true,
      "timestamp": "2025-01-19T14:30:00Z",
      "approved_by": "Bob (Tech Lead)"
    }
  }
}
```

---

## Epic Decomposition (v2.8.1)

### Step-by-Step Guide

#### 1. Verify Epic Approval

```bash
/speckit.epic.status multi-tenant-saas
```

Check approvals section:
```
📋 APPROVALS
├─ Planning: ✅ Approved (2025-01-18 by Alice)
├─ Architecture: ✅ Approved (2025-01-19 by Bob)
```

If not approved, fix issues and get stakeholder sign-off before proceeding.

#### 2. Run Epic Decomposition

```bash
/speckit.epic.decompose multi-tenant-saas
```

**Interactive Mode** (default):
- Prompts: "Which features should be decomposed?"
- Select: All features OR specific IDs (001, 002, 003)
- For each feature:
  - Confirm ADR applicability
  - Add feature-specific ADR implications
  - Verify integration contracts

**Quick Mode** (auto-approve all):
```bash
/speckit.epic.decompose multi-tenant-saas -y
```

**Selective Mode** (specific features):
```bash
/speckit.epic.decompose multi-tenant-saas --features 001,002,003
```

#### 3. Review Generated Feature Specs

For each feature (e.g., `001-tenant-isolation`):

**Review spec.md**:
- [ ] **Epic Reference**: Correct epic link, feature position in wave
- [ ] **Architectural Decisions**: Relevant ADRs inherited with feature-specific implications
- [ ] **Integration Requirements**: Upstream dependencies clear, downstream deliverables defined
- [ ] **Constitutional Requirements**: TDD, integration-first, prohibited patterns documented
- [ ] **User Stories**: Customize auto-generated stubs
- [ ] **Acceptance Criteria**: Add EARS requirements beyond integration contracts

**Example Customization**:
```markdown
<!-- AUTO-GENERATED -->
## Architectural Decisions (From Epic)

### AD-EPIC-001: PostgreSQL Row-Level Security
**Decision**: Use PostgreSQL RLS with tenant_id column
**Feature-Specific Implications**:
- Implement RLS policies on all tables with tenant_id column

<!-- CUSTOMIZE THIS -->
**Feature 001-Specific Implementation**:
- Create migration: `20250115_enable_rls_policies.sql`
- Tables: users, organizations, subscriptions, invoices
- Test: Verify cross-tenant access blocked via integration test
- Performance: Add tenant_id index for RLS performance
```

#### 4. Approve Feature Specs

For each feature:
```bash
cd .specify/features/001-tenant-isolation
# Review spec.md
# Update spec-metadata.json:
```

```json
{
  "approvals": {
    "specification": {
      "approved": true,
      "timestamp": "2025-02-01T09:00:00Z"
    }
  }
}
```

---

## Epic Monitoring (v2.8.2)

### Daily Standup Usage

**Quick Health Check** (default view):
```bash
/speckit.epic.status multi-tenant-saas
```

Output:
```
📊 OVERVIEW
├─ Progress: ████████░░░░ 40% (2/5 features complete)
├─ Status: 🟢 ACTIVE

🌊 WAVE PROGRESS
Wave 2: Core Features - ACTIVE WAVE
├─ Progress: █████░░░░░░░ 33% (1/3 complete)
└─ Blocked: None

💡 SMART RECOMMENDATIONS
🔴 CRITICAL: Resolve 2 security issues in Feature 002
🟠 HIGH: Complete Feature 003 planning (blocking Wave 2)
```

**Standup Questions Answered**:
- What's our overall progress? → **40% (2/5 complete)**
- Which wave are we in? → **Wave 2 (Core Features)**
- Any blockers? → **None (but Feature 003 at risk)**
- What's critical today? → **Security issues in 002, planning for 003**

### Weekly Status Report

**Comprehensive View** (for stakeholder updates):
```bash
/speckit.epic.status multi-tenant-saas --all
```

Includes:
- **--features**: Detailed feature-by-feature phase progression
- **--integration**: Integration point health (🟢/🟡/🔴)
- **--quality**: Quality metrics aggregation
- **--timeline**: Gantt chart, milestone tracking

### Integration Point Monitoring

**Focus on Dependencies**:
```bash
/speckit.epic.status multi-tenant-saas --integration
```

Output:
```
🤝 INTEGRATION POINTS

From: 001 → To: 002 (ITenantService)
Status: 🟢 DELIVERED (on time)

From: 002 → To: 005 (Dashboard UI)
Status: 🟡 AT RISK (due in 5 days, 50% complete)

INTEGRATION HEALTH: 67% (4/6 on track)
```

**Action Items**:
- **🟢 DELIVERED**: No action needed
- **🟡 AT RISK**: Daily check-in with team, expedite if needed
- **🔴 BLOCKED**: Immediate escalation, replan timeline

### Quality Tracking

**Aggregate Quality View**:
```bash
/speckit.epic.status multi-tenant-saas --quality
```

Output:
```
📊 QUALITY METRICS

Overall: GOOD (8.2/10)
Critical Issues: 3 total
Test Coverage: 87.5% ✅

Per-Feature:
001: EXCELLENT (9.5/10) - 0 issues
002: GOOD (8.0/10) - 2 issues (security)
004: GOOD (7.1/10) - 1 issue (performance)
```

**Quality Gate**:
- **EXCELLENT/GOOD**: Proceed
- **ACCEPTABLE**: Address issues before launch
- **POOR**: Stop, refactor required

---

## Team Coordination

### Single Team (Sequential Development)

**Timeline**: 10 weeks (Wave 1 → Wave 2 → Wave 3)

```
Week 1-3: Feature 001 (Foundation)
Week 4-6: Feature 003 (Critical path)
Week 7-8: Feature 002 (Parallel work)
Week 9: Feature 004 (Quick win)
Week 10: Feature 005 (Integration)
```

**Daily Workflow**:
1. Morning: `/speckit.epic.status` → Check recommendations
2. Development: Work on current feature
3. End of day: Update progress, run `/speckit.epic.status` again

### Multi-Team (Parallel Development)

**Timeline**: 6 weeks (40% reduction via parallelization)

**Team Assignment**:
```
Wave 1 (Week 1-3): Team A
├─ Feature 001: Tenant Isolation

Wave 2 (Week 4-5): Teams A + B (parallel)
├─ Team A:
│   ├─ Feature 002: Tenant Dashboard
│   └─ Feature 004: Tenant Admin
└─ Team B:
    └─ Feature 003: Tenant Billing

Wave 3 (Week 6): Team A
└─ Feature 005: Tenant Onboarding
```

**Coordination Points**:
1. **Wave 1 → Wave 2 Handoff**:
   - Verify Feature 001 delivers all integration contracts
   - Check: `/speckit.epic.status --integration`
   - Gate: All integration points 🟢 DELIVERED before Wave 2 starts

2. **Wave 2 (Parallel Development)**:
   - Daily sync between Team A and Team B
   - Monitor: `/speckit.epic.status --integration` (cross-team dependencies)
   - Example: Team A's Feature 002 may need Team B's Feature 003 API

3. **Wave 2 → Wave 3 Handoff**:
   - Verify Features 002, 003, 004 complete
   - Integration tests: Features work together
   - Gate: All Wave 2 features ✅ COMPLETE before Wave 3 starts

**Communication Cadence**:
- **Daily**: Each team runs `/speckit.epic.status` (their features)
- **Daily**: Tech lead runs `/speckit.epic.status --integration` (cross-team)
- **Weekly**: Stakeholder update via `/speckit.epic.status --all`

---

## Best Practices

### 1. Epic Planning

✅ **DO**:
- Start with clear vision and business goals
- Limit epic to 3-7 features (manageable scope)
- Document ADRs with context, not just decisions
- Identify critical path early (bottleneck features)
- Get stakeholder approval before decomposition

❌ **DON'T**:
- Create epics for single features (use `/speckit.specify`)
- Include exploratory work in epic (too much uncertainty)
- Skip dependency analysis (leads to integration failures)
- Over-engineer architecture (simplicity wins)

### 2. Architectural Decisions

✅ **DO**:
- Document "why" not just "what" (ADR context)
- Provide implementation examples (code snippets)
- Update ADRs when decisions change (mark superseded)
- Link to external resources (RFCs, blog posts, docs)

❌ **DON'T**:
- Create ADRs for trivial decisions (noise)
- Skip alternatives considered (context lost)
- Make decisions without team input (buy-in)
- Ignore constitutional principles (TDD, simplicity, etc.)

### 3. Dependency Management

✅ **DO**:
- Define integration contracts explicitly (TypeScript interfaces)
- Set delivery dates for integration points
- Monitor integration health daily (`--integration`)
- Use feature flags for incremental integration

❌ **DON'T**:
- Assume implicit dependencies (document everything)
- Wait until end to integrate (continuous integration)
- Ignore 🟡 AT RISK warnings (escalate early)
- Create circular dependencies (validate during epic creation)

### 4. Quality Management

✅ **DO**:
- Run `/speckit.analyze` after each feature implementation
- Address critical issues before moving to next feature
- Track test coverage per feature (epic-wide targets)
- Conduct security review for HIGH-risk epics

❌ **DON'T**:
- Skip quality analysis (defer all issues to end)
- Batch quality fixes (harder to fix later)
- Ignore constitutional violations (Step 10.4 blockers)
- Lower quality bar for "later" features (technical debt)

### 5. Monitoring & Communication

✅ **DO**:
- Run `/speckit.epic.status` daily (health check)
- Use flags for focused views (`--integration`, `--quality`)
- Share status dashboard with stakeholders weekly
- Act on 🔴 CRITICAL recommendations immediately

❌ **DON'T**:
- Wait for weekly meetings to check status
- Ignore 🟡 AT RISK warnings (become 🔴 BLOCKED)
- Manual status reports (use `/speckit.epic.status --all`)
- Surprise stakeholders with delays (proactive communication)

---

## Common Pitfalls

### Pitfall 1: Skipping Epic Approval

**Problem**: Start decomposition before epic approved → Rework when stakeholders reject vision or architecture

**Solution**:
```bash
# WRONG
/speckit.epic.create → /speckit.epic.decompose  # No approval step!

# RIGHT
/speckit.epic.create
# → Review with stakeholders
# → Update epic-status.json approvals
# → Then: /speckit.epic.decompose
```

### Pitfall 2: Ignoring Integration Contracts

**Problem**: Feature 002 assumes Feature 001 API, but no contract → Integration fails

**Solution**:
- Define integration contracts in `dependency-map.md`
- Monitor via `/speckit.epic.status --integration`
- Set delivery dates and track 🟢/🟡/🔴 status

### Pitfall 3: Parallel Development Without Coordination

**Problem**: Team A and Team B build incompatible implementations → Integration nightmare

**Solution**:
- Daily sync between teams during Wave 2
- Monitor integration points: `/speckit.epic.status --integration`
- Use ADRs to enforce consistency (shared patterns)

### Pitfall 4: Deferring Quality Issues

**Problem**: "We'll fix quality issues later" → Technical debt accumulates, launch delayed

**Solution**:
- Run `/speckit.analyze` after each feature
- Address critical issues before next feature
- Track via `/speckit.epic.status --quality`

### Pitfall 5: Not Updating epic-status.json

**Problem**: Manual tracking diverges from epic-status.json → `/speckit.epic.status` shows stale data

**Solution**:
- Update epic-status.json when features complete
- Update integration point status when contracts delivered
- Automate via CI/CD (future enhancement)

---

## Troubleshooting

### Issue: Epic Creation Fails

**Symptoms**:
- Error: "Circular dependency detected"
- Error: "Less than 2 features"

**Diagnosis**:
```bash
# Check feature dependencies in epic-spec.md
# Look for: 002 depends on 003, 003 depends on 002 (circular)
```

**Solution**:
- Remove circular dependencies (rethink architecture)
- Add at least 2 features (or use `/speckit.specify` for single feature)

### Issue: Feature Decomposition Overwrites Custom Specs

**Symptoms**:
- Warning: "Feature 001 already has spec. Overwrite?"

**Diagnosis**:
- Feature spec exists, decomposition will replace it

**Solution**:
```bash
# Option 1: Skip this feature (keep custom spec)
# Select "n" at prompt

# Option 2: Force overwrite (lose custom changes)
/speckit.epic.decompose --force

# Option 3: Manually merge (recommended)
# Compare custom spec vs auto-generated, merge manually
```

### Issue: Integration Point Shows 🔴 BLOCKED

**Symptoms**:
- Feature 005 blocked, dependency not delivered

**Diagnosis**:
```bash
/speckit.epic.status --integration
# From: 002 → To: 005 (Dashboard UI)
# Status: 🔴 BLOCKED (Feature 002 only 50% complete)
```

**Solution**:
1. Expedite Feature 002 (add resources, reduce scope)
2. Replan timeline (delay Feature 005 start)
3. Find workaround (mock integration for Feature 005 development)

### Issue: Quality Metrics Not Showing

**Symptoms**:
- `/speckit.epic.status --quality` shows "No quality data"

**Diagnosis**:
- Features not analyzed yet via `/speckit.analyze`

**Solution**:
```bash
# Analyze each feature after implementation
/speckit.analyze 001-tenant-isolation
/speckit.analyze 002-tenant-dashboard
# Then: /speckit.epic.status --quality
```

---

## Case Study: Multi-Tenant SaaS Platform

### Epic Overview

**Epic**: EPIC-001 Multi-Tenant SaaS Platform
**Features**: 5 features (001-005)
**Timeline**: 10 weeks sequential → 6 weeks parallelized (40% reduction)
**Risk**: 🔴 HIGH (score: 10/12)
**Teams**: 2 teams (parallel Wave 2 development)

### Week 1-2: Epic Planning

**Command**:
```bash
/speckit.epic.create multi-tenant-saas
```

**Key Outputs**:

**epic-architecture.md** (ADRs):
- **AD-EPIC-001**: PostgreSQL Row-Level Security for multi-tenancy
- **AD-EPIC-002**: JWT with tenant claim for authentication
- **AD-EPIC-003**: Subdomain-based tenant resolution

**dependency-map.md** (Critical Path):
```
Critical Path: 001 → 003 → 005 (7 weeks)
Wave 1: 001 (Week 1-3)
Wave 2: 002, 003, 004 (Week 4-5, parallel)
Wave 3: 005 (Week 6)
```

**Approvals**:
- Product: ✅ Approved 2025-01-18 (vision, business goals)
- Tech Lead: ✅ Approved 2025-01-19 (ADRs, technology choices)
- Security: ✅ Approved 2025-01-20 (OWASP compliance)

### Week 3-4: Feature Spec Generation

**Command**:
```bash
/speckit.epic.decompose multi-tenant-saas
```

**Generated Specs** (5 features):
- `.specify/features/001-tenant-isolation/spec.md` (with ADR inheritance)
- `.specify/features/002-tenant-dashboard/spec.md` (integration contracts)
- `.specify/features/003-tenant-billing/spec.md` (constitutional requirements)
- `.specify/features/004-tenant-admin/spec.md`
- `.specify/features/005-tenant-onboarding/spec.md`

**Review**: Engineers customize user stories, acceptance criteria

### Week 5-7: Wave 1 (Foundation)

**Team A**: Feature 001 (Tenant Isolation)

**Commands**:
```bash
/speckit.plan 001-tenant-isolation
/speckit.implement 001-tenant-isolation
/speckit.analyze 001-tenant-isolation
/speckit.reconcile 001-tenant-isolation
```

**Daily Monitoring**:
```bash
/speckit.epic.status multi-tenant-saas
# Progress: ░░░░░░░░░░░░░░░░░░░░ 0% → 20%
```

**Integration Contract Delivery** (End of Week 7):
- `ITenantService` interface → Feature 002, 004
- `tenant_id` schema + RLS policies → Feature 003
- Status: 🟢 DELIVERED

### Week 8-10: Wave 2 (Core Features - Parallel)

**Team A**: Features 002, 004
**Team B**: Feature 003

**Commands** (Team A):
```bash
/speckit.plan 002-tenant-dashboard
/speckit.implement 002-tenant-dashboard
/speckit.analyze 002-tenant-dashboard
```

**Commands** (Team B):
```bash
/speckit.plan 003-tenant-billing
/speckit.implement 003-tenant-billing
/speckit.analyze 003-tenant-billing
```

**Daily Coordination**:
```bash
# Tech Lead monitors cross-team dependencies
/speckit.epic.status multi-tenant-saas --integration

# Week 9 Issue: Feature 002 at risk
From: 002 → To: 005 (Dashboard UI)
Status: 🟡 AT RISK (due in 5 days, 50% complete)

# Action: Add engineer to Team A, expedite Feature 002
```

**End of Wave 2**:
- Feature 002: ✅ COMPLETE (Week 9)
- Feature 003: ✅ COMPLETE (Week 10)
- Feature 004: ✅ COMPLETE (Week 9)
- All integration contracts: 🟢 DELIVERED

### Week 11: Wave 3 (Integration)

**Team A**: Feature 005 (Tenant Onboarding)

**Commands**:
```bash
/speckit.plan 005-tenant-onboarding
/speckit.implement 005-tenant-onboarding
/speckit.reconcile 005-tenant-onboarding
```

**Integration Testing**:
- End-to-end flow: Create tenant → Dashboard → Billing → Admin
- All features working together ✅

### Final Status Report

**Command**:
```bash
/speckit.epic.status multi-tenant-saas --all
```

**Results**:
```
📊 OVERVIEW
├─ Progress: ████████████████████ 100% (5/5 complete)
├─ Status: ✅ COMPLETE

📊 QUALITY METRICS
├─ Overall: GOOD (8.5/10)
├─ Critical Issues: 0 (all resolved)
└─ Test Coverage: 88.2% ✅

⏱️ TIMELINE
├─ Original Estimate: 6 weeks (parallelized)
├─ Actual: 6 weeks ✅ ON TIME
└─ Savings vs Sequential: 4 weeks (40% reduction)
```

**Business Impact**:
- ✅ Revenue: Platform supports 100+ tenants (10x growth)
- ✅ Cost: Per-customer infrastructure cost reduced 70%
- ✅ Speed: Onboarding time <1 hour (vs 3 days)

---

## Conclusion

Epic management in Spec-Kit enables:
- **Consistent Architecture**: ADRs prevent divergence
- **Dependency Clarity**: Integration contracts prevent failures
- **Timeline Optimization**: 40% reduction via parallelization
- **Quality Assurance**: Continuous monitoring and validation
- **Team Coordination**: Clear handoffs, progress visibility

**Next Steps**:
1. Review [MIGRATION.md](./MIGRATION.md) for upgrading existing projects
2. See workflow examples in [EPIC-WORKFLOW-EXAMPLES.md](./EPIC-WORKFLOW-EXAMPLES.md)
3. Run `/speckit.epic.create` to create your first epic

---

**Guide Version**: 2.8.3
**Framework Version**: Spec-Kit v2.8.3
**Last Updated**: 2025-01-15
