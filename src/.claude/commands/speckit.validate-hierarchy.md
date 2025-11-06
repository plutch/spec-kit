# Validate Hierarchy - Strict Validation of Supplementary Specs

**Command**: `/speckit.validate-hierarchy`
**Phase**: Any (typically after `/speckit.plan` or `/speckit.supplement`)
**Purpose**: Strict validation of parent-child spec relationships
**Mode**: STRICT (errors block workflow per user decision)

---

## Overview

The validate-hierarchy command performs comprehensive validation of hierarchical specifications, ensuring that supplementary specs (UI-SPEC.md, API-SPEC.md, TECHNICAL-SPEC.md) are properly integrated with the parent spec.md and referenced in workflow files (plan.md, tasks.md).

**Validation Mode**: **STRICT**
- Errors block workflow (exit code 1)
- No warnings-only mode
- Enforce best practices and spec integrity

---

## When to Use

**Mandatory** after:
- ✅ `/speckit.supplement` - Immediately validate new supplementary spec
- ✅ `/speckit.plan` - Ensure plan.md references all supplementary specs
- ✅ `/speckit.reconcile` - Validate reconciliation edits to supplementary specs

**Recommended** before:
- ✅ `/speckit.implement` - Block implementation if specs invalid
- ✅ Git commit/PR merge - CI/CD gate

**Optional** during:
- Development (check spec integrity at any time)

---

## Usage

```bash
/speckit.validate-hierarchy [feature-dir]
```

**Examples**:
```bash
# Validate current feature
/speckit.validate-hierarchy

# Validate specific feature
/speckit.validate-hierarchy specs/000003-billing-console/

# CI/CD usage (strict mode exits with code 1 on errors)
/speckit.validate-hierarchy || exit 1
```

---

## Auto-Execution Mode (NEW v2.3)

**Validation Execution**: Commands automatically execute bash validation checks using the Bash tool

**Auto-Detection** (v2.3):
- **Feature Directory**: Automatically detect from git branch or `$SPECIFY_FEATURE`
  ```bash
  # Auto-detect feature
  BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)
  if [[ "$BRANCH" =~ ^feature/(.+)$ ]]; then
    FEATURE_DIR="specs/${BASH_REMATCH[1]}"
  fi
  ```

- **Supplementary Specs**: Automatically discover `*-SPEC.md` files
  ```bash
  find "$FEATURE_DIR" -maxdepth 1 -name "*-SPEC.md" -type f
  ```

- **Validation Checks**: Execute all bash validation checks automatically
  - Use Bash tool for live validation (not just examples)
  - Real-time error detection and reporting
  - Exit on first error (strict mode)

**Benefits**:
- ✅ No manual feature path specification needed
- ✅ Live validation with actual bash execution
- ✅ Immediate error feedback
- ✅ Workflow integration (blocks on failure)

---

## Validation Checks

### Check 1: Parent Spec Exists

**Rule**: `spec.md` must exist in feature directory

**Validation**:
```bash
if [ ! -f "${FEATURE_DIR}/spec.md" ]; then
  echo "❌ ERROR: Parent spec.md not found"
  exit 1
fi
```

**Error Example**:
```
❌ ERROR: Parent spec.md not found at specs/000003-billing-console/spec.md

RESOLUTION:
  Run /speckit.specify to create parent specification
```

---

### Check 2: Supplementary Spec Discovery

**Rule**: Find all `*-SPEC.md` files in feature directory

**Validation**:
```bash
SUPP_SPECS=$(find "${FEATURE_DIR}" -maxdepth 1 -name "*-SPEC.md")

if [ -z "$SUPP_SPECS" ]; then
  echo "✓ PASS: No supplementary specs (validation not applicable)"
  exit 0
fi
```

**Info Output**:
```
Found 1 supplementary spec(s):
  - UI-SPEC.md
```

---

### Check 3: Frontmatter Validation

**Rule**: All supplementary specs MUST have valid YAML frontmatter

**Required Fields**:
- `parent`: Must be "spec.md"
- `scope`: Must be valid scope (ui-ux-implementation, api-contracts, technical-architecture)
- `functional_requirements`: Must reference parent FRs
- `target_agents`: Array of agent identifiers (at least 1)
- `version`: Version number
- `last_updated`: ISO date

**Validation** (for each supplementary spec):
```bash
# Extract frontmatter
FRONTMATTER=$(sed -n '/^---$/,/^---$/p' "${SPEC_FILE}")

# Check parent field
PARENT=$(echo "$FRONTMATTER" | grep '^parent:' | awk '{print $2}')
if [ "$PARENT" != "spec.md" ]; then
  echo "❌ ERROR: ${SPEC_FILE}: Invalid or missing 'parent' field"
  echo "   Expected: parent: spec.md"
  echo "   Found: parent: ${PARENT}"
  exit 1
fi

# Check scope field
SCOPE=$(echo "$FRONTMATTER" | grep '^scope:' | awk '{print $2}')
VALID_SCOPES=("ui-ux-implementation" "api-contracts" "technical-architecture")
if [[ ! " ${VALID_SCOPES[@]} " =~ " ${SCOPE} " ]]; then
  echo "❌ ERROR: ${SPEC_FILE}: Invalid 'scope' field"
  echo "   Valid scopes: ui-ux-implementation, api-contracts, technical-architecture"
  echo "   Found: ${SCOPE}"
  exit 1
fi

# Check functional_requirements field
FR_MAPPING=$(echo "$FRONTMATTER" | grep '^functional_requirements:')
if [ -z "$FR_MAPPING" ]; then
  echo "❌ ERROR: ${SPEC_FILE}: Missing 'functional_requirements' field"
  echo "   All supplementary specs MUST map to parent FRs"
  exit 1
fi

# Check target_agents field
TARGET_AGENTS=$(echo "$FRONTMATTER" | grep -A5 '^target_agents:')
if [ -z "$TARGET_AGENTS" ]; then
  echo "❌ ERROR: ${SPEC_FILE}: Missing 'target_agents' field"
  echo "   At least one target agent must be specified"
  exit 1
fi

echo "✓ PASS: ${SPEC_FILE}: Valid frontmatter"
```

**Error Example**:
```
❌ ERROR: UI-SPEC.md: Missing 'parent' field in frontmatter

RESOLUTION:
  Add frontmatter to UI-SPEC.md:
  ---
  parent: spec.md
  scope: ui-ux-implementation
  functional_requirements: [FR-001 through FR-080]
  target_agents:
    - frontend-architect
  version: 1.0
  last_updated: 2025-11-01
  ---
```

---

### Check 4: FR Coverage Validation

**Rule**: Functional requirements in supplementary spec must exist in parent spec.md

**Validation**:
```bash
# Extract FRs from parent spec.md
PARENT_FRS=$(grep -oP 'FR-\d{3}' "${FEATURE_DIR}/spec.md" | sort -u)

# Extract FR range from supplementary spec frontmatter
# Example: [FR-001 through FR-080] → FR-001, FR-002, ..., FR-080
SUPP_FR_RANGE=$(echo "$FRONTMATTER" | grep 'functional_requirements:' | sed 's/.*\[//;s/\].*//')

# Validate FRs exist in parent
for FR in $(echo "$SUPP_FR_RANGE" | tr ',' '\n'); do
  FR_NUM=$(echo "$FR" | grep -oP 'FR-\d{3}')
  if ! echo "$PARENT_FRS" | grep -q "$FR_NUM"; then
    echo "❌ ERROR: ${SPEC_FILE}: References non-existent FR: $FR_NUM"
    echo "   Parent spec.md does not contain $FR_NUM"
    exit 1
  fi
done

echo "✓ PASS: ${SPEC_FILE}: FR coverage validated"
```

**Error Example**:
```
❌ ERROR: UI-SPEC.md: References non-existent FR: FR-085

Parent spec.md contains: FR-001 through FR-080
UI-SPEC.md claims: FR-001 through FR-085

RESOLUTION:
  Update UI-SPEC.md frontmatter to match parent FR range
```

---

### Check 5: plan.md Integration

**Rule**: plan.md MUST reference all supplementary specs

**Validation**:
```bash
# Check if plan.md exists
if [ ! -f "${FEATURE_DIR}/plan.md" ]; then
  echo "⚠️  SKIP: plan.md not found (run /speckit.plan first)"
  # Not an error, just skip this check
  return
fi

# Check for "Supplementary Specifications" section
if ! grep -q '## Supplementary Specifications' "${FEATURE_DIR}/plan.md"; then
  echo "❌ ERROR: plan.md: Missing 'Supplementary Specifications' section"
  echo "   Run /speckit.plan to regenerate plan.md with supplementary spec references"
  exit 1
fi

# Check each supplementary spec is referenced
for SPEC_FILE in $SUPP_SPECS; do
  SPEC_NAME=$(basename "$SPEC_FILE")
  if ! grep -q "\[${SPEC_NAME}\]" "${FEATURE_DIR}/plan.md"; then
    echo "❌ ERROR: plan.md: Does not reference ${SPEC_NAME}"
    echo "   Run /speckit.plan to regenerate plan.md"
    exit 1
  fi
done

echo "✓ PASS: plan.md references all supplementary specs"
```

**Error Example**:
```
❌ ERROR: plan.md: Missing 'Supplementary Specifications' section

RESOLUTION:
  Run /speckit.plan to regenerate plan.md with auto-discovery:
  /speckit.plan

  This will add a "Supplementary Specifications" section referencing:
    - UI-SPEC.md
    - API-SPEC.md
    - TECHNICAL-SPEC.md
```

---

### Check 6: tasks.md Integration

**Rule**: tasks.md SHOULD reference supplementary spec sections (warning for now, may become error in future)

**Validation**:
```bash
# Check if tasks.md exists
if [ ! -f "${FEATURE_DIR}/tasks.md" ]; then
  echo "⚠️  SKIP: tasks.md not found (run /speckit.tasks first)"
  return
fi

# Check for references to supplementary specs
for SPEC_FILE in $SUPP_SPECS; do
  SPEC_NAME=$(basename "$SPEC_FILE")
  REF_COUNT=$(grep -c "\[${SPEC_NAME}\]" "${FEATURE_DIR}/tasks.md" || echo "0")

  if [ "$REF_COUNT" -eq 0 ]; then
    echo "⚠️  WARNING: tasks.md: No references to ${SPEC_NAME}"
    echo "   Consider adding task references like: [${SPEC_NAME} Section X.Y]"
    # Not an error in strict mode, just informational
  else
    echo "✓ PASS: tasks.md: Contains ${REF_COUNT} reference(s) to ${SPEC_NAME}"
  fi
done
```

---

### Check 7: Agent Configuration

**Rule**: Agents listed in `target_agents` must exist in agent routing configuration

**Validation**:
```bash
# Load agent routing config from .specify/config.yml
AGENT_ROUTING=$(yq eval '.specify.supplementary_specs.agent_routing' .specify/config.yml)

# Check each target agent
for AGENT in $(echo "$TARGET_AGENTS" | grep '^  -' | awk '{print $2}'); do
  if ! echo "$AGENT_ROUTING" | grep -q "$AGENT"; then
    echo "⚠️  WARNING: ${SPEC_FILE}: Agent '${AGENT}' not in routing config"
    echo "   Add to .specify/config.yml agent_routing section"
    # Warning only, not blocking
  fi
done

echo "✓ PASS: ${SPEC_FILE}: Target agents validated"
```

---

### Check 8: Orphaned Spec Detection

**Rule**: Supplementary specs must be referenced in at least one workflow file

**Validation**:
```bash
# Check if spec is referenced anywhere
SPEC_NAME=$(basename "$SPEC_FILE")
REFERENCES=0

# Check plan.md
if [ -f "${FEATURE_DIR}/plan.md" ] && grep -q "${SPEC_NAME}" "${FEATURE_DIR}/plan.md"; then
  ((REFERENCES++))
fi

# Check tasks.md
if [ -f "${FEATURE_DIR}/tasks.md" ] && grep -q "${SPEC_NAME}" "${FEATURE_DIR}/tasks.md"; then
  ((REFERENCES++))
fi

if [ "$REFERENCES" -eq 0 ]; then
  echo "❌ ERROR: ${SPEC_FILE}: Orphaned (not referenced in any workflow files)"
  echo "   Run /speckit.plan to integrate supplementary spec"
  exit 1
fi

echo "✓ PASS: ${SPEC_FILE}: Referenced in ${REFERENCES} workflow file(s)"
```

**Error Example**:
```
❌ ERROR: API-SPEC.md: Orphaned (not referenced in plan.md or tasks.md)

RESOLUTION:
  Run /speckit.plan to regenerate plan.md with API-SPEC.md reference
```

---

### Check 9: File Size Validation

**Rule**: Warn if supplementary spec is too small (likely incomplete)

**Validation**:
```bash
SPEC_SIZE_KB=$(du -k "$SPEC_FILE" | cut -f1)
MIN_SIZE_KB=5  # Minimum 5KB for a meaningful spec

if [ "$SPEC_SIZE_KB" -lt "$MIN_SIZE_KB" ]; then
  echo "⚠️  WARNING: ${SPEC_FILE}: File size is ${SPEC_SIZE_KB}KB (expected >5KB)"
  echo "   Spec may be incomplete. Review PLACEHOLDER sections."
  # Warning only, not blocking
fi
```

---

## Validation Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate hierarchical spec validation completeness before presenting results.

### Evidence Collection (Mandatory)

❓ **"Were all supplementary specs discovered?"**
Action Required:
  - List all *-SPEC.md files found in feature directory
  - Show ACTUAL file paths
  - Report: Count and filenames

Expected Evidence:
  ✓ File discovery completed (find command executed)
  ✓ Supplementary spec count: [N] files
  ✓ File list: [UI-SPEC.md, API-SPEC.md, TECHNICAL-SPEC.md...]

❓ **"Did all validation checks execute?"**
Action Required:
  - Verify all 6 check types ran
  - Show ACTUAL check results (PASS/FAIL for each)
  - Report: Check execution summary

Expected Evidence:
  ✓ Check 1 - Parent Spec Exists: [PASS/FAIL]
  ✓ Check 2 - Frontmatter Validation: [PASS/FAIL]
  ✓ Check 3 - FR Coverage Validation: [PASS/FAIL]
  ✓ Check 4 - plan.md Integration: [PASS/SKIP]
  ✓ Check 5 - tasks.md Integration: [PASS/SKIP]
  ✓ Check 6 - Orphaned Spec Detection: [PASS/FAIL]

❓ **"Are errors surfaced with evidence?"**
Action Required:
  - Count errors by check type
  - Show ACTUAL error messages
  - Report: Error count and descriptions

Expected Evidence:
  ✓ Total errors: [N]
  ✓ Error breakdown by check:
    - Frontmatter errors: [N]
    - FR coverage errors: [N]
    - Integration errors: [N]
    - Orphaned specs: [N]
  ✓ Error messages include: File path, specific issue, resolution steps

❓ **"Is strict mode enforced correctly?"**
Action Required:
  - Verify strict mode blocking logic
  - Show ACTUAL exit code/status
  - Report: Blocking status if errors found

Expected Evidence:
  ✓ Strict mode: ENABLED (errors block workflow)
  ✓ Exit status: [0 = no errors, 1 = errors found, blocks workflow]
  ✓ IF errors > 0: Workflow BLOCKED with error list

IF any evidence is MISSING:
  ❌ CANNOT report completion
  → Gather missing evidence first
  → Re-run this step with complete evidence

### Hallucination Prevention (7 Red Flags for Validation)

```yaml
Detect and BLOCK these patterns:

🚨 "Validation passed" WITHOUT showing which checks ran
   → Self-correction: "Wait, I need to show all 6 check results"

🚨 "No errors" WITHOUT showing file discovery count
   → Self-correction: "Must show how many supplementary specs were found"

🚨 "Hierarchy valid" WITHOUT checking frontmatter
   → Self-correction: "Need to verify YAML frontmatter for each spec"

🚨 Claiming "all specs validated" WITHOUT file list
   → Self-correction: "I must list actual supplementary spec files found"

🚨 "Ready to proceed" WITH errors in strict mode
   → Self-correction: "Errors BLOCK workflow in strict mode, cannot proceed"

🚨 Skipping circular dependency check
   → Self-correction: "Must check for circular parent references"

🚨 "Validation complete" WITHOUT showing check-by-check results
   → Self-correction: "Need detailed check results for transparency"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

✅ **READY (Validation Passed)**:
```yaml
Criteria (ALL must be met):
  - All supplementary specs discovered and listed
  - All 6 validation checks executed
  - No errors found (error count = 0)
  - Frontmatter valid for all specs
  - FR coverage verified
  - Integration checks passed (or skipped if files missing)
  - No orphaned specs detected

IF ALL criteria met:
  → Workflow can continue
  → Present success report
```

⚠️ **WARNINGS Present (Non-Strict Mode)**:
```yaml
Criteria:
  - Validation checks passed
  - Minor warnings present (non-blocking)
  - tasks.md has no references to supplementary specs (warning only)
  - File size warnings (spec too small, likely incomplete)

IF criteria met:
  → Present warnings to user
  → Workflow can continue with awareness
```

❌ **NOT READY (Errors Block Workflow)**:
```yaml
Criteria (ANY triggers NOT READY in STRICT MODE):
  - Frontmatter missing or invalid (no parent: field)
  - FR coverage mismatch (spec references non-existent FRs)
  - plan.md missing supplementary spec references
  - Orphaned specs (not referenced in workflow files)
  - Parent spec.md not found

IF NOT READY:
  → Exit code 1 (blocks CI/CD)
  → Present errors with resolution steps
  → Recommend: "Fix errors before continuing workflow"
  → STOP workflow progression
```

### Output Format (Present to User - ONLY if evidence provided)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Hierarchical Spec Validation Review
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ READY | ⚠️ WARNINGS | ❌ NOT READY]

**Feature**: specs/[FEATURE_ID]/
**Mode**: STRICT (errors block workflow)
**Exit Code**: [0 = success, 1 = failure]

**Supplementary Specs Discovered**: [N]
  - [UI-SPEC.md]
  - [API-SPEC.md]
  - [TECHNICAL-SPEC.md]

**Validation Checks Executed**:
  [✅ PASS | ❌ FAIL | ⚠️ SKIP] Check 1 - Parent Spec Exists
  [✅ PASS | ❌ FAIL | ⚠️ SKIP] Check 2 - Frontmatter Validation
  [✅ PASS | ❌ FAIL | ⚠️ SKIP] Check 3 - FR Coverage Validation
  [✅ PASS | ❌ FAIL | ⚠️ SKIP] Check 4 - plan.md Integration
  [✅ PASS | ❌ FAIL | ⚠️ SKIP] Check 5 - tasks.md Integration
  [✅ PASS | ❌ FAIL | ⚠️ SKIP] Check 6 - Orphaned Spec Detection

**Error Summary**:
  - Total errors: [N]
  - Frontmatter errors: [N]
  - FR coverage errors: [N]
  - Integration errors: [N]
  - Orphaned specs: [N]

[IF errors > 0]
**Errors Found**:
1. [Error description]
   File: [SPEC_FILE]
   Fix: [Resolution steps]

2. [Error description]
   File: [SPEC_FILE]
   Fix: [Resolution steps]

[IF warnings > 0]
**Warnings**:
- [Warning 1]
- [Warning 2]

Next Action: [Workflow continues OR Fix errors before proceeding]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Validation Report

**Output Format**:

```
=========================================
  Hierarchical Spec Validation
=========================================
Feature: specs/000003-billing-console/
Mode: STRICT (errors block workflow)

--- Checking Parent Spec ---
✓ PASS: Parent spec.md exists (97KB, 80 FRs)

--- Discovering Supplementary Specs ---
Found 1 supplementary spec(s):
  - UI-SPEC.md (182KB)

--- Checking Frontmatter ---
✓ PASS: UI-SPEC.md: Has valid parent reference (spec.md)
✓ PASS: UI-SPEC.md: Has valid scope (ui-ux-implementation)
✓ PASS: UI-SPEC.md: Has functional requirements mapping
✓ PASS: UI-SPEC.md: Has target agents defined (frontend-architect)
✓ PASS: UI-SPEC.md: Has version (1.0)
✓ PASS: UI-SPEC.md: Has last_updated (2025-11-01)

--- Validating FR Coverage ---
✓ PASS: UI-SPEC.md: All FRs exist in parent spec.md
  Coverage: FR-001 through FR-080 (80 FRs)

--- Checking plan.md Integration ---
✓ PASS: plan.md: Has 'Supplementary Specifications' section
✓ PASS: plan.md: References UI-SPEC.md

--- Checking tasks.md Integration ---
✓ PASS: tasks.md: Contains 15 reference(s) to UI-SPEC.md

--- Checking Agent Configuration ---
✓ PASS: UI-SPEC.md: Target agent 'frontend-architect' in routing config

--- Checking for Orphans ---
✓ PASS: UI-SPEC.md: Referenced in 2 workflow file(s) (plan.md, tasks.md)

--- File Size Check ---
✓ PASS: UI-SPEC.md: File size is 182KB (well above minimum)

=========================================
  Validation Summary
=========================================
Feature: specs/000003-billing-console/
Supplementary specs: 1
Total checks: 12
Passed: 12
Warnings: 0
Errors: 0

✅ Validation PASSED - All checks successful!

Workflow can continue.
```

---

## Error Example (Strict Mode Blocking)

```
=========================================
  Hierarchical Spec Validation
=========================================
Feature: specs/000003-billing-console/
Mode: STRICT (errors block workflow)

--- Checking Parent Spec ---
✓ PASS: Parent spec.md exists

--- Discovering Supplementary Specs ---
Found 1 supplementary spec(s):
  - UI-SPEC.md

--- Checking Frontmatter ---
❌ ERROR: UI-SPEC.md: Missing 'parent' field in frontmatter

Expected frontmatter:
---
parent: spec.md
scope: ui-ux-implementation
functional_requirements: [FR-001 through FR-080]
target_agents:
  - frontend-architect
version: 1.0
last_updated: 2025-11-01
---

--- Checking plan.md Integration ---
❌ ERROR: plan.md: Does not reference UI-SPEC.md

Run /speckit.plan to regenerate plan.md

=========================================
  Validation Summary
=========================================
Feature: specs/000003-billing-console/
Supplementary specs: 1
Total checks: 5
Passed: 2
Warnings: 0
Errors: 2

❌ Validation FAILED - Workflow BLOCKED

Fix errors above before continuing.
Exit code: 1
```

**CI/CD Integration**:
```yaml
# .github/workflows/validate-specs.yml
- name: Validate Hierarchical Specs
  run: |
    for feature_dir in specs/*/; do
      if ls "$feature_dir"*-SPEC.md 1> /dev/null 2>&1; then
        /speckit.validate-hierarchy "$feature_dir" || exit 1
      fi
    done
```

---

## Integration with Other Commands

### Auto-Run After Commands

**Recommended**: Run validation automatically after these commands:

```bash
# After supplement
/speckit.supplement ui-ux "..." && /speckit.validate-hierarchy

# After plan
/speckit.plan && /speckit.validate-hierarchy

# After reconcile
/speckit.reconcile && /speckit.validate-hierarchy
```

### Block Implementation

**Required**: Validation must pass before implementation:

```bash
# Before implement
/speckit.validate-hierarchy || {
  echo "❌ Validation failed. Fix errors before implementing."
  exit 1
}

/speckit.implement
```

---

## Configuration

**Config**: `.specify/config.yml`

```yaml
specify:
  supplementary_specs:
    enabled: true
    patterns: ["*-SPEC.md"]
    require_parent_reference: true
    validate_fr_coverage: true

    validation:
      strict_mode: true  # Per user decision: errors block workflow
      run_after_plan: true  # Auto-validate after /speckit.plan
      run_before_implement: true  # Block /speckit.implement if validation fails
```

---

## Best Practices

### 1. Validate Early and Often

Run validation immediately after creating or modifying supplementary specs:

```bash
# Create supplementary spec
/speckit.supplement ui-ux "..."

# Validate immediately
/speckit.validate-hierarchy

# Fix any errors before continuing
```

### 2. CI/CD Gate

Add validation as a pre-merge check:

```yaml
# .github/workflows/pr-validation.yml
name: PR Validation

on:
  pull_request:
    paths:
      - 'specs/**/*-SPEC.md'
      - 'specs/**/plan.md'
      - 'specs/**/tasks.md'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Validate Hierarchical Specs
        run: |
          # Install SpecKit (if not in repo)
          # Run validation on changed features
          for feature_dir in $(git diff --name-only origin/main... | grep 'specs/' | cut -d'/' -f1-2 | sort -u); do
            if ls "${feature_dir}/"*-SPEC.md 1> /dev/null 2>&1; then
              /speckit.validate-hierarchy "${feature_dir}/" || exit 1
            fi
          done
```

### 3. Fix Errors Immediately

Strict mode blocks workflow, so fix errors right away:

```
❌ ERROR: UI-SPEC.md: Missing parent reference

↓ IMMEDIATE FIX:
1. Edit UI-SPEC.md
2. Add frontmatter with parent: spec.md
3. Re-run /speckit.validate-hierarchy
4. Continue workflow
```

### 4. Keep Specs in Sync

If parent spec.md changes (FRs added/removed), update supplementary spec frontmatter:

```yaml
# Update FR range in UI-SPEC.md frontmatter
functional_requirements: [FR-001 through FR-085]  # Was FR-080, now FR-085
```

---

## Troubleshooting

### Issue: Validation Fails But Looks Correct

**Problem**: Frontmatter looks correct but validation fails

**Solution**: Check for invisible characters, incorrect YAML syntax

```bash
# Debug frontmatter parsing
sed -n '/^---$/,/^---$/p' UI-SPEC.md | cat -A

# Look for ^M (carriage returns), extra spaces, tabs
```

### Issue: plan.md Missing Supplementary Section

**Problem**: Validation says plan.md doesn't reference supplementary specs

**Solution**: Regenerate plan.md

```bash
# Backup existing plan
cp specs/000003-billing-console/plan.md specs/000003-billing-console/plan.md.bak

# Regenerate plan (will auto-discover supplementary specs)
/speckit.plan

# Validate
/speckit.validate-hierarchy
```

### Issue: FR Coverage Validation Fails

**Problem**: Supplementary spec references FR that doesn't exist in parent

**Solution**: Either add FR to parent spec.md or fix supplementary spec frontmatter

```bash
# Option A: Add missing FR to parent spec.md
# (Use /speckit.reconcile or manual edit)

# Option B: Fix supplementary spec frontmatter
# Edit UI-SPEC.md frontmatter to remove non-existent FRs
```

---

## Exit Codes

| Code | Meaning | Workflow Action |
|------|---------|-----------------|
| 0 | All checks passed | Continue workflow |
| 1 | Validation errors found | **BLOCK workflow** (strict mode) |
| 2 | Invalid usage (missing args) | Fix command usage |

**Strict Mode**: Exit code 1 blocks CI/CD pipelines and prevents workflow continuation.

---

## Related Commands

- `/speckit.supplement` - Create supplementary specs (validate after)
- `/speckit.plan` - Auto-discovers supplementary specs (validate after)
- `/speckit.reconcile` - Updates supplementary specs (validate after)
- `/speckit.implement` - Should be blocked if validation fails

---

## Example: Full Validation Flow

```bash
# 1. Create supplementary spec
/speckit.supplement ui-ux "Complete UI/UX specifications"

✓ Created UI-SPEC.md

# 2. Edit UI-SPEC.md to fill in PLACEHOLDER sections
# (Manual editing)

# 3. Validate (STRICT mode)
/speckit.validate-hierarchy

❌ FAILED: Missing frontmatter fields
Fix errors...

# 4. Add frontmatter, validate again
/speckit.validate-hierarchy

✅ PASSED: All checks successful

# 5. Generate plan (auto-discovers UI-SPEC.md)
/speckit.plan

# 6. Validate plan integration
/speckit.validate-hierarchy

✅ PASSED: plan.md references UI-SPEC.md

# 7. Generate tasks
/speckit.tasks

# 8. Final validation before implementation
/speckit.validate-hierarchy || exit 1

✅ PASSED: Ready to implement

# 9. Implement (validation passed, workflow continues)
/speckit.implement
```

---

**Command Version**: 2.3.0
**Last Updated**: 2025-11-05
**Compatibility**: SpecKit v2.3+
**Validation Mode**: STRICT (errors block workflow)
**New in v2.3**: Auto-detection of feature directory from git branch, automatic bash execution for validation checks, live error reporting
