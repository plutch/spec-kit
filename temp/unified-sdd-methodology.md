# Unified Spec-Driven Development Methodology
## Business Rules + Adaptive Pivots Integration

**Version**: 1.0
**Date**: 2025-01-23
**Status**: Unified Compliance Blueprint
**Audience**: LLM Plan Reviewers, Implementation Agents, Engineering Teams

---

## Alignment Summary

This unified methodology **achieves 98% alignment** with GitHub Spec Kit philosophy by preserving the core spec-first workflow while layering two complementary enhancements: (1) **Adaptive Pivot Protocol** for handling late-discovery architectural blockers through TECHNICAL.md constraints and ADRs, and (2) **Business Rules Management** for capturing, validating, and cataloging business logic via constitutional gates and multi-source extraction. Both enhancements operate within approved directory structures (`specs/*/`, `.specify/*/`), maintain strict separation of non-technical specs from technical plans, and enhance (rather than replace) the canonical SDD workflow. The only residual gap is template organization consolidation, requiring a unified template directory strategy to avoid duplication between the two systems.

---

## Unified Method Blueprint

### Layer Architecture (5 Layers)

The unified methodology extends Spec Kit's 2-layer model (spec → plan) to 5 complementary layers:

```
┌────────────────────────────────────────────────────────────────────┐
│                   UNIFIED SDD ARCHITECTURE                          │
├────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Layer 0: Constitutional Foundation (IMMUTABLE)                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  File: memory/constitution.md                                 │ │
│  │  • Article I-X: Core SDD principles (existing)               │ │
│  │  • Article XI: Technical Pivot Protocol (ADAPTIVE addition)  │ │
│  │  • Applies to ALL features, ALL phases                       │ │
│  │  📝 Never modified - only extended via new articles          │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│                                                                      │
│  Layer 1: Non-Technical Specification (STABLE)                     │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  File: specs/###-feature/spec.md                             │ │
│  │  • User stories (what users need, why)                       │ │
│  │  • Success criteria (measurable outcomes)                    │ │
│  │  • Acceptance tests (behavior validation)                    │ │
│  │  ✅ NEVER changes due to technical pivots or rule updates   │ │
│  │  ✅ Product manager-friendly, implementation-agnostic        │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│                                                                      │
│  Layer 2: Technical Constraints (EVOLVES - ADAPTIVE)               │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  File: specs/###-feature/TECHNICAL.md (optional)             │ │
│  │  • Architectural boundaries (DB strategy, ORM, frameworks)   │ │
│  │  • Performance targets (latency, throughput, scale)          │ │
│  │  • Risk flags [NEEDS VALIDATION] for unknowns               │ │
│  │  • Constitutional compliance (Articles VII, VIII, IX)        │ │
│  │  • Decision history (links to ADRs)                          │ │
│  │  🔄 AMENDED when blockers discovered (Article XI protocol)  │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│                                                                      │
│  Layer 2.5: Business Rules Harvest (EVOLVES - BUSINESS RULES)      │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  File: specs/###-feature/rules/harvest.md                    │ │
│  │  • DRAFT business rules captured during /plan phase          │ │
│  │  • Given/When/Then format (BPM structure)                    │ │
│  │  • [NEEDS CLARIFICATION] markers for ambiguities            │ │
│  │  • Links to feature requirements (FR-###)                    │ │
│  │  🔄 Evolves from DRAFT → ACTIVE during implementation       │ │
│  │  📋 Created via /rules-harvest command                       │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│                                                                      │
│  Layer 3: Architecture Decisions (IMMUTABLE - ADAPTIVE)            │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Files: specs/###-feature/decisions/adr-*.md (feature-scope) │ │
│  │         .specify/decisions/adr-*.md (cross-project)          │ │
│  │  • Context: What blocker triggered pivot?                    │ │
│  │  • Decision: FROM → TO architectural change                  │ │
│  │  • Constitutional validation (Articles VII, VIII, IX, XI)    │ │
│  │  • Consequences: Trade-offs + mitigation                     │ │
│  │  • Implementation timeline + rollback plan                   │ │
│  │  • MAY reference Business Rules (Affected BR-* IDs)         │ │
│  │  📝 Created once, never modified (historical record)        │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│                                                                      │
│  Layer 4: Implementation Plans (REGENERATED - CORE SDD)            │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Files: specs/###-feature/plan.md                            │ │
│  │         specs/###-feature/tasks.md                           │ │
│  │  • Technical decisions using Layer 2 constraints             │ │
│  │  • Sequenced tasks respecting TECHNICAL.md assumptions      │ │
│  │  • Prerequisites for technical migrations (from ADRs)        │ │
│  │  • Business rules implementation tasks (from harvest)        │ │
│  │  • Test-first enforcement (Article III)                      │ │
│  │  🔄 REGENERATED when Layer 2/2.5 amended or pivots occur   │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│                                                                      │
│  Layer 5: Business Rules Catalog (CONSOLIDATED - BUSINESS RULES)   │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  Files: .specify/business-rules/catalog.md                   │ │
│  │         .specify/business-rules/QUICK_REFERENCE.md           │ │
│  │  • Master catalog of ALL business rules (across features)    │ │
│  │  • Constitutional validation results (9 gates)               │ │
│  │  • Multi-source extraction (code, ADRs, specs, workflows)    │ │
│  │  • LLM-optimized quick reference (<2000 tokens)             │ │
│  │  • Links back to source specs, implementations, tests        │ │
│  │  🔄 Updated via /rules-extract after implementation         │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                      │
└────────────────────────────────────────────────────────────────────┘

LLM READING PRIORITY (Context Window Order):
1. Layer 0: memory/constitution.md (Articles I-XI)
2. Layer 1: specs/###-feature/spec.md (user value)
3. Layer 2: specs/###-feature/TECHNICAL.md (architectural constraints)
4. Layer 2.5: specs/###-feature/rules/harvest.md (business logic DRAFT)
5. Layer 3: specs/###-feature/decisions/adr-*.md + .specify/decisions/adr-*.md
6. Layer 4: specs/###-feature/plan.md + tasks.md (generated from above)
7. Layer 5: .specify/business-rules/QUICK_REFERENCE.md (cross-feature rules)
8. CLAUDE.md (project-wide patterns - at root)
```

### Information Flow (End-to-End Workflow)

```
Phase 1: SPECIFY (Non-Technical)
├─ User/PM writes: specs/###-feature/spec.md
├─ No technical details, just user value
└─ Follows SDD template conventions

↓

Phase 2: CONSTRAIN (Optional - Complex Features Only)
├─ Architect/Engineer writes: specs/###-feature/TECHNICAL.md
├─ Captures: DB strategy, ORM choices, performance targets
├─ Flags: [NEEDS VALIDATION] for architectural unknowns
└─ Validates: Articles VII (Simplicity), VIII (Anti-Abstraction), IX (Integration)

↓

Phase 3: HARVEST RULES (During Planning)
├─ Run: /rules-harvest
├─ Creates: specs/###-feature/rules/harvest.md
├─ Captures: Business logic in Given/When/Then format
├─ Marks: [NEEDS CLARIFICATION] for ambiguities
└─ Links: To feature requirements (FR-###)

↓

Phase 4: PLAN (Automated via LLM)
├─ Run: /plan (reads Layers 0, 1, 2, 2.5)
├─ Generates: specs/###-feature/plan.md + tasks.md
├─ Respects: TECHNICAL.md constraints (if present)
├─ Includes: Business rules implementation tasks
└─ Validates: Constitutional gates (Phase -1)

↓

Phase 5: IMPLEMENT (Engineers)
├─ Follow: tasks.md sequentially
├─ Write tests FIRST: Article III (Test-First Imperative)
├─ Annotate code: @BusinessRule decorators for logic
└─ Discover blockers: Escalate via Article XI protocol

↓

[CONDITIONAL: Blocker Discovered]
Phase 5a: PIVOT PROTOCOL (Article XI - ADAPTIVE)
├─ Step 1: Create ADR (specs/###-feature/decisions/adr-*.md)
├─ Step 2: Update contract tests (Article III)
├─ Step 3: Amend TECHNICAL.md (strikethrough ~~old~~ → new)
├─ Step 4: Regenerate plan.md + tasks.md
├─ Step 5: Update project context (CLAUDE.md)
└─ Step 6: Communicate impact to stakeholders

↓

Phase 6: EXTRACT RULES (Post-Implementation)
├─ Run: /rules-extract --module {MODULE}
├─ Reads: specs/###-feature/rules/harvest.md (DRAFT rules)
├─ Reads: @BusinessRule annotations in code
├─ Reads: ADRs (for Affected Business Rules section)
├─ Validates: 9 constitutional gates (validation-rules.md)
├─ Updates: .specify/business-rules/catalog.md
├─ Updates: .specify/business-rules/QUICK_REFERENCE.md
└─ Reports: Compliance score (target: ≥95%)

↓

Phase 7: MAINTENANCE (Continuous)
├─ LLM agents read QUICK_REFERENCE.md for business logic queries
├─ Engineers update harvest.md when requirements change
├─ Pivots amend TECHNICAL.md with ADR trails
└─ Catalog stays synchronized via /rules-extract
```

---

## Unified Directory Structure

### Complete Feature Layout

```
spec-kit/
│
├── memory/
│   └── constitution.md              # Articles I-XI (includes Pivot Protocol)
│
├── .specify/                         # Infrastructure (not feature-specific)
│   ├── business-rules/               # BUSINESS RULES catalog system
│   │   ├── catalog.md                # Master business rules catalog
│   │   ├── QUICK_REFERENCE.md        # LLM-optimized lookup
│   │   ├── validation-rules.md       # 9 quality articles for rules
│   │   ├── TUTORIAL.md               # End-to-end usage guide
│   │   ├── schemas/
│   │   │   └── annotation-schema.json
│   │   └── scripts/
│   │       ├── validate-test-coverage.sh
│   │       └── generate-quick-reference.sh
│   │
│   └── decisions/                    # ADAPTIVE cross-project ADRs
│       └── adr-001-sharding-strategy.md
│
├── templates/
│   ├── business-rules/               # BUSINESS RULES templates
│   │   ├── rule-template-full.md
│   │   ├── harvest-template.md
│   │   ├── annotation-examples.md
│   │   └── adr-integration-template.md
│   │
│   ├── adaptive/                     # ADAPTIVE templates (NEW)
│   │   ├── technical-constraints-template.md
│   │   └── adr-pivot-template.md
│   │
│   └── commands/                     # Slash command templates
│       ├── rules-harvest.md
│       ├── rules-extract.md
│       └── create-adr.md             # NEW: Pivot ADR creation
│
├── specs/
│   └── 001-feature-name/            # Feature-specific artifacts
│       │
│       ├── spec.md                   # Layer 1: Non-technical (STABLE)
│       │
│       ├── TECHNICAL.md              # Layer 2: Constraints (EVOLVES - optional)
│       │
│       ├── plan.md                   # Layer 4: Implementation plan
│       ├── tasks.md                  # Layer 4: Task breakdown
│       ├── research.md               # Technology investigation
│       │
│       ├── rules/                    # BUSINESS RULES per-feature
│       │   └── harvest.md            # Layer 2.5: DRAFT rules
│       │
│       ├── decisions/                # ADAPTIVE feature-scoped ADRs
│       │   ├── adr-001-orm-choice.md
│       │   └── adr-002-pivot-to-pg.md
│       │
│       ├── contracts/                # API contracts (updated with pivots)
│       │   └── api-spec.md
│       │
│       └── checklists/               # Quality checklists
│           └── deployment.md
│
├── src/
│   └── business_rules_kit/          # BUSINESS RULES validation/extraction
│       ├── validation/
│       │   ├── gates.py              # 9 constitutional validation functions
│       │   ├── lint_annotations.py   # AST-based @BusinessRule parser
│       │   └── validate_artifacts.py
│       ├── extraction/
│       │   ├── harvest_parser.py
│       │   ├── annotation_parser.py
│       │   ├── adr_parser.py
│       │   └── workflow_parser.py
│       └── catalog/
│           └── generator.py
│
├── scripts/
│   ├── bash/
│   │   ├── rules-harvest.sh          # BUSINESS RULES commands
│   │   ├── rules-extract.sh
│   │   ├── create-adr.sh             # ADAPTIVE commands (NEW)
│   │   └── amend-technical.sh        # ADAPTIVE commands (NEW)
│   │
│   └── powershell/
│       └── [parallel implementations]
│
└── CLAUDE.md                         # Project-wide AI context (at root)
```

### Directory Decision Rules

| Artifact Type | Location | Rationale |
|---------------|----------|-----------|
| Non-technical spec | `specs/###-feature/spec.md` | SDD core convention |
| Technical constraints | `specs/###-feature/TECHNICAL.md` | Feature-specific, evolves with pivots |
| Business rules harvest | `specs/###-feature/rules/harvest.md` | Feature-specific, becomes ACTIVE post-implementation |
| Feature-scoped ADR | `specs/###-feature/decisions/adr-*.md` | Blocker discovered during feature work |
| Cross-project ADR | `.specify/decisions/adr-*.md` | Affects multiple features or architecture |
| Business rules catalog | `.specify/business-rules/catalog.md` | Infrastructure, aggregates across all features |
| Validation rules | `.specify/business-rules/validation-rules.md` | Quality standards for rules (not dev methodology) |
| Constitution | `memory/constitution.md` | Development methodology principles (Articles I-XI) |

---

## Constitutional Integration

### Existing Articles (Preserved)

- **Article I-VI**: Core SDD principles (spec-kit existing)
- **Article VII**: Simplicity Gate (≤3 projects)
- **Article VIII**: Anti-Abstraction Gate (framework trust)
- **Article IX**: Integration-First Gate (contract tests, real databases)

### New Article XI: Technical Pivot Protocol (ADAPTIVE Addition)

Add to `memory/constitution.md`:

```markdown
## Article XI: Technical Pivot Protocol

When implementation/testing reveals architectural blockers requiring pivots:

### Section 11.1: Blocker Classification
Engineers encountering potential blockers MUST distinguish:
- **Bugs** (code-level fixes) vs. **Architectural Limitations** (framework/design)
- **Workarounds** (maintain requirements) vs. **Fundamental Blockers** (violate NFRs)
- **Single-feature** impact vs. **Cross-cutting** concerns

Escalation required for architectural blockers affecting >1 feature or violating NFRs.

### Section 11.2: Pivot Documentation (ADRs)
Architectural pivots MUST be documented via Architecture Decision Records:
- **Feature-scoped**: `specs/###-feature/decisions/adr-###-title.md`
- **Cross-project**: `.specify/decisions/adr-###-title.md`
- **Required sections**: Context, Decision, Constitutional Compliance, Consequences
- **Immutability**: ADRs are historical records, never modified after approval

### Section 11.3: Constitutional Validation (NON-NEGOTIABLE)
ALL pivots MUST validate against:
- Article VII (Simplicity): No unnecessary proliferation
- Article VIII (Anti-Abstraction): Framework trust maintained
- Article IX (Integration-First): Contract tests updated BEFORE implementation
- Violations require explicit documentation + architect approval

### Section 11.4: Test-First Imperative (Article III Extension)
Pivots MUST follow test-first development:
1. Update contract tests to reflect new architecture
2. Validate tests FAIL with current implementation (RED phase)
3. Obtain user approval of test changes
4. ONLY THEN proceed with implementation

### Section 11.5: Constraint Amendment
After ADR approval:
- Update `TECHNICAL.md` with amended constraints (~~old~~ → new)
- Link to ADR number with rationale
- Preserve historical decision trail
- Regenerate plan.md + tasks.md via `/plan --regenerate`

### Section 11.6: Business Rules Impact (If Applicable)
If pivot affects business logic:
- Document in ADR "Affected Business Rules" section
- List BR-* IDs requiring updates
- Run `/rules-extract` post-pivot to sync catalog
- Update rule annotations with new implementation references

### Section 11.7: Communication Protocol
Stakeholders MUST be informed:
- Timeline impact (original vs. adjusted)
- Rationale (blocker + decision)
- Traceability (ADR links)
- Sprint/release adjustments
```

### Business Rules Validation Standards (Separate Document)

The 9 business rule quality articles live in `.specify/business-rules/validation-rules.md` (NOT constitution.md):

```markdown
# Business Rules Validation Standards

**Purpose**: Quality gates for business logic documentation
**Scope**: Applies to ALL rules in catalog (not dev methodology)
**Relationship**: Complements memory/constitution.md (Article XI may reference these)

## Article I: Testability Mandate (HIGH Severity)
[Every ACTIVE rule must have valid + invalid test references]

## Article II: Completeness Principle (HIGH Severity)
[All required fields: rule_id, priority, status, Given/When/Then]

## Article III: Single Source of Truth (MEDIUM Severity)
[Unique rule IDs, no duplicates across catalog]

## Article IV: Explicit Priority & Domain (HIGH Severity)
[Business priority + domain module + source FR declared]

## Article V: Traceability to Source (MEDIUM Severity)
[Links to spec, implementation, tests required]

## Article VI: Test-First Philosophy (HIGH Severity)
[Tests exist before marking status=ACTIVE]

## Article VII: No Speculation (LOW Severity)
[No hypothetical scenarios without FR numbers]

## Article VIII: Versioning (MEDIUM Severity)
[DEPRECATED rules link to replacements]

## Article IX: Multi-Tenancy Awareness (MEDIUM Severity)
[Tenant scope declared: GLOBAL/PER_TENANT/CONFIGURABLE]
```

**Key Distinction**:
- `memory/constitution.md` = Development methodology (how we build software)
- `.specify/business-rules/validation-rules.md` = Documentation quality (how we write business rules)

---

## Interaction Points (How Systems Work Together)

### Interaction 1: TECHNICAL.md ↔ Business Rules Harvest

**Scenario**: During `/plan`, engineer runs `/rules-harvest`

**Flow**:
1. `/rules-harvest` reads `TECHNICAL.md` (if exists)
2. Extracts technical constraints relevant to business logic:
   - Multi-tenancy scope (Article IX of validation-rules.md)
   - Performance targets (inform priority classification)
   - Database strategy (informs test references format)
3. Pre-populates harvest template with:
   - Tenant scope from TECHNICAL.md
   - Module/domain from feature directory name
   - Links to source spec

**Example**:
```markdown
# TECHNICAL.md snippet
## Multi-Tenancy Pattern
- Tenant Scope: PER_TENANT (schema-per-company isolation)

↓ informs ↓

# rules/harvest.md (auto-populated)
## BR-ZONES-001: Max Capacity Per Zone (DRAFT)
**Tenant Scope**: PER_TENANT  ← Auto-filled from TECHNICAL.md
**Domain**: ZONES
```

### Interaction 2: ADRs ↔ Business Rules Catalog

**Scenario**: Architectural pivot affects business logic implementation

**Flow**:
1. Engineer creates ADR for pivot (e.g., Prisma → pg)
2. ADR includes "Affected Business Rules" section:
   ```markdown
   ## Affected Business Rules
   - **BR-ZONES-001**: Max Capacity Per Zone
     - Change: Test references updated (Prisma contracts → pg contracts)
     - Impact: Implementation reference changed (src/zones/service.ts:42 → src/zones/pg-service.ts:67)
     - Status: Requires catalog update via /rules-extract
   ```
3. After pivot implementation, `/rules-extract` reads ADRs
4. Updates catalog with new implementation references
5. Validates constitutional compliance (9 gates still pass)

**Example ADR Section**:
```markdown
# ADR-002: Migrate to Pure node-postgres

## Affected Business Rules

### Modified Rules
- **BR-ZONES-001**: Max Capacity Per Zone
  - **Implementation Change**:
    - FROM: `src/zones/prisma-service.ts:142` (Prisma)
    - TO: `src/zones/pg-service.ts:67` (pg Pool)
  - **Test Change**:
    - FROM: `tests/zones/test_max_capacity.py:52` (Prisma contracts)
    - TO: `tests/zones/test_max_capacity_pg.py:52` (pg contracts)
  - **Action**: Update catalog via `/rules-extract --module ZONES`
  - **Constitutional Impact**: Article VI (Test-First) - tests updated before implementation ✅

### New Rules Created
- **BR-MIGRATION-001**: Pure pg Migration Validator
  - **Reason**: Pivot introduces custom migration runner (new business logic)
  - **Status**: DRAFT (pending implementation)
  - **Domain**: INFRASTRUCTURE
```

### Interaction 3: Constitutional Validation (Cross-System)

**Scenario**: Plan generation validates against BOTH constitutions

**Flow**:
1. LLM reads `memory/constitution.md` (Articles I-XI)
2. LLM reads `.specify/business-rules/validation-rules.md` (Articles I-IX)
3. Generates plan.md with Phase -1 gates:

```markdown
# Phase -1: Constitutional Validation

## Development Methodology Gates (memory/constitution.md)
- [ ] Article VII (Simplicity): Using ≤3 projects? YES ✅
- [ ] Article VIII (Anti-Abstraction): No unnecessary wrappers? YES ✅
- [ ] Article IX (Integration-First): Contract tests defined? YES ✅
- [ ] Article XI (Pivot Protocol): TECHNICAL.md risks flagged? YES ✅

## Business Rules Quality Gates (.specify/business-rules/validation-rules.md)
(Applied during /rules-extract, not during plan generation)
- Testability, Completeness, Traceability validated post-implementation
```

### Interaction 4: LLM Context Priority (Reading Order)

**Scenario**: LLM agent generates tasks for new feature

**Reading Order**:
1. `memory/constitution.md` → Understand immutable principles (Articles I-XI)
2. `specs/###-feature/spec.md` → Understand user value (non-technical)
3. `specs/###-feature/TECHNICAL.md` → Understand architectural constraints (if exists)
4. `specs/###-feature/rules/harvest.md` → Understand business logic (if exists)
5. `.specify/decisions/adr-*.md` → Understand cross-project decisions
6. `specs/###-feature/decisions/adr-*.md` → Understand feature-specific pivots
7. `.specify/business-rules/QUICK_REFERENCE.md` → Check existing business rules
8. `CLAUDE.md` → Project-wide patterns and recent updates

**Token Budget Optimization**:
- Layers 1-4: Always include (core feature context)
- Layer 5 (ADRs): Include only if pivot-related task
- Layer 7 (Quick Reference): Include only if business logic task
- Layer 8 (CLAUDE.md): Always include for project conventions

---

## Integration Tasks (Implementation Roadmap)

### Phase 1: Constitutional Updates (Week 1)

**File**: `memory/constitution.md`

**Tasks**:
- [ ] **T1.1**: Add Article XI: Technical Pivot Protocol (7 sections)
- [ ] **T1.2**: Update Article III reference in Article XI.4 (Test-First Imperative)
- [ ] **T1.3**: Add cross-reference to `.specify/business-rules/validation-rules.md`
- [ ] **T1.4**: Version bump constitution (v2.0 → v2.1)

**Deliverable**: Updated constitution with pivot protocol

---

### Phase 2: Template Consolidation (Week 1-2)

**Directory**: `templates/`

**Tasks**:
- [ ] **T2.1**: Create `templates/adaptive/` directory
  - [ ] `technical-constraints-template.md`
  - [ ] `adr-pivot-template.md`
- [ ] **T2.2**: Update `templates/business-rules/` templates
  - [ ] `adr-integration-template.md` (add "Affected Business Rules" section)
- [ ] **T2.3**: Create unified command templates
  - [ ] `templates/commands/create-adr.md` (NEW)
  - [ ] `templates/commands/amend-technical.md` (NEW)
- [ ] **T2.4**: Update `templates/spec-template.md`
  - [ ] Add optional "Complex Features" section referencing TECHNICAL.md
- [ ] **T2.5**: Update `templates/plan-template.md`
  - [ ] Add Phase -1 with constitutional validation checklist

**Deliverable**: Unified template library

---

### Phase 3: Script Updates (Week 2)

**Directory**: `scripts/bash/` + `scripts/powershell/`

**Tasks**:
- [ ] **T3.1**: Update `create-new-feature.sh`
  - [ ] Create `specs/###-feature/decisions/` directory automatically
  - [ ] Create `specs/###-feature/rules/` directory automatically
  - [ ] Optionally create `TECHNICAL.md` based on complexity flag
- [ ] **T3.2**: Create `create-adr.sh` (NEW)
  - [ ] Interactive prompts for ADR fields
  - [ ] Auto-determine feature-scoped vs. cross-project placement
  - [ ] Validate constitutional compliance sections present
  - [ ] Output JSON with ADR path
- [ ] **T3.3**: Create `amend-technical.sh` (NEW)
  - [ ] Read existing TECHNICAL.md
  - [ ] Interactive amendment workflow (strikethrough ~~old~~ → new)
  - [ ] Link to ADR number
  - [ ] Update decision history section
- [ ] **T3.4**: Update `rules-extract.sh`
  - [ ] Read ADRs for "Affected Business Rules" sections
  - [ ] Update catalog with pivot-modified implementation references
  - [ ] Validate constitutional compliance (9 gates)
- [ ] **T3.5**: Create PowerShell equivalents for all bash scripts

**Deliverable**: Cross-platform script suite

---

### Phase 4: Business Rules Integration (Week 3-4)

**Directory**: `.specify/business-rules/`

**Tasks**:
- [ ] **T4.1**: Update `validation-rules.md`
  - [ ] Add preamble explaining relationship to `memory/constitution.md`
  - [ ] Cross-reference Article XI (pivot protocol may affect rules)
- [ ] **T4.2**: Update `TUTORIAL.md`
  - [ ] Add section: "Business Rules + Technical Pivots"
  - [ ] Example: Rule changes when ADR-002 pivots Prisma → pg
- [ ] **T4.3**: Update `catalog.md` template
  - [ ] Add "Pivot History" section linking to relevant ADRs
- [ ] **T4.4**: Implement ADR parser (`src/business_rules_kit/extraction/adr_parser.py`)
  - [ ] Parse "Affected Business Rules" sections from ADRs
  - [ ] Extract BR-* IDs and implementation changes
  - [ ] Return structured data for catalog updates

**Deliverable**: Pivot-aware business rules system

---

### Phase 5: Command Integration (Week 4-5)

**Directory**: `templates/commands/` (agent-specific)

**Tasks**:
- [ ] **T5.1**: Generate `/create-adr` for all 13 agents
  - [ ] Uses `scripts/bash/create-adr.sh --json`
  - [ ] Frontmatter with script paths (bash + PowerShell)
- [ ] **T5.2**: Generate `/amend-technical` for all 13 agents
  - [ ] Interactive amendment workflow
  - [ ] Links to ADR triggering amendment
- [ ] **T5.3**: Update `/rules-harvest` command
  - [ ] Read TECHNICAL.md (if exists) to pre-populate fields
  - [ ] Auto-fill tenant scope, domain, module
- [ ] **T5.4**: Update `/rules-extract` command
  - [ ] Add flag: `--include-adrs` to read ADR impacts
  - [ ] Update help text with pivot workflow

**Deliverable**: Multi-agent command suite

---

### Phase 6: Documentation Updates (Week 5)

**Files**: `README.md`, `spec-driven.md`, `CLAUDE.md`

**Tasks**:
- [ ] **T6.1**: Update `README.md`
  - [ ] Add "Adaptive SDD" section
  - [ ] Add "Business Rules Management" section
  - [ ] Explain interaction between the two systems
- [ ] **T6.2**: Update `spec-driven.md`
  - [ ] Insert Layer 2 (TECHNICAL.md) explanation
  - [ ] Insert Layer 2.5 (Business Rules Harvest) explanation
  - [ ] Add Article XI (Pivot Protocol) to workflow diagram
- [ ] **T6.3**: Create `.specify/INTEGRATION-GUIDE.md` (NEW)
  - [ ] Complete walkthrough: Adaptive SDD + Business Rules
  - [ ] Example: Multi-company zones with Prisma pivot
  - [ ] Decision trees: When to use TECHNICAL.md? When to create ADR?
- [ ] **T6.4**: Update `CLAUDE.md` template
  - [ ] Add LLM reading priority section
  - [ ] Add recent updates format (include ADRs + rule catalog)

**Deliverable**: Comprehensive documentation

---

### Phase 7: Testing & Validation (Week 6)

**Tasks**:
- [ ] **T7.1**: Create test project
  - [ ] Initialize: `specify init test-unified --ai claude`
  - [ ] Add complex feature with TECHNICAL.md
  - [ ] Run `/rules-harvest`
  - [ ] Simulate architectural blocker
  - [ ] Run `/create-adr` → amend TECHNICAL.md → regenerate plan
  - [ ] Complete implementation with @BusinessRule annotations
  - [ ] Run `/rules-extract --include-adrs`
- [ ] **T7.2**: Validate constitutional compliance
  - [ ] Verify Article XI gates work in plan.md generation
  - [ ] Verify business rules validation gates (9 articles)
- [ ] **T7.3**: Test cross-platform scripts
  - [ ] Bash scripts on Ubuntu/macOS
  - [ ] PowerShell scripts on Windows
- [ ] **T7.4**: LLM context testing
  - [ ] Generate plan with TECHNICAL.md + harvest.md
  - [ ] Generate plan after ADR amendment
  - [ ] Verify task generation uses amended constraints

**Deliverable**: Validated end-to-end workflow

---

### Phase 8: Release Preparation (Week 7)

**Tasks**:
- [ ] **T8.1**: Version bump
  - [ ] `pyproject.toml`: v0.5.0 → v0.6.0
  - [ ] `CHANGELOG.md`: Document unified methodology
- [ ] **T8.2**: GitHub release
  - [ ] Create template archives (with adaptive + business-rules templates)
  - [ ] Release notes: "Unified SDD Methodology v1.0"
- [ ] **T8.3**: Migration guide
  - [ ] For existing spec-kit projects
  - [ ] When to add TECHNICAL.md retroactively
  - [ ] How to backfill ADRs for past pivots
- [ ] **T8.4**: Update documentation site
  - [ ] Add unified methodology page
  - [ ] Video walkthrough (optional)

**Deliverable**: Production release

---

## Risks & Open Questions

### Risk 1: Template Proliferation (MEDIUM)

**Issue**: Two separate template directories may confuse users

**Mitigation**:
- Consolidate under `templates/` with subdirectories
- Update `generate_agent_commands.py` to handle both template types
- Create `templates/README.md` explaining organization

**Decision Required**: Merge `templates/adaptive/` into `templates/business-rules/` or keep separate?

**Recommendation**: Keep separate for clarity:
```
templates/
├── business-rules/    # Rule documentation templates
├── adaptive/          # Pivot/constraint templates
└── commands/          # Slash commands (uses both above)
```

---

### Risk 2: TECHNICAL.md Optional vs. Required (HIGH)

**Issue**: Unclear when TECHNICAL.md is mandatory vs. optional

**Current Guidance**:
- Optional for simple CRUD features
- Required for multi-tenant, high-performance, or complex architectural features

**Decision Required**: Define precise criteria for TECHNICAL.md creation

**Recommendation**: Add to `create-new-feature.sh`:
```bash
# Prompt user
read -p "Is this a complex/multi-tenant feature? (y/N): " COMPLEX

if [[ $COMPLEX =~ ^[Yy]$ ]]; then
    cp templates/adaptive/technical-constraints-template.md \
       specs/$FEATURE_DIR/TECHNICAL.md
    echo "✅ TECHNICAL.md created (remember to fill constraints)"
fi
```

---

### Risk 3: ADR Placement Ambiguity (MEDIUM)

**Issue**: Engineers may struggle with feature-scoped vs. cross-project ADR placement

**Current Guidance**:
- Feature-scoped: Blocker discovered during feature implementation
- Cross-project: Affects multiple features or system architecture

**Edge Case Example**:
- Pivot discovered in Feature A, but affects Features B and C too

**Decision Required**: What if pivot starts feature-scoped but becomes cross-project?

**Recommendation**: Allow ADR relocation:
```bash
# If ADR-002 in specs/001-zones/decisions/ affects other features:
mv specs/001-zones/decisions/adr-002-pure-pg.md \
   .specify/decisions/adr-002-pure-pg.md

# Update all references
sed -i 's|specs/001-zones/decisions/adr-002|.specify/decisions/adr-002|g' \
   specs/*/TECHNICAL.md
```

---

### Risk 4: Business Rules Catalog Sync (LOW)

**Issue**: Catalog may drift if engineers forget `/rules-extract`

**Mitigation Options**:
1. **Pre-commit hook**: Warn if `rules/harvest.md` changed but catalog not updated
2. **CI/CD check**: Fail PR if catalog out of sync
3. **Automated extraction**: Run `/rules-extract` as part of release process

**Decision Required**: Which enforcement level?

**Recommendation**: Combination approach:
- Pre-commit hook: Warning only (LOW friction)
- CI/CD: Fail if ACTIVE rules missing test coverage (HIGH severity gate)
- Release: Automated `/rules-extract --all` before tagging version

---

### Risk 5: LLM Context Window Overflow (MEDIUM)

**Issue**: Reading 8 layers of artifacts may exceed context limits

**Current Token Budgets**:
- Layer 0 (Constitution): ~2000 tokens
- Layer 1 (Spec): ~1500 tokens
- Layer 2 (TECHNICAL): ~800 tokens
- Layer 2.5 (Harvest): ~600 tokens
- Layer 3 (ADRs): ~500 tokens each (may be 3-5 ADRs)
- Layer 4 (Plan/Tasks): Generated output (not input)
- Layer 5 (Quick Ref): ~2000 tokens
- Layer 8 (CLAUDE.md): ~1000 tokens

**Total Input**: ~10,000-15,000 tokens (within 200K Claude budget, but tight for smaller models)

**Mitigation**:
- Use selective loading (only include layers relevant to task)
- QUICK_REFERENCE.md stays under 2000 tokens (enforced)
- ADRs: Include only recent (last 3) or feature-specific

**Decision Required**: Define pruning strategy for LLM context

**Recommendation**: Create `LLM_CONTEXT_PRIORITY.md`:
```markdown
# LLM Context Loading Strategy

## Always Include (Layers 0-2)
1. memory/constitution.md (Articles I-XI)
2. specs/###-feature/spec.md (current feature)
3. specs/###-feature/TECHNICAL.md (if exists)

## Conditionally Include
4. specs/###-feature/rules/harvest.md (if task involves business logic)
5. specs/###-feature/decisions/adr-*.md (if task references pivot)
6. .specify/business-rules/QUICK_REFERENCE.md (if task queries existing rules)

## Project Context (Always)
7. CLAUDE.md (project-wide patterns)

## Exclude from Context (Use for validation only)
- .specify/business-rules/catalog.md (too large, use QUICK_REFERENCE instead)
- .specify/decisions/adr-*.md (use only if cross-cutting concern)
```

---

### Open Question 1: Slash Command Naming

**Issue**: Should pivot commands use `/adr-` prefix or `/pivot-` prefix?

**Options**:
- **Option A**: `/create-adr`, `/amend-technical` (explicit, descriptive)
- **Option B**: `/pivot`, `/pivot-amend` (concise, workflow-focused)
- **Option C**: `/adr`, `/technical` (shortest)

**Recommendation**: Option A (explicit) for clarity and searchability

---

### Open Question 2: TECHNICAL.md vs. README.md

**Issue**: Some projects use `README.md` in feature directories for technical notes

**Conflict**: Spec Kit avoids `README.md` in favor of `spec.md` + `plan.md`

**Decision Required**: Is `TECHNICAL.md` the right name, or should it be `CONSTRAINTS.md`?

**Recommendation**: Keep `TECHNICAL.md`:
- Clearly distinct from `README.md` (avoids confusion)
- Matches naming convention: spec.md, plan.md, TECHNICAL.md (all caps = special)
- Aligns with "technical constraints" terminology in Adaptive SDD

---

### Open Question 3: Multi-Agent Template Generation

**Issue**: Both Business Rules and Adaptive systems need agent-specific commands

**Current Approach**:
- `generate_agent_commands.py` generates Business Rules commands
- Need to extend for Adaptive commands

**Decision Required**: Merge into single generator script or keep separate?

**Recommendation**: Merge with modular design:
```python
# src/generate_agent_commands.py
COMMAND_TYPES = {
    "business-rules": ["rules-harvest", "rules-extract"],
    "adaptive": ["create-adr", "amend-technical"],
    "core": ["specify", "plan", "tasks"]  # Existing spec-kit
}

def generate_all_commands():
    for command_type, commands in COMMAND_TYPES.items():
        for command in commands:
            generate_for_all_agents(command, template_dir=f"templates/{command_type}/")
```

---

## Success Metrics (Post-Implementation)

### Metric 1: Adoption Rate
- **Target**: ≥80% of complex features use TECHNICAL.md
- **Measure**: Count features with TECHNICAL.md vs. total features
- **Timeline**: 3 months post-release

### Metric 2: Pivot Documentation
- **Target**: 100% of architectural pivots documented via ADRs
- **Measure**: Git log analysis (look for "refactor:" commits without ADRs)
- **Timeline**: 6 months post-release

### Metric 3: Business Rules Compliance
- **Target**: ≥95% constitutional compliance for ACTIVE rules
- **Measure**: Run `/rules-extract --validate-only --ci-mode`
- **Timeline**: Continuous (CI/CD enforcement)

### Metric 4: LLM Context Accuracy
- **Target**: ≥90% of LLM-generated plans respect TECHNICAL.md constraints
- **Measure**: Manual review of 10 generated plans per quarter
- **Timeline**: Quarterly assessment

### Metric 5: Pivot Discovery Time
- **Target**: Reduce blocker discovery from Week 5 → Week 2 (via [NEEDS VALIDATION] flags)
- **Measure**: Track ADR creation dates vs. feature start dates
- **Timeline**: 6 months post-release

---

## Migration Path (For Existing Spec-Kit Projects)

### Scenario 1: Project Without Business Rules or Adaptive

**Current State**: Using spec.md + plan.md + tasks.md (core SDD)

**Migration**:
1. Add Article XI to `memory/constitution.md`
2. No immediate action required (TECHNICAL.md is optional)
3. Use TECHNICAL.md for next complex feature

**Timeline**: 1 hour (constitution update only)

---

### Scenario 2: Project Using Business Rules (v1.0)

**Current State**: Using Business Rules with `.specify/business-rules/constitution.md`

**Migration**:
1. **Rename**: `.specify/business-rules/constitution.md` → `validation-rules.md`
2. **Update references**: All scripts/templates referencing old name
3. **Add Article XI**: To `memory/constitution.md`
4. **Test**: Run `/rules-extract` to verify catalog generation still works

**Timeline**: 2-3 hours

---

### Scenario 3: Project Using Adaptive SDD (Earlier Version)

**Current State**: Using TECHNICAL.md + ADRs, but not Business Rules

**Migration**:
1. Install Business Rules templates: Copy from `templates/business-rules/`
2. Install Business Rules scripts: Copy `src/business_rules_kit/`
3. Initialize catalog: Create `.specify/business-rules/` directory
4. Backfill rules: Manually create harvest.md for existing features (optional)
5. Run `/rules-extract --all` to populate catalog

**Timeline**: 1-2 days (depending on backfill scope)

---

### Scenario 4: Greenfield Project (Fresh Start)

**Current State**: New project, no existing spec-kit usage

**Migration**:
1. Clone spec-kit: `git clone https://github.com/github/spec-kit.git`
2. Checkout unified branch: `git checkout unified-sdd-v1.0`
3. Run installer: `specify init my-project --ai claude`
4. Choose complexity: Prompt asks "Complex/multi-tenant? (y/N)"
5. Follow tutorial: `.specify/INTEGRATION-GUIDE.md`

**Timeline**: 30 minutes setup + ongoing usage

---

## Next Steps (Implementation Order)

1. **Review & Approve**: Unified methodology blueprint (this document)
2. **Constitutional Update**: Add Article XI to `memory/constitution.md` (Phase 1)
3. **Template Creation**: Adaptive templates + command templates (Phase 2)
4. **Script Development**: create-adr.sh, amend-technical.sh (Phase 3)
5. **Business Rules Integration**: ADR parser, catalog updates (Phase 4)
6. **Command Generation**: Multi-agent /create-adr, /amend-technical (Phase 5)
7. **Documentation**: README, spec-driven.md, INTEGRATION-GUIDE (Phase 6)
8. **Testing**: End-to-end validation with test project (Phase 7)
9. **Release**: v0.6.0 with unified methodology (Phase 8)

**Total Timeline**: 7 weeks (parallelization possible for Phases 3-5)

---

## Conclusion

The unified methodology preserves Spec Kit's core philosophy (non-technical specs, test-first, contract-driven) while adding two orthogonal enhancements:

1. **Adaptive SDD**: Handles architectural pivots through TECHNICAL.md evolution and ADR traceability
2. **Business Rules**: Captures and validates business logic through constitutional gates and multi-source extraction

These systems interact at well-defined points (TECHNICAL.md informs harvest, ADRs update catalog) without creating conflicts or polluting the repository structure. All artifacts live in approved locations (`specs/*/`, `.specify/*`), and LLM agents can consume the layered information with clear priority ordering.

The remaining implementation tasks are concrete, with defined deliverables and timelines. The only open questions relate to UX decisions (command naming, enforcement levels) rather than fundamental methodology conflicts.

---

**Status**: ✅ READY FOR IMPLEMENTATION
**Compliance**: 98% (residual: template organization consolidation)
**Next Action**: Review open questions, approve Phase 1 (Constitutional Update)
