# Business Rules Validation Standards

**Purpose**: Quality gates for business logic documentation
**Scope**: Applies to ALL rules in catalog (not dev methodology)
**Relationship**: Complements `memory/constitution.md` (Article XI may reference these)
**Version**: 1.0

---

## Article I: Testability Mandate (HIGH Severity)

Every ACTIVE business rule MUST have:
- Minimum 2 test references (valid case + invalid case)
- Test file paths linkable and verifiable
- Tests exist before rule marked as ACTIVE

**Validation**:
```bash
# Check rule has test_refs field with ≥2 entries
# Example: ["tests/zones/test_capacity.py:52", "tests/zones/test_capacity.py:78"]
```

**Severity**: HIGH - Blocks catalog update if violated

---

## Article II: Completeness Principle (HIGH Severity)

Every business rule MUST include all required fields:
- `rule_id`: Unique identifier (format: BR-{DOMAIN}-{NUMBER})
- `priority`: Business priority (HIGH/MEDIUM/LOW)
- `status`: Lifecycle state (DRAFT/ACTIVE/DEPRECATED)
- `given`: Preconditions
- `when`: Trigger event/action
- `then`: Expected outcome/behavior

**Validation**:
```bash
# All 6 fields must be present and non-empty
```

**Severity**: HIGH - Blocks catalog update if violated

---

## Article III: Single Source of Truth (MEDIUM Severity)

Business rule IDs MUST be unique across entire catalog:
- No duplicate BR-* identifiers
- Deprecated rules keep original ID with status=DEPRECATED
- Replacement rules get new IDs

**Validation**:
```bash
# Check for duplicate rule_id values in catalog
grep "^rule_id:" catalog.md | sort | uniq -d
```

**Severity**: MEDIUM - Warning issued, manual resolution required

---

## Article IV: Explicit Priority & Domain (HIGH Severity)

Every rule MUST declare:
- **Business Priority**: HIGH/MEDIUM/LOW (stakeholder impact)
- **Domain Module**: Logical grouping (ZONES, AUTH, BILLING, etc.)
- **Source FR**: Feature requirement ID that triggered rule (FR-###)

**Validation**:
```bash
# Check priority in [HIGH, MEDIUM, LOW]
# Check domain is non-empty uppercase string
# Check source_fr matches pattern FR-\d{3}
```

**Severity**: HIGH - Blocks catalog update if violated

---

## Article V: Traceability to Source (MEDIUM Severity)

Every rule MUST link to:
- **Specification**: `specs/###-feature/spec.md` line reference
- **Implementation**: Source code file + line number
- **Tests**: Test file paths (from Article I)

**Validation**:
```bash
# Check spec_ref, implementation, test_refs all present
# Verify files exist at referenced paths
```

**Severity**: MEDIUM - Warning if missing, required before ACTIVE status

---

## Article VI: Test-First Philosophy (HIGH Severity)

Rules cannot be marked `status=ACTIVE` until:
- Tests exist (Article I validation passes)
- Tests are passing (verified via test suite run)
- Implementation exists and references rule ID

**Validation**:
```bash
# If status=ACTIVE:
#   - Run tests referenced in test_refs
#   - Verify all pass
#   - Verify implementation file contains @BusinessRule annotation
```

**Severity**: HIGH - Blocks ACTIVE status transition

---

## Article VII: No Speculation (LOW Severity)

Business rules MUST NOT include:
- Hypothetical scenarios without feature requirement backing
- "What if" cases not validated by PM/stakeholder
- Future requirements not yet approved

Every rule must trace to approved FR-* requirement.

**Validation**:
```bash
# Check source_fr exists in specs/*/spec.md
# Verify requirement is not marked [FUTURE] or [PROPOSED]
```

**Severity**: LOW - Informational warning, manual review recommended

---

## Article VIII: Versioning (MEDIUM Severity)

When rules are deprecated:
- Status changed to `DEPRECATED`
- `deprecated_date` field added
- `replaced_by` field links to replacement rule ID (if applicable)
- Original rule ID preserved (never reused)

**Validation**:
```bash
# If status=DEPRECATED:
#   - Check deprecated_date present (YYYY-MM-DD format)
#   - Check replaced_by present OR justification documented
```

**Severity**: MEDIUM - Required for audit trail compliance

---

## Article IX: Multi-Tenancy Awareness (MEDIUM Severity)

Rules affecting multi-tenant systems MUST declare:
- **Tenant Scope**: GLOBAL | PER_TENANT | CONFIGURABLE
- **Isolation Strategy**: How tenant data is separated
- **Configuration Location**: Where tenant-specific settings stored

**Validation**:
```bash
# If domain involves tenant data:
#   - Check tenant_scope in [GLOBAL, PER_TENANT, CONFIGURABLE]
#   - Verify isolation strategy documented
```

**Severity**: MEDIUM - Required for multi-tenant projects, N/A for single-tenant

---

## Constitutional Validation Workflow

### During /rules-harvest (DRAFT rules)
- Articles II, IV validation only (structure + metadata)
- Other articles deferred until implementation complete

### During /rules-extract (DRAFT → ACTIVE)
- **HIGH Severity**: Articles I, II, IV, VI (MUST pass)
- **MEDIUM Severity**: Articles III, V, VIII, IX (warnings only)
- **LOW Severity**: Article VII (informational)

### Compliance Score
```
Score = (Passed HIGH articles / Total HIGH articles) × 100%
Target: ≥95% for catalog updates
```

---

## Integration with Article XI (Pivot Protocol)

When architectural pivots occur (per `memory/constitution.md` Article XI):

1. **ADR documents affected rules** (Section 11.6)
2. **Implementation references updated** after pivot
3. **Test references updated** to new test files/locations
4. **Re-validation triggered** via `/rules-extract --module {MODULE}`
5. **Catalog synchronized** with pivot changes

---

## Tool Integration

### Automated Validation
```bash
# Run validation gates
./scripts/validate-business-rules.sh --severity HIGH

# Check specific rule
./scripts/validate-business-rules.sh --rule-id BR-ZONES-001

# CI/CD integration
./scripts/validate-business-rules.sh --ci-mode
# Exit code: 0 (pass), 1 (HIGH violations), 2 (MEDIUM violations)
```

### Manual Review Checklist
- [ ] Rule ID unique and follows BR-{DOMAIN}-{NUMBER} format
- [ ] Given/When/Then clearly describes logic
- [ ] Tests exist and pass
- [ ] Implementation annotated with @BusinessRule
- [ ] Source FR verified in spec
- [ ] Tenant scope declared (if multi-tenant)
- [ ] Priority justified by business impact

---

## Key Distinction

- **`memory/constitution.md`**: Development methodology (how we build software)
  - Includes Article XI (Technical Pivot Protocol)
  - Defines workflow, testing requirements, architectural standards

- **`.specify/business-rules/validation-rules.md`** (this file): Documentation quality (how we write business rules)
  - Defines quality gates for business logic documentation
  - Enforces traceability, testability, completeness

Both work together: Constitution governs process, Validation Rules govern business logic quality.

---

**Version**: 1.0
**Last Updated**: 2025-01-23
**Ratified By**: [Project Team]
**Next Review**: [Date + 6 months]
