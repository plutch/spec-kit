# /speckit.epic.create - Create Multi-Feature Epic

**Version**: 2.8.0
**Category**: Epic Management
**Purpose**: Guide epic planning for multi-feature initiatives with shared architecture, dependency tracking, and coordinated delivery

---

## Usage

```bash
/speckit.epic.create [epic-name] [flags]
```

**Flags**:
- `-y, --yes`: Quick mode (skip interactive prompts, use defaults)
- `--features <feature-ids>`: Pre-specify feature IDs to include (comma-separated)
- `--risk <HIGH|MEDIUM|LOW>`: Pre-specify risk level

**Examples**:
```bash
/speckit.epic.create multi-tenant-saas
/speckit.epic.create payment-processing --features 042,043,044 --risk HIGH
/speckit.epic.create analytics-dashboard -y
```

---

## What This Command Does

This command creates the foundation for multi-feature epic coordination:

1. **Epic Specification** (`epic-spec.md`): Vision, business goals, feature breakdown, success criteria, timeline
2. **Epic Architecture** (`epic-architecture.md`): Architectural decisions (ADRs), shared patterns, integration points, technology choices
3. **Dependency Map** (`dependency-map.md`): Feature dependencies, critical path, parallelization strategy, risk propagation
4. **Epic Status Tracker** (`epic-status.json`): Structured data for epic progress tracking

**When to use this command**:
- You have 2+ related features that share architecture/dependencies
- Features need to be delivered in a coordinated sequence
- Architectural decisions impact multiple features
- You need dependency tracking and parallelization planning

**When NOT to use this command**:
- Single independent feature → Use `/speckit.specify` instead
- Features with no shared architecture → Develop independently

---

## Prerequisites

- [ ] At least 2 feature specifications exist or are planned
- [ ] Shared architectural concerns identified (data model, APIs, patterns)
- [ ] Business goals for the epic are defined
- [ ] Constitutional validation complete (if using project constitution)

---

## Workflow Overview

```
Step 1: Context Loading (v2.7)
    ↓
Step 2: Epic Metadata Gathering
    ↓
Step 3: Feature Identification & Dependency Analysis
    ↓
Step 4: Epic Risk Assessment
    ↓
Step 5: Epic Specification Generation (epic-spec.md)
    ↓
Step 6: Epic Architecture Planning (epic-architecture.md)
    ↓
Step 7: Dependency Map Creation (dependency-map.md)
    ↓
Step 8: Epic Status Initialization (epic-status.json)
    ↓
Step 9: Constitutional Validation (v2.4+)
    ↓
Step 10: Summary & Next Steps
```

---

## Step 1: Context Loading (v2.7 - Context Optimization)

**Current Phase**: `epic_planning`

**Load Memory Files**:
- **Strategic**: constitution.md (FULL) - core principles, tech stack, standards
- **Strategic**: epic-planning-principles.md (if exists) - epic-specific guidance
- **Tactical**: SKIP (not needed during epic planning)
- **Reference**: SKIP (not needed during epic planning)

**Token Budget**: 15-20KB (strategic only)

**Action**:
1. Read `.specify/config.yml` (if exists) to check context budget settings
2. Load memory files with `inclusion_mode: always` and `load_phases: epic_planning`
3. Filter to strategic sections only (skip `<!-- context_level: tactical -->` and `<!-- context_level: reference -->`)

**Expected Token Savings**: ~70% (25-35KB → 5-8KB) compared to loading all context

---

## Step 2: Epic Metadata Gathering

**Purpose**: Collect high-level epic information

**Questions to Ask** (Interactive Mode):

1. **Epic Name**: What is the epic name? (e.g., "Multi-Tenant SaaS Platform", "Payment Processing System")
2. **Epic ID**: What is the epic ID? (format: EPIC-001, EPIC-042) - **Auto-generate if not provided**
3. **Target Date**: When should this epic be complete? (YYYY-MM-DD or Q# YYYY)
4. **Vision**: What is this epic trying to achieve? Why does it matter to the business and users? (1-2 paragraphs)
5. **Business Goals**: What are the measurable business goals? (3-5 goals with metrics)

**Quick Mode (-y)**: Use defaults:
- Epic ID: Auto-generate next available (EPIC-001, EPIC-002, etc.)
- Target Date: "TBD"
- Vision: "To be documented"
- Business Goals: Empty list (to be filled in epic-spec.md)

**Output**: Collect epic metadata for later steps

---

## Step 3: Feature Identification & Dependency Analysis

**Purpose**: Identify features in this epic and their dependencies

**Questions to Ask** (Interactive Mode):

1. **Feature List**: Which features are part of this epic?
   - **Option A**: Feature IDs already exist (e.g., 001, 002, 003)
   - **Option B**: Feature names only (will create specs later via `/speckit.epic.decompose`)

2. **For Each Feature**:
   - **Feature Name**: What is the feature name?
   - **Risk Level**: 🔴 HIGH / 🟠 MEDIUM / 🟢 LOW
   - **Complexity**: XL (>3 weeks) / L (2-3 weeks) / M (1-2 weeks) / S (<1 week)
   - **Effort Estimate**: How many weeks?
   - **Dependencies**: Which features must complete BEFORE this feature can start?

3. **Dependency Validation**:
   - Check for circular dependencies (A → B → A)
   - Identify critical path (longest dependency chain)
   - Identify parallelization opportunities (features with no dependencies can run in parallel)

**Quick Mode (-y)**:
- Prompt for feature list only (required)
- Use default risk=MEDIUM, complexity=M, effort=2 weeks for all features
- Assume sequential dependencies (001 → 002 → 003)

**Output**:
- Feature list with metadata
- Dependency graph data
- Critical path identification
- Parallelization waves

---

## Step 4: Epic Risk Assessment

**Purpose**: Calculate epic-level risk score (0-12 scale)

**Risk Criteria** (same as feature-level, but epic-wide):

| Criterion | Score 0 | Score 1 | Score 2 |
|-----------|---------|---------|---------|
| **Data Sensitivity** | No PII/financial data | Some PII (names, emails) | Critical PII (SSN, payment info) |
| **Access Control** | Public/read-only | Authenticated users | Admin/privileged access |
| **External Integration** | None | 1-2 APIs | 3+ APIs or critical dependencies |
| **Performance Impact** | Low traffic | Medium traffic | High traffic/real-time |
| **Complexity** | 2-3 features, simple deps | 4-5 features, moderate deps | 6+ features, complex deps |
| **Business Impact** | Low revenue impact | Moderate revenue impact | Critical business function |

**Epic Risk Calculation**:
1. **Aggregate Feature Risks**: If ANY feature is HIGH → Epic is HIGH
2. **Sum Individual Criteria**: Add scores across all 6 criteria
3. **Classification**:
   - 🔴 **HIGH**: Score 8-12 (requires extra scrutiny, architectural review, security audit)
   - 🟠 **MEDIUM**: Score 4-7 (standard process, moderate oversight)
   - 🟢 **LOW**: Score 0-3 (fast-track, minimal oversight)

**Output**: Epic risk level and score for epic-spec.md

---

## Step 5: Epic Specification Generation (epic-spec.md)

**Purpose**: Create comprehensive epic specification from template

**Template**: `.specify/templates/epics/epic-spec-template.md`

**Sections to Populate**:

1. **Header**:
   - Epic ID, Created Date, Target Date, Risk Level, Status (🟡 PLANNING)

2. **Vision**:
   - 1-2 paragraphs from Step 2 vision question
   - Example: "Enable SaaS multi-tenancy to support 100+ customers on shared infrastructure..."

3. **Business Goals**:
   - List from Step 2 business goals question
   - Format: `1. **[Category]**: [Measurable goal]`
   - Example: "1. **Revenue**: Support 10x customer growth without infrastructure rebuild"

4. **Success Criteria**:
   - Convert business goals → measurable criteria with targets
   - Include priority (🔴 CRITICAL / 🟠 HIGH / 🟡 MEDIUM)
   - Example: "Tenant Isolation | Zero cross-tenant data leaks in prod | 100% | 🔴 CRITICAL"

5. **Feature Breakdown**:
   - Table from Step 3 feature analysis
   - Columns: ID, Feature, Risk, Complexity, Effort, Dependencies
   - Calculate total effort (sequential vs parallelized)

6. **Epic-Level Risks**:
   - Identify risks that impact multiple features
   - Include probability, impact, mitigation strategy
   - Example: "Cross-tenant data leak | 🟡 Medium | 🔴 Critical | Comprehensive testing, audit logging"

7. **Shared Architecture**:
   - Reference to epic-architecture.md
   - Key decisions (brief list)
   - Example: "Multi-tenancy approach: PostgreSQL Row-Level Security (RLS)"

8. **Dependency Map**:
   - Reference to dependency-map.md
   - Critical path summary
   - Example: "001 (Foundation) → [002, 003, 004] (Parallel) → 005 (Integration)"

9. **Timeline**:
   - Wave-based timeline from dependency analysis
   - Milestones with target dates
   - Example: "Week 1-3: Feature 001 (Tenant Isolation) - Foundation"

10. **Epic Approval Checklist**:
    - Planning Approval (vision, feature breakdown, timeline)
    - Constitutional Validation (v2.7+)
    - All checkboxes unchecked (to be filled by stakeholders)

**Action**:
1. Read epic-spec-template.md
2. Replace all placeholders with actual data
3. Write to `.specify/epics/[epic-name]/epic-spec.md`

---

## Step 6: Epic Architecture Planning (epic-architecture.md)

**Purpose**: Document shared architectural decisions and patterns

**Template**: `.specify/templates/epics/epic-architecture-template.md`

**Sections to Populate**:

1. **Architectural Decisions (ADRs)**:
   - **Prompt User**: "What are the key architectural decisions for this epic? (e.g., database choice, multi-tenancy approach, authentication strategy)"
   - **For Each Decision**:
     - AD-[EPIC]-001 format
     - Context: Why this decision needed?
     - Decision: What was decided?
     - Consequences: Pros and cons
     - Alternatives Considered: What was rejected?
     - Implementation Guidance: Code examples
     - Affected Features: Which features must follow this?

2. **Cross-Feature Patterns**:
   - **Prompt User**: "What patterns will be shared across features? (e.g., error handling, logging, context propagation)"
   - **For Each Pattern**:
     - Use case, problem, solution
     - Code example
     - Used by which features

3. **Integration Points**:
   - **Auto-Generate from Dependency Map**: For each dependency relationship, create integration point
   - Provider Feature → Consumer Feature
   - Contract (interface, API)
   - Stability (🔒 Locked / ⚠️ May change / 🚧 Under development)

4. **Technology Choices**:
   - **Prompt User**: "What are the key technology choices? (database, authentication, caching, queue, etc.)"
   - Table: Category, Choice, Rationale, Impact (features affected)

5. **Constitutional Compliance (Epic-Wide)** (v2.4+):
   - **If constitution.md exists**:
     - Extract prohibited patterns
     - Define epic-specific enforcement
     - Example: "Use PostgreSQL RLS directly, no custom multi-tenant framework (Article VIII: Simplicity)"
   - **If no constitution.md**: Skip this section

6. **Quality Standards (Epic-Wide)**:
   - Test coverage targets (per-feature and overall)
   - Performance benchmarks (if applicable)
   - Security requirements (OWASP Top 10, compliance)
   - Accessibility (WCAG 2.1 AA for UI features)

**Action**:
1. Read epic-architecture-template.md
2. Prompt user for architectural decisions (interactive mode)
3. Replace placeholders with actual data
4. Write to `.specify/epics/[epic-name]/epic-architecture.md`

**Quick Mode (-y)**: Generate stub sections with "To be documented" placeholders

---

## Step 7: Dependency Map Creation (dependency-map.md)

**Purpose**: Visualize feature dependencies and critical path

**Template**: `.specify/templates/epics/dependency-map-template.md`

**Sections to Populate**:

1. **Dependency Graph (Mermaid)**:
   - **Auto-Generate from Step 3 Data**:
     - Nodes: Features with ID, name, risk color, effort estimate
     - Edges: Dependencies (arrows pointing from dependency to dependent)
     - Styling: Color by risk level (red=HIGH, orange=MEDIUM, yellow=LOW)

2. **Dependency Table**:
   - **Auto-Generate from Step 3 Data**:
     - Feature ID, Name, Depends On, Blocks, Type (Hard/Soft), Rationale

3. **Critical Path Analysis**:
   - **Auto-Calculate**:
     - Critical path = longest dependency chain
     - Total duration (critical path) = sum of efforts on critical path
     - Parallelization opportunities = features with no dependencies (Wave 1), features depending only on Wave 1 (Wave 2), etc.
     - Time savings = sequential duration - parallelized duration

4. **Risk Propagation Matrix**:
   - **Auto-Generate**:
     - For each feature: "If delayed by 1 week" → impact on downstream features
     - "If fails completely" → impact on epic
     - Mitigation strategies (user input or defaults)

5. **Parallelization Strategy**:
   - **Wave-Based Timeline**:
     - Wave 1: Features with no dependencies
     - Wave 2: Features depending only on Wave 1
     - Wave 3: Features depending on Wave 2, etc.
   - **Team Assignment** (if multiple teams):
     - Prompt user: "How many teams are available?"
     - Assign features to teams by wave

6. **Integration Points & Handoffs**:
   - **Auto-Generate from Dependencies**:
     - From Feature → To Feature → Contract → Delivery Date → Status (🟢/🟡/🔴)

**Action**:
1. Read dependency-map-template.md
2. Auto-generate dependency graph, tables, critical path
3. Write to `.specify/epics/[epic-name]/dependency-map.md`

---

## Step 8: Epic Status Initialization (epic-status.json)

**Purpose**: Create structured epic status tracker

**Schema**: `.specify/templates/epics/epic-status-schema.json`

**Data to Populate**:

```json
{
  "epic_id": "[EPIC-ID from Step 2]",
  "epic_name": "[Epic name from Step 2]",
  "version": "1.0.0",
  "created_date": "[Today's date]",
  "target_date": "[Target date from Step 2]",
  "risk_level": "[HIGH/MEDIUM/LOW from Step 4]",
  "risk_score": [0-12 from Step 4],
  "phase": "planning",
  "status": "PLANNING",
  "features": [
    {
      "feature_id": "[001]",
      "feature_name": "[Feature name from Step 3]",
      "risk_level": "[HIGH/MEDIUM/LOW]",
      "complexity": "[XL/L/M/S]",
      "effort_weeks": [N],
      "dependencies": ["[002]"],
      "blocks": ["[003]"],
      "status": "NOT_STARTED",
      "phase": "specification",
      "spec_file": ".specify/features/[feature-id]-[feature-slug]/spec.md",
      "plan_file": ""
    }
  ],
  "approvals": {
    "planning": {
      "generated": true,
      "approved": false,
      "timestamp": null,
      "approved_by": null
    },
    "architecture": {
      "generated": true,
      "approved": false,
      "timestamp": null,
      "approved_by": null
    },
    "constitutional": {
      "generated": false,
      "approved": false,
      "timestamp": null,
      "approved_by": null
    }
  },
  "timeline": {
    "total_effort_weeks_sequential": [Sum of all feature efforts],
    "total_effort_weeks_parallelized": [Critical path duration],
    "critical_path": ["[001]", "[003]", "[005]"],
    "waves": [
      {
        "wave_number": 1,
        "features": ["001"],
        "duration_weeks": 3,
        "dependencies": [],
        "status": "NOT_STARTED"
      }
    ],
    "milestones": [
      {
        "name": "Foundation Complete",
        "features": ["001"],
        "target_date": "[Calculated from start + wave 1 duration]",
        "status": "NOT_STARTED"
      }
    ]
  },
  "dependencies": {
    "integration_points": [
      {
        "from_feature": "001",
        "to_feature": "002",
        "contract": "[Auto-generated or user input]",
        "delivery_date": "[Calculated from timeline]",
        "status": "ON_TRACK"
      }
    ]
  },
  "quality": {
    "overall_quality": null,
    "critical_issues_count": 0,
    "test_coverage_average": null,
    "security_review_status": "NOT_STARTED"
  },
  "metadata": {
    "stakeholders": ["[User input or empty]"],
    "business_goals": ["[From Step 2]"],
    "epic_spec_file": ".specify/epics/[epic-name]/epic-spec.md",
    "epic_architecture_file": ".specify/epics/[epic-name]/epic-architecture.md",
    "dependency_map_file": ".specify/epics/[epic-name]/dependency-map.md",
    "last_updated": "[ISO 8601 timestamp]",
    "updated_by": "Claude Code (Spec-Kit v2.8)"
  }
}
```

**Action**:
1. Create `.specify/epics/[epic-name]/` directory
2. Populate JSON with data from Steps 2-7
3. Write to `.specify/epics/[epic-name]/epic-status.json`
4. Validate against epic-status-schema.json

---

## Step 9: Constitutional Validation (v2.4+)

**Purpose**: Ensure epic complies with project constitution

**If constitution.md exists**:

1. **Load Constitution**:
   - Read `.specify/memory/constitution.md`
   - Extract constitutional principles (Articles I-X)
   - Extract prohibited patterns (if defined)

2. **Epic-Level Constitutional Questions**:
   - **Article I (TDD)**: "Will ALL features in this epic follow strict TDD (RED-GREEN-REFACTOR)?"
   - **Article III (Integration-First)**: "Will integration tests use real dependencies (Docker/Testcontainers), not mocks?"
   - **Article VIII (Simplicity)**: "Are you avoiding custom frameworks/abstractions in favor of language/library built-ins?"
   - **Prohibited Patterns**: "Does epic-architecture.md avoid prohibited patterns from constitution?"

3. **Validation Result**:
   - ✅ **PASS**: Epic complies with all constitutional principles
   - ⚠️ **CONDITIONAL**: Minor concerns, may proceed with caution
   - ❌ **FAIL**: Epic violates constitution, must revise before proceeding

4. **Update epic-spec.md**:
   - Add Constitutional Validation section to Epic Approval checklist
   - Document result

**If no constitution.md**: Skip this step

---

## Step 9.5: Epic Creation Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate epic creation completion with actual evidence to prevent claiming "all done" without files created.

**Hallucination Prevention (7 Red Flags for Epic Creation)**:

```yaml
Detect and BLOCK these patterns:

🚨 "Epic complete" WITHOUT showing file listings
   → Self-correction: "Wait, I need to verify all epic files exist with ls command"

🚨 "ADRs documented" WITHOUT quoting actual ADR titles/content
   → Self-correction: "Let me quote ADR titles from epic-architecture.md to prove they exist"

🚨 "Dependencies analyzed" WITHOUT showing dependency graph or table
   → Self-correction: "I need to show the actual dependency relationships from dependency-map.md"

🚨 "Constitutional validation passed" WITHOUT showing Step 9 result
   → Self-correction: "Let me display the actual validation outcome (PASS/CONDITIONAL/FAIL)"

🚨 Skipping file existence checks
   → Self-correction: "I must run ls to verify epic-spec.md, epic-architecture.md, dependency-map.md, epic-status.json all exist"

🚨 "Critical path calculated" WITHOUT showing timeline comparison
   → Self-correction: "Let me show sequential vs parallelized duration with actual numbers"

🚨 "Probably correct" or "should be" statements
   → Self-correction: "Need to verify with actual file reads, not assume"

IF detected: STOP → Gather evidence → Report honestly
```

**Required Evidence (ALL must be provided)**:

1. **File Existence Proof**:
   ```bash
   ls -lh .specify/epics/[EPIC_NAME]/{epic-spec.md,epic-architecture.md,dependency-map.md,epic-status.json}
   ```
   → ALL 4 files must exist with non-zero file sizes

2. **ADR Proof** (Quote from epic-architecture.md):
   - Show at least 2-3 ADR titles
   - Example: "AD-EPIC-001: PostgreSQL Row-Level Security for Multi-Tenancy"
   - Example: "AD-EPIC-002: JWT with tenant claim for authentication"

3. **Dependency Graph Proof** (From dependency-map.md):
   - Show critical path summary OR Mermaid diagram
   - Example: "Critical Path: 001 → 003 → 005 (6 weeks parallelized vs 10 weeks sequential)"
   - Example: Feature dependency table with Hard/Soft/Optional types

4. **Constitutional Validation Proof** (Step 9 result):
   - Quote actual validation outcome
   - Example: "✅ PASS - Epic architecture complies with Article V (Integration-First Testing)"
   - Example: "⚠️ CONDITIONAL - Minor concern: Feature 003 may need mock justification"
   - If no constitution.md: "N/A - No constitution present"

**Evidence-Based Validation Questions**:

1. **"Did I actually CREATE all 4 epic files?"**
   - Required Evidence: Output of `ls -lh .specify/epics/[epic-name]/`
   - ✅ PASS: All 4 files exist with >0 bytes
   - ❌ FAIL: Missing files or 0-byte files

2. **"Can I quote at least 2 ADR titles from epic-architecture.md?"**
   - Required Evidence: Quoted ADR titles (AD-EPIC-001, AD-EPIC-002, etc.)
   - ✅ PASS: At least 2 ADRs with titles + status
   - ❌ FAIL: Cannot quote ADRs or epic-architecture.md empty

3. **"Can I show the critical path with duration?"**
   - Required Evidence: Critical path summary from dependency-map.md
   - ✅ PASS: Critical path identified with timeline (e.g., "001 → 003 → 005 = 6 weeks")
   - ❌ FAIL: No critical path or no duration estimate

4. **"Did Step 9 constitutional validation actually run?"**
   - Required Evidence: Step 9 validation result (PASS/CONDITIONAL/FAIL/N/A)
   - ✅ PASS: Clear validation outcome documented
   - ❌ FAIL: Step 9 skipped or no result shown

**Output Format** (Present to User - ONLY if ALL evidence provided):

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Epic Creation Complete - Review Gate Passed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Evidence Summary

### 1. Epic Files Created

[File listing output showing all 4 files]

### 2. Architectural Decisions (ADRs)

- AD-EPIC-001: [Title]
- AD-EPIC-002: [Title]
- [Additional ADRs...]

Total ADRs: [N]

### 3. Dependency Analysis

**Critical Path**: [Feature IDs] = [X weeks parallelized] vs [Y weeks sequential]
**Parallelization Savings**: [Z%]

[Dependency table or Mermaid diagram summary]

### 4. Constitutional Validation

[Step 9 result: ✅ PASS / ⚠️ CONDITIONAL / ❌ FAIL / N/A]

## Review Gate Status: ✅ READY

All evidence requirements met. Epic is ready for decomposition or implementation.

## Next Steps

→ Ready: /speckit.epic.decompose [EPIC_NAME]
→ Or: Create individual feature specs with /speckit.specify
→ Monitor: /speckit.epic.status [EPIC_NAME] (v2.8)
```

**Review Gate Logic**:

```yaml
IF all 4 evidence questions = ✅ PASS:
  → Overall Status: ✅ READY
  → Proceed to Step 10 (Summary & Next Steps)

ELSE IF 1-2 questions = ❌ FAIL:
  → Overall Status: ⚠️ NEEDS REVIEW
  → Fix missing evidence before proceeding
  → Re-run failed steps (e.g., regenerate empty files)

ELSE IF 3+ questions = ❌ FAIL:
  → Overall Status: ❌ NOT READY
  → Major issues: Epic creation likely incomplete
  → Review all steps and regenerate epic
```

---

## Step 10: Summary & Next Steps

**Summary Output**:

```
✅ Epic Created: [EPIC-ID] - [Epic Name]

📁 Files Generated:
  - .specify/epics/[epic-name]/epic-spec.md
  - .specify/epics/[epic-name]/epic-architecture.md
  - .specify/epics/[epic-name]/dependency-map.md
  - .specify/epics/[epic-name]/epic-status.json

📊 Epic Metrics:
  - Features: [N]
  - Total Effort (Sequential): [N] weeks
  - Total Effort (Parallelized): [N] weeks
  - Time Savings: [X]%
  - Critical Path: [Feature IDs]
  - Risk Level: [🔴/🟠/🟢] [HIGH/MEDIUM/LOW] (Score: [N]/12)

⚠️ Next Steps:

1. **Review Epic Specification**:
   - Review epic-spec.md with stakeholders
   - Approve vision, business goals, success criteria
   - Validate feature breakdown and timeline

2. **Review Epic Architecture**:
   - Review epic-architecture.md with tech lead
   - Approve architectural decisions (ADRs)
   - Validate technology choices

3. **Review Dependency Map**:
   - Review dependency-map.md with engineering team
   - Validate critical path and parallelization strategy
   - Confirm integration points

4. **Create Feature Specifications** (If Not Already Existing):
   - Use /speckit.epic.decompose to generate feature specs from epic
   - OR manually create specs via /speckit.specify for each feature

5. **Start Epic Implementation**:
   - Begin with Wave 1 features (no dependencies)
   - Use /speckit.plan, /speckit.tasks, /speckit.implement for each feature
   - Track progress with /speckit.epic.status (v2.8 - coming soon)

6. **Constitutional Compliance** (If Applicable):
   - Ensure all features adhere to epic-architecture.md decisions
   - Validate prohibited patterns not introduced
   - Review constitutional compliance at Step 10.4 of /speckit.implement

📚 Related Commands:
  - /speckit.epic.decompose - Generate feature specs from epic (v2.8)
  - /speckit.epic.status - View epic progress and feature status (v2.8)
  - /speckit.specify - Create individual feature specification
  - /speckit.plan - Plan feature implementation
  - /speckit.implement - Execute feature implementation
```

---

## Approval Gates

Before proceeding to epic implementation:

- [ ] **Epic Specification Approved**: Vision, business goals, feature breakdown validated by stakeholders
- [ ] **Epic Architecture Approved**: Architectural decisions reviewed and approved by tech lead
- [ ] **Dependency Map Approved**: Critical path and parallelization strategy validated by engineering team
- [ ] **Constitutional Validation Passed** (v2.4+): Epic complies with project constitution
- [ ] **Timeline Feasible**: Engineering capacity confirmed for target date

**Gate Logic**:
- ✅ **All approved** → Proceed to `/speckit.epic.decompose` or `/speckit.specify` for individual features
- ⚠️ **Partially approved** → Revise unapproved sections, re-submit for approval
- ❌ **Not approved** → Major revision needed, revisit vision or feature scope

---

## Error Handling

**Common Errors**:

1. **Circular Dependency Detected**:
   - **Error**: "Feature [ID] depends on [ID2], which depends on [ID], creating a circular dependency"
   - **Fix**: Review dependency relationships, break the cycle

2. **Less Than 2 Features**:
   - **Error**: "Epics require at least 2 features. Use /speckit.specify for single features."
   - **Fix**: Add more features or use single-feature workflow

3. **Invalid Epic ID Format**:
   - **Error**: "Epic ID must match format EPIC-XXX (e.g., EPIC-001)"
   - **Fix**: Use correct format

4. **Epic Directory Already Exists**:
   - **Error**: ".specify/epics/[epic-name]/ already exists"
   - **Fix**: Choose different epic name or delete existing epic

5. **Missing Templates**:
   - **Error**: "Template file not found: .specify/templates/epics/epic-spec-template.md"
   - **Fix**: Ensure spec-kit templates are installed in .specify/templates/epics/

---

## Token Economy (v2.8)

**Cost Breakdown**:

| Step | Tokens | Cumulative | Notes |
|------|--------|------------|-------|
| Context Loading | 5-8K | 5-8K | Strategic memory only (v2.7 savings) |
| Metadata Gathering | 2-3K | 7-11K | Interactive prompts |
| Feature Analysis | 5-10K | 12-21K | Dependency graph generation |
| Risk Assessment | 1-2K | 13-23K | 6-criteria scoring |
| Epic Spec Generation | 10-15K | 23-38K | Template population |
| Architecture Planning | 15-20K | 38-58K | ADRs, patterns, integration points |
| Dependency Map | 10-15K | 48-73K | Mermaid graph, critical path |
| Status Initialization | 3-5K | 51-78K | JSON generation |
| Constitutional Validation | 5-10K | 56-88K | If constitution exists |
| Summary | 2-3K | 58-91K | Next steps |

**Total Cost**: 58K-91K tokens (~$1.10-$1.75)

**ROI**: Creating an epic manually (without this command) would require:
- 3-5 hours of manual template copying, data entry, dependency analysis
- High risk of inconsistencies between epic-spec.md, epic-architecture.md, dependency-map.md
- No automated dependency validation or critical path calculation

**Break-Even**: Using this command vs manual epic creation = 100-200% time savings + guaranteed consistency

---

## Best Practices

1. **Start with Vision**: Clear vision statement drives all downstream decisions
2. **Validate Dependencies Early**: Catch circular dependencies before feature work begins
3. **Document Architectural Decisions**: ADRs prevent future confusion and rework
4. **Prioritize Critical Path**: Focus on bottleneck features first
5. **Plan for Parallelization**: Identify features that can be developed simultaneously
6. **Define Integration Contracts**: Prevent integration failures by documenting handoffs
7. **Constitutional Compliance**: Ensure epic-level decisions align with project principles (v2.4+)
8. **Use Context Optimization**: Load only strategic memory during epic planning (v2.7)

---

## Troubleshooting

### Issue: Epic Directory Already Exists

**Problem**: Command fails with "Epic directory already exists" error

**Solution**: Choose one of three options

```bash
# Option A: Delete existing epic (DESTRUCTIVE - lose all epic files)
rm -rf .specify/epics/[EPIC_NAME]
/speckit.epic.create [EPIC_NAME]

# Option B: Use different epic name
/speckit.epic.create [EPIC_NAME]-v2

# Option C: Resume partial epic creation (if files incomplete)
# Manually check which files are missing:
ls -la .specify/epics/[EPIC_NAME]/
# Re-create only missing files manually or delete partial epic
```

**Prevention**: Check for existing epics before creating:
```bash
ls .specify/epics/
```

### Issue: Template Files Not Found

**Problem**: Error "Template not found: .specify/templates/epics/[template].md"

**Solution**: Verify Spec-Kit installation is complete

```bash
# Check if template directory exists
ls -la .specify/templates/epics/

# Expected files:
# - epic-spec-template.md
# - epic-architecture-template.md
# - dependency-map-template.md
# - epic-status-schema.json

# If missing, re-copy from spec-kit distribution:
cp -r /path/to/spec-kit/src/.specify/templates/epics .specify/templates/
```

### Issue: Invalid Feature Count or IDs

**Problem**: Epic created with wrong feature count or invalid IDs

**Solution**: Edit epic-spec.md and epic-status.json manually

```bash
# 1. Edit epic-spec.md to fix feature list
nano .specify/epics/[EPIC_NAME]/epic-spec.md

# 2. Edit epic-status.json to match
nano .specify/epics/[EPIC_NAME]/epic-status.json

# 3. Update dependency-map.md if feature count changed
nano .specify/epics/[EPIC_NAME]/dependency-map.md

# 4. Verify consistency
/speckit.epic.status [EPIC_NAME] --validate
```

### Issue: JSON Schema Validation Errors

**Problem**: epic-status.json fails schema validation

**Solution**: Fix JSON syntax or regenerate from template

```bash
# Option A: Validate JSON manually
cat .specify/epics/[EPIC_NAME]/epic-status.json | jq .
# If error: "parse error" → fix JSON syntax

# Option B: Regenerate from schema template
cp .specify/templates/epics/epic-status-schema.json .specify/epics/[EPIC_NAME]/epic-status.json
# Manually fill in epic details
```

### Issue: Constitutional Validation Fails (Step 9)

**Problem**: Epic architecture violates project constitution

**Solution**: Review and fix ADRs in epic-architecture.md

```bash
# 1. Read constitution to understand violation
cat .specify/memory/constitution.md

# 2. Edit epic-architecture.md to align with constitution
nano .specify/epics/[EPIC_NAME]/epic-architecture.md

# 3. Common violations:
#    - ADR recommends prohibited pattern (e.g., API mocking)
#    - ADR contradicts simplicity principle
#    - ADR introduces custom abstraction without justification

# 4. Re-run constitutional validation
# (Re-run /speckit.epic.create from Step 9)
```

### Issue: Git Conflicts During Epic Creation

**Problem**: Multiple team members creating epics simultaneously

**Solution**: Coordinate epic creation or use git conflict resolution

```bash
# 1. Pull latest changes before creating epic
git pull

# 2. If conflict occurred, resolve manually
git status  # Check conflicting files
git add .specify/epics/[EPIC_NAME]/*
git commit -m "feat: resolve epic creation conflict"

# 3. Prevention: Coordinate epic creation via team communication
```

### Issue: Epic Name Contains Special Characters

**Problem**: Epic directory name with spaces or special characters causes errors

**Solution**: Use kebab-case for epic names

```bash
# ❌ Invalid:
/speckit.epic.create "Multi Tenant SaaS"

# ✅ Valid:
/speckit.epic.create multi-tenant-saas

# If already created with invalid name:
mv ".specify/epics/Multi Tenant SaaS" .specify/epics/multi-tenant-saas
# Update all file references to new path
```

### Issue: Dependency Map Mermaid Syntax Errors

**Problem**: dependency-map.md Mermaid diagram doesn't render

**Solution**: Fix Mermaid syntax

```bash
# 1. Validate Mermaid syntax online
# Copy dependency graph to: https://mermaid.live/

# 2. Common issues:
#    - Missing semicolons (not required but helps readability)
#    - Invalid node IDs (must be alphanumeric + underscores)
#    - Incorrect arrow syntax (use --> for dependencies)

# 3. Fix syntax in dependency-map.md
nano .specify/epics/[EPIC_NAME]/dependency-map.md
```

### Issue: Parallel Epic Creation by Multiple Teams

**Problem**: Two teams creating different epics simultaneously causing merge conflicts

**Solution**: Use feature branches and merge sequentially

```bash
# Team A:
git checkout -b epic/multi-tenant-saas
/speckit.epic.create multi-tenant-saas
git add .specify/epics/multi-tenant-saas
git commit -m "feat: create multi-tenant SaaS epic"
git push origin epic/multi-tenant-saas
# Create PR

# Team B:
git checkout -b epic/payment-processing
/speckit.epic.create payment-processing
git add .specify/epics/payment-processing
git commit -m "feat: create payment processing epic"
git push origin epic/payment-processing
# Create PR

# Merge PRs sequentially (no conflicts since different epic names)
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.8.0 | 2025-01-15 | Initial release - Epic creation workflow |

---

**Command Status**: ✅ Ready for Use (v2.8.0)
**Last Updated**: 2025-01-15
**Maintainer**: Spec-Kit Framework
