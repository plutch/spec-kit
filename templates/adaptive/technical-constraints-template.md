# Technical Constraints: [Feature Name]

**Feature ID**: ###-feature-name
**Created**: YYYY-MM-DD
**Last Updated**: YYYY-MM-DD
**Status**: [Active | Amended | Superseded]

---

## Purpose

This document captures **architectural boundaries and technical constraints** for implementing the feature described in [`spec.md`](spec.md). Unlike the specification (what/why), this defines implementation boundaries (how/with what).

**When to use TECHNICAL.md**:
✅ Multi-tenancy/sharding requirements
✅ Performance targets (P95 <250ms, >1000 RPS)
✅ New/untested frameworks
✅ Regulatory constraints (GDPR, SOC2, HIPAA)
✅ Architectural unknowns
✅ Cross-team dependencies

❌ Skip for simple CRUD features with known tech stack

---

## Multi-Tenancy Strategy

> **Required if**: Feature handles tenant-specific data

### Pattern
- **Isolation Model**: [Schema-per-tenant | Row-level security | Database-per-tenant | Single schema with tenant_id]
- **Tenant Identification**: [Subdomain | Path parameter | JWT claim | API key]
- **Connection Strategy**: [Connection pooling | Schema switching | Dynamic routing]

### Example
```typescript
// Schema-per-tenant with SET search_path
await client.query('SET search_path TO company_$1', [tenantId]);
```

### Compliance
- [ ] Data isolation verified (no cross-tenant queries)
- [ ] Tenant context propagated through all layers
- [ ] Migration strategy handles tenant schemas

---

## Performance Targets

> **Required if**: Feature has specific performance requirements

### Latency
- **API Response**: P95 <[X]ms, P99 <[Y]ms
- **Database Queries**: [N] queries max per request
- **External APIs**: Timeout [X]ms with fallback

### Throughput
- **Peak Load**: [N] requests/second
- **Sustained Load**: [N] requests/second
- **Concurrent Users**: Support [N] simultaneous users

### Resource Limits
- **Memory**: <[X]MB per request
- **CPU**: <[X]% average utilization
- **Storage**: [X]GB total, [Y]MB per tenant

### Measurement
```bash
# Load test command
artillery run load-test.yml --target https://api.example.com

# Expected results
# P95: <250ms ✅
# P99: <500ms ✅
# Success rate: >99.9% ✅
```

---

## Technology Assumptions

> **Required if**: Feature uses specific technologies/frameworks

### Primary Stack
- **Runtime**: [Node.js 20+ | Python 3.11+ | Go 1.21+]
- **Framework**: [Express | FastAPI | Gin]
- **Database**: [PostgreSQL 15+ | MySQL 8+ | MongoDB 6+]
- **ORM/Query**: [Prisma | SQLAlchemy | GORM | Raw SQL]

### Infrastructure
- **Deployment**: [Cloudflare Workers | AWS Lambda | Kubernetes]
- **Caching**: [Redis | Memcached | In-memory | None]
- **Queue**: [Bull/Redis | AWS SQS | None]
- **Storage**: [S3 | R2 | Local filesystem]

### Example
```typescript
// Using node-postgres for multi-tenant queries
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  database: 'tenants',
  max: 100 // Connection pool size
});
```

---

## Regulatory Constraints

> **Required if**: Feature handles regulated data

### Compliance Requirements
- [ ] **GDPR**: Right to erasure, data portability, consent tracking
- [ ] **HIPAA**: PHI encryption, audit logs, access controls
- [ ] **SOC 2**: Security controls, monitoring, incident response
- [ ] **PCI DSS**: Cardholder data protection, secure transmission
- [ ] **[Custom]**: [Project-specific regulations]

### Data Retention
- **Retention Period**: [N days/months/years]
- **Deletion Process**: [Automated | Manual with approval]
- **Backup Strategy**: [Frequency, retention, encryption]

### Audit Requirements
- [ ] All data access logged
- [ ] Log retention: [N] months minimum
- [ ] Audit reports: [Frequency and format]

---

## Risk Flags

> **Required**: Document architectural unknowns

Use `[NEEDS VALIDATION]` markers for areas requiring investigation before implementation.

### Architecture Risks
- [NEEDS VALIDATION] **Multi-tenant scaling**: Current ORM may not support 10,000+ schemas efficiently
  - **Investigation**: Load test with 1000 tenants
  - **Deadline**: Week 2 of implementation
  - **Fallback**: Migrate to raw SQL if ORM doesn't scale

- [NEEDS VALIDATION] **RLS performance**: Row-level security impact on query latency unknown
  - **Investigation**: Benchmark RLS vs schema-per-tenant
  - **Deadline**: During planning phase
  - **Fallback**: Implement schema-per-tenant if RLS >100ms overhead

### Technology Risks
- [NEEDS VALIDATION] **Framework compatibility**: New framework version untested with project
  - **Investigation**: Prototype in isolated environment
  - **Deadline**: Week 1 of implementation
  - **Fallback**: Stay on current version

### Integration Risks
- [NEEDS VALIDATION] **External API reliability**: Third-party API SLA unknown
  - **Investigation**: Contact vendor for SLA documentation
  - **Deadline**: Before implementation starts
  - **Fallback**: Implement circuit breaker pattern

---

## Constitutional Compliance

> **Required**: Validate against project constitution

### Article VII: Simplicity Gate
- **Projects Used**: [N] (must be ≤3 for most projects)
- **Justification**: [Why additional complexity is warranted]
- **Approval**: [ ] Required | [X] Not Required

### Article VIII: Anti-Abstraction Gate
- **Frameworks**: [List all frameworks/libraries]
- **Trust Level**: [ ] Official | [X] Community-maintained | [ ] Custom
- **Maintenance Status**: [Active | Stable | Deprecated]

### Article IX: Integration-First Gate
- **Contract Tests**: [ ] Defined | [ ] In Progress | [X] Pending implementation
- **Real Dependencies**: [ ] Required | [X] Mocked for unit tests only
- **Test Database**: [PostgreSQL | MySQL | In-memory]

### [Article X: Project-Specific]
- [Add project-specific constitutional validation]

---

## Decision History

> **Updated when**: Architectural pivots occur (Article XI)

### Decision Log
Track all architectural decisions that modify these constraints:

#### [Date]: [Decision Title]
- **Trigger**: [What blocker/issue prompted the decision]
- **Change**: ~~Old approach~~ → New approach
- **ADR**: See [ADR-###](../decisions/adr-###-title.md)
- **Rationale**: [Brief explanation]
- **Impact**: [Timeline, affected components]

#### Example:
#### 2025-01-30: Migrate from Prisma to node-postgres
- **Trigger**: Prisma multi-tenant connection pooling causes 500ms latency (target: <100ms)
- **Change**: ~~Prisma ORM~~ → Pure node-postgres with `SET search_path`
- **ADR**: See [ADR-002](../decisions/adr-002-prisma-to-pg.md)
- **Rationale**: Prisma requires separate client per schema; pg supports connection-level schema switching
- **Impact**: +2 weeks timeline, test references updated, lost type safety (mitigated with DTOs)

---

## Integration Points

> **Optional**: Document how this feature integrates with others

### Upstream Dependencies
- **Feature**: [Feature ID/Name]
- **Dependency**: [What this feature needs from upstream]
- **Contract**: [API spec or data format]

### Downstream Consumers
- **Feature**: [Feature ID/Name]
- **Provides**: [What this feature exposes]
- **Contract**: [API spec or data format]

### Cross-Cutting Concerns
- **Logging**: [Winston | Pino | Custom]
- **Metrics**: [Prometheus | StatsD | OpenTelemetry]
- **Error Tracking**: [Sentry | Rollbar | Custom]
- **Authentication**: [JWT | OAuth | Session]
- **Authorization**: [RBAC | ABAC | Custom]

---

## Development Guidelines

> **Optional**: Technology-specific guidance for implementers

### Database Schema Design
```sql
-- Example: Multi-tenant table structure
CREATE TABLE zones (
  id SERIAL PRIMARY KEY,
  company_id INTEGER NOT NULL REFERENCES companies(id),
  name VARCHAR(255) NOT NULL,
  capacity INTEGER NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),

  -- Ensure tenant isolation
  CONSTRAINT fk_company_id FOREIGN KEY (company_id) REFERENCES companies(id)
);

-- Index for performance
CREATE INDEX idx_zones_company_id ON zones(company_id);
```

### API Patterns
```typescript
// Example: Tenant-aware API endpoint
app.get('/api/:companySlug/zones', async (req, res) => {
  const { companySlug } = req.params;

  await withCompany(env, companySlug, async ({ prisma, companyId }) => {
    const zones = await prisma.zone.findMany({
      where: { company_id: companyId }, // Enforce tenant isolation
      orderBy: { created_at: 'desc' }
    });

    res.json({ zones });
  });
});
```

### Error Handling
```typescript
// Example: Structured error responses
class TenantIsolationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'TenantIsolationError';
  }
}

// Usage
if (!companyId) {
  throw new TenantIsolationError('Request missing tenant context');
}
```

---

## Validation Checklist

Before marking TECHNICAL.md as complete:

- [ ] All `[NEEDS VALIDATION]` flags have investigation plans with deadlines
- [ ] Multi-tenancy strategy defined (if applicable)
- [ ] Performance targets quantified with measurement plan
- [ ] Constitutional compliance validated (Articles VII, VIII, IX)
- [ ] Technology stack documented with version requirements
- [ ] Regulatory constraints identified and addressed
- [ ] Integration points mapped to upstream/downstream features
- [ ] Risk mitigation plans defined for all known risks

---

## Amendment Process

When architectural pivots occur (Article XI):

1. **Create ADR**: Document decision in `decisions/adr-###-title.md`
2. **Update this file**: Use ~~strikethrough~~ → new pattern
3. **Link to ADR**: Add to Decision History section above
4. **Regenerate plans**: Run `/plan --regenerate` to update tasks
5. **Update rules catalog**: Run `/rules-extract` if business logic affected

---

**Version**: 1.0
**Last Amended**: YYYY-MM-DD
**Amendment Count**: 0
**Status**: [Active | Amended | Superseded]
