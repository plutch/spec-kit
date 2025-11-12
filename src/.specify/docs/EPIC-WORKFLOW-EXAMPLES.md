# Epic Workflow Examples

**Version**: 2.8.3
**Last Updated**: 2025-01-15
**Purpose**: End-to-end examples of epic management workflows

---

## Table of Contents

1. [Example 1: Multi-Tenant SaaS Platform (Complete Workflow)](#example-1-multi-tenant-saas-platform-complete-workflow)
2. [Example 2: Payment Processing System (High-Risk Epic)](#example-2-payment-processing-system-high-risk-epic)
3. [Example 3: Analytics Dashboard (Single Team, Sequential)](#example-3-analytics-dashboard-single-team-sequential)
4. [Example 4: Microservices Migration (Retrofit Existing Features)](#example-4-microservices-migration-retrofit-existing-features)

---

## Example 1: Multi-Tenant SaaS Platform (Complete Workflow)

**Scenario**: Building multi-tenant capabilities for existing SaaS application
**Features**: 5 features (Isolation, Dashboard, Billing, Admin, Onboarding)
**Teams**: 2 teams (parallel Wave 2 development)
**Timeline**: 6 weeks (parallelized) vs 10 weeks (sequential)
**Risk**: 🔴 HIGH (score: 10/12)

---

### Week 1-2: Epic Planning & Architecture

#### Day 1: Epic Creation

**Command**:
```bash
/speckit.epic.create multi-tenant-saas
```

**Interactive Prompts**:
```
Epic Name: Multi-Tenant SaaS Platform
Epic ID: EPIC-001 (auto-generated)
Target Date: Q2 2025

Vision (1-2 paragraphs):
Enable SaaS multi-tenancy to support 100+ customers on shared infrastructure.
This allows us to scale revenue 10x without rebuilding our platform, reduce
per-customer costs by 70%, and onboard new customers in under 1 hour vs 3 days currently.

Business Goals:
1. Revenue: Support 10x customer growth without infrastructure rebuild
2. Cost Efficiency: Reduce per-customer infrastructure cost by 70%
3. Time-to-Value: Onboard new customers in <1 hour (vs 3 days currently)

Features (enter feature IDs and names):
001 - Tenant Isolation
002 - Tenant Dashboard
003 - Tenant Billing
004 - Tenant Administration
005 - Tenant Onboarding

For Feature 001 (Tenant Isolation):
  Risk Level: 🔴 HIGH
  Complexity: XL (>3 weeks)
  Effort Estimate: 3 weeks
  Dependencies: None (foundation feature)

For Feature 002 (Tenant Dashboard):
  Risk Level: 🟠 MEDIUM
  Complexity: L (2-3 weeks)
  Effort Estimate: 2 weeks
  Dependencies: 001

[Repeat for Features 003, 004, 005...]

Architectural Decisions:
Decision 1: Multi-tenancy database approach?
  → Use PostgreSQL Row-Level Security (RLS) with tenant_id column

Decision 2: Tenant resolution method?
  → Subdomain-based (e.g., acme.yourapp.com)

Decision 3: Authentication strategy?
  → JWT with tenant claim in payload
```

**Generated Files**:
```
.specify/epics/multi-tenant-saas/
├── epic-spec.md
├── epic-architecture.md
├── dependency-map.md
└── epic-status.json
```

#### Day 2-3: Epic Review & Refinement

**Review epic-architecture.md** - Add implementation guidance to ADRs:

```markdown
### AD-EPIC-001: Multi-Tenancy Data Isolation Strategy

**Status**: Accepted
**Date**: 2025-01-15
**Deciders**: Tech Lead, DBA, Security Team
**Impact**: All features (001-005)

**Context**:
We need to isolate data between tenants in our SaaS application. Options include
database-per-tenant, schema-per-tenant, or row-level security. We have 100+
expected tenants, limited DBA resources, and strict data isolation requirements.

**Decision**:
Use PostgreSQL Row-Level Security (RLS) with tenant_id column in all tables.

**Consequences**:
- **Positive**:
  - Single database = easier ops, backups, migrations
  - RLS enforced at database level = hard to bypass
  - Cost-effective for 100+ tenants
- **Negative**:
  - Performance overhead (~10-20ms per query)
  - Must remember tenant_id in every query
  - Harder to give tenant direct DB access

**Alternatives Considered**:
1. **Database-per-tenant**: Rejected - 100+ databases = ops nightmare
2. **Schema-per-tenant**: Rejected - schema migrations complex, connection pooling issues

**Implementation Guidance**:
```sql
-- Enable RLS on all tables
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  -- other columns
);

ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_policy ON orders
  USING (tenant_id = current_setting('app.current_tenant')::UUID);

-- Set tenant context in application
SET app.current_tenant = '[tenant-uuid]';
```

**Affected Features**: 001 (Isolation), 002 (Dashboard), 003 (Billing), 004 (Admin), 005 (Onboarding)
```

**Review dependency-map.md** - Verify critical path:

```markdown
**Critical Path**: 001 → 003 → 005 (7 weeks)

**Parallelization Strategy**:
- Wave 1 (Week 1-3): Feature 001 (Foundation)
- Wave 2 (Week 4-5): Features 002, 003, 004 (Parallel, 2 teams)
- Wave 3 (Week 6): Feature 005 (Integration)

**Time Savings**: 10 weeks sequential → 6 weeks parallelized (40% reduction)
```

#### Day 4-5: Stakeholder Approval

**Product Review** (Day 4):
- Review epic-spec.md with Product Owner
- Validate business goals, success criteria
- Approve vision and feature breakdown

**Update epic-status.json**:
```json
{
  "approvals": {
    "planning": {
      "generated": true,
      "approved": true,
      "timestamp": "2025-01-18T10:00:00Z",
      "approved_by": "Alice (Product Owner)"
    }
  }
}
```

**Tech Lead Review** (Day 5):
- Review epic-architecture.md with Tech Lead
- Validate ADRs, technology choices
- Discuss implementation risks

**Update epic-status.json**:
```json
{
  "approvals": {
    "architecture": {
      "generated": true,
      "approved": true,
      "timestamp": "2025-01-19T14:30:00Z",
      "approved_by": "Bob (Tech Lead)"
    }
  }
}
```

---

### Week 3-4: Feature Spec Generation

#### Day 10: Epic Decomposition

**Command**:
```bash
/speckit.epic.decompose multi-tenant-saas
```

**Interactive Prompts**:
```
Which features should be decomposed?
[ ] All features (001-005)
[x] Select specific features

Select features:
[x] 001 - Tenant Isolation
[x] 002 - Tenant Dashboard
[x] 003 - Tenant Billing
[x] 004 - Tenant Administration
[x] 005 - Tenant Onboarding

For Feature 001 (Tenant Isolation):
  AD-EPIC-001 affects this feature. Feature-specific implications?
  → Implement RLS policies on users, organizations, subscriptions, invoices tables
  → Create setTenantContext(db, tenantId) helper function
  → Write integration tests to verify cross-tenant access blocked

[Repeat for Features 002-005...]

Generating feature specs...
✅ 001-tenant-isolation/spec.md created
✅ 002-tenant-dashboard/spec.md created
✅ 003-tenant-billing/spec.md created
✅ 004-tenant-admin/spec.md created
✅ 005-tenant-onboarding/spec.md created
```

**Generated Files**:
```
.specify/features/
├── 001-tenant-isolation/
│   ├── spec.md (with ADR inheritance)
│   └── spec-metadata.json (epic linkage)
├── 002-tenant-dashboard/
│   ├── spec.md (with integration contracts)
│   └── spec-metadata.json
├── 003-tenant-billing/
│   ├── spec.md (with constitutional requirements)
│   └── spec-metadata.json
├── 004-tenant-admin/
│   ├── spec.md
│   └── spec-metadata.json
└── 005-tenant-onboarding/
    ├── spec.md
    └── spec-metadata.json
```

#### Day 11-14: Feature Spec Review & Customization

**Review 001-tenant-isolation/spec.md**:

Check auto-generated sections:
```markdown
## Epic Reference
**Epic**: [EPIC-001: Multi-Tenant SaaS Platform](../../../epics/multi-tenant-saas/epic-spec.md)
**Wave**: 1 (Week 1-3)
**Dependencies**: None (foundation feature)

## Architectural Decisions (From Epic)
### AD-EPIC-001: RLS for Multi-Tenancy
[Auto-generated from epic-architecture.md]

## Integration Requirements
### Provides To (Downstream Features)
| Feature | Contract | Delivery | What This Feature Must Deliver |
|---------|----------|----------|-------------------------------|
| 002 | ITenantService | 2025-02-10 | getTenantContext(req) helper, CRUD operations |
| 003 | tenant_id schema | 2025-02-10 | All tables have tenant_id, RLS policies active |

## Constitutional Requirements
### Article I: TDD
- Requirement: ALL code must follow RED-GREEN-REFACTOR
- Validation: Git history analysis at Step 10.4.1
```

**Customize User Stories & Acceptance Criteria**:
```markdown
## User Scenarios & Testing

### US-001: Tenant Data Isolation (P1 - CRITICAL)
As a SaaS administrator, I want tenant data to be strictly isolated at the database level, so that no tenant can access another tenant's data under any circumstances.

**Acceptance Criteria (EARS Format)**:
- REQ-ISO-1: WHEN a query is executed THEN it SHALL only return rows WHERE tenant_id matches current_setting('app.current_tenant')
- REQ-ISO-2: WHEN tenant context is not set THEN database SHALL reject query with error
- REQ-ISO-3: WHEN attempting to access another tenant's data THEN RLS policy SHALL block access (zero results)

**Test Scenarios**:
1. Happy Path: User queries data for their tenant → Returns only their data
2. Edge Case: User attempts SQL injection to bypass RLS → Blocked by RLS
3. Error Case: Application forgets to set tenant context → Database error (not silent failure)
```

**Approve Specification**:
```json
// .specify/features/001-tenant-isolation/spec-metadata.json
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

### Week 5-7: Wave 1 (Foundation)

#### Day 15: Feature 001 Planning

**Command**:
```bash
/speckit.plan 001-tenant-isolation
```

**Output** (`plan.md` generated):
- Phase 0: Context Memory Check
- Phase -1: Constitutional Validation ✅ (TDD, prohibited patterns)
- Phase 1: Requirements Analysis (extracts EARS requirements)
- Phase 2: Implementation Strategy (database migrations, RLS policies, helper functions)
- Phase 2.5: Test Strategy Planning (integration tests with Testcontainers PostgreSQL)
- Phase 2.6: Test Strategy Validation ✅ (Article V compliance)
- Phase 3: Security & Compliance Review ✅ (OWASP Top 10)
- Phase 4: Implementation Tasks (18 tasks, RED-GREEN-REFACTOR order)

#### Day 16-30: Feature 001 Implementation

**Command**:
```bash
/speckit.implement 001-tenant-isolation
```

**Daily Monitoring**:
```bash
/speckit.epic.status multi-tenant-saas

# Day 16 Output:
🌊 WAVE PROGRESS
Wave 1: Foundation (Week 5-7)
├─ Status: 🟢 IN PROGRESS
├─ Progress: ░░░░░░░░░░░░░░░░░░░░ 5% (0/1 complete)
└─ 001 - Tenant Isolation [█░░░░░░░░░░░░░░░░░░░] 5% (Planning)

# Day 20 Output:
└─ 001 - Tenant Isolation [██████░░░░░░░░░░░░░░] 30% (Implementation)

# Day 25 Output:
└─ 001 - Tenant Isolation [██████████████░░░░░░] 70% (Reconciliation)

# Day 30 Output:
Wave 1: Foundation
├─ Status: ✅ COMPLETE
├─ Progress: ████████████████████ 100% (1/1 complete)
└─ 001 - Tenant Isolation [████████████████████] 100% (Complete) ✅
```

**Integration Contract Delivery** (End of Day 30):
```bash
/speckit.epic.status multi-tenant-saas --integration

# Output:
🤝 INTEGRATION POINTS

From: 001 → To: 002 (ITenantService interface)
Status: 🟢 DELIVERED (2025-02-10, on time)
Contract: getTenantContext(req), tenant CRUD operations

From: 001 → To: 003 (tenant_id schema + RLS)
Status: 🟢 DELIVERED (2025-02-10, on time)
Contract: All tables have tenant_id, RLS policies active
```

---

### Week 8-10: Wave 2 (Core Features - Parallel)

#### Team Assignment

**Team A** (2 engineers):
- Feature 002: Tenant Dashboard
- Feature 004: Tenant Admin

**Team B** (2 engineers):
- Feature 003: Tenant Billing

#### Day 31: Wave 2 Kickoff

**Team A**:
```bash
/speckit.plan 002-tenant-dashboard
/speckit.plan 004-tenant-admin
```

**Team B**:
```bash
/speckit.plan 003-tenant-billing
```

**Daily Sync**: Tech lead monitors cross-team integration points

```bash
/speckit.epic.status multi-tenant-saas --integration

# Day 31 Output:
From: 002 → To: 005 (Dashboard UI component)
Status: 🟡 AT RISK (Due: 2025-03-07, Feature 002 in planning, 14 days away)
```

#### Day 32-50: Parallel Development

**Team A Progress**:
```bash
# Day 35
/speckit.implement 002-tenant-dashboard
/speckit.implement 004-tenant-admin

# Day 40
/speckit.epic.status multi-tenant-saas --features

002 - Tenant Dashboard    [████████░░░░░░░░░░░░] 40% (Implementation)
004 - Tenant Admin        [██████████████████░░] 90% (Reconciliation)
```

**Team B Progress**:
```bash
# Day 35
/speckit.implement 003-tenant-billing

# Day 40
003 - Tenant Billing       [██████░░░░░░░░░░░░░░] 30% (Implementation)
```

#### Day 43: Integration Risk Detected

**Command**:
```bash
/speckit.epic.status multi-tenant-saas --integration
```

**Output**:
```
From: 002 → To: 005 (Dashboard UI component)
Status: 🔴 BLOCKED (Due: 2025-03-07, 4 days away, Feature 002 only 50% complete)
Risk: HIGH - Unlikely to meet deadline, may delay Wave 3
```

**Action**: Team lead escalates, adds 1 engineer from Team B to Team A (expedite Feature 002)

#### Day 50: Wave 2 Complete

**Command**:
```bash
/speckit.epic.status multi-tenant-saas

# Output:
🌊 WAVE PROGRESS

Wave 2: Core Features (Week 8-10)
├─ Status: ✅ COMPLETE
├─ Progress: ████████████████████ 100% (3/3 complete)
├─ 002 - Tenant Dashboard [████████████████████] 100% (Complete) ✅
├─ 003 - Tenant Billing   [████████████████████] 100% (Complete) ✅
└─ 004 - Tenant Admin     [████████████████████] 100% (Complete) ✅

Integration Points: 100% delivered (6/6 on track)
```

---

### Week 11: Wave 3 (Integration)

#### Day 51: Feature 005 Planning & Implementation

**Commands**:
```bash
/speckit.plan 005-tenant-onboarding
/speckit.implement 005-tenant-onboarding
```

**Integration Testing** (Day 54):
- End-to-end tenant onboarding flow:
  1. Create tenant (Feature 001)
  2. Generate dashboard (Feature 002)
  3. Set up billing (Feature 003)
  4. Create admin user (Feature 004)
  5. Complete onboarding (Feature 005)

**All features working together** ✅

#### Day 57: Epic Complete

**Command**:
```bash
/speckit.epic.status multi-tenant-saas --all
```

**Final Output**:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   EPIC STATUS: EPIC-001 Multi-Tenant SaaS Platform
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 OVERVIEW
├─ Status: ✅ COMPLETE
├─ Progress: ████████████████████ 100% (5/5 features complete)
├─ Timeline: 6 weeks (40% savings vs 10 weeks sequential)
└─ Quality: GOOD (8.5/10 average)

🌊 WAVE PROGRESS
Wave 1: ✅ COMPLETE (1/1)
Wave 2: ✅ COMPLETE (3/3)
Wave 3: ✅ COMPLETE (1/1)

📊 QUALITY METRICS
├─ Overall: GOOD (8.5/10)
├─ Critical Issues: 0 (all resolved)
├─ Test Coverage: 88.2% ✅
└─ Security: ✅ PASSED (OWASP Top 10)

🎯 BUSINESS GOALS ACHIEVED
✅ Revenue: Platform supports 100+ tenants
✅ Cost: Per-customer cost reduced 72%
✅ Speed: Onboarding time <45 minutes
```

---

## Example 2: Payment Processing System (High-Risk Epic)

**Scenario**: Building payment processing with Stripe integration
**Features**: 4 features (Gateway Integration, Subscription Management, Invoice Generation, Payment History)
**Risk**: 🔴 HIGH (score: 11/12 - financial data, PCI-DSS compliance)
**Timeline**: 8 weeks (HIGH-risk features require extra scrutiny)

### Key Differences from Example 1

**Epic Creation**:
- **Security Review**: Required before decomposition
- **Compliance Validation**: PCI-DSS, SOC 2 requirements documented in epic-architecture.md

**ADRs with Security Focus**:
```markdown
### AD-EPIC-001: Never Store Credit Card Data

**Decision**: Use Stripe's tokenization - never store raw credit card data

**Constitutional Compliance**:
- Article V Prohibited Patterns: `credit_card_number` column in any table

**Validation**: Step 10.4.5 scans codebase for prohibited patterns
```

**Gap Analysis** (Mandatory for HIGH-risk):
```bash
# Run gap analysis for ALL features (not optional)
/speckit.validate-gap 001-stripe-integration
/speckit.validate-gap 002-subscription-management
/speckit.validate-gap 003-invoice-generation
/speckit.validate-gap 004-payment-history
```

**Security Review Checkpoints**:
- Week 2: Epic-level security review (OWASP Top 10)
- Week 4: Feature-level penetration testing (per feature)
- Week 8: Final security audit before launch

---

## Example 3: Analytics Dashboard (Single Team, Sequential)

**Scenario**: Building analytics dashboard with 3 data visualization features
**Features**: 3 features (Data Aggregation, Chart Library, Dashboard UI)
**Teams**: 1 team (3 engineers, sequential development)
**Timeline**: 6 weeks (no parallelization)

### Simplified Workflow

**No Parallelization** (single team):
- Wave 1: Feature 001 (Data Aggregation) - Week 1-2
- Wave 2: Feature 002 (Chart Library) - Week 3-4
- Wave 3: Feature 003 (Dashboard UI) - Week 5-6

**Daily Status**:
```bash
/speckit.epic.status analytics-dashboard

# Simpler output (no parallel coordination)
📊 OVERVIEW
├─ Progress: ████████░░░░░░░░░░░░ 40% (1/3 complete)
└─ Current: Wave 2 (Feature 002 in progress)

💡 RECOMMENDATIONS
🟢 Ready to Start: None (sequential workflow)
🔴 Blocked: Feature 003 (waiting for Feature 002)
```

**Less Integration Complexity**:
- Features tightly coupled (Chart Library uses Data Aggregation, Dashboard UI uses both)
- No cross-team coordination needed
- Epic still provides: ADRs, dependency tracking, progress monitoring

---

## Example 4: Microservices Migration (Retrofit Existing Features)

**Scenario**: Migrate monolith to microservices, 4 services already implemented
**Features**: 4 existing features (User Service, Order Service, Inventory Service, Notification Service)
**Goal**: Create epic retroactively for documentation and future coordination

### Retrofit Workflow

#### Step 1: Create Epic Retroactively

```bash
/speckit.epic.create microservices-migration --features 001,002,003,004
```

**Mark Features as COMPLETE**:
```json
{
  "features": [
    {
      "feature_id": "001",
      "feature_name": "User Service",
      "status": "COMPLETE",
      "phase": "complete",
      "start_date": "2024-10-01",
      "completion_date": "2024-10-15"
    }
    // ... (002, 003, 004)
  ]
}
```

#### Step 2: Document Shared Architecture (Manual)

**Add ADRs based on existing implementation**:
```markdown
### AD-EPIC-001: gRPC for Inter-Service Communication

**Status**: Accepted
**Date**: 2024-10-01 (retrospective)
**Decision**: Use gRPC for synchronous service-to-service calls

**Rationale**: Already implemented in all 4 services, provides type safety and performance

**Implementation Guidance**:
[Extract code examples from existing services]
```

#### Step 3: Link Existing Features to Epic

**Update spec-metadata.json** (per feature):
```json
{
  "epic_id": "EPIC-002",
  "epic_name": "Microservices Migration",
  "retrofitted_to_epic": true,
  "retrofit_date": "2025-01-15"
}
```

#### Step 4: Use Epic for Future Features

**New Feature 005** (Billing Service):
```bash
# Generate spec from epic
/speckit.epic.decompose microservices-migration --features 005

# Inherits ADRs from epic (gRPC, service mesh, etc.)
# Integration contracts with existing services (001-004)
```

**Benefit**: New services consistent with existing architecture (via ADR inheritance)

---

## Summary

These examples demonstrate:
1. **Complete Epic Workflow** (Multi-Tenant SaaS): Full lifecycle from planning to completion
2. **HIGH-Risk Epic** (Payment Processing): Security-focused workflow with extra validation
3. **Single-Team Epic** (Analytics Dashboard): Simplified workflow without parallelization
4. **Retrofit Epic** (Microservices Migration): Applying epic to existing features

**Key Takeaways**:
- Epic management scales from simple (3 features, 1 team) to complex (5+ features, 2+ teams)
- ADRs ensure architectural consistency across features
- Integration point monitoring prevents "works in isolation, fails in integration"
- Daily status checks (`/speckit.epic.status`) keep teams aligned

**Next Steps**:
- Review [EPIC-MANAGEMENT-GUIDE.md](./EPIC-MANAGEMENT-GUIDE.md) for detailed guidance
- Review [MIGRATION-v2.8.md](./MIGRATION-v2.8.md) for upgrade instructions
- Create your first epic: `/speckit.epic.create <your-epic-name>`

---

**Examples Version**: 2.8.3
**Last Updated**: 2025-01-15
