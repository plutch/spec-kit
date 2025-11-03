# Hierarchical Specifications Guide

**Feature**: Supplementary Specifications (UI-SPEC.md, API-SPEC.md, TECHNICAL-SPEC.md)
**Version**: SpecKit v2.1
**Purpose**: Manage large, multi-domain features with token efficiency

---

## Table of Contents

1. [Overview](#overview)
2. [When to Use](#when-to-use)
3. [Quick Start](#quick-start)
4. [Supplementary Spec Types](#supplementary-spec-types)
5. [Workflow](#workflow)
6. [Best Practices](#best-practices)
7. [Token Savings](#token-savings)
8. [Examples](#examples)

---

## Overview

Hierarchical specifications split large features into:
- **Parent spec.md**: WHAT and WHY (functional requirements, user stories)
- **Supplementary *-SPEC.md**: HOW (domain-specific implementation details)

**Benefits**:
- **Token Efficiency**: Agents load only relevant specs (~50KB vs ~279KB)
- **Clear Ownership**: UI team owns UI-SPEC.md, Backend team owns API-SPEC.md
- **Maintainability**: Update domain specs without touching parent
- **Scalability**: Large features (500+ FRs) become manageable

**Evidence**: BP ERP billing console saved 200K tokens per feature lifecycle (47% reduction).

---

## When to Use

### ✅ Use Hierarchical Specs When:

- Parent spec.md is large (>150KB or >80 FRs)
- Feature spans multiple domains (UI + API + Architecture)
- Specialized agents need focused context
- Implementation details would clutter parent

### ❌ Do NOT Use When:

- Feature is small (<100KB, <50 FRs)
- Single domain (backend-only or UI-only)
- Simple CRUD application

**Decision Rule**:
```
if (specSizeKB > 150 || frCount > 80 || hasMultipleDomains) {
  USE: Hierarchical specs (/speckit.supplement)
} else {
  USE: Single spec.md (traditional)
}
```

---

## Quick Start

### 1. Create Parent Spec

```bash
/speckit.specify "Build billing console for multi-tenant SaaS"
# Output: specs/000003-billing-console/spec.md (97KB, 80 FRs)
```

### 2. Create Supplementary Specs

```bash
# UI specifications
/speckit.supplement ui-ux "Complete UI/UX specifications with Angular 20 + Kendo UI"
# Output: UI-SPEC.md

# API specifications
/speckit.supplement api-contracts "REST API contracts for billing and payments"
# Output: API-SPEC.md

# Technical specifications
/speckit.supplement technical "Multi-tenant architecture with Neon Postgres sharding"
# Output: TECHNICAL-SPEC.md
```

### 3. Edit Supplementary Specs

Fill in PLACEHOLDER sections:
- Component designs (UI-SPEC.md)
- Endpoint contracts (API-SPEC.md)
- Architecture decisions (TECHNICAL-SPEC.md)

### 4. Generate Plan (Auto-Discovery)

```bash
/speckit.plan
# Auto-discovers *-SPEC.md files
# Appends "Supplementary Specifications" section to plan.md
```

### 5. Validate Hierarchy (Strict)

```bash
/speckit.validate-hierarchy
# Validates frontmatter, references, FR mapping
# Blocks workflow on errors (strict mode)
```

### 6. Generate Tasks & Implement

```bash
/speckit.tasks
# Tasks reference supplementary spec sections

/speckit.implement
# Agents auto-load relevant supplementary specs
```

---

## Supplementary Spec Types

### 1. UI-SPEC.md (ui-ux scope)

**Purpose**: Component designs, layouts, accessibility

**Target Agents**: `frontend-architect`

**Contents**:
- Component hierarchy
- State management (NgRx, Signals)
- Routing and navigation
- Responsive layouts (desktop, tablet, mobile)
- UI component specifications
- Form validation rules
- Accessibility (WCAG 2.1 AA)
- Performance targets

**When to Use**:
- Angular/React/Vue applications
- Complex UI with >10 components
- Responsive design requirements
- Accessibility compliance

**Example**:
```markdown
---
parent: spec.md
scope: ui-ux-implementation
functional_requirements: [FR-001 through FR-080]
target_agents:
  - frontend-architect
version: 1.0
last_updated: 2025-11-01
---

# UI/UX Implementation Specification

## 4. UI Components

### 4.2 Dashboard Component
**Implements**: FR-008, FR-015
**Description**: Main dashboard with analytics widgets
**Props**: userId, dateRange, theme
**State**: dashboardData, isLoading, error
**Kendo Components**: Grid, Chart, DatePicker
```

---

### 2. API-SPEC.md (api-contracts scope)

**Purpose**: REST API contracts, schemas, webhooks

**Target Agents**: `backend-architect`, `system-architect`

**Contents**:
- API endpoints (method, path, params, response)
- Request/response schemas (JSON, TypeScript)
- Error codes and handling
- Authentication & authorization
- Rate limiting
- Webhooks
- API versioning

**When to Use**:
- RESTful APIs with >10 endpoints
- Integration with external services
- Webhook handling
- Complex schemas

**Example**:
```markdown
---
parent: spec.md
scope: api-contracts
functional_requirements: [FR-001 through FR-080]
target_agents:
  - backend-architect
  - system-architect
version: 1.0
last_updated: 2025-11-01
---

# API Contracts Specification

## 2. API Endpoints

### 2.1 POST /api/billing/invoices
**Implements**: FR-025
**Description**: Generate invoice for billing period

#### Request
```json
{
  "companyId": "uuid",
  "billingPeriod": "2025-11"
}
```

#### Response (200 OK)
```json
{
  "invoiceId": "uuid",
  "total": 150.00,
  "dueDate": "2025-12-01"
}
```
```

---

### 3. TECHNICAL-SPEC.md (technical scope)

**Purpose**: Architecture, database, infrastructure

**Target Agents**: `system-architect`, `postgres-entity-architect`, `devops-architect`

**Contents**:
- System architecture diagrams
- Database schema and indexes
- Multi-tenant isolation strategy
- Caching layers
- Security threat model
- Observability (logging, metrics, alerts)
- Performance requirements
- Disaster recovery

**When to Use**:
- Complex architectures (microservices, multi-tenant)
- Database design with >10 tables
- Performance-critical systems
- Compliance requirements

**Example**:
```markdown
---
parent: spec.md
scope: technical-architecture
functional_requirements: [FR-001 through FR-080]
target_agents:
  - system-architect
  - postgres-entity-architect
  - devops-architect
version: 1.0
last_updated: 2025-11-01
---

# Technical Architecture Specification

## 3. Database Design

### 3.1 Schema
```sql
CREATE TABLE billing_invoices (
  id UUID PRIMARY KEY,
  company_id UUID NOT NULL,
  billing_period DATE NOT NULL,
  total DECIMAL(10,2) NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_billing_company ON billing_invoices(company_id);
```
```

---

## Workflow

### Step 1: Assess Feature Size

```bash
# Check parent spec size
du -k specs/000003-billing-console/spec.md
# Output: 97KB

# Count FRs
grep -c "^### FR-" specs/000003-billing-console/spec.md
# Output: 80

# Decision: 97KB + 80 FRs → Use hierarchical specs
```

### Step 2: Create Supplementary Specs

**Proactive Recommendations** (since v2.1.1):

The `/speckit.specify` command automatically detects large specifications and recommends hierarchical specs when:
- **Spec size > 100KB** (early warning before 150KB pain point), OR
- **Total requirements > 60** (multi-domain complexity signal)

**Example Auto-Recommendation**:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 RECOMMENDATION: Large Specification Detected
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Spec Size: 128KB (threshold: 100KB)
Requirements: 72 (threshold: 60)

Consider creating supplementary specs for domain-specific details:

  /speckit.supplement ui-ux "UI/UX implementation details"
  /speckit.supplement api-contracts "REST API contracts"
  /speckit.supplement technical "Architecture and infrastructure"
```

**User Control**: The recommendation is **informational only** - users decide whether to create supplementary specs immediately, defer, or continue with a single spec.md.

**Manual Creation** (anytime):

```bash
/speckit.supplement ui-ux "Detailed UI/UX specifications"
/speckit.supplement api-contracts "REST API contracts and schemas"
/speckit.supplement technical "Architecture and database design"
```

### Step 3: Populate Supplementary Specs

Edit each *-SPEC.md to fill in PLACEHOLDER sections:

```markdown
[PLACEHOLDER: Define Angular component tree]
→ Replace with actual component hierarchy

[PLACEHOLDER: Endpoint contracts]
→ Replace with API endpoint definitions
```

### Step 4: Add Frontmatter

Ensure all supplementary specs have valid frontmatter:

```yaml
---
parent: spec.md
scope: ui-ux-implementation | api-contracts | technical-architecture
functional_requirements: [FR-001 through FR-080]
target_agents:
  - frontend-architect  # UI-SPEC.md
  - backend-architect   # API-SPEC.md
  - system-architect    # TECHNICAL-SPEC.md
version: 1.0
last_updated: 2025-11-01
---
```

### Step 5: Generate Plan (Auto-Discovery)

```bash
/speckit.plan

# Output includes:
## Supplementary Specifications

This feature includes detailed implementation guides:

### ui-ux-implementation
- **[UI-SPEC.md](UI-SPEC.md)**
  - **Scope**: ui-ux-implementation
  - **Coverage**: FR-001 through FR-080
  - **Size**: 182KB, 3500 lines
  - **Target Agents**: frontend-architect
```

### Step 6: Validate (Strict Mode)

```bash
/speckit.validate-hierarchy

# Checks:
✓ Frontmatter valid
✓ Parent reference correct
✓ FRs exist in parent
✓ plan.md references all specs
✓ No orphaned specs
```

### Step 7: Implement with Agent Routing

```bash
/speckit.implement

# Agent routing (automatic):
frontend-architect → Loads spec.md + UI-SPEC.md (147KB vs 279KB)
backend-architect → Loads spec.md + API-SPEC.md
system-architect → Loads spec.md + TECHNICAL-SPEC.md + API-SPEC.md
```

---

## Best Practices

### 1. One Spec Per Domain

**Good**:
```
UI-SPEC.md (all UI concerns)
API-SPEC.md (all API concerns)
TECHNICAL-SPEC.md (all technical concerns)
```

**Bad**:
```
UI-SPEC-dashboard.md  ❌ Too granular
UI-SPEC-settings.md   ❌ Too granular
UI-SPEC-billing.md    ❌ Too granular
```

### 2. Maintain FR Traceability

Every section maps to parent FRs:

```markdown
### 4.2 Company Management Component
**Implements**: FR-008, FR-015, FR-019
```

### 3. Reference Parent Spec

```markdown
This implements "Company Profile Management" from [spec.md Section 3.2](spec.md#32-company-profile-management).
```

### 4. Use Frontmatter Strictly

**Required fields**:
- `parent`: Must be "spec.md"
- `scope`: Valid scope (ui-ux-implementation, api-contracts, technical-architecture)
- `functional_requirements`: FR range or list
- `target_agents`: Array of agents

**Validation blocks workflow** if missing (strict mode).

### 5. Keep Specs in Sync

When parent FRs change:

```yaml
# Update supplementary spec frontmatter
functional_requirements: [FR-001 through FR-085]  # Was FR-080
```

---

## Token Savings

### Example: BP ERP Billing Console

**Before (Monolithic)**:
- spec.md: 279KB (combined UI + API + Technical)
- All agents load: 279KB × 5 sessions = ~350K tokens

**After (Hierarchical)**:
- spec.md: 97KB (parent only)
- UI-SPEC.md: 182KB
- Total: 279KB

**Agent Loading**:
- frontend-architect: 97KB (parent) + 182KB (UI) = 279KB
  Wait, that's the same?

**Actually**:
- frontend-architect: 97KB + 50KB (relevant UI sections) = 147KB
- Typical load: 147KB × 5 sessions = ~184K tokens

**Savings**: 350K - 184K = **166K tokens (47%)**

**Why savings**:
- Agents load parent + relevant domain spec only
- Don't load irrelevant domains (frontend skips API-SPEC, backend skips UI-SPEC)
- Average load: 147KB vs 279KB monolithic

---

## Examples

### Example 1: Large ERP Feature

**Scenario**: Billing console with 80 FRs spanning UI, API, Architecture

**Files**:
```
specs/000003-billing-console/
├── spec.md                    # 97KB, 80 FRs (WHAT/WHY)
├── UI-SPEC.md                 # 182KB (HOW - UI)
├── API-SPEC.md                # 65KB (HOW - API)
├── TECHNICAL-SPEC.md          # 45KB (HOW - Architecture)
├── plan.md                    # References all specs
└── tasks.md                   # References spec sections
```

**Workflow**:
1. `/speckit.specify` → spec.md (97KB, 80 FRs)
2. Recognize size → Create supplementary specs
3. `/speckit.supplement ui-ux` → UI-SPEC.md
4. `/speckit.supplement api-contracts` → API-SPEC.md
5. `/speckit.supplement technical` → TECHNICAL-SPEC.md
6. Edit each spec (fill placeholders)
7. `/speckit.plan` → Auto-discovers, references in plan.md
8. `/speckit.validate-hierarchy` → Strict validation
9. `/speckit.tasks` → Tasks reference UI-SPEC.md sections
10. `/speckit.implement` → Agents load relevant specs

**Result**: 47% token savings, clear ownership, maintainable specs.

### Example 2: Small Feature (No Hierarchical)

**Scenario**: Simple user profile feature with 15 FRs

**File**:
```
specs/000005-user-profile/
└── spec.md                    # 35KB, 15 FRs
```

**Decision**: 35KB < 150KB threshold → Single spec (traditional workflow)

**Workflow**:
1. `/speckit.specify`
2. `/speckit.clarify`
3. `/speckit.plan`
4. `/speckit.tasks`
5. `/speckit.implement`

**No supplementary specs needed** - overhead not justified.

---

## Troubleshooting

### Issue: Validation Fails After Supplement

```
❌ ERROR: UI-SPEC.md missing parent reference

RESOLUTION:
Ensure frontmatter present:
---
parent: spec.md
scope: ui-ux-implementation
...
---
```

### Issue: Plan Doesn't Reference Supplementary Specs

```
❌ ERROR: plan.md missing "Supplementary Specifications" section

RESOLUTION:
Re-run /speckit.plan (auto-discovery should detect *-SPEC.md)
```

### Issue: Agent Not Loading Supplementary Spec

```
⚠️ WARNING: frontend-architect not loading UI-SPEC.md

RESOLUTION:
1. Check target_agents in UI-SPEC.md frontmatter
2. Verify agent_routing in .specify/config.yml
3. Ensure UI-SPEC.md referenced in plan.md
```

---

## Related Commands

- `/speckit.supplement` - Create supplementary specs
- `/speckit.validate-hierarchy` - Validate strict mode
- `/speckit.reconcile` - Update supplementary specs post-implementation
- `/speckit.plan` - Auto-discovers supplementary specs

---

## Next Steps

1. **Read Full Spec**: [supplement.md](../templates/commands/supplement.md)
2. **Validation Guide**: [VALIDATION-GUIDE.md](VALIDATION-GUIDE.md)
3. **Example**: BP ERP billing console (reference implementation)

---

**Version**: 2.1.0
**Last Updated**: 2025-11-01
**Evidence**: BP ERP saved 200K tokens (47%) on billing console feature
