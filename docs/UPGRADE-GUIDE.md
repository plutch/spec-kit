# Spec-Kit Upgrade Guide

## Upgrading to Deep Review (v2.1+)

This guide walks you through updating an existing spec-kit installation to include the new deep review clarification functionality.

**⚠️ Important:** This upgrade is **100% backward compatible**. Your existing specifications, workflows, and commands will continue to work exactly as before.

---

## Table of Contents

- [Pre-Upgrade Checklist](#pre-upgrade-checklist)
- [Quick Upgrade (5 minutes)](#quick-upgrade-5-minutes)
- [Detailed Upgrade Steps](#detailed-upgrade-steps)
- [Verification & Testing](#verification--testing)
- [Post-Upgrade Configuration](#post-upgrade-configuration)
- [Rollback Procedure](#rollback-procedure)
- [FAQ](#faq)

---

## Pre-Upgrade Checklist

**Before you begin, verify:**

- [ ] Current working directory is your project root (where `.git/` exists)
- [ ] No uncommitted changes in spec-kit templates (run `git status`)
- [ ] You have a backup or can revert changes (git history is sufficient)
- [ ] You know your current spec-kit version (check `git log --oneline | head -5`)

**Recommended (but optional):**

- [ ] Complete any in-progress clarification sessions
- [ ] Commit current feature specs to avoid conflicts

---

## Quick Upgrade (5 minutes)

**For users who want the new functionality with minimal fuss:**

```bash
# 1. Navigate to your project root
cd /path/to/your/project

# 2. Pull latest spec-kit updates
git pull origin main  # Or your default branch

# If you installed via submodule:
# git submodule update --remote spec-kit

# 3. Add new deep review command template
# (If it doesn't exist after pull)
curl -o templates/commands/clarify-deep.md \
  https://raw.githubusercontent.com/your-org/spec-kit/main/templates/commands/clarify-deep.md

# 4. Add example configuration (optional)
mkdir -p .specify
curl -o .specify/config.example.yml \
  https://raw.githubusercontent.com/your-org/spec-kit/main/.specify/config.example.yml

# 5. Test that existing commands still work
/speckit.specify "Test feature: upgrade verification"
/speckit.clarify  # Should work exactly as before

# 6. Test new deep review command
/speckit.clarify --deep  # New functionality

# Done! 🎉
```

**What changed:**
- ✅ New command available: `clarify-deep.md` (opt-in)
- ✅ Legacy `clarify.md` unchanged (still default)
- ✅ New documentation added
- ✅ Example configuration added

**What stayed the same:**
- ✅ All existing specs work unchanged
- ✅ Default clarify command behavior unchanged
- ✅ All other commands (specify, plan, tasks) unchanged

---

## Detailed Upgrade Steps

### Step 1: Check Current Installation

```bash
# Navigate to project root
cd /path/to/your/project

# Check current spec-kit version/commit
git log --oneline templates/commands/clarify.md | head -1

# Example output:
# ca49e6f feat: add Unified SDD Methodology v2.1

# Check if deep review already exists
ls -la templates/commands/clarify-deep.md
# If exists: Already upgraded! Skip to verification.
# If not found: Continue with upgrade.
```

---

### Step 2: Backup Current State (Optional but Recommended)

```bash
# Option A: Create git branch for safety
git checkout -b pre-upgrade-backup
git checkout main  # Or your working branch

# Option B: Tag current state
git tag pre-deep-review-upgrade

# Option C: Backup templates directory
tar -czf spec-kit-templates-backup-$(date +%Y%m%d).tar.gz templates/
```

---

### Step 3: Pull Latest Changes

**If installed via Git clone:**

```bash
# Fetch latest changes
git fetch origin

# Merge changes (preserves your customizations)
git merge origin/main

# If conflicts arise (rare):
# - Accept incoming changes for new files (clarify-deep.md)
# - Keep your version for customized files (clarify.md if you modified it)
```

**If installed via Git submodule:**

```bash
# Update submodule to latest version
git submodule update --remote spec-kit

# Or update specific submodule
git submodule update --remote templates/

# Commit submodule update
git add .gitmodules templates/
git commit -m "chore: upgrade spec-kit to v2.1 (deep review)"
```

**If installed manually (copied files):**

Download new files directly:

```bash
# Download new deep review command
curl -o templates/commands/clarify-deep.md \
  https://raw.githubusercontent.com/sigent-amazon/spec-kit/main/templates/commands/clarify-deep.md

# Download example configuration
mkdir -p .specify
curl -o .specify/config.example.yml \
  https://raw.githubusercontent.com/sigent-amazon/spec-kit/main/.specify/config.example.yml

# Download documentation (optional)
curl -o docs/deep-review-configuration.md \
  https://raw.githubusercontent.com/sigent-amazon/spec-kit/main/docs/deep-review-configuration.md

curl -o docs/deep-review-example.md \
  https://raw.githubusercontent.com/sigent-amazon/spec-kit/main/docs/deep-review-example.md

curl -o docs/MIGRATION-GUIDE.md \
  https://raw.githubusercontent.com/sigent-amazon/spec-kit/main/docs/MIGRATION-GUIDE.md

curl -o docs/legacy-vs-deep-review-comparison.md \
  https://raw.githubusercontent.com/sigent-amazon/spec-kit/main/docs/legacy-vs-deep-review-comparison.md

curl -o docs/ROLLOUT-STRATEGY.md \
  https://raw.githubusercontent.com/sigent-amazon/spec-kit/main/docs/ROLLOUT-STRATEGY.md
```

---

### Step 4: Verify New Files Added

```bash
# Check new files exist
ls -la templates/commands/clarify-deep.md
ls -la .specify/config.example.yml
ls -la docs/deep-review-*.md
ls -la docs/MIGRATION-GUIDE.md

# Expected output:
# -rw-r--r-- 1 user group  [size] [date] templates/commands/clarify-deep.md
# -rw-r--r-- 1 user group  [size] [date] .specify/config.example.yml
# -rw-r--r-- 1 user group  [size] [date] docs/deep-review-configuration.md
# -rw-r--r-- 1 user group  [size] [date] docs/deep-review-example.md
# -rw-r--r-- 1 user group  [size] [date] docs/MIGRATION-GUIDE.md
# ... etc
```

---

### Step 5: Preserve Custom Configurations

**If you customized `templates/commands/clarify.md`:**

```bash
# Compare your version with new version (if updated)
git diff templates/commands/clarify.md

# If you made customizations you want to keep:
# 1. Note your customizations
# 2. Check if upstream changed (unlikely for clarify.md)
# 3. Re-apply your customizations if needed
```

**If you have custom domain patterns or config:**

Your custom configurations are NOT affected. They live in:
- `.specify/config.yml` (your custom config, not touched)
- `memory/` directory (project-specific knowledge, not touched)
- Individual feature specs in `specs/` (not touched)

---

### Step 6: Install Dependencies (If Using CLI Enhancement)

**Only needed if you want CLI flag support (`--deep` flag):**

This is **optional**. Deep review works without CLI changes by using the command name directly.

```bash
# If using Python CLI (specify_cli)
cd src/specify_cli
pip install -r requirements.txt  # Installs PyYAML if needed

# Test CLI still works
specify --help
```

---

## Verification & Testing

### Test 1: Legacy Behavior Unchanged

```bash
# Create a test spec
/speckit.specify "Test feature: upgrade verification - simple CRUD user profile"

# Run legacy clarify (default behavior)
/speckit.clarify

# Expected: Should work exactly as before
# - Taxonomy-based questions
# - Fast execution (30-60s)
# - Updates spec normally
```

**✅ Pass criteria:**
- Clarify command completes successfully
- Questions are asked and answered normally
- Spec is updated with clarifications
- No errors or warnings

---

### Test 2: Deep Review Available

**Option A: If using separate command (no CLI changes):**

You would invoke deep review by manually loading the template in your AI agent.

**Option B: If CLI supports flags:**

```bash
# Use same test spec from Test 1
/speckit.clarify --deep

# Expected: Deep review mode activates
# - Shows "Deep Review Mode" header
# - Runs sequential thinking analysis
# - Generates questions with trade-off analysis
# - Takes longer (60-180s depending on mode)
```

**✅ Pass criteria:**
- Deep review analysis runs (shows thinking process)
- Questions include recommendations and trade-offs
- Spec is updated with clarifications
- Analysis report shown at end

---

### Test 3: Auto-Detection (If Configured)

```bash
# Create config file
cp .specify/config.example.yml .specify/config.yml

# Edit config.yml to enable auto-detection:
# clarify:
#   deep_review:
#     enabled: true
#     default_mode: "balanced"

# Create complex spec
/speckit.specify "Multi-tenant authentication with SSO and MFA"

# Run clarify (should auto-detect complexity)
/speckit.clarify

# Expected: Auto-activates deep review (balanced mode)
# Because spec has "authentication" and "multi-tenant" keywords
```

**✅ Pass criteria:**
- Auto-detection triggers deep review
- Mode selection shown (e.g., "Mode: Balanced")
- Rationale displayed (e.g., "Detected authentication domain")

---

### Test 4: Existing Specs Unaffected

```bash
# List existing feature specs
ls specs/

# Open an existing spec
cat specs/001-some-feature/spec.md

# Verify:
# - Spec format unchanged
# - Clarifications section intact
# - No corruption or unexpected changes
```

**✅ Pass criteria:**
- All existing specs readable
- No formatting corruption
- Clarifications preserved

---

## Post-Upgrade Configuration

### Option 1: Keep Legacy as Default (Gradual Adoption)

**Recommended for:** Teams wanting to try deep review on select features first.

**Configuration:**
```yaml
# .specify/config.yml (create from example)
clarify:
  deep_review:
    enabled: true  # Allows opt-in via command/flag
    default_mode: "balanced"

  # Legacy is default, deep review is opt-in
  # Use: /speckit.clarify --deep
```

**Usage:**
```bash
# Most features use legacy (default)
/speckit.clarify

# Complex features opt-in to deep review
/speckit.clarify --deep
```

---

### Option 2: Make Deep Review Default (Full Adoption)

**Recommended for:** Teams ready to use deep review for most features.

**Configuration:**
```bash
# Replace default command
cd templates/commands
mv clarify.md clarify-legacy.md
mv clarify-deep.md clarify.md

# Legacy still available via flag:
# /speckit.clarify --fast
```

**Or via config:**
```yaml
# .specify/config.yml
clarify:
  deep_review:
    enabled: true
    default_mode: "balanced"

  # Deep review is default, legacy is opt-out
  # Use: /speckit.clarify --fast for legacy
```

---

### Option 3: Auto-Detection Hybrid (Smart Default)

**Recommended for:** Best of both worlds - simple features use legacy, complex use deep review.

**Configuration:**
```yaml
# .specify/config.yml
clarify:
  deep_review:
    enabled: true
    default_mode: "balanced"

    auto_enable:
      complex_features_section: true
      keywords:
        - authentication
        - payment
        - multi-tenant
        - integration
      min_requirements: 15
      min_complexity_score: 50
```

**Behavior:**
- Simple CRUD specs → Legacy mode (fast, cheap)
- Complex/critical specs → Deep review (thorough, catches issues)
- User can override with `--fast` or `--deep` flags

---

### Customize for Your Tech Stack

```yaml
# .specify/config.yml
clarify:
  deep_review:
    # Add domain patterns specific to your projects
    domain_patterns:
      # Example: If you use Next.js heavily
      nextjs_ssr:
        keywords: ["server-side rendering", "SSR", "next.js", "ISR"]
        questions:
          - "Should this use static generation or SSR?"
          - "What are the revalidation requirements?"
          - "Are there client-side hydration concerns?"

      # Example: If you use Kubernetes
      kubernetes_deployment:
        keywords: ["deployment", "kubernetes", "k8s", "helm"]
        questions:
          - "What are the scaling requirements (HPA config)?"
          - "Are there resource limits (CPU/memory)?"
          - "What health check endpoints are needed?"
```

---

## Rollback Procedure

### If Issues Arise

**Immediate Rollback (< 5 minutes):**

```bash
# Option 1: Use git to revert
git log --oneline -5  # Find commit before upgrade
git revert <commit-hash>

# Option 2: Restore from backup tag
git checkout pre-deep-review-upgrade

# Option 3: Restore backup tarball
tar -xzf spec-kit-templates-backup-20251024.tar.gz

# Option 4: Remove new files, keep legacy
rm templates/commands/clarify-deep.md
# Legacy clarify.md continues to work
```

**Partial Rollback (keep upgrade, disable deep review):**

```yaml
# .specify/config.yml
clarify:
  deep_review:
    enabled: false  # Disable deep review

# Or delete config file entirely
rm .specify/config.yml

# Deep review becomes inaccessible, legacy continues
```

---

## FAQ

### Q: Will this upgrade break my existing specifications?

**A: No.** All existing specs remain 100% compatible. The upgrade only adds new optional functionality.

---

### Q: Do I need to re-run clarify on existing specs?

**A: No.** Existing clarifications are complete and valid. Only use deep review on new features going forward.

---

### Q: What if I don't want deep review at all?

**A: No problem.** Simply don't use the new command/flag. Your workflow continues unchanged. You can delete `clarify-deep.md` if desired.

---

### Q: Can I try deep review on one feature without committing to it?

**A: Yes.** Use `/speckit.clarify --deep` on a single feature. If you don't like it, continue using legacy for other features.

---

### Q: Will deep review work without the MCP sequential thinking server?

**A: No, but it degrades gracefully.** If MCP server unavailable, deep review automatically falls back to legacy mode. No errors, no disruption.

---

### Q: How much will this increase my token costs?

**A: Depends on usage:**
- **Legacy only:** 0 increase (nothing changes)
- **Selective deep review:** Minimal increase (only features you opt-in)
- **Deep review default:** ~3-5x token cost for clarify phase, but 2-3x ROI from prevented rework

See [Token Economics](deep-review-configuration.md#token-economics) for detailed analysis.

---

### Q: Can I customize which mode (fast/balanced/thorough) is used?

**A: Yes.** Edit `.specify/config.yml`:
```yaml
clarify:
  deep_review:
    default_mode: "fast"  # or "balanced" or "thorough"
```

Or override per-feature:
```bash
/speckit.clarify --balanced
/speckit.clarify --thorough
```

---

### Q: What if my team is confused by the new options?

**A: Keep it simple:**
1. Don't replace legacy command (keep as default)
2. Introduce deep review as optional advanced feature
3. Provide quick reference card (see [ROLLOUT-STRATEGY.md](ROLLOUT-STRATEGY.md#team-training))
4. Run training session when team is ready

---

### Q: Can I upgrade just one project, not all projects?

**A: Yes.** Upgrade is per-project. Each project with spec-kit installation upgrades independently.

---

### Q: What if I customized the legacy clarify.md template?

**A: Your customizations are preserved.** New `clarify-deep.md` is a separate file. Your `clarify.md` is unchanged unless you explicitly replace it.

---

### Q: How do I know if upgrade was successful?

**A: Run verification tests:**
```bash
# Test 1: Legacy works
/speckit.clarify

# Test 2: Deep review available
/speckit.clarify --deep  # Or manually load clarify-deep.md

# Both should complete without errors
```

---

### Q: Can I upgrade during an active sprint?

**A: Yes, safely.** Since legacy behavior is unchanged:
- In-progress features continue with legacy
- New features can opt-in to deep review
- Zero disruption to current work

---

### Q: Do I need to update my Claude/AI agent configuration?

**A: Not necessarily.** If you're using agent-specific config files (`.claude/`, `.cursor/`, etc.), they continue to work. Deep review is invoked via command template, not agent config.

**Optional:** If you want deep review available in agent slash commands, add:
```markdown
<!-- .claude/commands.md or similar -->
## /clarify-deep
Load and execute templates/commands/clarify-deep.md
```

---

## Version History

### v2.1 (2025-10-24) - Deep Review Release

**Added:**
- ✨ Deep review clarification with sequential thinking
- ✨ Three analysis modes (fast, balanced, thorough)
- ✨ Auto-detection based on complexity
- ✨ Domain-specific patterns (auth, payment, multi-tenant, etc.)
- ✨ Optimization suggestion engine
- ✨ Trade-off analysis for all questions
- 📚 Comprehensive documentation suite

**Changed:**
- Nothing (100% backward compatible)

**Deprecated:**
- Nothing (legacy clarify fully supported)

---

### v2.0 (Previous)

**Added:**
- Unified SDD Methodology
- Adaptive SDD
- Business Rules validation

---

## Next Steps After Upgrade

1. **Read Documentation:**
   - [Deep Review Configuration](deep-review-configuration.md) - Full feature guide
   - [Migration Guide](MIGRATION-GUIDE.md) - Adoption strategies
   - [Comparison](legacy-vs-deep-review-comparison.md) - See the difference

2. **Try It Out:**
   - Pick your next feature
   - Use `/speckit.clarify --deep`
   - Compare experience to legacy

3. **Decide Strategy:**
   - Keep legacy default? (Gradual adoption)
   - Make deep review default? (Full adoption)
   - Use auto-detection? (Hybrid)

4. **Configure:**
   - Customize `.specify/config.yml`
   - Add your tech stack domain patterns
   - Set token limits based on budget

5. **Train Team:**
   - Share quick reference card
   - Run 1-hour training session (optional)
   - Collect feedback after 5 features

---

## Support

### Issues During Upgrade

**File an issue:**
- GitHub: https://github.com/your-org/spec-kit/issues
- Include: Error messages, git log output, steps to reproduce

**Community Support:**
- Discussions: https://github.com/your-org/spec-kit/discussions
- Slack: #spec-kit channel (if available)

### Successful Upgrade?

**Help others:**
- Share your experience in discussions
- Document any custom configurations that work well
- Contribute domain patterns for your tech stack

---

## Upgrade Checklist

Use this to track your upgrade:

### Pre-Upgrade
- [ ] Verified current version/commit
- [ ] Created backup (git tag or tarball)
- [ ] No uncommitted changes blocking update
- [ ] Completed any in-progress clarifications

### Upgrade Execution
- [ ] Pulled latest changes (git pull / submodule update / manual download)
- [ ] Verified new files added (clarify-deep.md, config.example.yml, docs/)
- [ ] Preserved any custom configurations
- [ ] Installed dependencies (if using CLI enhancement)

### Verification
- [ ] Test 1: Legacy clarify works unchanged
- [ ] Test 2: Deep review available (--deep flag or manual)
- [ ] Test 3: Auto-detection works (if configured)
- [ ] Test 4: Existing specs readable and intact

### Configuration
- [ ] Created .specify/config.yml from example
- [ ] Set default_mode based on team preference
- [ ] Added custom domain patterns (optional)
- [ ] Configured auto-enable thresholds (optional)

### Team Rollout
- [ ] Decided rollout strategy (gradual/immediate/hybrid)
- [ ] Announced upgrade to team
- [ ] Shared documentation links
- [ ] Scheduled training session (optional)

### Post-Upgrade
- [ ] Monitored first 5 features using deep review
- [ ] Collected feedback
- [ ] Tuned configuration based on usage
- [ ] Updated internal team docs

---

**Upgrade completed?** 🎉

You now have access to deep review clarification while preserving full backward compatibility with existing workflows.

**Next:** Try it on your next complex feature and see the difference!

---

**Last Updated:** 2025-10-24
**Upgrade Version:** v2.1 (Deep Review Release)
**Compatibility:** Backward compatible with all v2.x
