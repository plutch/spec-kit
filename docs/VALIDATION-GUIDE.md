# Validation Guide - Strict Hierarchical Spec Validation

**Command**: `/speckit.validate-hierarchy`
**Version**: SpecKit v2.1
**Mode**: STRICT (errors block workflow)

---

## Table of Contents

1. [Overview](#overview)
2. [When to Run](#when-to-run)
3. [Validation Checks](#validation-checks)
4. [Strict Mode](#strict-mode)
5. [Examples](#examples)
6. [CI/CD Integration](#cicd-integration)
7. [Troubleshooting](#troubleshooting)

---

## Overview

The validate-hierarchy command performs comprehensive validation of hierarchical specifications, ensuring parent-child relationships are correct and workflow files properly reference supplementary specs.

**Validation Mode**: **STRICT** (per user decision)
- Errors block workflow (exit code 1)
- No warnings-only mode
- Enforces best practices

**Purpose**: Prevent broken hierarchies, ensure spec integrity, maintain traceability.

---

## When to Run

### Mandatory After:

1. **`/speckit.supplement`** - Immediately validate new supplementary spec
2. **`/speckit.plan`** - Ensure plan.md references all specs
3. **`/speckit.reconcile`** - Validate edits to supplementary specs

### Recommended Before:

1. **`/speckit.implement`** - Block implementation if specs invalid
2. **Git commit/PR merge** - CI/CD gate

### Optional Anytime:

- Development (check spec integrity)
- After manual edits to *-SPEC.md files

---

## Validation Checks

### Check 1: Parent Spec Exists

**Rule**: `spec.md` must exist in feature directory

**Pass**:
```
✓ PASS: Parent spec.md exists (97KB, 80 FRs)
```

**Fail**:
```
❌ ERROR: Parent spec.md not found at specs/000003-billing-console/spec.md

RESOLUTION:
  Run /speckit.specify to create parent specification
```

---

### Check 2: Frontmatter Validation

**Rule**: All *-SPEC.md files MUST have valid YAML frontmatter

**Required Fields**:
- `parent`: Must be "spec.md"
- `scope`: Valid scope (ui-ux-implementation, api-contracts, technical-architecture)
- `functional_requirements`: Must reference parent FRs
- `target_agents`: Array with ≥1 agent
- `version`: Version number
- `last_updated`: ISO date

**Pass**:
```
✓ PASS: UI-SPEC.md: Valid frontmatter
  - parent: spec.md
  - scope: ui-ux-implementation
  - FRs: FR-001 through FR-080
  - agents: frontend-architect
```

**Fail**:
```
❌ ERROR: UI-SPEC.md: Missing 'parent' field in frontmatter

Expected:
---
parent: spec.md
scope: ui-ux-implementation
functional_requirements: [FR-001 through FR-080]
target_agents:
  - frontend-architect
version: 1.0
last_updated: 2025-11-01
---

RESOLUTION:
  Add frontmatter to UI-SPEC.md
```

---

### Check 3: FR Coverage Validation

**Rule**: FRs in supplementary spec must exist in parent spec.md

**Pass**:
```
✓ PASS: UI-SPEC.md: All FRs exist in parent
  Coverage: FR-001 through FR-080 (80 FRs)
```

**Fail**:
```
❌ ERROR: UI-SPEC.md: References non-existent FR: FR-085

Parent contains: FR-001 through FR-080
UI-SPEC claims: FR-001 through FR-085

RESOLUTION:
  Update UI-SPEC.md frontmatter to match parent FR range:
  functional_requirements: [FR-001 through FR-080]
```

---

### Check 4: plan.md Integration

**Rule**: plan.md MUST reference all supplementary specs

**Pass**:
```
✓ PASS: plan.md has 'Supplementary Specifications' section
✓ PASS: plan.md references UI-SPEC.md
```

**Fail**:
```
❌ ERROR: plan.md: Missing 'Supplementary Specifications' section

RESOLUTION:
  Run /speckit.plan to regenerate plan.md with auto-discovery
```

---

### Check 5: Orphaned Spec Detection

**Rule**: Supplementary specs must be referenced in ≥1 workflow file

**Pass**:
```
✓ PASS: UI-SPEC.md referenced in 2 workflow files
  - plan.md
  - tasks.md
```

**Fail**:
```
❌ ERROR: API-SPEC.md: Orphaned (not referenced in plan.md or tasks.md)

RESOLUTION:
  Run /speckit.plan to integrate API-SPEC.md
```

---

### Check 6: Agent Configuration

**Rule**: Agents in `target_agents` should exist in routing config

**Pass**:
```
✓ PASS: UI-SPEC.md: Agent 'frontend-architect' in routing config
```

**Warning** (not blocking):
```
⚠️ WARNING: UI-SPEC.md: Agent 'custom-agent' not in routing config

Add to .specify/config.yml:
specify:
  supplementary_specs:
    agent_routing:
      custom-agent:
        - UI-SPEC.md
```

---

## Strict Mode

**User Decision**: Validation errors block workflow

### Exit Codes:

| Code | Meaning | Action |
|------|---------|--------|
| 0 | All checks passed | Continue workflow |
| 1 | Errors found | **BLOCK workflow** |
| 2 | Invalid usage | Fix command usage |

### CI/CD Integration:

```yaml
# .github/workflows/validate-specs.yml
- name: Validate Hierarchical Specs
  run: |
    /speckit.validate-hierarchy || exit 1  # Exit code 1 blocks pipeline
```

### Blocking Workflow:

```bash
# Before implement
/speckit.validate-hierarchy || {
  echo "❌ Validation failed. Fix errors before implementing."
  exit 1
}

/speckit.implement  # Only runs if validation passed
```

---

## Examples

### Example 1: All Checks Pass

```
=========================================
  Hierarchical Spec Validation
=========================================
Feature: specs/000003-billing-console/
Mode: STRICT

--- Checking Parent Spec ---
✓ PASS: Parent spec.md exists (97KB, 80 FRs)

--- Discovering Supplementary Specs ---
Found 1 supplementary spec(s):
  - UI-SPEC.md (182KB)

--- Checking Frontmatter ---
✓ PASS: UI-SPEC.md: Valid parent reference
✓ PASS: UI-SPEC.md: Valid scope (ui-ux-implementation)
✓ PASS: UI-SPEC.md: Has FR mapping
✓ PASS: UI-SPEC.md: Has target agents

--- Validating FR Coverage ---
✓ PASS: UI-SPEC.md: All FRs exist in parent

--- Checking plan.md Integration ---
✓ PASS: plan.md: Has 'Supplementary Specifications' section
✓ PASS: plan.md: References UI-SPEC.md

--- Checking for Orphans ---
✓ PASS: UI-SPEC.md: Referenced in 2 files

=========================================
  Validation Summary
=========================================
Supplementary specs: 1
Total checks: 8
Passed: 8
Errors: 0

✅ Validation PASSED - Workflow continues
```

---

### Example 2: Validation Fails (Strict Mode Blocks)

```
=========================================
  Hierarchical Spec Validation
=========================================
Feature: specs/000003-billing-console/
Mode: STRICT

--- Checking Parent Spec ---
✓ PASS: Parent spec.md exists

--- Discovering Supplementary Specs ---
Found 1 supplementary spec(s):
  - UI-SPEC.md

--- Checking Frontmatter ---
❌ ERROR: UI-SPEC.md: Missing 'parent' field

Expected frontmatter:
---
parent: spec.md
scope: ui-ux-implementation
...
---

--- Checking plan.md Integration ---
❌ ERROR: plan.md: Does not reference UI-SPEC.md

Run /speckit.plan to regenerate

=========================================
  Validation Summary
=========================================
Supplementary specs: 1
Total checks: 5
Passed: 2
Errors: 2

❌ Validation FAILED - Workflow BLOCKED

Fix errors above before continuing.
Exit code: 1
```

**Result**: Workflow cannot continue until errors fixed.

---

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/pr-validation.yml
name: PR Validation

on:
  pull_request:
    paths:
      - 'specs/**/*-SPEC.md'
      - 'specs/**/plan.md'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Validate Hierarchical Specs
        run: |
          for feature_dir in specs/*/; do
            if ls "${feature_dir}"*-SPEC.md 1> /dev/null 2>&1; then
              /speckit.validate-hierarchy "${feature_dir}" || exit 1
            fi
          done
```

### GitLab CI

```yaml
# .gitlab-ci.yml
validate-specs:
  stage: test
  script:
    - |
      for feature_dir in specs/*/; do
        if ls "${feature_dir}"*-SPEC.md 1> /dev/null 2>&1; then
          /speckit.validate-hierarchy "${feature_dir}" || exit 1
        fi
      done
  only:
    refs:
      - merge_requests
    changes:
      - specs/**/*-SPEC.md
      - specs/**/plan.md
```

---

## Troubleshooting

### Issue: Frontmatter Looks Correct But Fails

**Problem**: Validation fails despite correct-looking frontmatter

**Diagnosis**:
```bash
# Check for invisible characters
sed -n '/^---$/,/^---$/p' UI-SPEC.md | cat -A

# Look for:
- ^M (carriage returns)
- Extra spaces/tabs
- Wrong quote types
```

**Resolution**: Re-save file with correct encoding (UTF-8, Unix line endings).

---

### Issue: plan.md Missing Supplementary Section

**Problem**: plan.md doesn't reference supplementary specs

**Diagnosis**:
```bash
grep "## Supplementary Specifications" plan.md
# No output = section missing
```

**Resolution**:
```bash
# Regenerate plan
/speckit.plan

# Validate
/speckit.validate-hierarchy
```

---

### Issue: FR Coverage Fails

**Problem**: Supplementary spec references non-existent FRs

**Resolution Option A** (Add FR to parent):
```bash
# Use reconcile to add missing FR
/speckit.reconcile

# Or manually edit spec.md
```

**Resolution Option B** (Fix supplementary spec):
```yaml
# Edit UI-SPEC.md frontmatter
functional_requirements: [FR-001 through FR-080]  # Was FR-085
```

---

### Issue: Orphaned Spec

**Problem**: Supplementary spec not referenced anywhere

**Diagnosis**:
```bash
grep "UI-SPEC.md" plan.md tasks.md
# No output = orphaned
```

**Resolution**:
```bash
# Regenerate plan (should auto-discover)
/speckit.plan

# Validate
/speckit.validate-hierarchy
```

---

## Best Practices

### 1. Validate Early and Often

```bash
# After creating supplementary spec
/speckit.supplement ui-ux "..."
/speckit.validate-hierarchy  # Immediate validation

# After editing
vim UI-SPEC.md
/speckit.validate-hierarchy  # Re-validate

# Before committing
git add .
/speckit.validate-hierarchy || exit 1  # Block commit on errors
git commit -m "..."
```

### 2. Fix Errors Immediately

Strict mode blocks workflow, so fix errors right away:

```
❌ ERROR: Missing parent reference

↓ IMMEDIATE FIX:
1. Edit UI-SPEC.md
2. Add frontmatter
3. Re-run validation
4. Continue workflow
```

### 3. Automate in CI/CD

Add validation as pre-merge check (examples above).

### 4. Keep Specs in Sync

When parent changes:

```bash
# Parent spec.md updated (FRs added)
# Update supplementary spec frontmatter
vim UI-SPEC.md
# Change: functional_requirements: [FR-001 through FR-085]

# Validate
/speckit.validate-hierarchy
```

---

## Configuration

**Config**: `.specify/config.yml`

```yaml
specify:
  supplementary_specs:
    validation:
      strict_mode: true  # Errors block workflow
      run_after_plan: true  # Auto-validate after /speckit.plan
      run_before_implement: true  # Block implement if invalid
```

---

## Related Commands

- `/speckit.supplement` - Create supplementary specs (validate after)
- `/speckit.plan` - Auto-discovers specs (validate after)
- `/speckit.reconcile` - Updates specs (validate after)
- `/speckit.implement` - Blocked if validation fails

---

## Next Steps

1. **Read Full Spec**: [validate-hierarchy.md](../templates/commands/validate-hierarchy.md)
2. **Hierarchical Guide**: [HIERARCHICAL-SPECS-GUIDE.md](HIERARCHICAL-SPECS-GUIDE.md)
3. **Reconcile Guide**: [RECONCILE-GUIDE.md](RECONCILE-GUIDE.md)

---

**Version**: 2.1.0
**Last Updated**: 2025-11-01
**Mode**: STRICT (errors block workflow per user decision)
