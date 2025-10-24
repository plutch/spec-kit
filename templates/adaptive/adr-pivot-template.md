# ADR-###: [Pivot Title]

**Status**: [Proposed | Accepted | Deprecated | Superseded]
**Date**: YYYY-MM-DD
**Feature**: `specs/###-feature-name/` OR `Cross-Project`
**Trigger**: [Week X of implementation | Testing phase | Integration testing]

---

## Blocker Summary (Article XI.1)

### Classification
- **Type**: [Architectural Limitation | Framework Limitation | Performance Blocker]
- **Scope**: [Single Feature | Cross-Cutting Concern]
- **NFR Impact**: [Violates latency requirement | Violates security requirement | No NFR violation]

### Discovery
**When**: [Implementation Week X | During testing | Code review]

**What**: [Concise description of the blocker - e.g., "Prisma multi-tenant connection pooling causes 500ms latency"]

**Evidence**:
```
[Paste error message, test output, or metrics showing the blocker]
```

---

## Decision (Article XI.2)

### Pivot: FROM → TO

**FROM**:
- Technology: [Current framework/library/pattern]
- Rationale for original choice: [Why this was selected initially]

**TO**:
- Technology: [New framework/library/pattern]
- Reason for change: [Primary blocker resolution]

### Alternatives Considered

1. **Option A**: [Workaround within current tech]
   - **Pros**: [No pivot needed, faster]
   - **Cons**: [Doesn't resolve core issue, technical debt]
   - **Decision**: ❌ Rejected

2. **Option B**: [Different pivot]
   - **Pros**: [Different benefits]
   - **Cons**: [Different tradeoffs]
   - **Decision**: ❌ Rejected

3. **Selected Option**: [Chosen pivot] ✅
   - **Pros**: [Resolves blocker + additional benefits]
   - **Cons**: [Timeline impact, learning curve]
   - **Decision**: ✅ Accepted

---

## Constitutional Validation (Article XI.3)

### Simplicity Gate (Article VII)
**Assessment**: [PASS | CONDITIONAL PASS | FAIL]
- Adding new dependency? [YES | NO]
- If YES, justification: [Why warranted despite complexity]
- Architect approval: [ ] Required | [ ] Not Required

### Anti-Abstraction Gate (Article VIII)
**Assessment**: [PASS | FAIL]
- Framework trust: [Well-maintained? Active community?]
- Custom wrappers needed? [YES | NO]
- If YES, justification: [Why wrapper necessary]

### Integration-First Gate (Article IX)
**Assessment**: [PASS | FAIL]
- [ ] Contract tests updated BEFORE implementation
- [ ] Tests show RED phase (current impl fails new tests)
- [ ] User approved test changes

**Test Changes**:
```diff
# File: tests/integration/test_zones.py
- def test_create_zone_prisma():
+ def test_create_zone_postgres():
      # New test expectations
```

---

## Test-First Imperative (Article XI.4)

### Updated Contract Tests

**Files Modified**:
- `tests/integration/test_zones.py:42-58`
- `tests/e2e/test_multi_company.py:123-145`

**Validation**:
```bash
# Run tests with CURRENT implementation (should FAIL)
pytest tests/integration/test_zones.py -v
# Expected: 2 tests FAIL (RED phase validated ✅)
```

**User Approval**:
- [ ] Test changes reviewed and approved
- Date: YYYY-MM-DD
- Approver: [Name/Role]

---

## TECHNICAL.md Amendment (Article XI.5)

### Constraint Changes

**Before**:
```markdown
## Database Access
- **ORM**: Prisma (connection pooling per tenant)
- **Multi-tenancy**: Row-Level Security (RLS)
```

**After** (amended):
```markdown
## Database Access
- ~~**ORM**: Prisma (connection pooling per tenant)~~ [DEPRECATED via ADR-002]
- **Database Driver**: node-postgres with `SET search_path` per tenant
- **Multi-tenancy**: PostgreSQL schemas (one per company)
- **Change**: ADR-002 - Prisma latency blocker
```

### Decision History Entry

Add to `TECHNICAL.md` Decision History:
```markdown
#### YYYY-MM-DD: Migrate Prisma → node-postgres
- **Trigger**: Prisma multi-tenant connection pooling causes 500ms latency (Week 3 testing)
- **Change**: ~~Prisma ORM~~ → Pure node-postgres with schema isolation
- **ADR**: [ADR-002](decisions/adr-002-prisma-to-pg.md)
- **Impact**: +2 weeks timeline, 15 test files updated
- **Business Rules**: BR-ZONES-001, BR-ZONES-003 implementation changed
```

---

## Affected Business Rules (Article XI.6)

### Modified Rules

#### BR-ZONES-001: Maximum Zone Capacity
- **Implementation Change**:
  - FROM: `src/services/zones/prisma-zones.ts:45`
  - TO: `src/services/zones/pg-zones.ts:38`
- **Test Change**:
  - FROM: `tests/integration/test_zones_prisma.py:52`
  - TO: `tests/integration/test_zones_pg.py:48`
- **Catalog Update**: Run `/rules-extract --module ZONES` after implementation

#### BR-ZONES-003: Isolation Guarantees
- **Implementation Change**:
  - FROM: `src/middleware/rls-middleware.ts:22` (Prisma RLS)
  - TO: `src/middleware/schema-middleware.ts:18` (Schema isolation)
- **Test Change**:
  - FROM: `tests/security/test_rls.py:89`
  - TO: `tests/security/test_schema_isolation.py:72`
- **Catalog Update**: Run `/rules-extract --module ZONES` after implementation

---

## Timeline Impact (Article XI.7)

### Original Plan
- **Total Duration**: 6 weeks
- **Completion**: Week 8

### Adjusted Plan
- **Pivot Work**: +2 weeks (tests update + migration + validation)
- **New Completion**: Week 10
- **Rationale**: Database layer refactor affects 15 test files, 8 service files

### Mitigation
- Parallel work on unaffected modules (AUTH, BILLING)
- Reuse existing pg patterns from other projects
- Pair programming for knowledge transfer

---

## Implementation Checklist

### Pre-Implementation (MUST complete before coding)
- [ ] Contract tests updated with new technology expectations
- [ ] Tests run and show RED phase (current impl fails)
- [ ] User approved test changes
- [ ] TECHNICAL.md amended with strikethrough ~~old~~ → new
- [ ] Decision history entry added to TECHNICAL.md
- [ ] plan.md regenerated via `/plan --regenerate` (reads amended TECHNICAL.md)
- [ ] tasks.md regenerated via `/tasks` (uses new plan)

### During Implementation
- [ ] Implement pivot following regenerated tasks.md
- [ ] Tests transition RED → GREEN
- [ ] Affected business rule implementations updated
- [ ] Code annotations (`@BusinessRule BR-*-***`) reference new file paths

### Post-Implementation (Article XI.6)
- [ ] Run `/rules-extract --module [AFFECTED_MODULES]` to update catalog
- [ ] Verify catalog shows updated implementation paths
- [ ] Verify all 9 constitutional gates pass for affected rules
- [ ] Update sprint/release timeline if needed

---

## Consequences

### Positive
- [Blocker resolved - specific metric improvement]
- [Performance gain - quantify]
- [Architectural benefit - e.g., better scalability]

### Negative
- [Timeline impact - quantify weeks/days]
- [Learning curve - team needs to learn new tech]
- [Testing effort - number of tests affected]

### Risks
- [Potential risks with new approach]
- [Mitigation strategies]

---

## Follow-up Actions

### Immediate (Week X)
- [ ] Update developer environment setup docs
- [ ] Team training session on new technology
- [ ] Update CI/CD pipeline for new tech

### Short-term (Month Y)
- [ ] Monitor performance metrics post-deployment
- [ ] Collect developer feedback on new approach
- [ ] Refine TECHNICAL.md with lessons learned

### Long-term (Quarter Z)
- [ ] Consider applying pattern to other features
- [ ] Update project constitution if pattern becomes standard
- [ ] Archive old implementation (reference branch)

---

## References

- **Blocker Thread**: [Link to GitHub issue/discussion]
- **Performance Data**: [Link to metrics/benchmarks]
- **Team Decision**: [Link to meeting notes]
- **Related ADRs**: [Links to related decisions]

---

**Pivot Protocol Version**: Article XI v1.0
**Template Version**: 1.0.0
