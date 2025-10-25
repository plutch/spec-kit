# Deep Review Configuration

## Overview

The Deep Review mode uses AI-powered sequential thinking to perform multi-pass requirements analysis during the clarification phase. This catches contradictions, suggests optimizations, and identifies high-impact ambiguities that traditional scanning might miss.

## Configuration File

Create or update `.specify/config.yml` in your project root:

```yaml
# Deep Review Configuration
clarify:
  deep_review:
    # Enable/disable deep review globally
    enabled: true

    # Default mode: "fast" | "balanced" | "thorough"
    # - fast: 8-12 thoughts, ~5,000 tokens, 30-60 seconds
    # - balanced: 15-20 thoughts, ~10,000 tokens, 60-120 seconds
    # - thorough: 25-30 thoughts, ~15,000 tokens, 120-180 seconds
    default_mode: "balanced"

    # Auto-enable conditions (deep review automatically activates when these are met)
    auto_enable:
      # Enable if spec has "Complex Features" section
      complex_features_section: true

      # Enable if these keywords are found in spec
      keywords:
        - authentication
        - authorization
        - payment
        - billing
        - multi-tenant
        - real-time
        - websocket
        - integration
        - external api
        - third-party

      # Enable if requirement count meets threshold
      min_requirements: 15

      # Enable if spec word count exceeds threshold
      min_word_count: 3000

      # Enable if spec contains multiple [NEEDS CLARIFICATION] markers
      min_clarification_markers: 3

      # Enable based on estimated feature complexity score
      # Calculated from: requirement count, section count, keyword matches, dependencies
      min_complexity_score: 50  # 0-100 scale

    # Custom domain detection
    # If spec mentions these terms, suggest domain-specific questions
    domain_patterns:
      authentication:
        keywords: ["login", "signup", "password", "oauth", "jwt", "session", "mfa"]
        questions:
          - "What password policy requirements apply? (complexity, length, expiration)"
          - "Is multi-factor authentication required or optional?"
          - "How long should user sessions remain active?"
          - "Are there rate limiting requirements for login attempts?"

      payment:
        keywords: ["payment", "checkout", "billing", "subscription", "invoice", "refund"]
        questions:
          - "Are payment operations idempotent (safe to retry)?"
          - "How should failed payment retries be handled?"
          - "What refund workflows are required?"
          - "Are webhooks needed for payment status updates?"

      multi_tenant:
        keywords: ["tenant", "organization", "workspace", "account", "isolation"]
        questions:
          - "How is data isolated between tenants?"
          - "Are there cross-tenant access scenarios (e.g., shared resources)?"
          - "How are tenant limits/quotas enforced?"
          - "Is tenant migration (import/export) required?"

    # Analysis pass configuration
    passes:
      logic_validation:
        enabled: true
        max_thoughts: 5

      completeness:
        enabled: true
        max_thoughts: 6

      optimization:
        enabled: true
        max_thoughts: 7

      synthesis:
        enabled: true
        max_thoughts: 2

    # Question generation
    questions:
      # Maximum questions to ask (spec-kit standard is 5)
      max_count: 5

      # Minimum impact score for question to be included (0-100)
      min_impact_score: 60

      # Balance category coverage (avoid asking multiple questions from same category)
      balance_categories: true

      # Prefer multiple-choice questions over short-answer
      prefer_multiple_choice: true

    # Optimization suggestions
    optimization:
      # Include alternative implementation suggestions
      suggest_alternatives: true

      # Identify over-engineering opportunities
      flag_over_engineering: true

      # Suggest scope reduction options
      suggest_scope_reduction: true

      # Flag premature optimization patterns
      detect_premature_optimization: true

    # Token management
    token_limits:
      # Hard limit for sequential thinking (stops even if needsMoreThoughts=true)
      max_tokens_per_review: 20000

      # Soft limit (triggers warning but allows continuation)
      warn_at_tokens: 15000

    # Performance tuning
    performance:
      # Cache taxonomy scan results to avoid re-analysis
      cache_taxonomy_scan: true

      # Skip deep review if recent analysis exists (within N hours)
      use_cached_analysis_within_hours: 24

      # Parallel processing (experimental - not yet supported)
      parallel_thinking: false
```

## Command-Line Overrides

Users can override configuration via command arguments:

### Explicit Mode Selection

```bash
# Force deep review with thorough mode
/speckit.clarify --deep

# Force deep review with balanced mode
/speckit.clarify --balanced

# Skip deep review (fast taxonomy-based questions only)
/speckit.clarify --fast

# Use configuration defaults (smart detection)
/speckit.clarify
```

### Additional Options

```bash
# Skip deep review entirely (legacy behavior)
/speckit.clarify --skip-deep

# Use thorough mode with extended thinking
/speckit.clarify --thorough

# Enable deep review with custom thought limit
/speckit.clarify --deep --max-thoughts 25

# Dry run (show what would be analyzed without executing)
/speckit.clarify --dry-run
```

## Auto-Detection Logic

When no explicit flag is provided, the system uses this decision tree:

```
1. Check user arguments
   ├─ Contains "--deep", "--thorough", "--balanced" → Use specified mode
   ├─ Contains "--fast", "--skip-deep" → Skip deep review
   └─ No flags → Continue to auto-detection

2. Check spec complexity indicators
   ├─ Has "Complex Features" section → Enable (balanced)
   ├─ ≥15 functional requirements → Enable (balanced)
   ├─ ≥3 [NEEDS CLARIFICATION] markers → Enable (thorough)
   ├─ Matches domain keywords (auth, payment, etc.) → Enable (balanced)
   └─ Word count >3000 → Enable (balanced)

3. Calculate complexity score
   ├─ Score = (requirements × 2) + (sections × 5) + (keywords × 10) + (dependencies × 15)
   ├─ Score ≥50 → Enable (balanced)
   └─ Score <50 → Enable (fast) or skip based on config

4. Default fallback
   └─ Use config.clarify.deep_review.enabled and default_mode
```

## Example Scenarios

### Scenario 1: Simple CRUD Feature

**Spec characteristics:**
- 8 requirements
- No complex features section
- 1,500 word count
- No domain keywords

**Auto-detection result:** Fast mode or skip (based on config)

**Estimated cost:** ~3,000 tokens (or 0 if skipped)

---

### Scenario 2: Authentication Feature

**Spec characteristics:**
- 18 requirements
- Contains "authentication", "password", "session" keywords
- 2,800 word count
- 2 [NEEDS CLARIFICATION] markers

**Auto-detection result:** Balanced mode

**Estimated cost:** ~10,000 tokens

**Expected questions:**
- Password policy requirements
- Session duration and expiration
- Rate limiting for login attempts
- MFA implementation approach

---

### Scenario 3: Multi-Tenant Payment System

**Spec characteristics:**
- 32 requirements
- Has "Complex Features" section
- Contains "multi-tenant", "payment", "webhook" keywords
- 5,200 word count
- 4 [NEEDS CLARIFICATION] markers

**Auto-detection result:** Thorough mode

**Estimated cost:** ~15,000 tokens

**Expected questions:**
- Tenant data isolation strategy
- Payment idempotency handling
- Webhook security and retry logic
- Cross-tenant resource sharing
- Refund workflow requirements

---

## Cost-Benefit Analysis

### Token Costs

| Mode | Thoughts | Est. Tokens | Time | Use Case |
|------|----------|-------------|------|----------|
| **Skip** | 0 | 0 | 0s | Simple features, tight budget |
| **Fast** | 8-12 | ~5,000 | 30-60s | Moderate complexity, quick review |
| **Balanced** | 15-20 | ~10,000 | 60-120s | Standard features, best ROI |
| **Thorough** | 25-30 | ~15,000 | 120-180s | Complex/critical features |

### Expected Savings

Based on catching issues early:

| Issue Type | Avg. Rework Cost | Deep Review Detection Rate | Expected Savings |
|------------|------------------|----------------------------|------------------|
| **Contradictory Requirements** | 5,000-10,000 tokens | 90% | 4,500-9,000 tokens |
| **Missing Critical Requirements** | 10,000-20,000 tokens | 75% | 7,500-15,000 tokens |
| **Over-Engineering** | 3,000-8,000 tokens | 60% | 1,800-4,800 tokens |
| **Ambiguous Success Criteria** | 2,000-5,000 tokens | 85% | 1,700-4,250 tokens |

**Total Expected Savings:** 15,500-33,050 tokens per feature

**Net ROI:**
- Fast mode: 2-6x return (~10,000-28,000 net savings)
- Balanced mode: 1.5-3x return (~5,500-23,000 net savings)
- Thorough mode: 1-2x return (~500-18,000 net savings)

### When to Use Each Mode

**Skip Deep Review:**
- Simple CRUD operations
- Well-defined specs with <10 requirements
- Prototypes or spikes
- Features similar to recently completed work
- Tight token budget

**Fast Mode:**
- Moderate complexity (10-15 requirements)
- Standard patterns (no unusual constraints)
- Experienced team with clear domain knowledge
- Quick validation needed

**Balanced Mode (Recommended Default):**
- Most features (15-25 requirements)
- First-time domain implementations
- Specs with some ambiguity
- Best cost-benefit ratio

**Thorough Mode:**
- Complex features (25+ requirements)
- Critical systems (auth, payment, security)
- High uncertainty or novel approaches
- Multi-system integrations
- Compliance-heavy domains

---

## Integration with Existing Workflow

### Before Deep Review

```
/speckit.specify → generates spec.md
  ↓
Taxonomy scan identifies ambiguities
  ↓
5 questions generated from taxonomy
  ↓
User answers questions
  ↓
Spec updated with clarifications
```

### With Deep Review

```
/speckit.specify → generates spec.md
  ↓
Taxonomy scan identifies ambiguities
  ↓
[NEW] Deep Review Analysis (sequential thinking)
  ├─ Pass 1: Logic validation
  ├─ Pass 2: Completeness analysis
  ├─ Pass 3: Optimization suggestions
  └─ Pass 4: Synthesis → Top 5 questions
  ↓
User answers questions (with better context/alternatives)
  ↓
Spec updated with clarifications + optimization notes
```

### Output Enhancements

The deep review adds:

1. **Pre-Question Report**:
   - Logic issues found (contradictions, impossible constraints)
   - Completeness gaps (missing requirements by domain)
   - Optimization opportunities (alternative approaches)

2. **Enhanced Questions**:
   - Questions have clearer rationale (why this matters)
   - Multiple-choice options include trade-off analysis
   - Recommendations backed by deep analysis findings

3. **Post-Question Summary**:
   - Token usage report
   - ROI assessment (issues caught vs. cost)
   - Remaining optimization opportunities
   - Suggested follow-up actions

---

## Migration Guide

### For Existing Projects

1. **Add configuration file** (optional):
   ```bash
   mkdir -p .specify
   cp docs/deep-review-configuration.md .specify/config.yml
   # Edit config.yml to set preferences
   ```

2. **Update existing specs**:
   - Deep review works with any existing spec
   - No changes required to spec format
   - Can run on specs that already have clarifications

3. **Try it out**:
   ```bash
   # On your next feature
   /speckit.specify "Add user profile page"

   # Then run clarify with deep review
   /speckit.clarify --balanced
   ```

4. **Evaluate results**:
   - Compare questions to what you would have asked manually
   - Assess if optimization suggestions are valuable
   - Adjust config based on your team's needs

### Backward Compatibility

- **Original clarify command remains unchanged** ([clarify.md](../templates/commands/clarify.md))
- **New command is opt-in** ([clarify-deep.md](../templates/commands/clarify-deep.md))
- **No breaking changes** to existing workflows
- **Can mix modes** (some features with deep review, others without)

---

## Troubleshooting

### "Sequential thinking MCP server not available"

**Solution:** Deep review requires the `mcp__sequential-thinking__sequentialthinking` MCP server.

Check if installed:
```bash
# List available MCP servers
claude-code --list-mcp-servers
```

If missing, the system will automatically fall back to fast mode (taxonomy-based questions).

### "Token limit exceeded"

**Solution:** Reduce mode or adjust token limits in config:
```yaml
clarify:
  deep_review:
    default_mode: "fast"  # Down from "balanced"
    token_limits:
      max_tokens_per_review: 10000  # Down from 20000
```

### "Deep review taking too long"

**Solution:**
- Use fast mode for time-sensitive features
- Enable caching to reuse recent analyses:
  ```yaml
  performance:
    use_cached_analysis_within_hours: 24
  ```

### "Questions not relevant to my domain"

**Solution:** Customize domain patterns in config:
```yaml
clarify:
  deep_review:
    domain_patterns:
      my_custom_domain:
        keywords: ["custom", "specific", "terms"]
        questions:
          - "Domain-specific question 1?"
          - "Domain-specific question 2?"
```

---

## Best Practices

1. **Start with balanced mode** for most features
2. **Use thorough mode** for critical/complex systems
3. **Skip or use fast mode** for simple CRUD operations
4. **Review optimization suggestions** even if not implementing immediately
5. **Track ROI** over time to adjust config for your team
6. **Customize domain patterns** for your specific technology stack
7. **Cache results** when iterating on the same spec
8. **Educate team** on when to use each mode

---

## Future Enhancements

Planned improvements:

- [ ] Parallel thinking (multiple passes simultaneously)
- [ ] Learning from past clarifications (pattern detection)
- [ ] Integration with `/speckit.analyze` for continuous validation
- [ ] Custom analysis pass plugins
- [ ] Team-level optimization libraries
- [ ] Visual analysis reports (diagrams/charts)
- [ ] Real-time streaming of thinking process
- [ ] A/B testing of question effectiveness

---

## Support

For issues or questions:
- GitHub Issues: https://github.com/your-org/spec-kit/issues
- Documentation: https://docs.spec-kit.dev/deep-review
- Community: https://discord.gg/spec-kit
