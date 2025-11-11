# Discovery Interview Gates - Implementation Summary
# Version 2.7.0 - Complete Integration Guide

---

## Overview

This document provides a quick reference for implementing Discovery Interview Gates in spec-kit v2.7.0.

**What This Adds:**
- Domain-specific discovery questions (auth, data, UI, API, performance)
- Mandatory specification approval gate
- Mandatory constitutional approval gate
- Complete audit trail of approvals and revisions

**Token Cost:** +500-900 per feature (~5-10% increase)
**ROI:** 400-800% (prevents 10K-30K tokens in rework)

---

## Files Created

All implementation details are in these files:

| File | Purpose | Lines |
|------|---------|-------|
| `INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md` | Enhanced pre-spec discovery with domain templates | ~600 |
| `INTEGRATION-STEP-8-REVIEW-GATE.md` | Specification review approval gate | ~400 |
| `INTEGRATION-CONSTITUTIONAL-GATE.md` | Constitutional approval gate for planning | ~350 |
| `INTEGRATION-CONFIG-SCHEMA.md` | Configuration schema and examples | ~400 |
| `INTEGRATION-TEST-CASES.md` | Comprehensive test suite | ~800 |

**Total Implementation:** ~2,550 lines of detailed code

---

## Quick Implementation Checklist

### Phase 1: Enhanced Discovery (Week 1)

**Files to Modify:**
- [ ] `src/.claude/commands/speckit.specify.md`
  - Location: Replace lines 22-167 (Step 1)
  - Source: `INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md`
  - Changes: 600 lines (domain detection + 5 templates)

**New Files to Create:**
- [ ] `src/.specify/gates/auth-template.md` (extract from Step 1)
- [ ] `src/.specify/gates/data-template.md` (extract from Step 1)
- [ ] `src/.specify/gates/ui-template.md` (extract from Step 1)
- [ ] `src/.specify/gates/api-template.md` (extract from Step 1)
- [ ] `src/.specify/gates/performance-template.md` (extract from Step 1)

**Testing:**
- [ ] Run Test Suite 1 (tests 1.1-1.7)
- [ ] Verify domain detection works
- [ ] Verify A/B option appears
- [ ] Verify insights flow to spec generation

**Token Budget:** +200-400 tokens (only if user chooses discovery)

---

### Phase 2: Specification Review Gate (Week 2)

**Files to Modify:**
- [ ] `src/.claude/commands/speckit.specify.md`
  - Location: INSERT before current Step 8 (line ~1076)
  - Source: `INTEGRATION-STEP-8-REVIEW-GATE.md`
  - Changes: +400 lines (new Step 8, renumber old Step 8 to Step 9)

**Files to Update:**
- [ ] `src/.specify/config.example.yml`
  - Add `discovery_gates.specification_review_gate` section
  - Source: `INTEGRATION-CONFIG-SCHEMA.md`

**Testing:**
- [ ] Run Test Suite 2 (tests 2.1-2.7)
- [ ] Verify review gate appears after spec generation
- [ ] Verify approval/revise/clarify flows work
- [ ] Verify metadata updated correctly

**Token Budget:** +300-500 tokens (mandatory, every feature)

---

### Phase 3: Constitutional Approval Gate (Week 3)

**Files to Modify:**
- [ ] `src/.claude/commands/speckit.plan.md`
  - Location: INSERT after Phase -1 validation
  - Source: `INTEGRATION-CONSTITUTIONAL-GATE.md`
  - Changes: +350 lines (approval gate after Phase -1)

- [ ] `src/.claude/commands/speckit.implement.md`
  - Location: Enhance Step 2.5 (Pre-Flight Check)
  - Source: `INTEGRATION-CONSTITUTIONAL-GATE.md` (Step -1.5)
  - Changes: +50 lines (validate planning approval)

**Files to Update:**
- [ ] `src/.specify/config.example.yml`
  - Add `discovery_gates.constitutional_approval_gate` section
  - Source: `INTEGRATION-CONFIG-SCHEMA.md`

**Testing:**
- [ ] Run Test Suite 3 (tests 3.1-3.7)
- [ ] Verify PASS/CONDITIONAL/FAIL statuses work
- [ ] Verify blocking on FAIL works
- [ ] Verify pre-flight check validates approval

**Token Budget:** +100-200 tokens (mandatory, every feature)

---

## File Change Summary

### Modifications Required

| File | Action | Lines | Backward Compatible |
|------|--------|-------|---------------------|
| `src/.claude/commands/speckit.specify.md` | Enhance Step 1 | 600 | ✅ Yes (A/B choice preserved) |
| `src/.claude/commands/speckit.specify.md` | Add Step 8 | 400 | ✅ Yes (can disable) |
| `src/.claude/commands/speckit.plan.md` | Add approval gate | 350 | ✅ Yes (can disable) |
| `src/.claude/commands/speckit.implement.md` | Enhance Step 2.5 | 50 | ✅ Yes (graceful fail) |
| `src/.specify/config.example.yml` | Add discovery_gates | 200 | ✅ Yes (optional section) |

### New Files Required

| File | Purpose | Lines |
|------|---------|-------|
| `src/.specify/gates/auth-template.md` | Auth discovery questions | 120 |
| `src/.specify/gates/data-template.md` | Data model questions | 100 |
| `src/.specify/gates/ui-template.md` | UI/UX questions | 90 |
| `src/.specify/gates/api-template.md` | API questions | 110 |
| `src/.specify/gates/performance-template.md` | Performance questions | 100 |
| `docs/DISCOVERY-GATES.md` | User guide | 300 |
| `docs/MIGRATION-v2.7.md` | Upgrade guide | 150 |

---

## Code Extraction Guide

### Step 1: Extract Domain Templates

From `INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md`, extract each domain template to separate files:

```bash
# Auth template
sed -n '/### 🔐 AUTHENTICATION DOMAIN TEMPLATE/,/---$/p' INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md > src/.specify/gates/auth-template.md

# Data template
sed -n '/### 💾 DATA\/CRUD DOMAIN TEMPLATE/,/---$/p' INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md > src/.specify/gates/data-template.md

# UI template
sed -n '/### 🎨 UI\/UX DOMAIN TEMPLATE/,/---$/p' INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md > src/.specify/gates/ui-template.md

# API template
sed -n '/### 🔌 API DOMAIN TEMPLATE/,/---$/p' INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md > src/.specify/gates/api-template.md

# Performance template
sed -n '/### ⚡ PERFORMANCE DOMAIN TEMPLATE/,/---$/p' INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md > src/.specify/gates/performance-template.md
```

### Step 2: Integrate into speckit.specify.md

Open `src/.claude/commands/speckit.specify.md`:

1. **Backup original:**
   ```bash
   cp src/.claude/commands/speckit.specify.md src/.claude/commands/speckit.specify.md.v2.6.backup
   ```

2. **Replace Step 1 (lines 22-167):**
   - Delete lines 22-167
   - Insert content from `INTEGRATION-STEP-1-ENHANCED-DISCOVERY.md`

3. **Insert Step 8 (before line ~1076):**
   - Insert content from `INTEGRATION-STEP-8-REVIEW-GATE.md`
   - Renumber old Step 8 to Step 9

4. **Verify:**
   ```bash
   # Check line count changed
   wc -l src/.claude/commands/speckit.specify.md
   # Should be ~1000 lines longer

   # Check Step 8 exists
   grep -n "Step 8:" src/.claude/commands/speckit.specify.md
   ```

### Step 3: Integrate into speckit.plan.md

Open `src/.claude/commands/speckit.plan.md`:

1. **Backup original:**
   ```bash
   cp src/.claude/commands/speckit.plan.md src/.claude/commands/speckit.plan.md.v2.6.backup
   ```

2. **Find Phase -1 end:**
   ```bash
   grep -n "Phase -1:" src/.claude/commands/speckit.plan.md
   # Note line number where Phase -1 validation completes
   ```

3. **Insert approval gate:**
   - After Phase -1 validation result
   - Before final plan write
   - Insert content from `INTEGRATION-CONSTITUTIONAL-GATE.md`

4. **Verify:**
   ```bash
   grep -n "WAITING_FOR_CONSTITUTIONAL_APPROVAL" src/.claude/commands/speckit.plan.md
   # Should find the gate
   ```

### Step 4: Update speckit.implement.md

Open `src/.claude/commands/speckit.implement.md`:

1. **Find Step 2.5:**
   ```bash
   grep -n "Step 2.5" src/.claude/commands/speckit.implement.md
   ```

2. **Enhance pre-flight check:**
   - Add planning approval validation
   - Add constitutional status check
   - Source: `INTEGRATION-CONSTITUTIONAL-GATE.md` Step -1.5

3. **Verify:**
   ```bash
   grep -n "planning.approved" src/.claude/commands/speckit.implement.md
   # Should find approval check
   ```

### Step 5: Update Configuration

1. **Add to config.example.yml:**
   ```bash
   cat INTEGRATION-CONFIG-SCHEMA.md >> src/.specify/config.example.yml
   ```

2. **Verify:**
   ```bash
   yq eval '.discovery_gates' src/.specify/config.example.yml
   # Should show discovery_gates section
   ```

---

## Testing Strategy

### Unit Tests (Per Component)

```bash
# Test domain detection
echo "Testing domain detection..."
/speckit.specify "add OAuth authentication"
# Expected: Authentication domain detected

# Test discovery A/B option
/speckit.specify "add something"
# Expected: A/B option appears

# Test review gate
/speckit.specify "add auth" # Choose A (skip discovery)
# Expected: Review gate appears after spec generation

# Test constitutional gate
/speckit.plan
# Expected: Approval gate appears after Phase -1
```

### Integration Tests (End-to-End)

Run complete test suite from `INTEGRATION-TEST-CASES.md`:

```bash
# Run all tests
bash test-discovery-gates.sh

# Or run individual test suites
bash test-suite-1-discovery.sh
bash test-suite-2-review-gate.sh
bash test-suite-3-constitutional-gate.sh
bash test-suite-4-config.sh
bash test-suite-5-integration.sh
```

### Performance Tests

Measure token costs:

```bash
# Track tokens per command
/speckit.specify "add auth" # Should add +200-400 tokens (discovery)
/speckit.specify "add auth" # Should add +300-500 tokens (review gate)
/speckit.plan                # Should add +100-200 tokens (const gate)

# Total: 500-900 tokens per feature
```

---

## Rollback Plan

If issues found during testing:

### Rollback Step 1 (Discovery)

```bash
# Restore original Step 1
git checkout src/.claude/commands/speckit.specify.md
# OR
cp src/.claude/commands/speckit.specify.md.v2.6.backup src/.claude/commands/speckit.specify.md
```

### Rollback Step 8 (Review Gate)

Remove Step 8 section from speckit.specify.md, renumber Step 9 back to Step 8.

### Rollback Constitutional Gate

```bash
# Restore original planning command
git checkout src/.claude/commands/speckit.plan.md
# OR
cp src/.claude/commands/speckit.plan.md.v2.6.backup src/.claude/commands/speckit.plan.md
```

### Rollback Configuration

Remove `discovery_gates` section from config.example.yml.

---

## Migration Path for Users

### Option 1: Gradual Adoption (Recommended)

**Week 1:** Enable discovery only
```yaml
discovery_gates:
  enabled: true
  pre_spec_discovery:
    mode: "adaptive"
  specification_review_gate:
    enabled: false  # Not yet
  constitutional_approval_gate:
    enabled: false  # Not yet
```

**Week 2:** Add specification review gate
```yaml
specification_review_gate:
  enabled: true  # Now enabled
```

**Week 3:** Add constitutional gate
```yaml
constitutional_approval_gate:
  enabled: true  # Now enabled
```

### Option 2: Full Adoption (Advanced Users)

```yaml
discovery_gates:
  enabled: true  # All gates enabled
```

### Option 3: Keep v2.6 Behavior (Conservative)

```yaml
legacy_mode:
  enabled: true  # Exact v2.6 behavior
```

---

## Success Metrics

### Quantitative

- [ ] **Rework Rate:** <10% (down from 15-20%)
- [ ] **Gate Skip Rate:** <10% (users find gates valuable)
- [ ] **Revision Rate:** 15-25% (healthy iteration)
- [ ] **Token Overhead:** 500-900 per feature (5-10%)

### Qualitative

- [ ] **User Confidence:** >80% report "high confidence in specs"
- [ ] **Perceived Value:** >70% say "gates are helpful"
- [ ] **Time Perception:** "Worth the extra time"

### Technical

- [ ] All test suites pass (100%)
- [ ] Backward compatibility verified
- [ ] Configuration overrides work
- [ ] Graceful degradation works (no constitution.md)

---

## Support & Troubleshooting

### Common Issues

**Issue:** Gates don't appear
**Solution:** Check `discovery_gates.enabled` in config, check `legacy_mode.enabled`

**Issue:** Auto-approve not working in CI/CD
**Solution:** Set `SPECKIT_AUTO_APPROVE=true` environment variable

**Issue:** Domain detection incorrect
**Solution:** Use more specific keywords in feature description

**Issue:** Approval timeout
**Solution:** Increase `approval_timeout_minutes` or use auto-approve

### Getting Help

1. Check `INTEGRATION-TEST-CASES.md` for test scenarios
2. Review `INTEGRATION-CONFIG-SCHEMA.md` for configuration
3. See implementation files for detailed code
4. Run test suite to identify specific failures

---

## Release Checklist

### Pre-Release

- [ ] All implementation files created
- [ ] All tests pass (Test Suites 1-5)
- [ ] Token costs validated (500-900 per feature)
- [ ] Backward compatibility verified
- [ ] Configuration schema validated
- [ ] Documentation complete

### Release

- [ ] Tag v2.7.0 in Git
- [ ] Update CHANGELOG.md
- [ ] Update README.md with v2.7 features
- [ ] Create GitHub release notes
- [ ] Announce in community channels

### Post-Release

- [ ] Monitor adoption metrics
- [ ] Collect user feedback
- [ ] Track skip rates and revision rates
- [ ] Iterate on templates based on data

---

## Quick Reference: What Goes Where

```
src/.claude/commands/
├── speckit.specify.md
│   ├── Step 1: Enhanced Discovery (MODIFIED)
│   │   └── Domain detection + 5 templates
│   ├── Step 8: Review Gate (NEW)
│   │   └── Approval loop with revise/clarify
│   └── Step 9: Completion (RENUMBERED from Step 8)

├── speckit.plan.md
│   └── Phase -1 Approval Gate (NEW)
│       └── Constitutional approval after validation

└── speckit.implement.md
    └── Step 2.5: Pre-Flight Check (ENHANCED)
        └── Validate planning approval

src/.specify/
├── config.example.yml
│   └── discovery_gates section (NEW)
├── gates/ (NEW DIRECTORY)
│   ├── auth-template.md
│   ├── data-template.md
│   ├── ui-template.md
│   ├── api-template.md
│   └── performance-template.md

docs/
├── DISCOVERY-GATES.md (NEW)
└── MIGRATION-v2.7.md (NEW)
```

---

## Next Steps

To implement Discovery Interview Gates:

1. **Review all implementation files** (5 files, ~2,550 lines total)
2. **Start with Phase 1** (Enhanced Discovery, Week 1)
3. **Test thoroughly** (Test Suite 1 from INTEGRATION-TEST-CASES.md)
4. **Proceed to Phase 2** (Review Gate, Week 2)
5. **Proceed to Phase 3** (Constitutional Gate, Week 3)
6. **Release v2.7.0** with complete documentation

**Estimated Timeline:** 3-4 weeks for full implementation + testing

**Estimated Effort:**
- Implementation: 40-50 hours
- Testing: 20-30 hours
- Documentation: 10-15 hours
- **Total: 70-95 hours**

---

## Contact & Questions

For implementation questions:
- Review detailed implementation files
- Check test cases for examples
- See configuration schema for options

For design decisions:
- See original integration plan document
- Review token economy analysis
- Check ROI calculations

---

**Spec-Kit Discovery Interview Gates v2.7.0**
**Complete Implementation Guide**
**Last Updated: 2025-11-11**
