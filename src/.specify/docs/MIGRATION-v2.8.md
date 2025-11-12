# Migration Guide: Upgrading to v2.8 Epic Management

**Target Audience**: Existing Spec-Kit users upgrading from v2.0-v2.7 to v2.8+
**Migration Complexity**: Low (backward compatible, opt-in)
**Estimated Time**: 15-30 minutes for framework update, 1-2 hours for first epic migration

---

## Table of Contents

1. [Overview](#overview)
2. [Breaking Changes](#breaking-changes)
3. [Installation](#installation)
4. [Migration Paths](#migration-paths)
5. [Feature-by-Feature Upgrade](#feature-by-feature-upgrade)
6. [Existing Projects to Epic Migration](#existing-projects-to-epic-migration)
7. [Rollback Plan](#rollback-plan)
8. [FAQ](#faq)

---

## Overview

**What's New in v2.8**:
- **v2.8.0**: Epic creation with ADRs, dependency tracking, parallelization
- **v2.8.1**: Epic decomposition with ADR inheritance, integration contracts
- **v2.8.2**: Epic status monitoring with wave tracking, quality aggregation

**Compatibility**:
- ✅ **Fully backward compatible** with v2.0-v2.7
- ✅ **Opt-in**: Existing single-feature workflow unchanged
- ✅ **No breaking changes**: All existing commands work as before
- ✅ **Gradual adoption**: Use epic management only when needed

**Who Should Upgrade**:
- Teams managing multi-feature initiatives (2+ related features)
- Projects with complex dependency relationships
- Teams needing architectural consistency across features
- Organizations doing multi-team parallel development

**Who Can Skip**:
- Single-feature projects (use existing `/speckit.specify` workflow)
- Exploratory/prototype work (too much upfront planning)
- Simple CRUD applications (<1 week per feature)

---

## Breaking Changes

**NONE** - v2.8 is fully backward compatible.

**Additions** (opt-in):
- 3 new commands: `/speckit.epic.create`, `/speckit.epic.decompose`, `/speckit.epic.status`
- 4 new templates: `epic-spec-template.md`, `epic-architecture-template.md`, `dependency-map-template.md`, `epic-status-schema.json`
- 1 updated template: `spec-template.md` (new epic sections, auto-skipped for non-epic features)
- 1 new documentation guide: `EPIC-MANAGEMENT-GUIDE.md`

**Existing Workflows** (unchanged):
- `/speckit.specify` → `/speckit.plan` → `/speckit.implement` still works exactly as before
- Feature specs generated without epic reference work identically
- No changes to constitutional enforcement (v2.4-v2.6)
- No changes to context optimization (v2.7)

---

## Installation

### Step 1: Backup Current Project

```bash
# Create backup of .claude and .specify directories
cp -r .claude .claude.backup
cp -r .specify .specify.backup

# Optional: Commit current state to git
git add .
git commit -m "chore: backup before v2.8 upgrade"
```

### Step 2: Update Framework Files

**Option A: Fresh Copy** (Recommended)

```bash
# From spec-kit repository
cd /path/to/spec-kit
git pull  # Get latest v2.8

# Copy to your project
cp -r src/.claude /path/to/your-project/
cp -r src/.specify/templates /path/to/your-project/.specify/
cp -r src/.specify/docs /path/to/your-project/.specify/
```

**Option B: Selective Update** (Preserve customizations)

```bash
# Copy only new epic commands
cp src/.claude/commands/speckit.epic.*.md /path/to/your-project/.claude/commands/

# Copy only new epic templates
cp -r src/.specify/templates/epics /path/to/your-project/.specify/templates/

# Copy updated spec-template.md (CAREFUL: may have customizations)
cp src/.specify/templates/spec-template.md /path/to/your-project/.specify/templates/spec-template.md.new
# Manually merge: spec-template.md.new → spec-template.md

# Copy documentation
cp src/.specify/docs/EPIC-MANAGEMENT-GUIDE.md /path/to/your-project/.specify/docs/
cp src/.specify/docs/MIGRATION-v2.8.md /path/to/your-project/.specify/docs/
```

### Step 3: Verify Installation

```bash
# Check new commands are available
ls .claude/commands/speckit.epic.*.md
# Should show: speckit.epic.create.md, speckit.epic.decompose.md, speckit.epic.status.md

# Check new templates exist
ls .specify/templates/epics/
# Should show: epic-spec-template.md, epic-architecture-template.md, dependency-map-template.md, epic-status-schema.json

# Test epic creation (dry run)
# This should work without errors:
# /speckit.epic.create test-epic
```

### Step 4: Update Project Documentation

Update your project's `README.md` or docs to mention epic management:

```markdown
## Spec-Driven Development

This project uses Spec-Kit v2.8 for spec-driven development.

**Single Feature Workflow**:
/speckit.specify → /speckit.plan → /speckit.implement

**Multi-Feature Epic Workflow** (v2.8+):
/speckit.epic.create → /speckit.epic.decompose → [feature development] → /speckit.epic.status
```

---

## Migration Paths

### Path 1: No Migration (Continue Single-Feature Workflow)

**Best For**: Projects with independent features, no shared architecture

**Action**: None required. Continue using `/speckit.specify` as before.

**Benefit**: Zero migration effort, no changes to existing workflow.

### Path 2: New Epics Only

**Best For**: Existing features stay as-is, new multi-feature initiatives use epics

**Action**:
1. Keep existing features in `.specify/features/` (unchanged)
2. Create new epics in `.specify/epics/` for new initiatives
3. Use epic workflow for new features, single-feature workflow for existing

**Benefit**: Gradual adoption, no disruption to existing features.

**Example**:
```
.specify/
├── features/
│   ├── 001-user-authentication/  (existing, single-feature)
│   ├── 002-dashboard/            (existing, single-feature)
│   └── 003-notifications/        (existing, single-feature)
└── epics/
    └── multi-tenant-saas/        (NEW, epic-managed)
        ├── epic-spec.md
        ├── epic-architecture.md
        └── ...
```

### Path 3: Retrofit Existing Features to Epic

**Best For**: Existing related features that would benefit from epic coordination

**Action**:
1. Create epic retroactively (`/speckit.epic.create`)
2. Document shared architecture (ADRs)
3. Manually link existing feature specs to epic
4. Update `spec-metadata.json` with epic references

**Benefit**: Apply epic benefits to existing features (documentation, monitoring).

**Effort**: 2-4 hours depending on number of features.

**See**: [Existing Projects to Epic Migration](#existing-projects-to-epic-migration)

---

## Feature-by-Feature Upgrade

### Upgrading spec-template.md

**Changes in v2.8**:
- Added 4 new sections (Epic Reference, Architectural Decisions, Integration Requirements, Constitutional Requirements)
- Marked with `*(auto-generated if part of epic)*` → skipped for non-epic features

**Migration Strategy**:

**Option A: Accept New Template** (Recommended if minimal customizations)

```bash
# Backup old template
cp .specify/templates/spec-template.md .specify/templates/spec-template.md.v2.7

# Copy new template
cp /path/to/spec-kit/src/.specify/templates/spec-template.md .specify/templates/

# Re-apply customizations if needed
# (Compare spec-template.md.v2.7 vs spec-template.md, merge manually)
```

**Option 2: Manual Merge** (Recommended if heavy customizations)

```bash
# Copy new template to temporary location
cp /path/to/spec-kit/src/.specify/templates/spec-template.md .specify/templates/spec-template.md.new

# Compare old vs new
diff .specify/templates/spec-template.md .specify/templates/spec-template.md.new

# Manually add new sections to your template:
```

Add these sections after "Risk Assessment":

```markdown
---

## Epic Reference *(auto-generated if part of epic)*

[Epic linkage section - see spec-template.md.new]

---

## Architectural Decisions (From Epic) *(auto-generated if part of epic)*

[ADR inheritance section - see spec-template.md.new]

---

## Integration Requirements *(auto-generated if part of epic)*

[Integration contracts section - see spec-template.md.new]

---

## Constitutional Requirements *(auto-generated if part of epic)*

[Constitutional propagation section - see spec-template.md.new]

---
```

**Impact on Existing Features**:
- **Non-epic features**: New sections ignored (marked `*(auto-generated if part of epic)*`)
- **Epic features**: Sections auto-populated by `/speckit.epic.decompose`

### Upgrading Commands

**No changes required** - all existing commands (v2.0-v2.7) work identically.

**New commands added**:
- `/speckit.epic.create`
- `/speckit.epic.decompose`
- `/speckit.epic.status`

**Optional**: Update any custom slash commands or hooks that reference spec workflow to include epic workflow.

---

## Existing Projects to Epic Migration

### Scenario: 3 Related Features Already Implemented

**Existing Structure**:
```
.specify/features/
├── 001-tenant-database-schema/
│   ├── spec.md
│   ├── plan.md
│   └── spec-metadata.json
├── 002-tenant-authentication/
│   ├── spec.md
│   ├── plan.md
│   └── spec-metadata.json
└── 003-tenant-dashboard/
    ├── spec.md
    ├── plan.md
    └── spec-metadata.json
```

**Goal**: Retroactively create epic for better documentation and future coordination.

### Step 1: Create Epic Retroactively

```bash
/speckit.epic.create multi-tenant-foundation --features 001,002,003
```

**Fill in prompts**:
- Epic Name: "Multi-Tenant Foundation"
- Epic ID: EPIC-001
- Target Date: [Already complete, use past date]
- Vision: "Establish multi-tenant foundation for SaaS platform"
- Business Goals: [Retrospective business value]
- Features: 001, 002, 003 (mark as COMPLETE)

**Generated Files**:
- `.specify/epics/multi-tenant-foundation/epic-spec.md`
- `.specify/epics/multi-tenant-foundation/epic-architecture.md`
- `.specify/epics/multi-tenant-foundation/dependency-map.md`
- `.specify/epics/multi-tenant-foundation/epic-status.json`

### Step 2: Document Shared Architecture (Manual)

Edit `epic-architecture.md` to add ADRs based on existing implementation:

```markdown
### AD-EPIC-001: Database Schema Design

**Status**: Accepted
**Date**: 2024-12-01 (retrospective)
**Deciders**: Tech Lead
**Impact**: Features 001, 002, 003

**Context**: Needed to support multi-tenancy at database level.

**Decision**: Use tenant_id column in all tables with PostgreSQL RLS.

**Consequences**:
- Positive: Strong data isolation, cost-effective
- Negative: Performance overhead (~10-20ms per query)

**Implementation Guidance**:
[Code examples from existing Feature 001 implementation]

**Affected Features**: 001 (Database Schema), 002 (Authentication), 003 (Dashboard)
```

### Step 3: Link Existing Features to Epic (Manual)

For each feature, update `spec-metadata.json`:

```json
{
  "version": "1.0.0",
  "feature_id": "001",
  "feature_name": "Tenant Database Schema",
  "epic_id": "EPIC-001",
  "epic_name": "Multi-Tenant Foundation",
  "epic_spec_file": "../../../epics/multi-tenant-foundation/epic-spec.md",
  "epic_architecture_file": "../../../epics/multi-tenant-foundation/epic-architecture.md",
  "phase": "complete",
  "metadata": {
    "architectural_decisions": ["AD-EPIC-001"],
    "generated_from_epic": false,
    "retrofitted_to_epic": true,
    "retrofit_date": "2025-01-15"
  }
}
```

### Step 4: Optional - Add Epic Sections to Existing Specs

Edit each `spec.md` to add epic reference section:

```markdown
---

## Epic Reference (Retrofitted)

**Epic**: [EPIC-001: Multi-Tenant Foundation](../../../epics/multi-tenant-foundation/epic-spec.md)

**Note**: This feature was implemented before epic management (v2.8). Epic documentation added retroactively for consistency.

**Architectural Decisions**:
- See [epic-architecture.md](../../../epics/multi-tenant-foundation/epic-architecture.md)
- AD-EPIC-001: Database Schema Design (tenant_id + RLS)

---
```

### Step 5: Update epic-status.json with Actual Dates

```json
{
  "features": [
    {
      "feature_id": "001",
      "feature_name": "Tenant Database Schema",
      "status": "COMPLETE",
      "phase": "complete",
      "start_date": "2024-12-01",
      "completion_date": "2024-12-15"
    },
    {
      "feature_id": "002",
      "feature_name": "Tenant Authentication",
      "status": "COMPLETE",
      "phase": "complete",
      "start_date": "2024-12-16",
      "completion_date": "2024-12-30"
    },
    {
      "feature_id": "003",
      "feature_name": "Tenant Dashboard",
      "status": "COMPLETE",
      "phase": "complete",
      "start_date": "2025-01-05",
      "completion_date": "2025-01-12"
    }
  ],
  "timeline": {
    "total_effort_weeks_sequential": 6,
    "total_effort_weeks_parallelized": 6,
    "milestones": [
      {
        "name": "Foundation Complete",
        "features": ["001", "002", "003"],
        "target_date": "2025-01-12",
        "actual_date": "2025-01-12",
        "status": "COMPLETE"
      }
    ]
  }
}
```

### Step 6: Verify Retrofit

```bash
/speckit.epic.status multi-tenant-foundation --all
```

Output should show:
```
📊 OVERVIEW
├─ Progress: ████████████████████ 100% (3/3 complete)
├─ Status: ✅ COMPLETE

🌊 WAVE PROGRESS
All waves complete ✅

📋 FEATURE STATUS
001: ✅ COMPLETE
002: ✅ COMPLETE
003: ✅ COMPLETE
```

---

## Rollback Plan

### If v2.8 Causes Issues

**Step 1: Restore Backup**

```bash
# Remove v2.8 files
rm -rf .claude .specify

# Restore backup
mv .claude.backup .claude
mv .specify.backup .specify
```

**Step 2: Verify Rollback**

```bash
# Check commands still work
ls .claude/commands/speckit.specify.md
# Should exist

# Verify epics removed
ls .specify/epics/
# Should not exist (or be empty)
```

**Step 3: Report Issue**

If you encounter bugs, please report at: https://github.com/anthropics/claude-code/issues

Include:
- Spec-Kit version (v2.8.x)
- Error message or unexpected behavior
- Steps to reproduce

### Partial Rollback (Keep Some v2.8 Features)

**Scenario**: Epic management works, but want to revert spec-template.md changes

```bash
# Restore old spec-template.md
cp .specify/templates/spec-template.md.v2.7 .specify/templates/spec-template.md

# Keep epic commands
# (Don't delete .claude/commands/speckit.epic.*.md)
```

**Impact**: Epic commands still work, but `/speckit.epic.decompose` won't add epic sections to specs (you'd add them manually).

---

## FAQ

### Q: Do I need to migrate existing features to epic management?

**A**: No. Epic management is opt-in. Existing features continue to work with single-feature workflow (`/speckit.specify`).

### Q: Can I mix epic features and non-epic features in the same project?

**A**: Yes. You can have:
- `.specify/features/` → Single-feature specs (existing workflow)
- `.specify/epics/` → Epic-managed features (new workflow)

Both coexist without conflict.

### Q: What happens if I run `/speckit.epic.decompose` on features with existing specs?

**A**: The command prompts: "Feature 001 already has spec. Overwrite? (y/n/skip)"
- **y**: Overwrite (destructive, lose custom changes)
- **n**: Keep existing spec, skip this feature
- **skip**: Move to next feature

Use `--force` flag to overwrite all without prompting (dangerous).

### Q: Do epic features work with constitutional enforcement (v2.4-v2.6)?

**A**: Yes. Epic features inherit constitutional requirements from both:
- Project-wide constitution (`.specify/memory/constitution.md`)
- Epic-specific requirements (`epic-architecture.md` → "Constitutional Compliance" section)

The constitutional reviewer (Step 10.4) validates both during implementation.

### Q: Do epic features work with context optimization (v2.7)?

**A**: Yes. All epic commands use context optimization:
- `/speckit.epic.create`: Loads strategic context only (70% savings)
- `/speckit.epic.decompose`: Loads strategic context only
- `/speckit.epic.status`: Loads strategic context only

Epic commands are optimized for token efficiency.

### Q: Can I create epics without running `/speckit.epic.decompose`?

**A**: Yes. You can:
1. Create epic via `/speckit.epic.create`
2. Manually create feature specs via `/speckit.specify` (per feature)
3. Manually link features to epic (update spec-metadata.json)

This is more work but gives you full control.

### Q: What if I delete epic files after creating features?

**A**: Epic files (epic-spec.md, epic-architecture.md, dependency-map.md) are documentation. Deleting them doesn't break feature specs, but you lose:
- Shared architecture documentation (ADRs)
- Dependency visualization
- Epic progress monitoring (`/speckit.epic.status` won't work)

Recommendation: Keep epic files for documentation value.

### Q: Can I rename an epic after creation?

**A**: Yes, but manual updates required:
1. Rename `.specify/epics/<old-name>/` → `.specify/epics/<new-name>/`
2. Update all feature `spec-metadata.json` → `epic_spec_file`, `epic_architecture_file` paths
3. Update `epic-status.json` → `epic_name`

Recommendation: Choose epic name carefully during creation.

### Q: How do I migrate from CCPM (if I was using it)?

**A**: Spec-Kit v2.8 was inspired by CCPM but uses different workflows:

| CCPM Concept | Spec-Kit v2.8 Equivalent |
|--------------|--------------------------|
| PRD | epic-spec.md |
| Epic Tasks | Features (001, 002, 003) |
| Git Worktrees (Parallel) | Parallelization Waves (coordination via epic-status) |
| Agent Specialization | Not included (use custom agents per project) |

**Migration**: Create new epic in Spec-Kit, reference CCPM documentation as needed.

---

## Support

**Documentation**:
- [EPIC-MANAGEMENT-GUIDE.md](./EPIC-MANAGEMENT-GUIDE.md) - Complete epic management guide
- [EPIC-WORKFLOW-EXAMPLES.md](./EPIC-WORKFLOW-EXAMPLES.md) - End-to-end examples
- [CLAUDE.md](../../CLAUDE.md) - Framework overview

**Community**:
- GitHub Issues: https://github.com/anthropics/claude-code/issues
- Discussions: https://github.com/anthropics/claude-code/discussions

**Version History**:
- v2.8.0 (2025-01-15): Epic Foundation
- v2.8.1 (2025-01-15): Epic Decomposition
- v2.8.2 (2025-01-15): Epic Status & Monitoring
- v2.8.3 (2025-01-15): Testing & Documentation

---

**Migration Guide Version**: 2.8.3
**Last Updated**: 2025-01-15
