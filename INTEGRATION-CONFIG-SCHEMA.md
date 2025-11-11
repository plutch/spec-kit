# Configuration Schema for Discovery Interview Gates
# File: src/.specify/config.example.yml
# Location: ADD new section after existing config sections

---

## Discovery Gates Configuration (v2.7+)

```yaml
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Discovery Interview Gates Configuration (v2.7.0)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#
# Discovery gates ensure specifications match user intent through
# structured interviews and mandatory approval checkpoints.
#
# Token Cost Impact: +500-900 tokens per feature (~5-10% increase)
# ROI: 400-800% (prevents 10K-30K tokens in rework from misunderstandings)
#
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

discovery_gates:
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # Global Settings
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  # Enable/disable all discovery gates globally
  # Set to false to disable all gates (v2.6 behavior)
  enabled: true

  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # Pre-Specification Discovery (Step 1)
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  pre_spec_discovery:
    # Discovery mode:
    # - "adaptive": Show A/B option based on vagueness detection (RECOMMENDED)
    # - "always": Always run discovery (no skip option, slower but most accurate)
    # - "disabled": Skip discovery entirely (fastest, least accurate, v2.6 behavior)
    mode: "adaptive"

    # Domain-specific templates
    domain_templates:
      # Use domain-specific question templates (auth, data, UI, API, performance)
      # Set false to use generic questions only
      enabled: true

      # Fallback to generic template if domain cannot be detected
      fallback_to_generic: true

      # Custom domain template directory (optional)
      # Place custom templates in this directory to override built-in templates
      # Example: .specify/gates/custom-auth.md
      custom_template_dir: ".specify/gates"

    # Vagueness detection thresholds
    vagueness_detection:
      # Minimum word count to skip vagueness check (high-detail descriptions)
      min_words: 30

      # Keywords that trigger vagueness detection
      uncertainty_keywords:
        - "maybe"
        - "thinking about"
        - "not sure"
        - "possibly"
        - "could"
        - "might"

      # Generic terms that trigger domain-specific discovery
      generic_terms:
        - "dashboard"
        - "authentication"
        - "api"
        - "admin panel"
        - "performance"
        - "caching"
        - "search"

  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # Specification Review Gate (Step 8)
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  specification_review_gate:
    # Enable specification approval gate
    # RECOMMENDED: Keep enabled to prevent misunderstandings
    enabled: true

    # Auto-approve without user interaction (DANGEROUS)
    # Only use for:
    # - CI/CD pipelines (non-interactive environments)
    # - Trusted automation workflows
    # - Testing/development
    # WARNING: Bypasses critical quality check
    auto_approve: false

    # Summary detail level
    # - "full": Show all details (requirements, risk, architecture, quality)
    # - "standard": Show key details only (requirements summary, risk, quality)
    # - "minimal": Show only essential details (requirements count, risk level)
    summary_detail: "standard"

    # Number of top requirements to show in summary (1-10)
    # More = better context, but longer summary
    top_requirements_count: 5

    # Show quality scores in summary
    show_quality_scores: true

    # Show architecture impact analysis
    show_architecture_impact: true

    # Revision tracking
    track_revisions: true

  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # Constitutional Approval Gate (Planning Phase)
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  constitutional_approval_gate:
    # Enable constitutional approval gate
    # RECOMMENDED: Keep enabled to prevent constitutional drift
    enabled: true

    # Auto-approve without user interaction (DANGEROUS)
    # Same warnings as specification_review_gate.auto_approve
    auto_approve: false

    # Block implementation if constitutional validation FAILS
    # Set false to allow proceeding with warnings (NOT RECOMMENDED)
    block_on_fail: true

    # Allow conditional approvals
    # If true, CONDITIONAL status can be approved
    # If false, treat CONDITIONAL as FAIL (stricter)
    allow_conditional: true

    # Show detailed article-by-article validation
    show_article_details: true

    # Graceful degradation when constitution.md missing
    graceful_degradation:
      # Allow approval without constitution.md (PARTIAL mode)
      enabled: true

      # Show recommendation to create constitution
      recommend_creation: true

  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # Approval Timeout (Non-Interactive Environments)
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  # Maximum time to wait for approval (minutes)
  # After timeout, command fails with error
  # Set to 0 for no timeout (wait indefinitely)
  approval_timeout_minutes: 60

  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # Audit & Analytics
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  audit:
    # Track approval decisions in spec-metadata.json
    track_approvals: true

    # Track revision history
    track_revisions: true

    # Track discovery insights
    track_discovery_insights: true

    # Log approval events to file (optional)
    log_to_file:
      enabled: false
      file_path: ".specify/logs/approvals.log"

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Backward Compatibility (Legacy Mode)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#
# Use this to maintain v2.6 behavior exactly
# Sets all discovery gates to v2.6 defaults
#
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

legacy_mode:
  # Enable legacy mode (v2.6 behavior)
  # When true, overrides all discovery_gates settings above
  enabled: false

  # What legacy mode does:
  # - pre_spec_discovery.mode: "disabled" (original optional behavior)
  # - specification_review_gate.enabled: false (no approval gate)
  # - constitutional_approval_gate.enabled: false (no approval gate)
  # - All gates auto-approved

  # Note: You can still enable individual gates by setting legacy_mode.enabled: false
  # and configuring discovery_gates sections above
```

---

## Environment Variable Overrides

Discovery gates can be controlled via environment variables for CI/CD integration:

```bash
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Auto-Approve All Gates
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Auto-approve all gates (specification + constitutional)
export SPECKIT_AUTO_APPROVE=true

# Equivalent to setting:
# - discovery_gates.specification_review_gate.auto_approve: true
# - discovery_gates.constitutional_approval_gate.auto_approve: true

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Skip All Gates
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Skip all gates entirely (legacy mode)
export SPECKIT_SKIP_GATES=true

# Equivalent to setting:
# - legacy_mode.enabled: true

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Discovery Mode Override
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Override discovery mode (adaptive | always | disabled)
export SPECKIT_DISCOVERY_MODE=disabled

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Approval Timeout
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Set approval timeout in minutes (0 = no timeout)
export SPECKIT_APPROVAL_TIMEOUT=30

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# CI/CD Integration Example
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# GitHub Actions workflow example:
# .github/workflows/spec-kit-ci.yml
#
# jobs:
#   generate-spec:
#     runs-on: ubuntu-latest
#     env:
#       SPECKIT_AUTO_APPROVE: true  # Non-interactive mode
#       SPECKIT_APPROVAL_TIMEOUT: 0  # No timeout (not waiting for user)
#     steps:
#       - name: Generate specification
#         run: |
#           /speckit.specify "Add feature from ticket description"
#
#       - name: Generate plan
#         run: |
#           /speckit.plan
```

---

## Command-Line Flag Overrides

Discovery gates can also be controlled via command-line flags:

```bash
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Skip All Gates (single command)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

/speckit.specify --skip-gates "Add OAuth integration"
/speckit.plan --skip-gates

# Skips discovery and approval gates for this invocation only

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Auto-Approve (single command)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

/speckit.specify --auto-approve "Add OAuth integration"
/speckit.plan --auto-approve

# Auto-approves gates but still runs discovery and validation

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Force Discovery Mode
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

/speckit.specify --force-discovery "Add authentication"
# Always runs discovery, no A/B option

/speckit.specify --no-discovery "Add authentication"
# Skips discovery entirely (fast path)

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Generic Discovery Only
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

/speckit.specify --generic-discovery "Add authentication"
# Uses generic questions even if domain detected
```

---

## Configuration Priority

When multiple configuration sources exist, priority is:

1. **Command-line flags** (highest priority)
   - `--skip-gates`, `--auto-approve`, `--force-discovery`, etc.

2. **Environment variables**
   - `SPECKIT_AUTO_APPROVE`, `SPECKIT_SKIP_GATES`, etc.

3. **Config file** (`.specify/config.yml`)
   - `discovery_gates.*` settings

4. **Default behavior** (lowest priority)
   - All gates enabled, adaptive discovery

Example priority resolution:

```bash
# Scenario: Config has auto_approve: false
# But environment variable SPECKIT_AUTO_APPROVE=true is set

/speckit.specify "Add feature"
# Result: Auto-approves (env var overrides config)

# Scenario: Config has enabled: true
# But command has --skip-gates flag

/speckit.specify --skip-gates "Add feature"
# Result: Skips gates (flag overrides config)
```

---

## Configuration Examples

### Example 1: Production (Strict)

```yaml
discovery_gates:
  enabled: true
  pre_spec_discovery:
    mode: "always"  # Force discovery, no skip option
  specification_review_gate:
    enabled: true
    auto_approve: false
  constitutional_approval_gate:
    enabled: true
    auto_approve: false
    block_on_fail: true
    allow_conditional: false  # Stricter: treat conditional as fail
```

Use for: Mission-critical projects, high-quality requirements

### Example 2: Development (Balanced)

```yaml
discovery_gates:
  enabled: true
  pre_spec_discovery:
    mode: "adaptive"  # Default: show A/B option
  specification_review_gate:
    enabled: true
    auto_approve: false
  constitutional_approval_gate:
    enabled: true
    auto_approve: false
    block_on_fail: true
    allow_conditional: true  # More flexible
```

Use for: Most projects, balanced quality and speed

### Example 3: CI/CD (Automated)

```yaml
discovery_gates:
  enabled: true
  pre_spec_discovery:
    mode: "disabled"  # Skip for speed
  specification_review_gate:
    enabled: true
    auto_approve: true  # Non-interactive
  constitutional_approval_gate:
    enabled: true
    auto_approve: true  # Non-interactive
    block_on_fail: false  # Allow proceeding with warnings
```

Use for: Automated pipelines, testing environments

### Example 4: Legacy (v2.6 Behavior)

```yaml
legacy_mode:
  enabled: true

# All discovery gates disabled, exact v2.6 behavior
```

Use for: Users upgrading who want gradual adoption

---

## Validation & Schema

To validate your configuration:

```bash
# Check config syntax
cat .specify/config.yml | yq eval '.'

# Validate discovery_gates section exists
cat .specify/config.yml | yq eval '.discovery_gates' -

# Check specific setting
cat .specify/config.yml | yq eval '.discovery_gates.enabled' -
```

Schema validation (JSON Schema):

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "discovery_gates": {
      "type": "object",
      "properties": {
        "enabled": {"type": "boolean"},
        "pre_spec_discovery": {
          "type": "object",
          "properties": {
            "mode": {"enum": ["adaptive", "always", "disabled"]},
            "domain_templates": {
              "type": "object",
              "properties": {
                "enabled": {"type": "boolean"},
                "fallback_to_generic": {"type": "boolean"},
                "custom_template_dir": {"type": "string"}
              }
            }
          }
        },
        "specification_review_gate": {
          "type": "object",
          "properties": {
            "enabled": {"type": "boolean"},
            "auto_approve": {"type": "boolean"},
            "summary_detail": {"enum": ["full", "standard", "minimal"]},
            "top_requirements_count": {"type": "integer", "minimum": 1, "maximum": 10}
          }
        },
        "constitutional_approval_gate": {
          "type": "object",
          "properties": {
            "enabled": {"type": "boolean"},
            "auto_approve": {"type": "boolean"},
            "block_on_fail": {"type": "boolean"},
            "allow_conditional": {"type": "boolean"}
          }
        },
        "approval_timeout_minutes": {"type": "integer", "minimum": 0}
      }
    },
    "legacy_mode": {
      "type": "object",
      "properties": {
        "enabled": {"type": "boolean"}
      }
    }
  }
}
```

---

## Troubleshooting

### Gates Not Working

**Symptom:** Discovery gates don't appear, commands proceed without approval

**Check:**
```bash
# 1. Check if gates are enabled in config
cat .specify/config.yml | yq eval '.discovery_gates.enabled' -
# Should output: true

# 2. Check if legacy mode is enabled
cat .specify/config.yml | yq eval '.legacy_mode.enabled' -
# Should output: false

# 3. Check environment variables
echo $SPECKIT_SKIP_GATES
# Should output: nothing or false

# 4. Check command flags
# Did you use --skip-gates or --auto-approve?
```

### Auto-Approval Not Working in CI/CD

**Symptom:** CI/CD pipeline hangs waiting for approval

**Solution:**
```bash
# Set environment variable in CI/CD config
export SPECKIT_AUTO_APPROVE=true

# OR use command flag
/speckit.specify --auto-approve "Add feature"

# OR configure in config.yml
discovery_gates:
  specification_review_gate:
    auto_approve: true
  constitutional_approval_gate:
    auto_approve: true
```

### Discovery Questions Not Domain-Specific

**Symptom:** Always getting generic questions instead of domain-specific

**Check:**
```bash
# 1. Check if domain templates enabled
cat .specify/config.yml | yq eval '.discovery_gates.pre_spec_discovery.domain_templates.enabled' -
# Should output: true

# 2. Check feature description has domain keywords
# Example: "Add authentication" should trigger auth domain
# Example: "Optimize database" should trigger data domain

# 3. Try explicit domain keywords
/speckit.specify "Add OAuth authentication with JWT tokens"
# Should trigger authentication domain
```
