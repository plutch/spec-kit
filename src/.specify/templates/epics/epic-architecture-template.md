# Epic Architecture: [EPIC_NAME]

**Epic ID**: EPIC-[NUMBER]
**Version**: 1.0.0
**Last Updated**: [YYYY-MM-DD]

---

## Architectural Decisions

### AD-[EPIC]-001: [Decision Title]

**Status**: [Proposed / Accepted / Superseded]
**Date**: [YYYY-MM-DD]
**Deciders**: [Names/Roles]
**Impact**: [All features / Specific features]

**Context**:
[Why did this decision need to be made? What were the constraints, requirements, or problems driving this decision?]

**Decision**:
[What was decided? Be specific and actionable. Include technical details.]

**Consequences**:
- **Positive**:
  - [Benefit 1]
  - [Benefit 2]
- **Negative**:
  - [Trade-off 1]
  - [Trade-off 2]

**Alternatives Considered**:
1. **[Alternative 1]**: [Why rejected - pros/cons]
2. **[Alternative 2]**: [Why rejected - pros/cons]

**Implementation Guidance**:
```typescript
// Example code showing how to apply this decision
// Include concrete examples that developers can copy/adapt
```

**Affected Features**: [Feature IDs that must follow this decision]

---

### AD-[EPIC]-002: [Another Decision Title]

[Same structure as above]

---

**Example** (Multi-Tenant SaaS):

### AD-EPIC-001: Multi-Tenancy Data Isolation Strategy

**Status**: Accepted
**Date**: 2025-01-15
**Deciders**: Tech Lead, DBA, Security Team
**Impact**: All features (001-005)

**Context**:
We need to isolate data between tenants in our SaaS application. Options include database-per-tenant, schema-per-tenant, or row-level security. We have 100+ expected tenants, limited DBA resources, and strict data isolation requirements.

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

---

## Cross-Feature Patterns

### Pattern 1: [Pattern Name]

**Use Case**: [When to use this pattern]

**Problem**: [What problem does this pattern solve?]

**Solution**:
[Description of the pattern]

**Implementation**:
```typescript
// Code example showing the pattern
```

**Used By**: [Features that use this pattern]

**Example**:

### Pattern 1: Tenant Context Propagation

**Use Case**: Every API request needs tenant context for RLS

**Problem**: How to safely pass tenant context from HTTP request through all layers (controller → service → repository) without passing tenant_id as function parameter everywhere?

**Solution**:
Use middleware to extract tenant from JWT, store in request context, and provide helper to retrieve tenant context in any layer.

**Implementation**:
```typescript
// Middleware (runs on every request)
export function tenantMiddleware(req: Request, res: Response, next: NextFunction) {
  const token = extractJWT(req.headers.authorization);
  const tenantId = token.claims.tenant_id;
  req.tenantContext = { tenantId, tenantSlug: token.claims.tenant_slug };
  next();
}

// Helper to get tenant context
export function getTenantContext(req: Request): TenantContext {
  if (!req.tenantContext) {
    throw new Error('Tenant context not set - did you forget tenant middleware?');
  }
  return req.tenantContext;
}

// Usage in any layer
export class OrderService {
  async createOrder(req: Request, data: CreateOrderDTO) {
    const tenant = getTenantContext(req);
    return this.orderRepo.create({ ...data, tenantId: tenant.tenantId });
  }
}
```

**Used By**: All features (001-005)

---

## Integration Points

### Integration Point 1: [Integration Name]

**Purpose**: [Why this integration point exists]

**Provider Feature**: [Feature ID that provides this]
**Consumer Features**: [Feature IDs that consume this]

**Contract**:
```typescript
// TypeScript interface or API contract
export interface [InterfaceName] {
  // Methods, properties, events
}
```

**Dependencies**: [Prerequisites for this integration]

**Stability**: [🔒 Locked / ⚠️ May change / 🚧 Under development]

**Breaking Change Policy**: [How breaking changes are handled]

---

**Example**:

### Integration Point 1: Tenant Management Service

**Purpose**: Centralized tenant CRUD operations used by all features

**Provider Feature**: 001 (Tenant Isolation)
**Consumer Features**: 002 (Dashboard), 003 (Billing), 004 (Admin), 005 (Onboarding)

**Contract**:
```typescript
export interface ITenantService {
  // Core operations
  createTenant(data: CreateTenantDTO): Promise<Tenant>;
  getTenant(tenantId: string): Promise<Tenant>;
  updateTenant(tenantId: string, data: UpdateTenantDTO): Promise<Tenant>;
  deleteTenant(tenantId: string): Promise<void>;

  // Tenant resolution
  resolveTenantBySubdomain(subdomain: string): Promise<Tenant>;
  resolveTenantByContext(req: Request): Promise<Tenant>;

  // Tenant validation
  validateTenantExists(tenantId: string): Promise<boolean>;
  checkTenantLimits(tenantId: string, resource: string): Promise<boolean>;
}

export interface Tenant {
  id: string;
  slug: string;
  name: string;
  subdomain: string;
  plan: 'free' | 'pro' | 'enterprise';
  createdAt: Date;
  settings: TenantSettings;
}
```

**Dependencies**: PostgreSQL with RLS enabled, JWT with tenant claim

**Stability**: 🔒 Locked after Feature 001 complete

**Breaking Change Policy**: Feature 001 contract must remain stable. If changes needed, create v2 interface and deprecate v1 with 6-month notice.

---

## Shared Libraries & Modules

### Module 1: [Module Name]

**Purpose**: [What this module does]

**Location**: `.specify/epics/[epic-name]/shared/[module-name]`

**Exports**:
```typescript
// Public API
export interface [...];
export function [...];
export class [...];
```

**Used By**: [Feature IDs]

**Ownership**: [Which feature creates/maintains this]

**Installation**:
```bash
# How features import/use this module
```

---

**Example**:

### Module 1: Tenant Context Library

**Purpose**: Shared utilities for tenant context management

**Location**: `.specify/epics/multi-tenant-dashboard/shared/tenant-context/`

**Exports**:
```typescript
// tenant-context.ts
export interface TenantContext {
  tenantId: string;
  tenantSlug: string;
  tenantName: string;
  tenantPlan: 'free' | 'pro' | 'enterprise';
  userId: string;
  userRole: 'admin' | 'member' | 'viewer';
}

export function tenantMiddleware(req, res, next): void;
export function getTenantContext(req): TenantContext;
export function setTenantContext(db, tenantId): Promise<void>;
```

**Used By**: All features (001-005)

**Ownership**: Feature 001 (Tenant Isolation) - other features consume only

**Installation**:
```typescript
// Import in any feature
import { getTenantContext, tenantMiddleware } from '@epic/tenant-context';
```

---

## Technology Choices

| Category | Choice | Rationale | Impact |
|----------|--------|-----------|--------|
| [e.g., Database] | [e.g., PostgreSQL 15+] | [Why this choice] | [Features affected] |
| [e.g., Authentication] | [e.g., JWT with custom claims] | [Why this choice] | [Features affected] |
| [e.g., Caching] | [e.g., Redis for tenant data] | [Why this choice] | [Features affected] |

**Example**:
| Category | Choice | Rationale | Impact |
|----------|--------|-----------|--------|
| Database | PostgreSQL 15+ with RLS | Native RLS support, battle-tested, cost-effective | All features |
| Authentication | JWT with tenant claim | Stateless, subdomain resolution via claim | All features |
| Caching | Redis for tenant lookup | Avoid DB hit on every request, <5ms tenant resolution | Features 001-005 |
| Queue | Bull (Redis-based) | Tenant-scoped background jobs (billing, notifications) | Features 003, 005 |

---

## Constitutional Compliance (Epic-Wide)

### Article I: Test-Driven Development (TDD)
**Epic Requirement**: ALL features MUST follow strict TDD (RED-GREEN-REFACTOR)

**Epic-Level Enforcement**:
- Each feature spec includes TDD requirements section
- Each feature plan includes RED-GREEN-REFACTOR checkpoints
- Step 10.4 Constitutional Reviewer validates TDD compliance
- Epic completion blocked if ANY feature violates TDD

### Article III: Integration-First Testing
**Epic Requirement**: Multi-tenant features MUST test with real tenant context, real database

**Prohibited Patterns** (Epic-Specific):
```typescript
// ❌ PROHIBITED - Mocked tenant context in integration tests
const mockTenantContext = jest.fn(() => ({ tenantId: 'fake-tenant' }));

// ✅ REQUIRED - Real tenant context via test fixtures
const testTenant = await createTestTenant({ name: 'Test Corp' });
const tenantContext = await resolveTenant(testTenant.subdomain);
```

**Epic-Level Validation**:
- Scan all feature test files for mocked tenant context (Phase 2.6)
- Validate integration tests use Testcontainers PostgreSQL
- Block epic if prohibited patterns found

### Article VIII: Simplicity Over Abstraction
**Epic Requirement**: Use PostgreSQL RLS directly, no custom multi-tenant framework

**Rationale**: Multi-tenancy frameworks add 10-20 dependencies, increase complexity, harder to debug

**Epic-Level Enforcement**:
- No custom ORM wrappers for tenant scoping
- Use raw SQL RLS policies (documented in epic-architecture.md)
- Share RLS patterns via epic memory file

---

## Quality Standards (Epic-Wide)

| Standard | Target | Validation | Blocking |
|----------|--------|------------|----------|
| Test Coverage | ≥85% (high-risk epic) | Per-feature coverage reports | Yes - if <80% |
| Performance | <50ms tenant resolution (p95) | Load testing with 100+ tenants | Yes - if >100ms |
| Security | Zero cross-tenant data leaks | Security audit + penetration testing | Yes - if leaks found |
| Accessibility | WCAG 2.1 AA (UI features) | Automated + manual accessibility testing | No - warn only |

---

## Migration & Rollback

### Data Migration Strategy
[How existing data will be migrated to the new architecture, if applicable]

### Feature Flags
[Which features will be behind feature flags, how they'll be enabled/disabled]

### Rollback Plan
[How to rollback if epic deployment fails]

**Example**:

### Data Migration Strategy
1. **Phase 1** (Week 1): Add tenant_id column to all tables (nullable)
2. **Phase 2** (Week 2): Backfill tenant_id for existing data (default tenant)
3. **Phase 3** (Week 3): Make tenant_id NOT NULL, enable RLS policies
4. **Phase 4** (Week 4-6): Deploy features, create new tenants

### Feature Flags
- `multi_tenant_enabled` (global) - Master switch for entire epic
- `tenant_dashboard_enabled` (Feature 002) - Per-tenant dashboard visibility
- `tenant_billing_enabled` (Feature 003) - Per-tenant billing features

### Rollback Plan
1. Disable feature flags (immediate rollback)
2. If RLS issues: Disable RLS policies, revert to single-tenant mode
3. If data corruption: Restore from backup (pre-migration snapshot)

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0.0 | [YYYY-MM-DD] | Initial architecture | [Name] |

---

**Epic Architecture Status**: [Draft / In Review / Approved / Superseded]
**Approved By**: [Name/Role]
**Approved Date**: [YYYY-MM-DD]
