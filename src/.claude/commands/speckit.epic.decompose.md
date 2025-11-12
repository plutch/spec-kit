# /speckit.epic.decompose - Generate Feature Specs from Epic

**Version**: 2.8.1
**Category**: Epic Management
**Purpose**: Generate individual feature specifications from epic blueprint with inherited architecture, linked integration contracts, and propagated constitutional requirements

---

## Usage

```bash
/speckit.epic.decompose <epic-name> [flags]
```

**Flags**:
- `-y, --yes`: Quick mode (skip interactive prompts, auto-approve all features)
- `--features <feature-ids>`: Decompose specific features only (comma-separated)
- `--force`: Overwrite existing feature specs (destructive)

**Examples**:
```bash
/speckit.epic.decompose multi-tenant-saas
/speckit.epic.decompose payment-processing --features 042,043
/speckit.epic.decompose analytics-dashboard -y
/speckit.epic.decompose multi-tenant-saas --force  # Regenerate all specs
```

---

## What This Command Does

This command generates individual feature specifications from an epic blueprint, ensuring consistency and architectural alignment across all features:

1. **Feature Specs** (`spec.md`): Full feature specifications with EARS requirements, inherited from epic
2. **Architectural Inheritance**: ADRs from `epic-architecture.md` → feature-level implementation guidance
3. **Integration Contracts**: Linked dependencies from `dependency-map.md` → explicit integration requirements
4. **Constitutional Propagation**: Requirements from `constitution.md` + `epic-architecture.md` → feature-level validation
5. **Metadata Initialization**: `spec-metadata.json` with epic linkage and phase tracking

**When to use this command**:
- After epic creation (`/speckit.epic.create`) to generate initial feature specs
- To regenerate feature specs after epic architecture changes
- To ensure all features inherit latest architectural decisions

**When NOT to use this command**:
- Features already have detailed custom specs → Manual updates preferred
- Single-feature projects → Use `/speckit.specify` instead

---

## Prerequisites

- [ ] Epic created via `/speckit.epic.create` (epic-spec.md, epic-architecture.md, dependency-map.md, epic-status.json exist)
- [ ] Epic approved (planning, architecture approvals in epic-status.json)
- [ ] Feature breakdown defined in epic-spec.md

---

## Workflow Overview

```
Step 1: Context Loading (v2.7)
    ↓
Step 2: Epic Discovery & Validation
    ↓
Step 3: Feature Selection
    ↓
Step 4: For Each Feature:
    4.1. Feature Spec Generation (spec.md)
    4.2. ADR Inheritance
    4.3. Integration Contract Linking
    4.4. Constitutional Requirement Propagation
    4.5. Metadata Initialization (spec-metadata.json)
    ↓
Step 5: Epic Status Update
    ↓
Step 6: Summary & Next Steps
```

---

## Step 1: Context Loading (v2.7 - Context Optimization)

**Current Phase**: `epic_decomposition`

**Load Memory Files**:
- **Strategic**: constitution.md (FULL) - core principles, tech stack, standards
- **Strategic**: epic-planning-principles.md (if exists) - epic-specific guidance
- **Tactical**: SKIP (not needed during decomposition)
- **Reference**: SKIP (not needed during decomposition)

**Token Budget**: 15-20KB (strategic only)

**Action**:
1. Read `.specify/config.yml` (if exists) to check context budget settings
2. Load memory files with `inclusion_mode: always` and `load_phases: epic_decomposition`
3. Filter to strategic sections only (skip `<!-- context_level: tactical -->` and `<!-- context_level: reference -->`)

**Expected Token Savings**: ~70% (25-35KB → 5-8KB) compared to loading all context

---

## Step 2: Epic Discovery & Validation

**Purpose**: Locate epic files and validate readiness for decomposition

**Actions**:

1. **Discover Epic Directory**:
   - Check `.specify/epics/<epic-name>/` exists
   - **If not found**: Error "Epic '<epic-name>' not found. Run /speckit.epic.create first."

2. **Load Epic Files**:
   - Read `epic-status.json` (required)
   - Read `epic-spec.md` (required)
   - Read `epic-architecture.md` (required)
   - Read `dependency-map.md` (required)
   - Read `constitution.md` (optional)

3. **Validate Epic Readiness**:
   - Check `epic-status.json` → `approvals.planning.approved === true`
   - Check `epic-status.json` → `approvals.architecture.approved === true`
   - **If not approved**: Warn "Epic not fully approved. Proceed anyway? (y/n)"

4. **Extract Feature List**:
   - Parse `epic-status.json` → `features[]` array
   - Extract: feature_id, feature_name, risk_level, complexity, effort_weeks, dependencies, spec_file

**Output**: Feature list with metadata for decomposition

---

## Step 3: Feature Selection

**Purpose**: Determine which features to decompose

**Interactive Mode** (default):
- Prompt: "Which features should be decomposed?"
- Show table of features with ID, Name, Risk, Complexity, Existing Spec Status
- Allow multi-select or "all"

**Quick Mode (-y)**:
- Decompose ALL features

**Filtered Mode (--features)**:
- Decompose only specified feature IDs

**Existing Spec Handling**:
- If `spec_file` exists and `--force` NOT set:
  - **Prompt**: "Feature [ID] already has spec at [path]. Overwrite? (y/n/skip)"
  - **y**: Regenerate spec (destructive)
  - **n**: Keep existing spec, skip this feature
  - **skip**: Move to next feature

**Output**: List of features to decompose

---

## Step 4: Feature Spec Generation (For Each Feature)

### Step 4.1: Feature Spec Generation (spec.md)

**Purpose**: Create comprehensive feature specification from epic blueprint

**Template Base**: `.specify/templates/spec-template.md`

**Data Sources**:
1. **Epic Spec** (`epic-spec.md`): Feature name, description, business goals, success criteria
2. **Epic Architecture** (`epic-architecture.md`): Architectural decisions, patterns, technology choices
3. **Dependency Map** (`dependency-map.md`): Dependencies, integration points

**Sections to Populate**:

1. **Header**:
   - Feature ID: From `epic-status.json`
   - Feature Name: From `epic-status.json`
   - Epic Reference: Link to parent epic
   ```markdown
   **Epic**: [EPIC-001 Multi-Tenant SaaS Platform](../../../epics/multi-tenant-saas/epic-spec.md)
   ```

2. **Overview**:
   - Description: Extract from epic-spec.md feature breakdown
   - Business Context: Inherit from epic business goals
   - Example: "This feature implements tenant isolation as part of EPIC-001 Multi-Tenant SaaS Platform. Supports business goal: Reduce per-customer infrastructure cost by 70%."

3. **User Stories**:
   - **Prompt User** (interactive): "Describe user stories for Feature [ID]"
   - **Quick Mode**: Generate stub "As a [user], I want [goal], So that [benefit]"
   - **Epic Context**: Reference epic vision in user stories

4. **Acceptance Criteria (EARS Format)**:
   - **Prompt User** (interactive): "Define acceptance criteria in EARS format"
   - **Quick Mode**: Generate stub EARS requirements
   - **Epic Integration**: Include integration contract requirements from Step 4.3

5. **Risk Assessment**:
   - Inherit risk_level from `epic-status.json` → features[i].risk_level
   - Calculate risk score (0-12) based on epic-level risks + feature-specific risks
   - **Mitigation**: Reference epic-level mitigations from epic-spec.md

6. **Architectural Decisions (Inherited from Epic)** - NEW SECTION:
   ```markdown
   ## Architectural Decisions (From Epic)

   This feature must adhere to the following epic-level architectural decisions:

   ### AD-EPIC-001: [Decision Title]
   **Status**: Accepted
   **Impact**: This feature
   **Decision**: [Summary from epic-architecture.md]
   **Implementation Guidance**: [Link to epic-architecture.md#AD-EPIC-001]

   **Feature-Specific Implications**:
   - [How this decision affects THIS feature specifically]
   - [What this feature must implement to comply]
   ```

7. **Integration Requirements** - NEW SECTION:
   ```markdown
   ## Integration Requirements

   ### Depends On (Upstream Features)
   | Feature | Contract | Delivery | What This Feature Needs |
   |---------|----------|----------|-------------------------|
   | [001] | [Interface/API] | [Date] | [Specific requirements] |

   ### Provides To (Downstream Features)
   | Feature | Contract | Delivery | What This Feature Must Deliver |
   |---------|----------|----------|-------------------------------|
   | [003] | [Interface/API] | [Date] | [Specific deliverables] |
   ```

8. **Constitutional Requirements** - NEW SECTION (if constitution.md exists):
   ```markdown
   ## Constitutional Requirements

   This feature must comply with the following constitutional principles:

   - **Article I (TDD)**: ALL code must follow RED-GREEN-REFACTOR workflow
   - **Article III (Integration-First)**: Integration tests use real dependencies (Docker/Testcontainers)
   - **Article V (Prohibited Patterns)**:
     - ❌ PROHIBITED: [List from constitution.md + epic-architecture.md]
     - ✅ REQUIRED: [Alternatives]
   ```

**Action**:
1. Create feature directory: `.specify/features/[feature-id]-[feature-slug]/`
2. Generate spec.md from template + epic data
3. Write to `.specify/features/[feature-id]-[feature-slug]/spec.md`

---

### Step 4.2: ADR Inheritance

**Purpose**: Link epic-level architectural decisions to feature implementation

**Data Source**: `epic-architecture.md` → "Architectural Decisions" section

**Process**:

1. **Extract ADRs**:
   - Parse all `### AD-EPIC-XXX:` sections from epic-architecture.md
   - For each ADR:
     - Extract: ID, Title, Status, Date, Deciders, Impact, Context, Decision, Consequences, Alternatives, Implementation Guidance, Affected Features

2. **Filter Relevant ADRs**:
   - Check `Affected Features` field
   - If contains "All features" OR current `feature_id` → Include in feature spec
   - Example: "Affected Features: All features (001-005)" → Include for all features
   - Example: "Affected Features: 001, 003" → Include only for 001, 003

3. **Generate Feature-Specific Implications**:
   - **Prompt User** (interactive): "How does AD-EPIC-001 affect Feature [ID]? What must be implemented?"
   - **Quick Mode**: Generate stub "This feature must implement [decision summary]"
   - **Examples**:
     - ADR: "Use PostgreSQL RLS for multi-tenancy"
     - Feature 001 Implication: "Implement RLS policies on all tables with tenant_id column"
     - Feature 002 Implication: "Use getTenantContext(req) helper for all database queries"

4. **Add to Spec.md**:
   - Append "Architectural Decisions (From Epic)" section with filtered ADRs
   - Include links to epic-architecture.md for full context

**Output**: Feature spec enriched with epic-level architectural guidance

---

### Step 4.3: Integration Contract Linking

**Purpose**: Explicitly define integration requirements from dependency relationships

**Data Source**: `dependency-map.md` → "Integration Points & Handoffs" section

**Process**:

1. **Extract Integration Points**:
   - Parse "Integration Points & Handoffs" table
   - Filter where `from_feature === current_feature_id` OR `to_feature === current_feature_id`

2. **Categorize Integration Requirements**:
   - **Depends On (Upstream)**: Where `to_feature === current_feature_id`
     - Example: Feature 002 depends on 001 → "002 needs ITenantService interface from 001"
   - **Provides To (Downstream)**: Where `from_feature === current_feature_id`
     - Example: Feature 001 provides to 002 → "001 must deliver ITenantService by Week 3"

3. **Generate EARS Requirements**:
   - Convert integration contracts → EARS acceptance criteria
   - Example:
     ```markdown
     REQ-INT-1: WHEN Feature 001 completes THEN `ITenantService` interface SHALL be available
     REQ-INT-2: WHEN calling `getTenantContext(req)` THEN tenant context SHALL be returned in <50ms
     ```

4. **Add to Spec.md**:
   - Append "Integration Requirements" section with upstream/downstream tables
   - Add integration contracts to "Acceptance Criteria" as EARS requirements (REQ-INT-X)

**Output**: Feature spec with explicit integration expectations

---

### Step 4.4: Constitutional Requirement Propagation

**Purpose**: Ensure feature inherits project-wide + epic-wide constitutional principles

**Data Sources**:
1. `constitution.md` (if exists) - Project-wide constitutional principles
2. `epic-architecture.md` → "Constitutional Compliance (Epic-Wide)" section

**Process**:

1. **Load Constitutional Principles**:
   - **If constitution.md exists**:
     - Parse Articles I-X
     - Extract prohibited patterns (if defined)
   - **If epic-architecture.md has "Constitutional Compliance" section**:
     - Extract epic-specific constitutional requirements
     - Extract epic-specific prohibited patterns

2. **Generate Feature-Level Constitutional Requirements**:
   - **Article I (TDD)**: "ALL code in this feature must follow RED-GREEN-REFACTOR workflow"
   - **Article III (Integration-First)**: "Integration tests must use real dependencies (Docker/Testcontainers)"
   - **Article V (Prohibited Patterns)**: List patterns from constitution + epic
   - **Article VIII (Simplicity)**: Epic-specific simplicity requirements (e.g., "Use PostgreSQL RLS directly, no custom framework")

3. **Add Validation Checkpoints**:
   - Reference Step 2.5 (Pre-Flight Check) and Step 10.4 (Constitutional Reviewer) from `/speckit.implement`
   - Example: "During implementation, Step 10.4 will validate TDD compliance via git history analysis"

4. **Add to Spec.md**:
   - Append "Constitutional Requirements" section with principles + prohibited patterns
   - Include references to constitution.md and epic-architecture.md

**Output**: Feature spec with constitutional compliance expectations

---

### Step 4.5: Metadata Initialization (spec-metadata.json)

**Purpose**: Create feature metadata with epic linkage and phase tracking

**Schema**: Same as existing spec-metadata.json, with epic-specific additions

**Data to Populate**:

```json
{
  "version": "1.0.0",
  "feature_id": "[feature-id]",
  "feature_name": "[Feature Name]",
  "epic_id": "[EPIC-001]",
  "epic_name": "[Epic Name]",
  "epic_spec_file": "../../../epics/[epic-name]/epic-spec.md",
  "epic_architecture_file": "../../../epics/[epic-name]/epic-architecture.md",
  "phase": "specification",
  "approvals": {
    "specification": {
      "generated": true,
      "approved": false,
      "timestamp": null
    },
    "gap_analysis": {
      "generated": false,
      "approved": false,
      "timestamp": null
    },
    "planning": {
      "generated": false,
      "approved": false,
      "timestamp": null
    },
    "implementation": {
      "generated": false,
      "approved": false,
      "timestamp": null
    },
    "reconciliation": {
      "generated": false,
      "approved": false,
      "timestamp": null
    }
  },
  "metadata": {
    "risk_level": "[HIGH/MEDIUM/LOW from epic-status.json]",
    "risk_score": null,
    "complexity": "[XL/L/M/S from epic-status.json]",
    "effort_weeks": "[N from epic-status.json]",
    "dependencies": ["[001]", "[002]"],
    "blocks": ["[003]", "[004]"],
    "integration_contracts": {
      "depends_on": [
        {
          "feature_id": "001",
          "contract": "ITenantService interface",
          "delivery_date": "2025-02-10"
        }
      ],
      "provides_to": [
        {
          "feature_id": "003",
          "contract": "Tenant dashboard UI component",
          "delivery_date": "2025-03-07"
        }
      ]
    },
    "architectural_decisions": ["AD-EPIC-001", "AD-EPIC-002"],
    "constitutional_requirements": ["Article I (TDD)", "Article III (Integration-First)", "Article V (Prohibited Patterns)"],
    "overall_quality": null,
    "critical_issues": null,
    "generated_from_epic": true,
    "generation_date": "[ISO 8601 timestamp]",
    "generated_by": "Claude Code (Spec-Kit v2.8.1)"
  }
}
```

**Action**:
1. Create `.specify/features/[feature-id]-[feature-slug]/spec-metadata.json`
2. Populate with feature data from epic-status.json + integration contracts + ADRs
3. Validate against spec-metadata schema

**Output**: Feature metadata with epic linkage

---

## Step 5: Epic Status Update

**Purpose**: Update epic-status.json with feature spec file paths

**Actions**:

1. **For Each Decomposed Feature**:
   - Update `epic-status.json` → `features[i].spec_file` = path to generated spec.md
   - Update `epic-status.json` → `features[i].status` = "NOT_STARTED" → "IN_PROGRESS" (if was NOT_STARTED)
   - Update `epic-status.json` → `features[i].phase` = "specification"

2. **Update Epic Metadata**:
   - Set `epic-status.json` → `metadata.last_updated` = current timestamp
   - Set `epic-status.json` → `metadata.updated_by` = "Claude Code (Spec-Kit v2.8.1)"

3. **Write Updated epic-status.json**:
   - Preserve all existing data
   - Only update spec_file paths and timestamps

**Output**: Epic status reflects feature spec generation

---

## Step 5.5: Feature Spec Generation Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate feature spec generation completion with actual evidence to prevent claiming "all decomposed" without specs created.

**Hallucination Prevention (7 Red Flags for Epic Decomposition)**:

```yaml
Detect and BLOCK these patterns:

🚨 "Features decomposed" WITHOUT showing file listings
   → Self-correction: "Wait, I need to verify all feature spec files exist with ls or find command"

🚨 "ADRs inherited" WITHOUT quoting which ADRs linked to which features
   → Self-correction: "Let me show actual ADR inheritance mapping from feature specs"

🚨 "Integration contracts linked" WITHOUT showing contract details
   → Self-correction: "I need to show the actual integration contract definitions from specs"

🚨 "Constitutional requirements propagated" WITHOUT proof in feature specs
   → Self-correction: "Let me read a sample spec to confirm constitutional section exists"

🚨 Skipping file count verification
   → Self-correction: "I must count feature spec files created vs expected from epic"

🚨 "Spec metadata generated" WITHOUT checking spec-metadata.json files
   → Self-correction: "Let me verify spec-metadata.json exists for each feature"

🚨 "All features ready" statements WITHOUT evidence
   → Self-correction: "Need to verify with actual file operations, not assume"

IF detected: STOP → Gather evidence → Report honestly
```

**Required Evidence (ALL must be provided)**:

1. **Feature Files Existence Proof**:
   ```bash
   find .specify/features/ -name "spec.md" -o -name "spec-metadata.json" | sort
   ```
   → Expected: 2 files per feature (spec.md + spec-metadata.json)
   → Verify count matches number of features decomposed

2. **ADR Inheritance Proof** (Quote from feature specs):
   - Show which ADRs affect which features
   - Example from spec.md: "### Inherited Architectural Decisions
     - AD-EPIC-001: PostgreSQL RLS for multi-tenancy
     - AD-EPIC-003: JWT authentication pattern"

3. **Integration Contract Proof** (From feature specs):
   - Show at least 1-2 integration contracts
   - Example: "Feature 001 provides: ITenantService interface, getTenantContext() helper"
   - Example: "Feature 002 depends on: tenant_id column, tenant isolation validated"

4. **Constitutional Propagation Proof** (Spot check 1 feature spec):
   - Read a sample feature spec.md
   - Verify "Constitutional Requirements" section exists
   - Quote 1-2 constitutional requirements
   - Example: "Article I (TDD): RED-GREEN-REFACTOR mandatory for all implementation tasks"

**Evidence-Based Validation Questions**:

1. **"Did I CREATE spec files for all features?"**
   - Required Evidence: File count from find command
   - ✅ PASS: Count = (Features × 2) [spec.md + spec-metadata.json per feature]
   - ❌ FAIL: Missing spec files or metadata files

2. **"Can I show which ADRs are inherited by which features?"**
   - Required Evidence: ADR inheritance mapping quoted from specs
   - ✅ PASS: At least 2 features show inherited ADRs
   - ❌ FAIL: No ADR references found in feature specs

3. **"Can I show at least 2 integration contracts?"**
   - Required Evidence: Integration contract details from specs
   - ✅ PASS: At least 2 contracts with provider/consumer defined
   - ❌ FAIL: No integration contracts or vague descriptions

4. **"Do feature specs contain constitutional requirements?"**
   - Required Evidence: Constitutional section quoted from sample feature spec
   - ✅ PASS: Constitutional Requirements section exists with at least 1 requirement
   - ❌ FAIL: No constitutional section or empty section

**Output Format** (Present to User - ONLY if ALL evidence provided):

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Epic Decomposition Complete - Review Gate Passed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Evidence Summary

### 1. Feature Spec Files Created

[File listing showing all spec.md and spec-metadata.json files]

Total Files: [N spec.md files] + [N spec-metadata.json files] = [2N files]
Expected: [2N files]
Status: ✅ Complete

### 2. ADR Inheritance Mapping

| Feature | Inherited ADRs |
|---------|----------------|
| 001 | AD-EPIC-001, AD-EPIC-003 |
| 002 | AD-EPIC-001, AD-EPIC-002 |
| [More features...] |

Total ADRs Propagated: [N]

### 3. Integration Contracts

[Integration contract table or list]

Example:
- 001 → 002: ITenantService interface, getTenantContext() helper
- 001 → 003: tenant_id column, RLS policies active

Total Contracts: [N]

### 4. Constitutional Requirements Propagation

[Quote from sample feature spec showing Constitutional Requirements section]

Example:
```
## Constitutional Requirements (Epic-Level)

This feature inherits the following constitutional requirements from EPIC-001:

- **Article I (TDD)**: RED-GREEN-REFACTOR mandatory
- **Article III (Integration-First)**: Real Auth0 test tenant, real database (Testcontainers)
```

## Review Gate Status: ✅ READY

All evidence requirements met. Features are ready for gap analysis or planning.

## Next Steps

→ Recommended: /speckit.validate-gap [FEATURE_ID] (for HIGH-risk features)
→ Then: /speckit.plan [FEATURE_ID] (for each feature)
→ Monitor: /speckit.epic.status [EPIC_NAME] (track epic progress)
```

**Review Gate Logic**:

```yaml
IF all 4 evidence questions = ✅ PASS:
  → Overall Status: ✅ READY
  → Proceed to Step 6 (Summary & Next Steps)

ELSE IF 1-2 questions = ❌ FAIL:
  → Overall Status: ⚠️ NEEDS REVIEW
  → Fix missing evidence (regenerate incomplete specs)

ELSE IF 3+ questions = ❌ FAIL:
  → Overall Status: ❌ NOT READY
  → Major issues: Decomposition likely incomplete
  → Review Steps 3-4 and regenerate feature specs
```

---

## Step 6: Summary & Next Steps

**Summary Output**:

```
✅ Epic Decomposed: [EPIC-ID] - [Epic Name]

📊 Features Decomposed: [N] / [Total]

📁 Generated Files:
  - .specify/features/001-[slug]/spec.md
  - .specify/features/001-[slug]/spec-metadata.json
  - .specify/features/002-[slug]/spec.md
  - .specify/features/002-[slug]/spec-metadata.json
  ...

🔗 Architectural Inheritance:
  - [N] ADRs linked to features
  - AD-EPIC-001: [Title] → Affects features [001, 002, 003]
  - AD-EPIC-002: [Title] → Affects features [001, 004]

🤝 Integration Contracts Linked:
  - 001 → 002: ITenantService interface
  - 001 → 003: tenant_id schema + RLS policies
  - 002 + 003 + 004 → 005: Onboarding integration

📜 Constitutional Requirements Propagated:
  - Article I (TDD): ALL features
  - Article III (Integration-First): Features 001, 002, 003
  - Article V (Prohibited Patterns): [N] patterns documented

⚠️ Next Steps:

1. **Review Generated Specs**:
   - Review each feature spec.md for accuracy
   - Customize user stories and acceptance criteria (auto-generated stubs)
   - Approve specifications via /speckit.status (per-feature)

2. **Gap Analysis** (Recommended for HIGH-risk features):
   - Run /speckit.validate-gap for each HIGH-risk feature
   - Assess implementation feasibility before planning

3. **Feature Planning**:
   - Run /speckit.plan for each feature (respects dependencies)
   - Plans will inherit epic architecture and constitutional requirements

4. **Epic Progress Tracking** (v2.8.2):
   - Use /speckit.epic.status to monitor epic progress (coming soon)
   - Track wave completion and integration point status

📚 Related Commands:
  - /speckit.clarify - Clarify ambiguities in generated specs
  - /speckit.validate-gap - Assess implementation feasibility (HIGH-risk features)
  - /speckit.plan - Plan feature implementation
  - /speckit.epic.status - View epic progress (v2.8.2 - coming soon)
```

---

## Approval Gates

Before proceeding to feature implementation:

- [ ] **All Feature Specs Reviewed**: User stories, acceptance criteria validated
- [ ] **Architectural Inheritance Verified**: ADRs correctly linked and feature-specific implications documented
- [ ] **Integration Contracts Validated**: Upstream/downstream dependencies clearly defined
- [ ] **Constitutional Requirements Clear**: All features understand compliance expectations
- [ ] **Dependency Order Understood**: Features scheduled according to dependency-map.md

**Gate Logic**:
- ✅ **All approved** → Proceed to `/speckit.plan` for each feature
- ⚠️ **Partially approved** → Revise unapproved feature specs
- ❌ **Not approved** → Regenerate specs with `--force` or manually edit

---

## Error Handling

**Common Errors**:

1. **Epic Not Found**:
   - **Error**: "Epic '<epic-name>' not found in .specify/epics/"
   - **Fix**: Run `/speckit.epic.create <epic-name>` first

2. **Epic Not Approved**:
   - **Error**: "Epic planning/architecture not approved in epic-status.json"
   - **Fix**: Approve epic via stakeholder review, update epic-status.json

3. **Missing Epic Files**:
   - **Error**: "Required file not found: epic-architecture.md"
   - **Fix**: Ensure all epic files exist (spec, architecture, dependency-map, status)

4. **Feature Spec Already Exists**:
   - **Warning**: "Feature 001 already has spec at [path]. Overwrite?"
   - **Fix**: Use `--force` to overwrite, or skip this feature

5. **Invalid Feature IDs in --features**:
   - **Error**: "Feature ID '999' not found in epic-status.json"
   - **Fix**: Use valid feature IDs from epic

6. **Circular Dependency Detected**:
   - **Warning**: "Feature 002 depends on 003, which depends on 002 (circular)"
   - **Fix**: This should be caught during epic creation, but warn if detected

---

## Token Economy (v2.8.1)

**Cost Breakdown (Per Feature)**:

| Step | Tokens | Notes |
|------|--------|-------|
| Context Loading | 5-8K | Strategic memory only (v2.7 savings) |
| Epic Discovery & Validation | 3-5K | Read epic files, validate approvals |
| Feature Selection | 1-2K | Interactive prompts or auto-select |
| Feature Spec Generation | 8-12K | Template population, epic data inheritance |
| ADR Inheritance | 5-8K | Parse ADRs, generate feature-specific implications |
| Integration Contract Linking | 3-5K | Parse dependency-map, generate EARS requirements |
| Constitutional Propagation | 3-5K | Parse constitution, epic compliance, generate feature requirements |
| Metadata Initialization | 2-3K | JSON generation |
| **Per Feature Total** | **30-48K** | ~$0.55-$0.90 per feature |

**Total Cost for Epic (5 features)**: 150K-240K tokens (~$2.75-$4.50)

**ROI**: Generating 5 feature specs manually would require:
- 5-10 hours of manual spec writing
- High risk of architectural inconsistencies
- No automated ADR linking or integration contract generation
- No constitutional requirement propagation

**Break-Even**: Using this command vs manual spec writing = 200-400% time savings + guaranteed consistency

**Combined Epic + Decomposition Cost** (5 features):
- Epic creation (v2.8.0): ~$1.50
- Epic decomposition (v2.8.1): ~$3.50
- **Total**: ~$5.00 for complete epic with 5 feature specs

---

## Best Practices

1. **Review Before Proceeding**: Generated specs are starting points, customize user stories and acceptance criteria
2. **Validate Integration Contracts**: Ensure upstream/downstream dependencies are accurate
3. **Verify ADR Applicability**: Not all epic-level ADRs may apply to all features
4. **Constitutional Compliance**: Understand constitutional requirements before planning
5. **Dependency Order**: Follow dependency-map.md when scheduling feature implementation
6. **Regenerate After Epic Changes**: If epic architecture changes significantly, regenerate feature specs with `--force`
7. **Use Gap Analysis**: Run `/speckit.validate-gap` for HIGH-risk features before planning

---

## Integration with Existing Workflow

**Epic Decomposition fits between Epic Creation and Feature Planning**:

```
/speckit.epic.create           # Create epic blueprint
    ↓
/speckit.epic.decompose         # ← NEW: Generate feature specs from epic
    ↓
/speckit.clarify (per feature)  # Clarify ambiguities (optional)
    ↓
/speckit.validate-gap (HIGH-risk) # Assess feasibility (optional)
    ↓
/speckit.plan (per feature)     # Plan feature implementation (inherits epic architecture)
    ↓
/speckit.implement (per feature) # Execute implementation (validates epic + constitutional compliance)
    ↓
/speckit.epic.status            # Monitor epic progress (v2.8.2)
```

---

## Troubleshooting

### Issue: Epic Files Not Found

**Problem**: Command fails with "Epic not found" or "epic-spec.md missing"

**Solution**: Verify epic was created successfully

```bash
# Check epic directory exists
ls -la .specify/epics/[EPIC_NAME]/

# Expected files:
# - epic-spec.md
# - epic-architecture.md
# - dependency-map.md
# - epic-status.json

# If missing, create epic first:
/speckit.epic.create [EPIC_NAME]
```

### Issue: Feature Specs Already Exist

**Problem**: Decomposition warns "Feature 001 already has spec. Overwrite?"

**Solution**: Choose appropriate action based on context

```bash
# Option A: Keep existing specs (recommended if manually created)
# When prompted: "Overwrite? (y/n/skip)"
# Press: n

# Option B: Overwrite with epic-generated specs (DESTRUCTIVE)
# When prompted: "Overwrite? (y/n/skip)"
# Press: y
# Backup first:
cp -r specs/[FEATURE_ID] specs/[FEATURE_ID].backup

# Option C: Skip conflicting features, generate only new ones
# When prompted for each feature: "skip"

# Option D: Force overwrite all (DANGEROUS - use with caution)
/speckit.epic.decompose [EPIC_NAME] --force
```

**Prevention**: Run decomposition before creating any feature specs manually

### Issue: ADR Inheritance Fails

**Problem**: Feature specs don't show ADRs from epic-architecture.md

**Solution**: Verify ADR format and re-run decomposition

```bash
# 1. Check ADR format in epic-architecture.md
cat .specify/epics/[EPIC_NAME]/epic-architecture.md

# Expected format:
# ### AD-EPIC-001: [Decision Title]
# **Affected Features**: 001, 002, 003

# 2. Fix ADR format if incorrect
nano .specify/epics/[EPIC_NAME]/epic-architecture.md

# 3. Re-run decomposition (will update existing specs)
/speckit.epic.decompose [EPIC_NAME]
```

### Issue: Integration Contracts Not Auto-Linked

**Problem**: Feature specs missing integration contract references

**Solution**: Check dependency-map.md format

```bash
# 1. Verify integration contracts table exists in dependency-map.md
cat .specify/epics/[EPIC_NAME]/dependency-map.md | grep "Integration Contract"

# Expected format:
# | From Feature | To Feature | Integration Contract | Delivery Date | Status |
# | 001 | 002 | ITenantService interface | 2025-02-10 | ON TRACK |

# 2. Fix table format if incorrect
nano .specify/epics/[EPIC_NAME]/dependency-map.md

# 3. Re-run decomposition
/speckit.epic.decompose [EPIC_NAME]
```

### Issue: Constitutional Requirements Not Propagated

**Problem**: Feature specs missing constitutional requirements section

**Solution**: Verify epic-architecture.md has constitutional compliance section

```bash
# 1. Check for "Constitutional Compliance" section in epic-architecture
grep -A 5 "Constitutional Compliance" .specify/epics/[EPIC_NAME]/epic-architecture.md

# 2. If missing, add section manually:
nano .specify/epics/[EPIC_NAME]/epic-architecture.md

# Add:
# ## Constitutional Compliance
# **Epic-Level Requirements**:
# - All features MUST follow TDD (RED-GREEN-REFACTOR)
# - No API mocking in integration tests
# - Simplicity over abstraction

# 3. Re-run decomposition
/speckit.epic.decompose [EPIC_NAME]
```

### Issue: Invalid Feature IDs in Epic

**Problem**: epic-status.json references features 001-005 but epic-spec.md only lists 3

**Solution**: Ensure epic files are consistent

```bash
# 1. Check feature count in epic-spec.md
grep -A 10 "Feature Breakdown" .specify/epics/[EPIC_NAME]/epic-spec.md

# 2. Check feature count in epic-status.json
cat .specify/epics/[EPIC_NAME]/epic-status.json | jq '.features | length'

# 3. Fix discrepancy by editing files:
nano .specify/epics/[EPIC_NAME]/epic-spec.md
nano .specify/epics/[EPIC_NAME]/epic-status.json

# 4. Verify consistency
/speckit.epic.status [EPIC_NAME] --validate
```

### Issue: Template Mismatch (spec-template.md Out of Date)

**Problem**: Generated specs missing epic sections (Epic Reference, Architectural Decisions, etc.)

**Solution**: Update spec-template.md to v2.8

```bash
# 1. Backup existing template
cp .specify/templates/spec-template.md .specify/templates/spec-template.md.backup

# 2. Copy v2.8 template from spec-kit
cp /path/to/spec-kit/src/.specify/templates/spec-template.md .specify/templates/

# 3. Re-run decomposition (will use updated template)
/speckit.epic.decompose [EPIC_NAME] --force

# 4. Review generated specs to ensure epic sections present
cat specs/[FEATURE_ID]/spec.md | grep "Epic Reference"
```

### Issue: Dependency Resolution Errors

**Problem**: Circular dependencies detected or wave assignments incorrect

**Solution**: Fix dependency graph in dependency-map.md

```bash
# 1. Validate Mermaid diagram
cat .specify/epics/[EPIC_NAME]/dependency-map.md
# Copy to https://mermaid.live/ to visualize

# 2. Identify circular dependencies
# Example: F001 → F002 → F003 → F001 (circular!)

# 3. Fix dependency arrows in dependency-map.md
nano .specify/epics/[EPIC_NAME]/dependency-map.md

# 4. Re-calculate waves (manual)
# Wave 1: Features with no dependencies
# Wave 2: Features depending only on Wave 1
# Wave 3: Features depending on Wave 2

# 5. Re-run decomposition
/speckit.epic.decompose [EPIC_NAME]
```

### Issue: Feature Spec Metadata Corrupted

**Problem**: spec-metadata.json malformed after decomposition

**Solution**: Regenerate metadata file

```bash
# 1. Backup corrupted file
cp specs/[FEATURE_ID]/spec-metadata.json specs/[FEATURE_ID]/spec-metadata.json.corrupted

# 2. Delete corrupted file
rm specs/[FEATURE_ID]/spec-metadata.json

# 3. Re-run decomposition (will recreate metadata)
/speckit.epic.decompose [EPIC_NAME]

# 4. Verify JSON syntax
cat specs/[FEATURE_ID]/spec-metadata.json | jq .
```

### Issue: Wave Assignments Incorrect

**Problem**: Features assigned to wrong waves (dependencies not respected)

**Solution**: Manually review and fix wave assignments

```bash
# 1. Check dependency graph
cat .specify/epics/[EPIC_NAME]/dependency-map.md

# 2. Recalculate waves manually:
# - Wave 1: Features with zero incoming dependencies
# - Wave 2: Features depending ONLY on Wave 1
# - Wave 3: Features depending on Wave 2

# 3. Update epic-status.json wave assignments
nano .specify/epics/[EPIC_NAME]/epic-status.json

# 4. Update feature spec metadata
nano specs/[FEATURE_ID]/spec-metadata.json
# Change: "wave": 2 → "wave": 1

# 5. Verify via epic status
/speckit.epic.status [EPIC_NAME]
```

### Issue: Multiple Decompositions Overwriting Work

**Problem**: Re-running decomposition loses manual edits to feature specs

**Solution**: Use version control and selective regeneration

```bash
# 1. Commit existing feature specs BEFORE re-running decomposition
git add specs/
git commit -m "feat: save feature specs before epic decomposition update"

# 2. Re-run decomposition
/speckit.epic.decompose [EPIC_NAME]

# 3. Review changes
git diff specs/

# 4. If manual edits lost, selectively restore:
git checkout HEAD -- specs/001-tenant-isolation/spec.md
# Keep epic-generated changes for other features

# Prevention: Use --dry-run flag (future enhancement)
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.8.1 | 2025-01-15 | Initial release - Epic decomposition workflow |

---

**Command Status**: ✅ Ready for Use (v2.8.1)
**Last Updated**: 2025-01-15
**Maintainer**: Spec-Kit Framework
