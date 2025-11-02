# Supplementary Spec Discovery Pattern

**Pattern**: Reusable logic for discovering and processing supplementary specifications
**Used By**: plan.md, reconcile.md, validate-hierarchy.md, implement.md
**Version**: 2.1.0

---

## Purpose

This pattern provides standardized logic for:
1. Discovering supplementary specs (*-SPEC.md files)
2. Parsing frontmatter metadata
3. Validating spec structure
4. Generating references

---

## Discovery Logic

### Step 1: Find Supplementary Specs

```bash
# Current feature directory
FEATURE_DIR=$(pwd)
# Or from argument: FEATURE_DIR=$1

# Glob pattern for supplementary specs
SUPP_SPECS=$(find "${FEATURE_DIR}" -maxdepth 1 -name "*-SPEC.md" -type f)

# Count discovered specs
SUPP_SPEC_COUNT=$(echo "$SUPP_SPECS" | wc -l)

if [ -z "$SUPP_SPECS" ]; then
  echo "No supplementary specs found (standard single-spec workflow)"
  SUPPLEMENTARY_ENABLED=false
  return 0
else
  echo "Discovered ${SUPP_SPEC_COUNT} supplementary spec(s):"
  echo "$SUPP_SPECS" | while read -r spec; do
    echo "  - $(basename "$spec")"
  done
  SUPPLEMENTARY_ENABLED=true
fi
```

---

## Frontmatter Parsing

### Step 2: Extract Frontmatter

```bash
# For each supplementary spec
for SPEC_FILE in $SUPP_SPECS; do
  SPEC_NAME=$(basename "$SPEC_FILE")

  # Extract YAML frontmatter (between first two --- markers)
  FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$SPEC_FILE" | head -n -1)

  if [ -z "$FRONTMATTER" ]; then
    echo "⚠️  WARNING: ${SPEC_NAME} missing frontmatter"
    continue
  fi

  # Parse individual fields
  PARENT=$(echo "$FRONTMATTER" | grep '^parent:' | awk '{print $2}')
  SCOPE=$(echo "$FRONTMATTER" | grep '^scope:' | awk '{print $2}')
  FR_LIST=$(echo "$FRONTMATTER" | grep '^functional_requirements:' | sed 's/.*\[//; s/\].*//')
  TARGET_AGENTS=$(echo "$FRONTMATTER" | sed -n '/^target_agents:/,/^[^ ]/p' | grep '^  -' | awk '{print $2}')
  VERSION=$(echo "$FRONTMATTER" | grep '^version:' | awk '{print $2}')
  LAST_UPDATED=$(echo "$FRONTMATTER" | grep '^last_updated:' | awk '{print $2}')

  # Store parsed data (example using associative array)
  declare -A SPEC_META
  SPEC_META["name"]=$SPEC_NAME
  SPEC_META["parent"]=$PARENT
  SPEC_META["scope"]=$SCOPE
  SPEC_META["fr_list"]=$FR_LIST
  SPEC_META["target_agents"]=$TARGET_AGENTS
  SPEC_META["version"]=$VERSION
  SPEC_META["last_updated"]=$LAST_UPDATED

  # Make available for downstream processing
  export SPEC_META_${SPEC_NAME/.md/}="${SPEC_META[@]}"
done
```

---

## Validation

### Step 3: Basic Validation

```bash
validate_supplementary_spec() {
  local SPEC_FILE=$1
  local SPEC_NAME=$(basename "$SPEC_FILE")
  local ERRORS=0

  # Check file exists
  if [ ! -f "$SPEC_FILE" ]; then
    echo "❌ ERROR: ${SPEC_NAME} not found at ${SPEC_FILE}"
    return 1
  fi

  # Check frontmatter exists
  FRONTMATTER=$(sed -n '/^---$/,/^---$/p' "$SPEC_FILE")
  if [ -z "$FRONTMATTER" ]; then
    echo "❌ ERROR: ${SPEC_NAME} missing YAML frontmatter"
    ((ERRORS++))
  fi

  # Check required fields
  PARENT=$(echo "$FRONTMATTER" | grep '^parent:' | awk '{print $2}')
  if [ "$PARENT" != "spec.md" ]; then
    echo "❌ ERROR: ${SPEC_NAME} invalid parent (expected: spec.md, found: ${PARENT})"
    ((ERRORS++))
  fi

  SCOPE=$(echo "$FRONTMATTER" | grep '^scope:')
  if [ -z "$SCOPE" ]; then
    echo "❌ ERROR: ${SPEC_NAME} missing 'scope' field"
    ((ERRORS++))
  fi

  FR_LIST=$(echo "$FRONTMATTER" | grep '^functional_requirements:')
  if [ -z "$FR_LIST" ]; then
    echo "❌ ERROR: ${SPEC_NAME} missing 'functional_requirements' field"
    ((ERRORS++))
  fi

  if [ $ERRORS -gt 0 ]; then
    return 1
  fi

  return 0
}

# Validate all discovered specs
for SPEC_FILE in $SUPP_SPECS; do
  validate_supplementary_spec "$SPEC_FILE" || {
    echo "Validation failed for $(basename "$SPEC_FILE")"
    exit 1
  }
done
```

---

## Reference Generation

### Step 4: Generate Markdown References

```bash
generate_supplementary_section() {
  local FEATURE_DIR=$1

  cat << 'EOF'
## Supplementary Specifications

This feature includes detailed implementation guides beyond the core spec.md:

EOF

  for SPEC_FILE in $(find "${FEATURE_DIR}" -maxdepth 1 -name "*-SPEC.md"); do
    SPEC_NAME=$(basename "$SPEC_FILE")

    # Parse frontmatter
    FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$SPEC_FILE" | head -n -1)
    SCOPE=$(echo "$FRONTMATTER" | grep '^scope:' | cut -d':' -f2- | xargs)
    FR_LIST=$(echo "$FRONTMATTER" | grep '^functional_requirements:' | sed 's/.*\[//; s/\].*//')
    TARGET_AGENTS=$(echo "$FRONTMATTER" | sed -n '/^target_agents:/,/^[^ ]/p' | grep '^  -' | awk '{print $2}' | tr '\n' ', ' | sed 's/, $//')

    # Get file stats
    SIZE_KB=$(du -k "$SPEC_FILE" | cut -f1)
    LINES=$(wc -l < "$SPEC_FILE")

    cat << EOF

### ${SCOPE}
- **[${SPEC_NAME}](${SPEC_NAME})**
  - **Scope**: ${SCOPE}
  - **Coverage**: ${FR_LIST}
  - **Size**: ${SIZE_KB}KB, ${LINES} lines
  - **Target Agents**: ${TARGET_AGENTS}

EOF
  done

  cat << 'EOF'

### Agent Usage Requirements

**CRITICAL**: All implementation tasks MUST reference relevant supplementary specs.

**Token Efficiency**: Load supplementary specs ONCE at session start, reference sections during implementation.

**Example Task Reference**:
```markdown
#### Task: Implement Dashboard Component
- [ ] Review component design in [UI-SPEC.md Section 4.2](UI-SPEC.md#42-dashboard-component)
- [ ] Implement according to specifications
- [ ] Verify against acceptance criteria in [spec.md AC-015](spec.md#ac-015)
```

EOF
}

# Usage in plan.md
if [ "$SUPPLEMENTARY_ENABLED" = true ]; then
  echo "" >> plan.md
  generate_supplementary_section "$FEATURE_DIR" >> plan.md
fi
```

---

## Agent Routing

### Step 5: Determine Which Specs to Load for Agent

```bash
get_specs_for_agent() {
  local AGENT_TYPE=$1
  local FEATURE_DIR=$2

  # Load agent routing config from .specify/config.yml
  # (Simplified - actual implementation would use yq or jq)

  case "$AGENT_TYPE" in
    frontend-architect)
      AGENT_SPECS="UI-SPEC.md"
      ;;
    backend-architect)
      AGENT_SPECS="API-SPEC.md"
      ;;
    system-architect)
      AGENT_SPECS="TECHNICAL-SPEC.md API-SPEC.md"
      ;;
    postgres-entity-architect)
      AGENT_SPECS="TECHNICAL-SPEC.md"
      ;;
    *)
      AGENT_SPECS=""
      ;;
  esac

  # Filter to only existing specs
  for SPEC_NAME in $AGENT_SPECS; do
    SPEC_PATH="${FEATURE_DIR}/${SPEC_NAME}"
    if [ -f "$SPEC_PATH" ]; then
      echo "$SPEC_PATH"
    fi
  done
}

# Usage in implement.md
AGENT_TYPE="frontend-architect"
SPECS_TO_LOAD=$(get_specs_for_agent "$AGENT_TYPE" "$FEATURE_DIR")

if [ -n "$SPECS_TO_LOAD" ]; then
  echo "Loading supplementary specs for ${AGENT_TYPE}:"
  for SPEC_PATH in $SPECS_TO_LOAD; do
    SPEC_NAME=$(basename "$SPEC_PATH")
    SIZE_KB=$(du -k "$SPEC_PATH" | cut -f1)
    echo "  ✓ Loaded ${SPEC_NAME} (~${SIZE_KB}KB)"

    # Read spec content for agent context
    SPEC_CONTENT=$(cat "$SPEC_PATH")
    # Add to agent context...
  done
fi
```

---

## File Size Analysis

### Step 6: Analyze Spec Sizes

```bash
analyze_spec_sizes() {
  local FEATURE_DIR=$1

  PARENT_SIZE=$(du -k "${FEATURE_DIR}/spec.md" | cut -f1)
  TOTAL_SUPP_SIZE=0

  echo "Specification Size Analysis:"
  echo "  Parent spec.md: ${PARENT_SIZE}KB"

  if [ "$SUPPLEMENTARY_ENABLED" = true ]; then
    echo "  Supplementary specs:"
    for SPEC_FILE in $SUPP_SPECS; do
      SPEC_NAME=$(basename "$SPEC_FILE")
      SIZE_KB=$(du -k "$SPEC_FILE" | cut -f1)
      TOTAL_SUPP_SIZE=$((TOTAL_SUPP_SIZE + SIZE_KB))
      echo "    - ${SPEC_NAME}: ${SIZE_KB}KB"
    done

    TOTAL_SIZE=$((PARENT_SIZE + TOTAL_SUPP_SIZE))
    echo "  Total: ${TOTAL_SIZE}KB (parent: ${PARENT_SIZE}KB + supplementary: ${TOTAL_SUPP_SIZE}KB)"

    # Calculate token estimate (~4 chars per token)
    TOTAL_TOKENS=$((TOTAL_SIZE * 1024 / 4))
    echo "  Estimated tokens: ~${TOTAL_TOKENS}"

    # Show savings if hierarchical
    if [ $SUPP_SPEC_COUNT -gt 0 ]; then
      AVG_SUPP_SIZE=$((TOTAL_SUPP_SIZE / SUPP_SPEC_COUNT))
      TYPICAL_LOAD=$((PARENT_SIZE + AVG_SUPP_SIZE))
      SAVINGS=$(((TOTAL_SIZE - TYPICAL_LOAD) * 100 / TOTAL_SIZE))
      echo "  Token savings per session: ~${SAVINGS}% (load parent + 1 domain-specific vs all)"
    fi
  else
    echo "  Total: ${PARENT_SIZE}KB (single spec)"
  fi
}
```

---

## Integration Examples

### Example 1: In /speckit.plan

```bash
# Discover supplementary specs
source .specify/templates/commands/shared/supplementary-spec-discovery.md

# Generate plan.md
cat plan-template.md > plan.md

# If supplementary specs exist, append section
if [ "$SUPPLEMENTARY_ENABLED" = true ]; then
  echo "" >> plan.md
  generate_supplementary_section "$FEATURE_DIR" >> plan.md
fi
```

### Example 2: In /speckit.reconcile

```bash
# Discover supplementary specs
source .specify/templates/commands/shared/supplementary-spec-discovery.md

# Determine which spec to edit based on gap type
GAP_TYPE="missing_navigation"

if [ "$GAP_TYPE" = "missing_navigation" ] && [ -f "${FEATURE_DIR}/UI-SPEC.md" ]; then
  # Edit UI-SPEC.md for navigation gaps
  TARGET_SPEC="${FEATURE_DIR}/UI-SPEC.md"
elif [ "$GAP_TYPE" = "incomplete_integration" ] && [ -f "${FEATURE_DIR}/API-SPEC.md" ]; then
  # Edit API-SPEC.md for integration gaps
  TARGET_SPEC="${FEATURE_DIR}/API-SPEC.md"
else
  # Fallback to parent spec.md
  TARGET_SPEC="${FEATURE_DIR}/spec.md"
fi

# Apply edits to TARGET_SPEC
```

### Example 3: In /speckit.validate-hierarchy

```bash
# Discover and validate all supplementary specs
source .specify/templates/commands/shared/supplementary-spec-discovery.md

VALIDATION_ERRORS=0

for SPEC_FILE in $SUPP_SPECS; do
  validate_supplementary_spec "$SPEC_FILE" || {
    ((VALIDATION_ERRORS++))
  }
done

if [ $VALIDATION_ERRORS -gt 0 ]; then
  echo "❌ Validation failed: ${VALIDATION_ERRORS} error(s)"
  exit 1
fi

echo "✅ All supplementary specs validated successfully"
```

---

## Configuration Reference

**Config**: `.specify/config.yml`

```yaml
specify:
  supplementary_specs:
    enabled: true
    patterns: ["*-SPEC.md"]

    # Agent routing (which agents load which specs)
    agent_routing:
      frontend-architect:
        - UI-SPEC.md
      backend-architect:
        - API-SPEC.md
      system-architect:
        - TECHNICAL-SPEC.md
        - API-SPEC.md
      postgres-entity-architect:
        - TECHNICAL-SPEC.md
```

---

## Output Templates

### Terminal Output

```bash
# Discovery output
echo "🔍 Discovering supplementary specs..."
echo "   Pattern: *-SPEC.md"
echo "   Location: ${FEATURE_DIR}"
echo ""
echo "📋 Found ${SUPP_SPEC_COUNT} supplementary spec(s):"
for SPEC_FILE in $SUPP_SPECS; do
  SPEC_NAME=$(basename "$SPEC_FILE")
  SIZE_KB=$(du -k "$SPEC_FILE" | cut -f1)
  SCOPE=$(sed -n '/^scope:/p' "$SPEC_FILE" | cut -d':' -f2 | xargs)
  echo "   ✓ ${SPEC_NAME} (${SIZE_KB}KB, scope: ${SCOPE})"
done
```

### JSON Output (for tooling)

```bash
generate_json_output() {
  cat << EOF
{
  "supplementary_enabled": ${SUPPLEMENTARY_ENABLED},
  "count": ${SUPP_SPEC_COUNT},
  "specs": [
EOF

  FIRST=true
  for SPEC_FILE in $SUPP_SPECS; do
    if [ "$FIRST" = false ]; then
      echo ","
    fi
    FIRST=false

    SPEC_NAME=$(basename "$SPEC_FILE")
    FRONTMATTER=$(sed -n '/^---$/,/^---$/{ /^---$/d; p; }' "$SPEC_FILE" | head -n -1)
    SCOPE=$(echo "$FRONTMATTER" | grep '^scope:' | cut -d':' -f2- | xargs)

    cat << SPEC_EOF
    {
      "name": "${SPEC_NAME}",
      "path": "${SPEC_FILE}",
      "scope": "${SCOPE}",
      "size_kb": $(du -k "$SPEC_FILE" | cut -f1)
    }
SPEC_EOF
  done

  cat << EOF

  ]
}
EOF
}
```

---

## Best Practices

1. **Always check SUPPLEMENTARY_ENABLED** before processing supplementary specs
2. **Validate frontmatter** before using metadata
3. **Handle missing files gracefully** (spec might be deleted between discovery and processing)
4. **Cache parsed metadata** to avoid re-parsing in loops
5. **Use absolute paths** when referencing specs across commands
6. **Log discovery results** for debugging and user feedback

---

**Pattern Version**: 2.1.0
**Last Updated**: 2025-11-01
**Used By**: plan.md, reconcile.md, validate-hierarchy.md, implement.md
