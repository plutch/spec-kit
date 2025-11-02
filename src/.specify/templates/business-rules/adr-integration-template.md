# ADR-###: [Title] - Business Rules Impact

**Status**: [Proposed | Accepted | Deprecated | Superseded]
**Date**: YYYY-MM-DD
**Feature**: `specs/###-feature/` OR Cross-Project
**Business Rules Affected**: [Count of BR-* rules impacted]

---

## Context

[Describe the architectural decision being made]

### Background
- Current implementation approach
- Why this decision is needed
- Which features/modules are affected

---

## Decision

We will [describe the decision].

### Technical Change
- **FROM**: [Current approach/technology]
- **TO**: [New approach/technology]

---

## Affected Business Rules (Article XI.6)

### Modified Rules

#### BR-[DOMAIN]-[NUMBER]: [Rule Name]

**Change Type**: [ ] Implementation Only | [ ] Test Changes | [ ] Both

**Implementation Changes**:
```diff
- FROM: src/services/zones/prisma-zones.ts:45
+ TO: src/services/zones/pg-zones.ts:38
```

**Test Changes**:
```diff
- FROM: tests/integration/test_zones_prisma.py:52
+ TO: tests/integration/test_zones_pg.py:48
```

**Catalog Update Required**:
```bash
/rules-extract --module [DOMAIN] --validate
```

**Constitutional Impact**:
- [ ] Article I: Testability (test references changed)
- [ ] Article V: Traceability (implementation paths changed)
- [ ] Article VI: Test-First (tests need RED → GREEN validation)
- [ ] No constitutional violations

---

#### BR-[DOMAIN]-[NUMBER]: [Another Rule Name]

[Repeat structure for each affected rule]

---

### New Rules Introduced

#### BR-[DOMAIN]-[NUMBER]: [New Rule Name]

**Reason**: [Why this decision introduces new business logic]

**Status**: DRAFT (will be ACTIVE after implementation)

**Given-When-Then**:
- **Given**: [Precondition]
- **When**: [Action]
- **Then**: [Expected outcome]

**Priority**: [HIGH | MEDIUM | LOW]

**Domain**: [DOMAIN]

**Source FR**: FR-###

**Provisional Implementation**: `src/path/to/new-file.ts:line` (to be created)

**Provisional Tests**:
- Valid case: `tests/path/test_new_rule.py:test_valid_case`
- Invalid case: `tests/path/test_new_rule.py:test_invalid_case`

---

### Deprecated Rules

#### BR-[DOMAIN]-[NUMBER]: [Rule Name Being Deprecated]

**Reason**: [Why this decision makes the rule obsolete]

**Status Change**: ACTIVE → DEPRECATED

**Replacement**: BR-[DOMAIN]-[NEW_NUMBER] OR None (no longer needed)

**Catalog Update**:
```markdown
## BR-[DOMAIN]-[NUMBER]: [Title]
**Status**: DEPRECATED (ADR-### - [Date])
**Replaced By**: BR-[DOMAIN]-[NEW_NUMBER]
**Reason**: [Concise reason]
```

---

### No Business Rules Affected
If this decision has no business logic impact, check this box:

- [ ] This ADR is purely architectural with no business rule changes
- Rationale: [Explain why - e.g., "Infrastructure change only, no business logic affected"]

---

## Business Rules Validation Checklist

Run this checklist AFTER implementing the ADR:

### Modified Rules
For each modified rule (BR-[DOMAIN]-[NUMBER]):

- [ ] Implementation path updated in code annotations
- [ ] Test references updated in code annotations
- [ ] Catalog updated via `/rules-extract --module [DOMAIN]`
- [ ] Article I validation passes (≥2 test references)
- [ ] Article V validation passes (traceability links valid)
- [ ] Article VI validation passes (tests exist and pass)

### New Rules
For each new rule (BR-[DOMAIN]-[NUMBER]):

- [ ] Rule harvested during planning (`/rules-harvest`)
- [ ] Status set to DRAFT initially
- [ ] Implementation includes `@BusinessRule BR-[DOMAIN]-[NUMBER]` annotation
- [ ] Minimum 2 tests reference the rule
- [ ] Tests pass before marking ACTIVE
- [ ] Catalog entry created via `/rules-extract`
- [ ] All 9 constitutional gates pass

### Deprecated Rules
For each deprecated rule (BR-[DOMAIN]-[NUMBER]):

- [ ] Status changed to DEPRECATED in catalog
- [ ] Replacement rule linked (if applicable)
- [ ] ADR number documented in deprecation reason
- [ ] Code annotations marked as deprecated
- [ ] Tests moved to `tests/deprecated/` or removed

---

## Post-Implementation Actions

### Immediate (Same Week)
```bash
# Update affected modules
/rules-extract --module [DOMAIN_1] --validate
/rules-extract --module [DOMAIN_2] --validate

# Verify no constitutional violations
grep "VALIDATION FAILED" .specify/business-rules/catalog.md
# Expected: No matches (all rules pass)

# Verify all modified rules updated
grep "BR-[DOMAIN]-[NUMBER]" .specify/business-rules/catalog.md | grep "src/new/path"
# Expected: Found (implementation path updated)
```

### Short-term (Next Sprint)
- [ ] Team review of business rules changes
- [ ] Update `QUICK_REFERENCE.md` if high-priority rules changed
- [ ] Developer training if new rule patterns introduced

### Long-term (Next Quarter)
- [ ] Monitor rule catalog growth (target: <200 ACTIVE rules per module)
- [ ] Consider module split if catalog exceeds 200 rules
- [ ] Refactor if deprecated rules accumulate (target: <10% deprecated)

---

## Integration Testing

### Business Rule Test Coverage
**Before ADR Implementation**:
```bash
pytest tests/ -k "test_BR_[DOMAIN]" --co -q | wc -l
# Record count: [NUMBER] tests
```

**After ADR Implementation**:
```bash
pytest tests/ -k "test_BR_[DOMAIN]" --co -q | wc -l
# Expected count: [NUMBER] tests (should be ≥ before count)
```

**Evidence**:
```bash
pytest tests/ -k "test_BR_[DOMAIN]" -v
# All affected business rule tests must PASS ✅
```

---

## References

- **Feature Spec**: `specs/###-feature/spec.md` (source requirements)
- **Business Rules Catalog**: `.specify/business-rules/catalog.md`
- **Validation Rules**: `.specify/business-rules/validation-rules.md`
- **Related ADRs**: [Links to related architectural decisions]

---

**Template Version**: 1.0.0
**Validation**: Business Rules Integration
