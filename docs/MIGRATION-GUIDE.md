# Migration Guide: Legacy Clarify → Deep Review

## Overview

This guide helps you migrate from the traditional taxonomy-based clarification workflow to the enhanced deep review mode with sequential thinking analysis.

**Migration Complexity**: Low
**Estimated Time**: 1-2 hours (includes testing)
**Breaking Changes**: None (backward compatible)
**Recommended Approach**: Gradual rollout with pilot team

---

## Table of Contents

- [What's Changing](#whats-changing)
- [Migration Strategies](#migration-strategies)
- [Step-by-Step Migration](#step-by-step-migration)
- [Testing & Validation](#testing--validation)
- [Team Training](#team-training)
- [Troubleshooting](#troubleshooting)
- [Rollback Procedures](#rollback-procedures)

---

## What's Changing

### Legacy Clarify Workflow

```
1. Load spec
2. Run taxonomy-based ambiguity scan (9 categories)
3. Generate 5 questions from Partial/Missing categories
4. Ask questions sequentially
5. Update spec with answers
```

**Characteristics:**
- Fast execution (~30-60 seconds)
- Low token cost (~2,000 tokens)
- Pattern-based question generation
- No optimization suggestions
- No contradiction detection

### Deep Review Workflow

```
1. Load spec
2. Run taxonomy-based ambiguity scan (same 9 categories)
3. [NEW] Deep Review Analysis (sequential thinking)
   ├─ Pass 1: Logic Validation
   ├─ Pass 2: Completeness Analysis
   ├─ Pass 3: Optimization Analysis
   └─ Pass 4: Synthesis → Top 5 questions
4. Ask questions sequentially (with trade-off analysis)
5. Update spec with answers
```

**Characteristics:**
- Moderate execution (~60-180 seconds, mode-dependent)
- Higher token cost (~5,000-15,000 tokens, mode-dependent)
- AI-reasoning-based question generation
- Optimization suggestions included
- Contradiction detection built-in

### Key Differences

| Aspect | Legacy | Deep Review |
|--------|--------|-------------|
| **Analysis Depth** | Pattern matching | Multi-pass AI reasoning |
| **Token Cost** | ~2,000 | ~5,000-15,000 (configurable) |
| **Execution Time** | 30-60s | 60-180s (mode-dependent) |
| **Contradictions** | Not detected | Explicitly identified |
| **Optimizations** | Not suggested | Alternatives + trade-offs |
| **Question Quality** | Coverage-based | Impact-scored |
| **Scope Reduction** | Not offered | Suggested when appropriate |
| **ROI** | Baseline | 2-3x via prevented rework |
| **Configuration** | None | Extensive (modes, domains) |
| **Fallback** | N/A | Degrades to legacy if MCP unavailable |

---

## Migration Strategies

### Strategy 1: Gradual Rollout (Recommended)

**Best for**: Production teams, risk-averse organizations

**Approach:**
1. Keep legacy clarify as default
2. Add deep review as opt-in (`/speckit.clarify --deep`)
3. Pilot with 3-5 team members on new features
4. Collect feedback for 2-4 weeks
5. Make deep review default (with opt-out)
6. Eventually deprecate legacy (6+ months)

**Pros:**
- ✅ Zero disruption to existing workflows
- ✅ Team can learn gradually
- ✅ Easy rollback if issues arise
- ✅ Gather real-world performance data

**Cons:**
- ❌ Slower adoption
- ❌ Requires maintaining two commands
- ❌ Team may resist switching

**Timeline:** 2-6 months

---

### Strategy 2: Feature Flag Rollout

**Best for**: Teams with existing feature flag infrastructure

**Approach:**
1. Add config flag: `clarify.use_deep_review: false` (default off)
2. Enable for pilot users/projects
3. Monitor performance and feedback
4. Gradually increase rollout percentage (25% → 50% → 100%)
5. Remove flag once stable

**Pros:**
- ✅ Gradual rollout with easy control
- ✅ Can roll back instantly
- ✅ Measure performance differences
- ✅ Users don't need to remember flags

**Cons:**
- ❌ Requires config infrastructure
- ❌ Inconsistent experience during rollout
- ❌ May confuse users (why different results?)

**Timeline:** 1-3 months

---

### Strategy 3: Immediate Replacement

**Best for**: Small teams, greenfield projects, token budget not constrained

**Approach:**
1. Replace `clarify.md` with `clarify-deep.md`
2. Keep legacy as `clarify-fast.md` (opt-out)
3. Announce change to team
4. Monitor first 5-10 feature specs closely

**Pros:**
- ✅ Fastest adoption
- ✅ Immediate benefits for all features
- ✅ Simple (one command to rule them all)
- ✅ Clean implementation

**Cons:**
- ❌ Potential shock to team workflow
- ❌ Higher token costs surprise users
- ❌ Harder rollback (affects all in-flight features)

**Timeline:** 1-2 weeks

---

### Strategy 4: Hybrid Approach (Auto-Detection)

**Best for**: Teams wanting smart defaults

**Approach:**
1. Replace `clarify.md` with auto-detecting version
2. Simple features use legacy mode automatically
3. Complex features use deep review automatically
4. Users can override with `--deep` or `--fast` flags

**Pros:**
- ✅ Best of both worlds (cost-effective for simple, deep for complex)
- ✅ No user decision fatigue
- ✅ Optimizes token spend automatically
- ✅ Smooth transition

**Cons:**
- ❌ Auto-detection may be wrong sometimes
- ❌ Inconsistent experience (by design)
- ❌ Requires tuning detection thresholds

**Timeline:** 1-2 weeks

---

## Step-by-Step Migration

### Option A: Gradual Rollout (Recommended)

#### Phase 1: Installation (Week 1)

**Step 1: Backup existing clarify command**

```bash
# From spec-kit repo root
cd templates/commands
cp clarify.md clarify-legacy.md
```

**Step 2: Add deep review as separate command**

```bash
# Keep both commands available
# clarify.md = legacy (default)
# clarify-deep.md = new (opt-in)
```

**Step 3: Create configuration file**

```bash
mkdir -p .specify
cp .specify/config.example.yml .specify/config.yml
```

Edit `.specify/config.yml`:
```yaml
clarify:
  deep_review:
    enabled: true  # Allow opt-in via --deep flag
    default_mode: "balanced"  # When explicitly enabled
```

**Step 4: Update CLI to support new command** (if separate command approach)

Edit `src/specify_cli/__init__.py`:
```python
# Add new command registration
@app.command(name="clarify-deep", help="Enhanced clarification with deep review")
def clarify_deep_command(args: str = ""):
    # Load clarify-deep.md template
    pass
```

Or keep single command with flag parsing:
```python
@app.command(name="clarify")
def clarify_command(
    args: str = "",
    deep: bool = typer.Option(False, "--deep", help="Enable deep review analysis"),
    fast: bool = typer.Option(False, "--fast", help="Skip deep review (legacy mode)")
):
    if deep:
        # Load clarify-deep.md
        pass
    else:
        # Load clarify.md (legacy)
        pass
```

**Step 5: Test installation**

```bash
# Test legacy still works
/speckit.specify "test feature"
/speckit.clarify

# Test deep review (if using flag approach)
/speckit.clarify --deep

# Or test separate command
/speckit.clarify-deep
```

#### Phase 2: Pilot Program (Weeks 2-3)

**Step 6: Select pilot features**

Criteria for pilot features:
- ✅ Medium-to-high complexity (15+ requirements)
- ✅ Not time-critical (can afford extra analysis time)
- ✅ Domain with existing patterns (auth, payment, multi-tenant)
- ✅ Team members open to feedback

**Step 7: Run pilot with 3-5 features**

For each pilot feature:
```bash
# Create spec as normal
/speckit.specify "feature description"

# Use deep review for clarification
/speckit.clarify --deep  # or /speckit.clarify-deep

# Document experience:
# - Token cost actual vs estimated
# - Time taken
# - Question quality (1-5 rating)
# - Optimization suggestions usefulness
# - Issues encountered
```

**Step 8: Collect structured feedback**

Use this template:

```markdown
## Deep Review Pilot Feedback

**Feature**: [Name]
**Date**: [YYYY-MM-DD]
**Reviewer**: [Name]

### Metrics
- Token cost: [actual] vs [estimated]
- Time taken: [actual] vs [estimated]
- Spec complexity: [Low/Medium/High]
- Mode used: [Fast/Balanced/Thorough]

### Quality Assessment (1-5 scale)
- Question relevance: [1-5]
- Optimization suggestions: [1-5]
- Trade-off analysis clarity: [1-5]
- Overall experience: [1-5]

### Specific Feedback
**What worked well:**
- [feedback]

**What didn't work:**
- [feedback]

**Suggestions for improvement:**
- [feedback]

**Would you use this for future features?** [Yes/No/Maybe]
```

**Step 9: Analyze pilot results**

Aggregate metrics:
- Average token cost vs legacy
- Average time increase
- Question quality improvement (if measurable)
- Adoption recommendation rate

Decision criteria:
- ✅ **Proceed to rollout** if:
  - >70% would use for future features
  - Token cost within acceptable range
  - No critical bugs/issues
  - Question quality noticeably better

- ⚠️ **Iterate** if:
  - 40-70% adoption recommendation
  - Token costs higher than expected
  - Minor issues identified
  - Mixed feedback on quality

- ❌ **Reconsider** if:
  - <40% adoption recommendation
  - Critical bugs or workflow blockers
  - Token costs prohibitive
  - No clear quality improvement

#### Phase 3: Full Rollout (Week 4-6)

**Step 10: Make deep review the default**

```bash
# Replace default clarify command
cd templates/commands
mv clarify.md clarify-legacy.md
mv clarify-deep.md clarify.md
```

Update documentation:
```bash
# Update README
# - Clarify command now uses deep review by default
# - Add --fast flag for legacy behavior
# - Document token cost expectations
```

**Step 11: Announce to team**

Email/Slack template:
```
📣 Spec-Kit Enhancement: Deep Review Clarification

Starting [date], the /speckit.clarify command now includes AI-powered deep review
analysis. This catches contradictions, suggests optimizations, and improves
specification quality.

What's changing:
- Analysis is more thorough (contradictions, completeness, optimizations)
- Takes 60-180 seconds instead of 30-60 seconds (mode-dependent)
- Uses 5,000-15,000 tokens instead of ~2,000 tokens
- Questions include trade-off analysis and recommendations

Benefits:
- 2-3x fewer issues discovered in planning/implementation
- Optimization suggestions can reduce scope 20-40%
- Better architecture decisions upfront

Pilot results:
- [X] features tested
- [Y]% of users recommend adoption
- Average [Z] tokens saved in downstream rework

If you need faster clarification (simple features):
  /speckit.clarify --fast

For complex/critical features (best analysis):
  /speckit.clarify --thorough

Questions? Contact [team lead] or see: docs/deep-review-configuration.md
```

**Step 12: Monitor adoption**

Track for first month:
- % of clarifications using deep review vs legacy
- Token costs (actual vs budgeted)
- User-reported issues
- Downstream rework reduction (qualitative)

**Step 13: Optimize configuration**

Based on actual usage, tune `.specify/config.yml`:
```yaml
clarify:
  deep_review:
    # Adjust default mode based on average feature complexity
    default_mode: "balanced"  # or "fast" if mostly simple features

    # Tune auto-enable thresholds based on sweet spot
    auto_enable:
      min_requirements: 15  # Increase if too many simple features trigger deep review
      min_complexity_score: 50  # Tune based on false positive rate

    # Adjust token limits based on budget
    token_limits:
      max_tokens_per_review: 20000
      warn_at_tokens: 15000
```

#### Phase 4: Refinement (Month 2+)

**Step 14: Add custom domain patterns**

Based on your tech stack:
```yaml
clarify:
  deep_review:
    domain_patterns:
      # Add custom domains for your projects
      ecommerce:
        keywords: ["cart", "checkout", "inventory", "shipping"]
        questions:
          - "How should cart abandonment be handled?"
          - "What inventory synchronization strategy is required?"
          - "Are there shipping calculation requirements?"

      analytics:
        keywords: ["dashboard", "metrics", "reporting", "analytics"]
        questions:
          - "What is the maximum acceptable data latency?"
          - "Are there data retention policies?"
          - "What export formats are required?"
```

**Step 15: Build team expertise**

- Document common patterns from clarification sessions
- Share examples of good optimization decisions
- Create internal best practices guide

**Step 16: Deprecate legacy (optional, 6+ months)**

If adoption is universal and no issues:
```bash
# Remove legacy command entirely
rm templates/commands/clarify-legacy.md

# Remove --fast flag from CLI
# Update documentation
```

---

### Option B: Immediate Replacement (Fast Track)

**For small teams or greenfield projects:**

```bash
# 1. Backup legacy
cd templates/commands
cp clarify.md clarify-legacy.md

# 2. Replace with deep review
mv clarify-deep.md clarify.md

# 3. Create config
mkdir -p .specify
cp .specify/config.example.yml .specify/config.yml

# 4. Announce to team
# 5. Monitor first 5-10 features closely

# Keep legacy available as opt-out:
# /speckit.clarify --legacy  (or --fast)
```

**Rollback** (if issues):
```bash
# Quick rollback
cd templates/commands
mv clarify.md clarify-deep-temp.md
mv clarify-legacy.md clarify.md
```

---

## Testing & Validation

### Pre-Migration Testing

**Test 1: Legacy Baseline**

```bash
# Create test spec with known issues
/speckit.specify "User authentication with SSO and password reset"

# Run legacy clarify, document:
/speckit.clarify
# - Questions asked
# - Time taken
# - Token cost
# - Issues NOT caught (contradictions, missing requirements)
```

**Test 2: Deep Review Comparison**

```bash
# Same spec, deep review
/speckit.clarify --deep
# - Questions asked (compare to legacy)
# - Time taken (compare)
# - Token cost (compare)
# - Issues caught (contradictions, optimizations)
```

**Test 3: Edge Cases**

Test with:
- Very simple spec (5 requirements) → should use fast mode
- Very complex spec (30+ requirements) → should use thorough mode
- Spec with obvious contradictions → should detect in Pass 1
- Spec with domain keywords (auth, payment) → should apply domain patterns

### Post-Migration Validation

**Validation Checklist:**

- [ ] Legacy clarify still accessible (if keeping as fallback)
- [ ] Deep review works for simple features (fast mode)
- [ ] Deep review works for complex features (thorough mode)
- [ ] Configuration file loads correctly
- [ ] Custom domain patterns apply when keywords detected
- [ ] Questions include trade-off analysis
- [ ] Spec updates correctly after each answer
- [ ] Token usage within expected range
- [ ] MCP sequential thinking server accessible (or fallback works)
- [ ] Documentation updated (README, help text)

### Regression Testing

Test that existing functionality still works:
- [ ] `/speckit.specify` creates specs normally
- [ ] `/speckit.clarify` (default) works end-to-end
- [ ] `/speckit.plan` works after clarification
- [ ] All scripts (bash/powershell) still execute
- [ ] Git integration unaffected
- [ ] Agent context files update correctly

---

## Team Training

### Training Session Outline (1 hour)

**Part 1: Overview (10 minutes)**
- What is deep review?
- Why are we adopting it?
- What changes in daily workflow?

**Part 2: Demo (20 minutes)**
- Live demo with real feature spec
- Show legacy clarify questions
- Show deep review questions (comparison)
- Highlight optimizations caught
- Show token cost vs savings

**Part 3: Hands-On (20 minutes)**
- Each person runs deep review on sample spec
- Discuss questions generated
- Evaluate optimization suggestions
- Compare results

**Part 4: Q&A (10 minutes)**
- When to use fast vs thorough mode?
- How to interpret trade-off tables?
- What if token costs are too high?
- How to customize for our projects?

### Training Materials

**Quick Reference Card:**

```
┌─────────────────────────────────────────────────────────────┐
│ Deep Review Quick Reference                                 │
├─────────────────────────────────────────────────────────────┤
│ DEFAULT:  /speckit.clarify                                  │
│           Auto-detects complexity, uses appropriate mode    │
│                                                             │
│ FAST:     /speckit.clarify --fast                          │
│           Legacy mode, ~2k tokens, 30-60s                   │
│           Use for: Simple CRUD, <10 requirements            │
│                                                             │
│ BALANCED: /speckit.clarify --balanced                      │
│           Recommended, ~10k tokens, 60-120s                 │
│           Use for: Standard features, 15-25 requirements    │
│                                                             │
│ THOROUGH: /speckit.clarify --deep                          │
│           Comprehensive, ~15k tokens, 120-180s              │
│           Use for: Auth, payment, multi-tenant, 25+ reqs    │
│                                                             │
│ TIPS:                                                       │
│ - Accept recommendations (type "yes" or "recommended")      │
│ - Read trade-offs table before answering                    │
│ - Optimization suggestions can reduce scope 20-40%          │
│ - Token cost pays for itself in prevented rework           │
└─────────────────────────────────────────────────────────────┘
```

**Video Tutorial** (optional):
- Record 10-minute walkthrough
- Show real example start-to-finish
- Upload to team wiki/knowledge base

---

## Troubleshooting

### Common Issues

#### Issue 1: "Sequential thinking MCP server not available"

**Symptoms:**
```
Error: mcp__sequential-thinking__sequentialthinking tool not found
Falling back to legacy clarify mode...
```

**Cause:** MCP server not installed or not running

**Solution:**
```bash
# Check MCP servers
claude-code --list-mcp-servers

# If missing, install sequential thinking server
# (installation method depends on your setup)

# Temporary workaround: use legacy mode
/speckit.clarify --fast
```

---

#### Issue 2: Token costs higher than expected

**Symptoms:**
- Balanced mode using 15,000+ tokens instead of ~10,000
- Warnings about token limit exceeded

**Cause:** Spec complexity higher than typical, or analysis finding many issues

**Solution:**
```yaml
# Reduce token limits in config
clarify:
  deep_review:
    default_mode: "fast"  # Down from "balanced"
    token_limits:
      max_tokens_per_review: 10000  # Down from 20000
```

Or use fast mode for that feature:
```bash
/speckit.clarify --fast
```

---

#### Issue 3: Deep review taking too long

**Symptoms:**
- Analysis taking 3-5 minutes instead of 1-2 minutes
- Blocking team workflow

**Cause:** Thorough mode activated unnecessarily, or complex spec

**Solution:**
```yaml
# Tune auto-detection to be less aggressive
clarify:
  deep_review:
    auto_enable:
      min_requirements: 20  # Up from 15
      min_complexity_score: 60  # Up from 50
```

Or explicitly use faster mode:
```bash
/speckit.clarify --balanced  # Instead of thorough
```

---

#### Issue 4: Questions seem generic or low-quality

**Symptoms:**
- Questions don't seem better than legacy
- No optimization suggestions
- Missing domain-specific questions

**Cause:** Domain patterns not configured, or spec lacks keywords

**Solution:**
```yaml
# Add custom domain patterns
clarify:
  deep_review:
    domain_patterns:
      your_domain:
        keywords: ["specific", "terms", "from", "your", "stack"]
        questions:
          - "Domain-specific question 1?"
          - "Domain-specific question 2?"
```

---

#### Issue 5: Spec updates not saving correctly

**Symptoms:**
- Answers recorded in Clarifications section
- But other sections not updated with details

**Cause:** Spec structure doesn't match expected template

**Solution:**
- Ensure spec has standard sections (Functional Requirements, Data Model, etc.)
- Check markdown structure is valid (no unclosed code blocks)
- Manually update sections if auto-update fails

---

#### Issue 6: Configuration not loading

**Symptoms:**
- Config changes have no effect
- Always uses default mode

**Cause:** Config file not in correct location or invalid YAML

**Solution:**
```bash
# Verify config file location
ls -la .specify/config.yml

# Validate YAML syntax
python -c "import yaml; yaml.safe_load(open('.specify/config.yml'))"

# Check for typos in keys
# - clarify (not clarrify)
# - deep_review (not deepreview)
```

---

### Debug Mode

Enable verbose logging:
```bash
# If CLI supports debug flag
/speckit.clarify --deep --debug

# Or set environment variable
export SPECIFY_DEBUG=1
/speckit.clarify --deep
```

---

## Rollback Procedures

### Emergency Rollback (Immediate)

**If critical issue blocking team:**

```bash
# 1. Restore legacy command
cd templates/commands
mv clarify.md clarify-deep-broken.md
mv clarify-legacy.md clarify.md

# 2. Notify team
echo "Deep review temporarily disabled due to [issue]. Using legacy clarify until resolved."

# 3. Document issue for investigation
# 4. Resume normal workflow
```

**Time to rollback:** <5 minutes

---

### Gradual Rollback

**If adoption isn't working but no critical issue:**

```bash
# 1. Change config to opt-in only
# Edit .specify/config.yml
clarify:
  deep_review:
    enabled: false  # Disable by default

# 2. Keep deep review available for those who want it
# /speckit.clarify --deep

# 3. Reassess in 4-8 weeks
```

---

### Partial Rollback

**If only certain modes have issues:**

```yaml
# Disable thorough mode, keep balanced/fast
clarify:
  deep_review:
    default_mode: "balanced"  # Down from "thorough"
    auto_enable:
      complex_features_section: false  # Disable thorough trigger
      min_clarification_markers: 5  # Increase threshold
```

---

## Success Metrics

Track these to measure migration success:

### Adoption Metrics
- [ ] % of features using deep review vs legacy (target: >80% by month 2)
- [ ] % of users who tried deep review that continue using it (target: >70%)
- [ ] Average time from migration announcement to full adoption (target: <2 months)

### Quality Metrics
- [ ] % reduction in contradictions found in planning (target: >60%)
- [ ] % reduction in missing requirements found in implementation (target: >40%)
- [ ] Average spec clarity rating (survey, 1-5 scale) (target: >4.0)

### Efficiency Metrics
- [ ] Average tokens invested in deep review (baseline for budgeting)
- [ ] Average tokens saved from prevented rework (target: >2x investment)
- [ ] Days from spec completion to implementation start (target: reduce 15%)
- [ ] Planning phase rework cycles (target: <1.5 per feature)

### Cost Metrics
- [ ] Total token spend on clarification (compare month-over-month)
- [ ] Token spend on rework (compare before/after)
- [ ] Net token savings (target: >10,000 per feature)
- [ ] Development time saved (hours per feature)

---

## Timeline Summary

### Gradual Rollout (Recommended)

| Week | Phase | Activities | Milestone |
|------|-------|------------|-----------|
| 1 | Installation | Setup, config, CLI changes | Deep review available opt-in |
| 2-3 | Pilot | 3-5 features tested, feedback collected | Decision to proceed |
| 4 | Rollout | Make default, team announcement | Deep review default |
| 5-8 | Refinement | Monitor, tune config, custom domains | Stable adoption |
| 12+ | Optimization | Deprecate legacy (optional) | Full migration |

**Total:** 2-6 months to full adoption

### Fast Track

| Week | Phase | Activities | Milestone |
|------|-------|------------|-----------|
| 1 | Replace | Swap commands, announce to team | Deep review default |
| 2-3 | Monitor | Track first 10 features closely | Stable operation |

**Total:** 2-3 weeks to full adoption

---

## Checklist

Use this to track migration progress:

### Pre-Migration
- [ ] Review deep review documentation
- [ ] Decide on migration strategy (gradual/immediate/hybrid)
- [ ] Identify pilot team members (if gradual)
- [ ] Create rollback plan
- [ ] Prepare training materials

### Installation
- [ ] Backup legacy clarify command
- [ ] Add deep review command/template
- [ ] Create configuration file
- [ ] Update CLI (if needed for flags)
- [ ] Test basic functionality

### Pilot (if applicable)
- [ ] Select 3-5 pilot features
- [ ] Run deep review on pilot features
- [ ] Collect structured feedback
- [ ] Analyze results
- [ ] Decide go/no-go for rollout

### Rollout
- [ ] Make deep review default (or enable feature flag)
- [ ] Announce to team
- [ ] Provide training session
- [ ] Share quick reference materials
- [ ] Monitor adoption metrics

### Post-Rollout
- [ ] Track token costs (actual vs budget)
- [ ] Monitor user-reported issues
- [ ] Tune configuration based on usage
- [ ] Add custom domain patterns
- [ ] Measure success metrics
- [ ] Document lessons learned

### Optimization (ongoing)
- [ ] Review token economics monthly
- [ ] Tune auto-detection thresholds
- [ ] Expand domain pattern library
- [ ] Build internal best practices guide
- [ ] Consider deprecating legacy (6+ months)

---

## Support & Resources

### Documentation
- [Deep Review Configuration](deep-review-configuration.md) - Full configuration reference
- [Deep Review Example](deep-review-example.md) - Realistic walkthrough
- [clarify-deep.md](../templates/commands/clarify-deep.md) - Command template

### Getting Help
- **Issues**: https://github.com/your-org/spec-kit/issues
- **Discussions**: https://github.com/your-org/spec-kit/discussions
- **Slack**: #spec-kit channel
- **Email**: spec-kit-support@yourorg.com

### Common Questions

**Q: Can I mix legacy and deep review for different features?**
A: Yes! Use `--deep` or `--fast` flags to override defaults per-feature.

**Q: What if my token budget is limited?**
A: Use fast mode by default, reserve deep review for complex/critical features only.

**Q: Can I customize the analysis passes?**
A: Not yet, but it's on the roadmap. You can tune token limits and enable/disable entire passes in config.

**Q: What if MCP sequential thinking isn't available?**
A: System automatically falls back to legacy clarify mode. No disruption.

**Q: How do I know if deep review is worth the cost for my team?**
A: Run pilot on 3-5 features and measure: (1) Token cost, (2) Issues caught early, (3) Downstream rework reduction. If rework savings > 2x cost, it's worth it.

---

## Appendix: Migration Decision Tree

```
Start here: Should I migrate to deep review?
│
├─ Do I have MCP sequential thinking available?
│  ├─ No → WAIT until MCP available, or use legacy only
│  └─ Yes → Continue
│
├─ What's my token budget constraint?
│  ├─ Very tight (<5k per feature) → Use legacy or fast mode only
│  ├─ Moderate (5-15k per feature) → Gradual rollout, use balanced mode
│  └─ Flexible (>15k per feature) → Immediate rollout, use thorough mode
│
├─ What's my team size?
│  ├─ 1-3 people → Immediate replacement (fast migration)
│  ├─ 4-10 people → Gradual rollout (pilot + rollout)
│  └─ 10+ people → Feature flag rollout (controlled)
│
├─ What's my typical feature complexity?
│  ├─ Mostly simple CRUD → Legacy sufficient, or use fast mode
│  ├─ Mix of simple + complex → Hybrid approach (auto-detection)
│  └─ Mostly complex → Deep review high value, immediate rollout
│
├─ What's my risk tolerance?
│  ├─ Low (production critical) → Gradual rollout with 4-6 week pilot
│  ├─ Medium → 2-week pilot, then rollout
│  └─ High (can tolerate issues) → Immediate replacement
│
└─ Recommendation:
   ├─ GRADUAL ROLLOUT if: Medium-large team, moderate budget, medium risk tolerance
   ├─ FEATURE FLAG if: Large team, existing flag infrastructure, low risk tolerance
   ├─ IMMEDIATE if: Small team, flexible budget, high risk tolerance
   └─ HYBRID (auto-detect) if: Mix of complexities, want smart defaults
```

---

**Last Updated:** 2025-10-24
**Version:** 1.0.0
**Next Review:** 2026-01-24 (3 months)
