# Epic: [EPIC_NAME]

**Epic ID**: EPIC-[NUMBER]
**Created**: [YYYY-MM-DD]
**Target**: [Q# YYYY or specific date]
**Risk Level**: [🔴 HIGH / 🟠 MEDIUM / 🟢 LOW] (Score: [0-12]/12)
**Status**: [🟡 Planning / 🟢 Active / 🔴 Blocked / ✅ Complete]

---

## Vision

[1-2 paragraphs: What is this epic trying to achieve? Why does it matter to the business and users?]

**Example**:
> Enable SaaS multi-tenancy to support 100+ customers on shared infrastructure. This allows us to scale revenue 10x without rebuilding our platform, reduce per-customer costs by 70%, and onboard new customers in under 1 hour vs 3 days currently.

---

## Business Goals

1. **[Goal Category]**: [Specific measurable goal]
2. **[Goal Category]**: [Specific measurable goal]
3. **[Goal Category]**: [Specific measurable goal]

**Example**:
1. **Revenue**: Support 10x customer growth without infrastructure rebuild
2. **Cost Efficiency**: Reduce per-customer infrastructure cost by 70%
3. **Time-to-Value**: Onboard new customers in <1 hour (vs 3 days currently)

---

## Success Criteria

| Criterion | Measure | Target | Priority |
|-----------|---------|--------|----------|
| [Criterion name] | [How measured] | [Specific target] | [🔴/🟠/🟡] |
| [Criterion name] | [How measured] | [Specific target] | [🔴/🟠/🟡] |
| [Criterion name] | [How measured] | [Specific target] | [🔴/🟠/🟡] |

**Example**:
| Criterion | Measure | Target | Priority |
|-----------|---------|--------|----------|
| Tenant Isolation | Zero cross-tenant data leaks in prod | 100% | 🔴 CRITICAL |
| Performance | Tenant resolution overhead | <50ms p95 | 🟠 HIGH |
| Onboarding | End-to-end onboarding time | <1 hour | 🟠 HIGH |
| Scalability | Concurrent tenants on single instance | 100+ | 🟡 MEDIUM |

---

## Feature Breakdown ([N] Features)

| ID | Feature | Risk | Complexity | Effort | Dependencies |
|----|---------|------|------------|--------|--------------|
| [###] | [Feature name] | [🔴/🟠/🟡] | [XL/L/M/S] | [N weeks] | [[###, ###]] |
| [###] | [Feature name] | [🔴/🟠/🟡] | [XL/L/M/S] | [N weeks] | [[###, ###]] |
| [###] | [Feature name] | [🔴/🟠/🟡] | [XL/L/M/S] | [N weeks] | [[###, ###]] |

**Total Effort**: [N] weeks (sequential) → [N] weeks (parallelized)

**Example**:
| ID | Feature | Risk | Complexity | Effort | Dependencies |
|----|---------|------|------------|--------|--------------|
| 001 | Tenant Isolation | 🔴 HIGH | XL | 3 weeks | [] |
| 002 | Tenant Dashboard | 🟠 MEDIUM | L | 2 weeks | [001] |
| 003 | Tenant Billing | 🔴 HIGH | XL | 3 weeks | [001] |
| 004 | Tenant Administration | 🟠 MEDIUM | M | 1 week | [001] |
| 005 | Tenant Onboarding | 🟡 LOW | M | 1 week | [002,003,004] |

**Total Effort**: 10 weeks (sequential) → **6 weeks (parallelized)**

---

## Epic-Level Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| [Risk description] | [🔴/🟠/🟡/🟢] | [🔴/🟠/🟡/🟢] | [Mitigation strategy] |
| [Risk description] | [🔴/🟠/🟡/🟢] | [🔴/🟠/🟡/🟢] | [Mitigation strategy] |

**Example**:
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Cross-tenant data leak | 🟡 Medium | 🔴 Critical | Comprehensive testing, audit logging, security review before launch |
| Performance degradation | 🟠 High | 🟠 Major | Load testing with 100+ tenants, caching strategy, database indexing |
| Complex migrations | 🟠 High | 🟡 Medium | Phased rollout, feature flags, documented rollback plan |
| Integration failures | 🟡 Medium | 🟡 Medium | Integration tests, contract testing, monitoring dashboards |

---

## Shared Architecture

See: [epic-architecture.md](./epic-architecture.md)

**Key Decisions**:
- [Decision 1]: [Brief description]
- [Decision 2]: [Brief description]
- [Decision 3]: [Brief description]

**Example**:
- **Multi-tenancy approach**: PostgreSQL Row-Level Security (RLS)
- **Tenant resolution**: Subdomain-based (e.g., acme.yourapp.com)
- **Authentication**: JWT with tenant claim in payload

---

## Dependency Map

See: [dependency-map.md](./dependency-map.md)

**Critical Path**: [###] → [[###, ###, ###]] → [###]

**Example**: 001 (Foundation) → [002, 003, 004] (Parallel) → 005 (Integration)

---

## Timeline

```
Week 1-3: Feature 001 (Tenant Isolation) - Foundation
Week 4-5: Features 002, 003, 004 (Parallel) - Core Features
Week 6: Feature 005 (Onboarding) - Integration
```

**Milestones**:
- [ ] Foundation Complete (Feature 001) - Week 3
- [ ] Core Features Complete (Features 002-004) - Week 5
- [ ] Epic Complete (Feature 005) - Week 6

---

## Epic Approval

### Planning Approval
- [ ] Epic vision approved by stakeholders
- [ ] Feature breakdown approved by product team
- [ ] Architecture decisions approved by tech lead
- [ ] Dependency map validated by engineering team
- [ ] Risk mitigation reviewed by security/ops
- [ ] Timeline feasible per engineering capacity

**Approved By**: [Name/Role]
**Approved Date**: [YYYY-MM-DD]

### Constitutional Validation (v2.7+)
- [ ] Epic complies with project constitution
- [ ] Shared architecture adheres to constitutional principles
- [ ] No prohibited patterns in epic architecture
- [ ] Integration-first testing strategy defined

**Validated By**: [Name/Role]
**Validated Date**: [YYYY-MM-DD]

---

## Notes & Context

[Additional context, constraints, assumptions, or decisions made during epic planning]

**Example**:
- Existing customers will be migrated to tenant model in Phase 2 (not part of this epic)
- Initial launch targets 10-20 tenants, scaling to 100+ over 6 months
- Single-tenant mode will be deprecated 12 months after epic completion

---

**Epic Version**: 1.0.0
**Last Updated**: [YYYY-MM-DD]
**Updated By**: [Name]
