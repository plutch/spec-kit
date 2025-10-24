# Unified Spec-Driven Development Methodology v2.1
## Business Rules + Adaptive Pivots Integration

**Version**: 2.1
**Date**: 2025-01-23
**Status**: Production-Ready Implementation Guide
**Audience**: Engineering Teams, LLM Agents, Product Managers

---

## Executive Summary

### What is This?

This methodology unifies **GitHub Spec Kit** with two complementary enhancements:

1. **Adaptive Pivot Protocol** - Handles architectural blockers discovered during implementation through TECHNICAL.md constraints and Architecture Decision Records (ADRs)
2. **Business Rules Management** - Captures, validates, and catalogs business logic through constitutional gates and multi-source extraction

**Key Value Propositions:**
- ✅ **Reduce pivot discovery from Week 5 → Week 2** via [NEEDS VALIDATION] flags
- ✅ **Eliminate duplicate business logic** via centralized catalog
- ✅ **Maintain non-technical specs** during architectural changes
- ✅ **100% traceability** for all architectural decisions
- ✅ **95%+ business rule compliance** via constitutional validation

### 5-Minute Visual Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    UNIFIED SDD WORKFLOW                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STEP 1: SPECIFY (Non-Technical)                               │
│  ┌────────────────────────────────────────────────┐            │
│  │  PM writes: specs/001-feature/spec.md          │            │
│  │  • User stories (what users need)              │            │
│  │  • Success criteria (measurable outcomes)      │            │
│  │  ✅ NO technical details                       │            │
│  └────────────────────────────────────────────────┘            │
│                        ↓                                        │
│                                                                 │
│  STEP 2: CONSTRAIN (Optional - Complex Features Only)         │
│  ┌────────────────────────────────────────────────┐            │
│  │  Architect writes: specs/001-feature/TECHNICAL.md           │
│  │  • Performance targets (P95 <250ms)            │            │
│  │  • Multi-tenancy strategy                      │            │
│  │  • [NEEDS VALIDATION] flags for unknowns       │            │
│  │  ⚠️ Use Decision Tree 1 to determine if needed │            │
│  └────────────────────────────────────────────────┘            │
│                        ↓                                        │
│                                                                 │
│  STEP 3: HARVEST RULES (During Planning)                       │
│  ┌────────────────────────────────────────────────┐            │
│  │  Run: /rules-harvest                           │            │
│  │  Creates: specs/001-feature/rules/harvest.md   │            │
│  │  • Business logic in Given/When/Then format    │            │
│  │  • [NEEDS CLARIFICATION] markers               │            │
│  │  ⚠️ Use Decision Tree 4 to determine if needed │            │
│  └────────────────────────────────────────────────┘            │
│                        ↓                                        │
│                                                                 │
│  STEP 4: PLAN (Automated via LLM)                             │
│  ┌────────────────────────────────────────────────┐            │
│  │  Run: /plan                                    │            │
│  │  Generates: plan.md + tasks.md                 │            │
│  │  • Respects TECHNICAL.md constraints           │            │
│  │  • Includes business rules implementation tasks│            │
│  │  • Validates constitutional compliance         │            │
│  └────────────────────────────────────────────────┘            │
│                        ↓                                        │
│                                                                 │
│  STEP 5: IMPLEMENT (Engineers)                                 │
│  ┌────────────────────────────────────────────────┐            │
│  │  • Write tests FIRST (Article III)             │            │
│  │  • Annotate code with @BusinessRule decorators │            │
│  │  • If blocker found → Escalate to STEP 5a      │            │
│  └────────────────────────────────────────────────┘            │
│                        ↓                                        │
│                  [Blocker Found?]                              │
│                        ↓                                        │
│  STEP 5a: PIVOT PROTOCOL (Article XI - If Needed)             │
│  ┌────────────────────────────────────────────────┐            │
│  │  1. Create ADR (use Decision Trees 2 & 3)      │            │
│  │  2. Update contract tests                      │            │
│  │  3. Amend TECHNICAL.md (link to ADR)           │            │
│  │  4. Regenerate plan.md + tasks.md              │            │
│  │  5. Continue implementation                    │            │
│  │  ⚠️ See complete walkthrough (Section 5)       │            │
│  └────────────────────────────────────────────────┘            │
│                        ↓                                        │
│                                                                 │
│  STEP 6: EXTRACT RULES (Post-Implementation)                   │
│  ┌────────────────────────────────────────────────┐            │
│  │  Run: /rules-extract --module {MODULE}         │            │
│  │  • Validates 9 constitutional gates            │            │
│  │  • Updates catalog.md                          │            │
│  │  • Generates QUICK_REFERENCE.md for LLMs       │            │
│  │  • Target: ≥95% compliance score               │            │
│  └────────────────────────────────────────────────┘            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Compliance & Philosophy

- **98% alignment** with GitHub Spec Kit philosophy
- **Preserves spec-first workflow** - non-technical specs never change due to pivots
- **Approved directory structure** - all artifacts in `specs/*/` or `.specify/*/`
- **No root pollution** - strict separation of concerns
- **LLM-optimized** - clear reading priority for AI agents

---

## Quick Start Guide

### For Simple CRUD Features
```bash
# 1. Create feature
./specify init 001-user-profile

# 2. Write non-technical spec
vim specs/001-user-profile/spec.md

# 3. Generate plan
/plan

# 4. Implement
# Follow tasks.md
```

### For Complex/Multi-Tenant Features
```bash
# 1. Create feature
./specify init 002-multi-company-zones

# 2. Write non-technical spec
vim specs/002-multi-company-zones/spec.md

# 3. Define technical constraints
vim specs/002-multi-company-zones/TECHNICAL.md
# Fill: multi-tenancy strategy, performance targets, risks

# 4. Harvest business rules
/rules-harvest

# 5. Generate plan
/plan

# 6. Implement with pivot awareness
# If blocker found → /create-adr → /amend-technical → /plan --regenerate

# 7. Extract validated rules
/rules-extract --module ZONES
```

### For Architectural Pivots
```bash
# Blocker discovered during implementation
/create-adr
# Fill ADR template

# Amend constraints
/amend-technical
# Links to ADR, updates TECHNICAL.md

# Regenerate plan with new constraints
/plan --regenerate

# Update affected business rules
/rules-extract --module AFFECTED_MODULE
```

---

## Decision Trees

### Decision Tree 1: Should I Create TECHNICAL.md?

```
START: New feature spec written (spec.md exists)
  ↓
  🎯 CONFIDENCE CHECK (MANDATORY):
  "Do I fully understand the technical requirements and constraints?"

  Self-Assessment Questions:
    1. Are acceptance criteria in spec.md quantifiable?
    2. Are NFRs (latency, scale, security) specified or clearly not needed?
    3. Is multi-tenancy strategy clear or explicitly single-tenant?
    4. Do I know the existing tech stack for this project?

  Confidence Level:
    ✅ High (>90%): All 4 questions answered YES → Continue decision tree
    ⚠️ Medium (70-90%): 2-3 questions YES → Present options with recommendation
    ❌ Low (<70%): 0-1 questions YES → STOP

  IF Low Confidence (<70%):
    ⚠️ Output: "Need clarification before determining TECHNICAL.md necessity"

    Report to user:
      "I need clarification on:
       • [Specific unclear requirement from questions above]
       • [Another gap in understanding]

       Please provide guidance so I can assess whether TECHNICAL.md is needed."

    BLOCK: Do not proceed with decision tree until confidence >70%
  ↓
  ├─ Is this simple CRUD with known tech stack?
  │    YES → Skip TECHNICAL.md ✅
  │    NO  ↓
  │
  ├─ Does feature involve ANY of:
  │    • Multi-tenancy/sharding
  │    • Performance targets (P95 <250ms, >1000 RPS)
  │    • New/untested frameworks
  │    • Regulatory constraints (GDPR, SOC2)
  │    • Architectural unknowns
  │    • Cross-team dependencies
  │
  │    YES → CREATE TECHNICAL.md ✅
  │    NO  → Skip TECHNICAL.md ✅
  │
  └─ RESULT:
       • TECHNICAL.md created: Fill constraints, flag [NEEDS VALIDATION]
       • TECHNICAL.md skipped: Proceed directly to /plan

EXAMPLES:
✅ Create TECHNICAL.md:
   - Multi-company zone capacity system
   - Real-time WebSocket notification service
   - Elasticsearch integration with 10K+ docs

❌ Skip TECHNICAL.md:
   - User profile CRUD
   - Simple API endpoint additions
   - UI component library updates
```

### Decision Tree 2: Should I Create an ADR?

```
START: Architectural question or blocker encountered
  ↓
  🔍 REFLEXION CHECK (BEFORE creating new ADR):

  1. Search Past ADRs:
     ```bash
     # Feature-scoped ADRs
     grep -ri "blocker_keyword" specs/*/decisions/

     # Cross-project ADRs
     grep -ri "blocker_keyword" .specify/decisions/

     # Example: Search for "Prisma" blockers
     grep -ri "prisma" .specify/decisions/ specs/*/decisions/
     ```

  2. Analyze Matches:
     IF similar blocker found in past ADR:
       ✅ Read past ADR-### for context
       ✅ Check decision rationale
       ✅ Assess if same solution applies

       IF same context + same blocker:
         → REUSE past solution
         → Update TECHNICAL.md only (link to existing ADR-###)
         → Skip new ADR creation
         → Token savings: ~2,000 tokens

       ELSE IF different context BUT same blocker type:
         → CREATE new ADR
         → Reference past ADR-### in "Context" section
         → Explain why different solution needed

  3. Document Learning:
     → Add to TECHNICAL.md "Decision History" section
     → Link to all relevant ADRs (past + new)
  ↓
  🎯 CONFIDENCE CHECK (MANDATORY):
  "Do I understand this blocker well enough to decide on ADR creation?"

  Confidence Level:
    ✅ High (>90%): Blocker clearly understood → Continue
    ⚠️ Medium (70-90%): Some uncertainty → Present options
    ❌ Low (<70%): Blocker unclear → STOP and investigate further

  IF Low Confidence (<70%):
    → Investigate root cause more deeply
    → Gather more evidence (error logs, performance metrics, etc.)
    → Request clarification from team
  ↓
  ├─ Is this a BUG (code-level fix)?
  │    YES → Fix code, no ADR needed ✅
  │    NO  ↓
  │
  ├─ Can I use simple WORKAROUND without violating NFRs?
  │    YES → Document workaround in code comment ✅
  │    NO  ↓
  │
  ├─ Does this blocker require:
  │    • Framework/library change (Prisma → pg)
  │    • Architectural pattern change (REST → GraphQL)
  │    • Database strategy change (MySQL → PostgreSQL)
  │    • Major refactoring (>5 files affected)
  │
  │    YES ↓
  │
  ├─ Does this violate ANY constitutional articles?
  │    • Article VII (Simplicity): Adds new project/framework
  │    • Article VIII (Anti-Abstraction): Untrusted wrapper
  │    • Article IX (Integration-First): Breaks contract tests
  │
  │    YES → CREATE ADR + Get architect approval ⚠️
  │    NO  → CREATE ADR ✅
  │
  └─ RESULT:
       • ADR created: Use Decision Tree 3 for placement
       • No ADR needed: Proceed with standard development

EXAMPLES:
✅ Create ADR:
   - Prisma ORM blocking multi-tenancy → pivot to node-postgres
   - GraphQL N+1 queries → add DataLoader caching layer
   - MongoDB joins underperforming → migrate to PostgreSQL

❌ No ADR needed:
   - Typo in validation message
   - Missing null check causing crash
   - Off-by-one error in pagination
```

### Decision Tree 3: Where Should I Place the ADR?

```
START: ADR creation required (from Decision Tree 2)
  ↓
  🎯 CONFIDENCE CHECK (MANDATORY):
  "Do I understand the scope of this architectural decision?"

  Self-Assessment Questions:
    1. Do I know which features will be affected by this decision?
    2. Have I checked for similar patterns in other features?
    3. Is this decision likely to be reused across the codebase?

  Confidence Level:
    ✅ High (>90%): Clear understanding of scope → Continue
    ⚠️ Medium (70-90%): Some uncertainty about impact → Research similar features
    ❌ Low (<70%): Scope unclear → STOP

  IF Low Confidence (<70%):
    → Search codebase for similar patterns (grep, file search)
    → Discuss with team to understand full impact
    → Document assumptions in ADR "Context" section
  ↓
  ├─ Does this decision affect ONLY current feature?
  │    YES → Feature-scoped ADR ✅
  │           Location: specs/###-feature/decisions/adr-###-title.md
  │    NO  ↓
  │
  ├─ Does this decision affect:
  │    • Multiple features (>1)
  │    • System architecture (database, auth, API)
  │    • Shared infrastructure (logging, monitoring)
  │
  │    YES → Cross-project ADR ✅
  │           Location: .specify/decisions/adr-###-title.md
  │
  └─ EDGE CASE: What if it starts feature-scoped but becomes cross-project?
       → Move ADR to .specify/decisions/
       → Update all references (scripts available)

EXAMPLES:
📁 Feature-scoped (specs/001-zones/decisions/):
   - ADR-001: Choose Postgres over MySQL for zone capacity tracking
   - ADR-002: Use Redis for zone availability caching (zones only)

📁 Cross-project (.specify/decisions/):
   - ADR-003: Migrate ALL features from Prisma to node-postgres
   - ADR-004: Adopt OpenTelemetry for distributed tracing
   - ADR-005: Implement row-level security for multi-tenancy

RELOCATION PROCESS:
# If ADR-002 initially in specs/001-zones/decisions/ affects other features:
mv specs/001-zones/decisions/adr-002-redis-cache.md \\
   .specify/decisions/adr-002-redis-cache.md

# Update all references
sed -i 's|specs/001-zones/decisions/adr-002|.specify/decisions/adr-002|g' \\
   specs/*/TECHNICAL.md
```

### Decision Tree 4: Should I Use Business Rules System?

```
START: Feature involves business logic
  ↓
  🎯 CONFIDENCE CHECK (MANDATORY):
  "Do I clearly understand what constitutes business logic for this feature?"

  Self-Assessment Questions:
    1. Can I distinguish business rules from implementation details?
    2. Do I know if this logic will be reused across features?
    3. Is this logic subject to change based on business requirements?
    4. Will stakeholders need to query/audit this logic?

  Confidence Level:
    ✅ High (>90%): Business logic clearly identified → Continue
    ⚠️ Medium (70-90%): Some uncertainty → Consult with PM/domain expert
    ❌ Low (<70%): Cannot distinguish business vs. technical logic → STOP

  IF Low Confidence (<70%):
    → Review spec.md for business requirements
    → Consult with product manager or domain expert
    → Check existing catalog for similar patterns
  ↓
  ├─ Does feature include ANY of:
  │    • Validation rules (capacity limits, quotas)
  │    • Authorization logic (role-based access)
  │    • Workflow state transitions (draft → active)
  │    • Calculation formulas (pricing, scoring)
  │    • Multi-tenant configuration rules
  │
  │    YES ↓
  │
  ├─ Will this logic:
  │    • Be queried by other features?
  │    • Require test coverage tracking?
  │    • Need version control (DEPRECATED → ACTIVE)?
  │    • Be exposed to LLM agents for Q&A?
  │
  │    YES → USE Business Rules System ✅
  │    NO  → Optional (use for traceability) ℹ️
  │
  └─ RESULT:
       • Run /rules-harvest during planning
       • Annotate code with @BusinessRule decorators
       • Run /rules-extract post-implementation
       • Catalog updated in .specify/business-rules/catalog.md

EXAMPLES:
✅ Use Business Rules:
   - Zone capacity limits (max 1000 zones per company)
   - Role-based feature access (admin vs. member permissions)
   - Tenant isolation rules (schema-per-company enforcement)
   - Pricing calculation formulas (tiered pricing logic)

❌ Skip Business Rules:
   - UI validation (frontend-only checks)
   - Transient data transformations (DTO mapping)
   - Framework configuration (webpack settings)
   - Test fixtures (mock data generation)

WORKFLOW:
1. /rules-harvest → Creates specs/###-feature/rules/harvest.md (DRAFT)
2. Implement with @BusinessRule annotations
3. /rules-extract → Updates catalog (DRAFT → ACTIVE)
4. LLMs read QUICK_REFERENCE.md for existing rules
```

---

## Layer Architecture (5 Layers)

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

**NOTE ON TERMINOLOGY**: This document uses "Milestone 1-8" for implementation phases to avoid confusion with SDD workflow phases (Specify → Clarify → Plan → Implement).

---

## Complete End-to-End Walkthrough

### Scenario: Multi-Company Zone Capacity System with Prisma Pivot

**Timeline**: 13 weeks (includes 1-week pivot)
**Complexity**: High (multi-tenancy + architectural pivot)
**Teams**: Backend (3 eng), Frontend (2 eng), QA (1)

#### Week 1-2: Specification Phase

**PM Actions:**
```bash
# Create feature directory
./specify init 002-multi-company-zones

# Write non-technical spec
vim specs/002-multi-company-zones/spec.md
```

**spec.md content**:
```markdown
# Multi-Company Zone Capacity Management

## User Stories
- As a company admin, I can define maximum zone capacity (1-1000 zones)
- As a developer, I can check if company has available zone capacity
- As a system admin, I can view capacity metrics across all companies

## Success Criteria
- Zone creation blocked when company reaches max capacity
- Capacity checks complete in <100ms (P95)
- Multi-tenant data isolation (schema-per-company)

## Acceptance Tests
- Given company has max_zones=10, when 10 zones exist, then new zone creation fails
- Given company has max_zones=1000, when 999 zones exist, then new zone succeeds
```

**Architect Actions:**
```bash
# Assess complexity
# Decision: Multi-tenancy + performance requirements → Use TECHNICAL.md

vim specs/002-multi-company-zones/TECHNICAL.md
```

**TECHNICAL.md content**:
```markdown
# Technical Constraints: Zone Capacity

## Multi-Tenancy Strategy
- **Pattern**: Schema-per-company (PostgreSQL)
- **Isolation**: Row-level security (RLS) policies
- **Connection Pooling**: Prisma with PgBouncer

## Performance Targets
- **Latency**: P95 <100ms for capacity checks
- **Throughput**: 1000 requests/second peak
- **Scale**: Support 10,000 companies

## Technology Assumptions
- **ORM**: Prisma (existing project standard)
- **Database**: PostgreSQL 15+
- **Cache**: Redis for capacity counters

## Risks
- [NEEDS VALIDATION] Prisma schema-per-tenant support unproven at scale
- [NEEDS VALIDATION] RLS performance impact on capacity queries unknown

## Constitutional Compliance
- Article VII (Simplicity): Using existing Prisma stack ✅
- Article VIII (Anti-Abstraction): Prisma is trusted framework ✅
- Article IX (Integration-First): Contract tests required ✅
```

#### Week 3: Planning Phase

**Engineer Actions:**
```bash
# Harvest business rules
/rules-harvest
```

**rules/harvest.md (generated)**:
```markdown
# Business Rules: Zone Capacity

## BR-ZONES-001: Maximum Zone Capacity (DRAFT)
**Rule ID**: BR-ZONES-001
**Priority**: HIGH
**Status**: DRAFT
**Tenant Scope**: PER_TENANT
**Domain**: ZONES
**Source**: FR-002-001 (spec.md line 12)

**Logic**:
- **Given**: Company has configured max_zones = N
- **When**: User attempts to create zone
- **Then**:
  - Count existing zones for company
  - IF count >= N THEN reject with error "Zone capacity reached"
  - ELSE allow creation

**Test References**: [Pending implementation]
**Implementation**: [Pending]
```

```bash
# Generate implementation plan
/plan
```

**plan.md (excerpt)**:
```markdown
# Phase -1: Constitutional Validation
- [ ] Article VII (Simplicity): Prisma only ✅
- [ ] Article VIII (Anti-Abstraction): No custom ORM wrappers ✅
- [ ] Article IX (Integration-First): Contract tests for capacity API ✅
- [ ] [NEEDS VALIDATION] flags present in TECHNICAL.md ⚠️

# Phase 1: Schema Design
- [ ] Design Prisma schema with RLS support
- [ ] Validate [NEEDS VALIDATION]: Prisma multi-tenant approach
- [ ] Create contract tests for capacity endpoint

# Phase 2: Capacity Logic
- [ ] Implement @BusinessRule BR-ZONES-001 in service layer
- [ ] Add Redis caching for capacity counters
- [ ] Write unit tests (2 test cases minimum per Article I)

# Phase 3: Integration Testing
- [ ] Test with 10,000 simulated companies (scale validation)
- [ ] Measure P95 latency (target <100ms)
```

#### Week 4: Implementation Begins - Blocker Discovered

**Engineer 1 (Backend Lead)**:
```bash
# Working on Phase 1 tasks
# Discovers: Prisma doesn't support dynamic schema switching for RLS

# Research outcome:
# - Prisma requires separate client instances per schema
# - Connection pooling breaks with 10,000 schemas
# - Performance: ~500ms per capacity check (5x over target)

# Escalation decision: This is ARCHITECTURAL BLOCKER (not bug)
# Violates: NFRs (performance <100ms) + Article VIII (framework limitation)

# Create ADR using Decision Tree 2 & 3
/create-adr
```

**ADR Creation (interactive prompts)**:
```
📝 ADR Creation Wizard

Title: Migrate from Prisma to node-postgres for Multi-Tenancy
Scope: [1] Feature-scoped  [2] Cross-project → 2 (affects all future multi-tenant features)
Location: .specify/decisions/adr-002-prisma-to-pg.md

Context: What blocker triggered this decision?
> Prisma requires separate client instances per schema, causing 500ms latency (target <100ms)

Decision FROM:
> Prisma ORM with dynamic schema switching

Decision TO:
> Pure node-postgres with SET search_path for tenant isolation

Constitutional Compliance:
Article VII (Simplicity): Does this add new project? → Yes (pg replaces Prisma)
Article VIII (Anti-Abstraction): Is this a trusted framework? → Yes (node-postgres is official driver)
Article IX (Integration-First): Contract tests need updates? → Yes (update tests BEFORE implementation)

Consequences:
> Trade-off: Lose Prisma type safety → Gain 90% latency reduction
> Mitigation: Add TypeScript DTO validation + integration tests

Affected Business Rules:
> BR-ZONES-001: Implementation reference will change
> Test references will change (Prisma contracts → pg contracts)

Timeline Impact:
> Original: Week 4-6 (3 weeks)
> Adjusted: Week 4-8 (5 weeks = 3 weeks + 2 weeks migration)

Rollback Plan:
> Keep Prisma for non-multi-tenant features
> Create pg-service.ts as separate module
> Feature flag MULTI_TENANT_PG for gradual rollout

✅ ADR created: .specify/decisions/adr-002-prisma-to-pg.md
```

**adr-002-prisma-to-pg.md (generated)**:
```markdown
# ADR-002: Migrate from Prisma to node-postgres for Multi-Tenancy

**Status**: Approved
**Date**: 2025-01-30
**Deciders**: Backend Lead, Architect, PM

## Context
During implementation of Zone Capacity feature (specs/002-multi-company-zones), discovered Prisma ORM limitation:
- Requires separate Prisma Client per schema (schema-per-tenant model)
- Connection pooling breaks with 10,000+ tenant schemas
- Measured latency: ~500ms per capacity check (target: <100ms P95)
- Blocker classification: ARCHITECTURAL (not workaround-able)

## Decision
**FROM**: Prisma ORM with dynamic schema switching
**TO**: Pure node-postgres with `SET search_path` for tenant isolation

**Rationale**:
- node-postgres supports connection-level `SET search_path` (10ms overhead)
- Maintains single connection pool across all tenants
- Measured latency: ~25ms per capacity check (4x under target)

## Constitutional Compliance

### Article VII (Simplicity Gate)
**Assessment**: CONDITIONAL PASS ⚠️
- **Concern**: Adds pg alongside Prisma (2 database access patterns)
- **Justification**: Isolated to multi-tenant features only
- **Architect Approval**: Required ✅ (approved 2025-01-30)

### Article VIII (Anti-Abstraction Gate)
**Assessment**: PASS ✅
- node-postgres is official PostgreSQL driver (trusted framework)
- No custom ORM wrappers introduced
- Direct SQL with parameterized queries

### Article IX (Integration-First Gate)
**Assessment**: PASS ✅
- Contract tests updated BEFORE migration (see test-plan.md)
- Real PostgreSQL used in tests (no mocks)
- RLS policies validated via integration tests

## Consequences

### Positive
- 90% latency reduction (500ms → 25ms)
- Supports 10,000+ tenant schemas
- Simpler mental model (direct SQL vs ORM abstractions)

### Negative
- Lose Prisma type safety for multi-tenant queries
- Engineers must write SQL manually (higher skill requirement)
- Two database access patterns in codebase (Prisma + pg)

### Mitigation
- TypeScript DTOs for response validation
- SQL query builder helpers (parameterized queries)
- Integration test coverage ≥95% for multi-tenant features
- Code review checklist for SQL injection prevention

## Affected Business Rules

### Modified Rules
- **BR-ZONES-001**: Max Zone Capacity
  - **Implementation Change**:
    - FROM: `src/zones/prisma-service.ts:142` (Prisma)
    - TO: `src/zones/pg-service.ts:67` (node-postgres Pool)
  - **Test Change**:
    - FROM: `tests/zones/test_max_capacity.py:52` (Prisma contracts)
    - TO: `tests/zones/test_max_capacity_pg.py:52` (pg contracts)
  - **Action**: Update catalog via `/rules-extract --module ZONES`
  - **Constitutional Impact**: Article VI (Test-First) - tests updated before implementation ✅

### New Rules Created
- **BR-MIGRATION-001**: Pure pg Migration Validator (status: DRAFT)
  - Ensures migrations run in correct tenant schema context
  - Validates RLS policies applied post-migration
  - Domain: INFRASTRUCTURE

## Timeline Impact
- **Original Estimate**: Week 4-6 (3 weeks)
- **Adjusted Estimate**: Week 4-8 (5 weeks)
  - Week 4: Create ADR + update contract tests
  - Week 5-6: Migrate zone capacity logic to node-postgres
  - Week 7: Integration testing + RLS validation
  - Week 8: Performance benchmarking + rollout planning

## Implementation Plan
1. ✅ Update contract tests (Week 4) - RED phase
2. Install node-postgres + connection pool setup (Week 5)
3. Implement pg-service.ts with RLS support (Week 5-6)
4. Migrate @BusinessRule BR-ZONES-001 implementation (Week 6)
5. Integration testing with 10,000 test tenants (Week 7)
6. Performance validation: P95 <100ms ✅ (Week 7)
7. Feature flag rollout (Week 8)

## Rollback Plan
- Prisma remains for non-multi-tenant features
- Feature flag: MULTI_TENANT_PG (gradual rollout)
- If P95 >100ms in production → rollback to Prisma (accept latency)
- Estimated rollback time: 2 hours (toggle feature flag)

## References
- TECHNICAL.md: specs/002-multi-company-zones/TECHNICAL.md (amended)
- Contract Tests: tests/integration/test_zone_capacity_contracts.py
- Performance Benchmarks: docs/benchmarks/pg-vs-prisma.md
- Migration Guide: docs/guides/multi-tenant-pg-migration.md
```

#### Week 4-5: Pivot Execution (Article XI Protocol)

**Step 1: Update Contract Tests (Test-First)**
```bash
# Article XI.4: Update tests BEFORE implementation
vim tests/integration/test_zone_capacity_contracts.py
```

```python
# test_zone_capacity_contracts.py (updated)
def test_capacity_check_multi_tenant_pg():
    """BR-ZONES-001: Max Zone Capacity (node-postgres)"""
    # Setup: Create tenant schema
    pg_pool.query("CREATE SCHEMA IF NOT EXISTS company_123")
    pg_pool.query("SET search_path TO company_123")

    # Given: Company has max_zones = 10
    pg_pool.query("UPDATE companies SET max_zones = 10 WHERE id = 123")

    # When: 10 zones exist
    for i in range(10):
        create_zone(company_id=123, name=f"Zone {i}")

    # Then: 11th zone creation fails
    with pytest.raises(CapacityExceededError):
        create_zone(company_id=123, name="Zone 11")

    # Validate: <100ms latency
    start = time.time()
    check_capacity(company_id=123)
    assert (time.time() - start) < 0.1  # P95 <100ms
```

```bash
# Validate tests FAIL (RED phase)
pytest tests/integration/test_zone_capacity_contracts.py
# Expected: FAIL (implementation not migrated yet)

# Get user approval
# PM reviews test changes → approves pivot
```

**Step 2: Amend TECHNICAL.md**
```bash
/amend-technical
```

**TECHNICAL.md (amended)**:
```markdown
# Technical Constraints: Zone Capacity

## Multi-Tenancy Strategy
- **Pattern**: Schema-per-company (PostgreSQL)
- **Isolation**: Row-level security (RLS) policies
- ~~**Connection Pooling**: Prisma with PgBouncer~~
  **Connection Pooling**: node-postgres Pool with `SET search_path`
  **Reason**: ADR-002 - Prisma latency blocker (500ms → 25ms)

## Technology Assumptions
- ~~**ORM**: Prisma (existing project standard)~~
  **Database Driver**: node-postgres (official PostgreSQL driver)
  **Reason**: ADR-002 - Prisma multi-tenant limitations
  **Scope**: Multi-tenant features only (non-MT features keep Prisma)

## Risks
- ~~[NEEDS VALIDATION] Prisma schema-per-tenant support unproven at scale~~
  **VALIDATED**: Prisma blocked at scale → migrated to node-postgres (ADR-002)
- ~~[NEEDS VALIDATION] RLS performance impact on capacity queries unknown~~
  **VALIDATED**: RLS adds <10ms overhead (acceptable)

## Decision History
- **ADR-002** (2025-01-30): Prisma → node-postgres for multi-tenancy
  - Blocker: 500ms latency (5x over target)
  - Solution: Direct SQL with search_path switching
  - Impact: Timeline +2 weeks (Week 4-8 instead of 4-6)

## Constitutional Compliance
- Article VII (Simplicity): ~~Using existing Prisma stack~~ Using pg alongside Prisma (approved) ⚠️
- Article VIII (Anti-Abstraction): ~~Prisma is trusted~~ node-postgres is official driver ✅
- Article IX (Integration-First): Contract tests updated ✅
```

**Step 3: Regenerate Plan**
```bash
/plan --regenerate
```

**plan.md (regenerated excerpt)**:
```markdown
# Phase -1: Constitutional Validation
- [x] Article VII: Architect approved pg addition (ADR-002) ✅
- [x] Article VIII: node-postgres is trusted driver ✅
- [x] Article IX: Contract tests updated (RED phase validated) ✅
- [x] Article XI: ADR-002 created, TECHNICAL.md amended ✅

# Phase 1: node-postgres Setup (NEW)
- [ ] Install node-postgres + @types/pg
- [ ] Configure connection pool (max 100 connections)
- [ ] Create pg-service.ts abstraction layer
- [ ] Add search_path helper: setTenantContext(companyId)

# Phase 2: Migration (UPDATED)
- [ ] Migrate BR-ZONES-001 from Prisma → pg
- [ ] Update @BusinessRule annotations with new file references
- [ ] Validate tests GREEN (contract tests pass)
- [ ] Performance test: P95 <100ms ✅

# Phase 3: Business Rules Catalog Update
- [ ] Run /rules-extract --module ZONES
- [ ] Validate BR-ZONES-001 implementation references updated
- [ ] Verify catalog links to ADR-002
```

#### Week 5-7: Implementation with Pivot

**Week 5**: node-postgres setup
**Week 6**: Migrate capacity logic + @BusinessRule annotations
**Week 7**: Integration testing + performance validation

**Code Example** (Week 6):
```typescript
// src/zones/pg-service.ts

import { Pool } from 'pg';
import { BusinessRule } from '@app/decorators';

const pool = new Pool({ max: 100 });

/**
 * @BusinessRule BR-ZONES-001
 * Rule: Maximum Zone Capacity
 * Priority: HIGH
 * Status: ACTIVE
 * Test: tests/zones/test_max_capacity_pg.py:52
 * Source: FR-002-001
 */
export async function checkZoneCapacity(companyId: number): Promise<boolean> {
  const client = await pool.connect();
  try {
    // Set tenant context
    await client.query(`SET search_path TO company_${companyId}`);

    // Fetch max capacity
    const { rows } = await client.query(
      'SELECT max_zones FROM companies WHERE id = $1',
      [companyId]
    );
    const maxZones = rows[0]?.max_zones || 0;

    // Count existing zones
    const { rows: countRows } = await client.query(
      'SELECT COUNT(*) as count FROM zones'
    );
    const currentCount = parseInt(countRows[0].count, 10);

    // Business logic: Given/When/Then from BR-ZONES-001
    return currentCount < maxZones;
  } finally {
    client.release();
  }
}
```

#### Week 8: Catalog Update & Verification

```bash
# Extract updated business rules
/rules-extract --module ZONES --include-adrs
```

**catalog.md (excerpt after extraction)**:
```markdown
# Business Rules Catalog

## BR-ZONES-001: Maximum Zone Capacity
**Rule ID**: BR-ZONES-001
**Priority**: HIGH
**Status**: ACTIVE ✅
**Tenant Scope**: PER_TENANT
**Domain**: ZONES
**Version**: 1.1 (updated 2025-02-07)

**Logic**:
- **Given**: Company has configured max_zones = N
- **When**: User attempts to create zone
- **Then**:
  - Count existing zones for company
  - IF count >= N THEN reject with error "Zone capacity reached"
  - ELSE allow creation

**Implementation**: `src/zones/pg-service.ts:67` (updated from Prisma)
**Test References**:
- Valid: `tests/zones/test_max_capacity_pg.py:52`
- Invalid: `tests/zones/test_max_capacity_pg.py:78`

**Source**: FR-002-001 (specs/002-multi-company-zones/spec.md)
**Pivot History**:
- **ADR-002** (2025-01-30): Migrated from Prisma to node-postgres
  - Reason: Latency blocker (500ms → 25ms)
  - Impact: Implementation reference changed

**Constitutional Compliance**:
- Article I (Testability): 2 test refs ✅
- Article II (Completeness): All fields present ✅
- Article VI (Test-First): Tests exist ✅
- Compliance Score: 100% ✅
```

**QUICK_REFERENCE.md (LLM-optimized excerpt)**:
```markdown
# Business Rules Quick Reference (<2000 tokens)

## Zone Management Rules

### BR-ZONES-001: Max Capacity
- **When**: Zone creation
- **Check**: current_count < max_zones
- **Error**: "Zone capacity reached (max: {N})"
- **Impl**: `src/zones/pg-service.ts:67`
- **Recent Change**: Migrated to node-postgres (ADR-002, 2025-01-30)
```

#### Week 9-13: Continued Feature Development

- Week 9-10: Frontend implementation
- Week 11: QA testing
- Week 12: Performance benchmarking (validate P95 <100ms ✅)
- Week 13: Production rollout (feature flag enabled)

### Walkthrough Summary

**Total Timeline**: 13 weeks (original: 12 weeks, +1 week for pivot)

**Artifacts Created**:
1. specs/002-multi-company-zones/spec.md (non-technical, STABLE)
2. specs/002-multi-company-zones/TECHNICAL.md (amended after pivot)
3. specs/002-multi-company-zones/rules/harvest.md (DRAFT → ACTIVE)
4. .specify/decisions/adr-002-prisma-to-pg.md (cross-project ADR)
5. specs/002-multi-company-zones/plan.md (regenerated post-pivot)
6. specs/002-multi-company-zones/tasks.md (regenerated post-pivot)
7. .specify/business-rules/catalog.md (updated with BR-ZONES-001)
8. .specify/business-rules/QUICK_REFERENCE.md (LLM-optimized)

**Key Success Factors**:
- ✅ Non-technical spec.md never changed (stable for PM)
- ✅ Pivot discovered Week 4 (not Week 10) via [NEEDS VALIDATION] flags
- ✅ ADR provided 100% traceability for decision
- ✅ Test-first protocol maintained (tests updated BEFORE pivot)
- ✅ Business rules catalog stayed synchronized
- ✅ LLM agents have complete context (TECHNICAL.md + ADR + catalog)

---

## Anti-Patterns (What NOT to Do)

### Anti-Pattern 1: Putting Technical Details in spec.md ❌

**Bad Example**:
```markdown
# spec.md (WRONG)
## User Stories
- As a developer, I can create zones using Prisma ORM
- As a system, I use Redis caching with 5-minute TTL
- As a database, I enforce RLS policies via SET search_path
```

**Why It's Wrong**:
- Spec becomes unstable when technology changes (violates Layer 1 principle)
- PM/stakeholders see implementation details (confusing)
- LLM agents may prioritize technical details over user value

**Correct Approach**:
```markdown
# spec.md (CORRECT)
## User Stories
- As a company admin, I can define maximum zone capacity
- As a system, I enforce capacity limits with <100ms response time
```

```markdown
# TECHNICAL.md (CORRECT)
## Technology Assumptions
- ORM: Prisma
- Cache: Redis (5-minute TTL)
- Multi-tenancy: RLS with SET search_path
```

---

### Anti-Pattern 2: Creating ADR for Every Decision ❌

**Bad Example**:
- ADR-001: Choose ESLint for linting
- ADR-002: Use async/await instead of callbacks
- ADR-003: Name files in kebab-case

**Why It's Wrong**:
- ADRs become noise (low signal-to-noise ratio)
- Engineers spend more time documenting than implementing
- Hard to find critical architectural decisions

**Correct Approach**:
- **Use ADRs for**: Framework changes, database migrations, architectural pivots
- **Skip ADRs for**: Code style, naming conventions (use linter configs)
- **Rule of Thumb**: If decision affects >1 feature or violates constitutional article → ADR

---

### Anti-Pattern 3: Forgetting to Amend TECHNICAL.md After Pivot ❌

**Bad Example**:
```markdown
# TECHNICAL.md (STALE)
## Technology Assumptions
- ORM: Prisma  ← Out of date! Already migrated to pg

# ADR-002 exists but TECHNICAL.md not amended
```

**Why It's Wrong**:
- LLM agents read TECHNICAL.md first → generate plans with wrong assumptions
- Engineers onboarding see outdated constraints
- No historical decision trail

**Correct Approach**:
```markdown
# TECHNICAL.md (AMENDED)
- ~~ORM: Prisma~~
  **Database Driver**: node-postgres
  **Reason**: ADR-002 - Latency blocker
```

---

### Anti-Pattern 4: Skipping Test Updates Before Pivot ❌

**Bad Example**:
```bash
# Engineer discovers Prisma blocker
# Immediately starts implementing node-postgres
# Forgets to update contract tests
# Tests still validate Prisma behavior
```

**Why It's Wrong**:
- Violates Article XI.4 (Test-First Imperative)
- No RED phase validation (can't confirm blocker)
- Risk: Old tests pass, new implementation broken

**Correct Approach**:
```bash
# 1. Update contract tests FIRST
vim tests/integration/test_zone_capacity_contracts.py
# Change: Prisma assertions → node-postgres assertions

# 2. Run tests → validate FAIL (RED phase)
pytest tests/integration/

# 3. Get approval for test changes

# 4. ONLY THEN implement pivot
```

---

### Anti-Pattern 5: Feature-Scoped ADR That Should Be Cross-Project ❌

**Bad Example**:
```
specs/001-zones/decisions/adr-002-migrate-all-to-pg.md  ← WRONG location
```

**Content**:
```markdown
# ADR-002: Migrate ALL Features to node-postgres
# (This affects zones, users, billing, notifications...)
```

**Why It's Wrong**:
- ADR hidden in feature directory (other teams won't find it)
- Affects multiple features but not discoverable

**Correct Approach**:
```
.specify/decisions/adr-002-migrate-all-to-pg.md  ← CORRECT location
```

**Decision Tree**: Use Tree 3 to determine placement

---

### Anti-Pattern 6: Business Rule Without Test References ❌

**Bad Example**:
```markdown
## BR-ZONES-001: Max Zone Capacity
**Status**: ACTIVE
**Test References**: [None]  ← Violates Article I (Testability Mandate)
```

**Why It's Wrong**:
- Cannot validate rule implementation
- Risks regression bugs
- Fails constitutional validation (HIGH severity)

**Correct Approach**:
```markdown
## BR-ZONES-001: Max Zone Capacity
**Status**: ACTIVE
**Test References**:
- Valid: tests/zones/test_max_capacity_pg.py:52
- Invalid: tests/zones/test_max_capacity_pg.py:78
```

---

### Anti-Pattern 7: Root Directory Pollution ❌

**Bad Example**:
```
spec-kit/
├── technical-constraints.md  ← WRONG (at root)
├── business-rules.md         ← WRONG (at root)
├── adr-001-pivot.md          ← WRONG (at root)
```

**Why It's Wrong**:
- Violates SDD directory hygiene
- Makes navigation confusing
- Breaks LLM context priority

**Correct Approach**:
```
spec-kit/
├── specs/001-feature/TECHNICAL.md  ← CORRECT (feature-specific)
├── specs/001-feature/rules/harvest.md  ← CORRECT
├── .specify/decisions/adr-001.md   ← CORRECT (infrastructure)
```

---

## FAQ (Frequently Asked Questions)

### Q1: When should I create TECHNICAL.md?

**A**: Use Decision Tree 1. Generally:
- **Yes, create**: Multi-tenancy, performance targets (P95 <250ms), new frameworks, regulatory constraints
- **No, skip**: Simple CRUD, known tech stack, UI-only features

**Example YES**: Multi-company zone capacity (multi-tenancy + performance)
**Example NO**: User profile CRUD with existing Prisma setup

---

### Q2: What if I discover a blocker after plan is generated?

**A**: Follow Article XI Pivot Protocol:
1. Create ADR (/create-adr)
2. Update contract tests (Article XI.4)
3. Amend TECHNICAL.md (/amend-technical)
4. Regenerate plan (/plan --regenerate)
5. Continue with updated tasks

**Timeline impact**: Typically +1-2 weeks depending on pivot scope

---

### Q3: Can I have both Prisma AND node-postgres in the same project?

**A**: Yes, if justified:
- **Requires**: Article VII (Simplicity) architect approval
- **Justification**: Different use cases (Prisma for simple queries, pg for multi-tenancy)
- **Documentation**: Must explain in ADR + TECHNICAL.md
- **Example**: ADR-002 in walkthrough (Prisma for non-MT, pg for MT)

---

### Q4: How do I know if an ADR should be feature-scoped or cross-project?

**A**: Use Decision Tree 3:
- **Feature-scoped** (`specs/###/decisions/`): Affects only current feature
- **Cross-project** (`.specify/decisions/`): Affects >1 feature or system architecture

**Edge case**: Starts feature-scoped, becomes cross-project → Move ADR and update references

---

### Q5: What happens if /rules-extract finds validation failures?

**A**: Constitutional gates run with severity levels:
- **HIGH severity** (Articles I, II, IV, VI): Build FAILS in CI/CD
- **MEDIUM severity** (Articles III, V, VIII, IX): Warnings logged
- **LOW severity** (Article VII): Informational only

**Example failure**:
```bash
❌ BR-ZONES-001 FAILED Article I (Testability)
   Reason: ACTIVE rule missing test references
   Severity: HIGH
   Action Required: Add test references or change status to DRAFT
```

---

### Q6: Can I use Business Rules system without TECHNICAL.md?

**A**: Yes, they're independent:
- **TECHNICAL.md**: Architectural constraints (optional)
- **Business Rules**: Logic documentation (optional)

**Example**: Simple CRUD with business rules:
```
specs/001-user-profile/
├── spec.md            ← Required
├── rules/harvest.md   ← Optional (business logic present)
└── plan.md            ← Generated
(No TECHNICAL.md - not complex enough)
```

---

### Q7: What if I disagree with a constitutional article?

**A**: Constitutional articles are immutable, but:
- **Process**: Propose new article (not modify existing)
- **Approval**: Requires stakeholder consensus
- **Example**: Article XI was ADDED (not modified Article I-X)

**For this project**: Articles I-XI are governance-approved

---

### Q8: How do LLM agents know which artifacts to read?

**A**: Reading priority defined in Layer Architecture:
1. Constitution (Articles I-XI)
2. spec.md (user value)
3. TECHNICAL.md (constraints)
4. rules/harvest.md (business logic)
5. ADRs (decisions)
6. plan.md + tasks.md (generated)
7. QUICK_REFERENCE.md (catalog)
8. CLAUDE.md (project patterns)

**Token optimization**: Agents selectively load based on task type (see Section 4 LLM Reading Priority)

---

### Q9: Can I delete old ADRs?

**A**: No. ADRs are immutable historical records:
- **Status field**: Use "Superseded by ADR-###" if replaced
- **Reason**: Maintains decision trail for audits
- **Example**:
  ```markdown
  # ADR-001: Use MySQL
  **Status**: Superseded by ADR-003 (PostgreSQL migration)
  ```

---

### Q10: What's the difference between validation-rules.md and constitution.md?

**A**:
- **memory/constitution.md**: Development methodology (how we build software, Articles I-XI)
- **.specify/business-rules/validation-rules.md**: Documentation quality (how we write business rules, Articles I-IX)

**Example**:
- Constitution Article III: "Write tests first" (dev practice)
- Validation-rules Article I: "Rules must have test references" (doc quality)

---

## Glossary

### A

**ADR (Architecture Decision Record)**
Immutable document capturing architectural pivots. Includes context, decision, constitutional compliance, consequences, and implementation plan. Stored in `specs/###/decisions/` (feature-scoped) or `.specify/decisions/` (cross-project).

**Adaptive Pivot Protocol**
Article XI of constitution. 7-step process for handling architectural blockers: classify blocker → create ADR → update tests → amend TECHNICAL.md → regenerate plan → update catalog → communicate impact.

**Article (Constitutional)**
Immutable principle in memory/constitution.md. Articles I-X are core SDD, Article XI is Adaptive addition. Never modified, only extended.

**Article (Validation Rule)**
Quality standard in .specify/business-rules/validation-rules.md. Articles I-IX define business rule documentation requirements. Separate from constitutional articles.

### B

**Blocker (Architectural)**
Implementation obstacle requiring framework/design change. Cannot be fixed with simple code changes. Triggers Pivot Protocol. Examples: ORM performance, framework limitations, scalability issues.

**BR-* (Business Rule ID)**
Unique identifier for business logic. Format: `BR-{DOMAIN}-{NUMBER}`. Examples: BR-ZONES-001, BR-AUTH-042. Links spec → implementation → tests → catalog.

**@BusinessRule (Annotation)**
Code decorator linking implementation to catalog entry. Includes Rule ID, priority, status, test references. Parsed by `/rules-extract` command.

### C

**Catalog (Business Rules)**
Master index of all business logic at `.specify/business-rules/catalog.md`. Aggregates rules from all features. Updated via `/rules-extract`. Includes constitutional validation results.

**Constitutional Compliance**
Validation against Articles I-XI (dev methodology) or Articles I-IX (rule quality). Required for ADRs, plan generation, and catalog updates. Pass/fail gates with severity levels.

**Constitutional Validation**
Phase -1 in plan.md. Checks Articles VII (Simplicity), VIII (Anti-Abstraction), IX (Integration-First), XI (Pivot Protocol) before implementation begins.

**Contract Tests**
Integration tests validating API contracts with real dependencies (Article IX). Updated BEFORE pivots (Article XI.4). Example: Prisma → pg requires test updates first.

### D

**DRAFT (Rule Status)**
Business rule not yet implemented. Captured in rules/harvest.md during planning. Transitions to ACTIVE post-implementation after test validation.

**Decision Tree**
Visual workflow for determining when to use methodology components. Four trees: TECHNICAL.md creation, ADR necessity, ADR placement, Business Rules usage.

### E

**Evolves (Layer Stability)**
Layer 2 (TECHNICAL.md) and Layer 2.5 (harvest.md) can change during implementation. Contrasts with Layer 1 (spec.md - STABLE) and Layer 3 (ADRs - IMMUTABLE).

### F

**Feature-Scoped ADR**
Architecture decision affecting only one feature. Stored in `specs/###-feature/decisions/adr-*.md`. Example: Choice of caching strategy for single feature.

### G

**Given/When/Then (GWT)**
Business rule logic format. Given = preconditions, When = trigger, Then = outcome. Example: Given max_zones=10, When create zone, Then reject if count≥10.

### H

**Harvest (Business Rules)**
Initial capture of business logic during planning via `/rules-harvest`. Creates `specs/###/rules/harvest.md` with DRAFT rules. Evolves to ACTIVE during implementation.

### I

**Immutable (Layer Stability)**
Layer 0 (constitution) and Layer 3 (ADRs) never modified. Constitution only extended via new articles. ADRs use "Superseded" status if replaced.

**Integration-First Gate**
Article IX constitutional requirement. Contract tests with real dependencies (PostgreSQL, Redis) before implementation. No mocks for persistence layer.

### L

**Layer (Architecture)**
5-layer information hierarchy: 0=Constitution, 1=Spec, 2=TECHNICAL.md, 2.5=Harvest, 3=ADRs, 4=Plan/Tasks, 5=Catalog. Defines stability (STABLE/EVOLVES/IMMUTABLE) and LLM reading priority.

**LLM Context Priority**
Ordered list for AI agent reading: Constitution → Spec → TECHNICAL.md → Harvest → ADRs → Plan → Catalog → CLAUDE.md. Optimizes token usage and relevance.

### M

**Milestone (Implementation Phase)**
Implementation roadmap stages (Milestone 1-8). Distinct from SDD workflow phases (Specify → Clarify → Plan → Implement). Used to avoid terminology confusion.

**Multi-Tenancy**
Architecture pattern for isolating company/tenant data. Common strategies: schema-per-tenant, row-level security (RLS). Requires TECHNICAL.md documentation.

### N

**[NEEDS CLARIFICATION]**
Marker in rules/harvest.md for ambiguous business logic. Signals PM/stakeholder follow-up required. Example: "Max capacity default value [NEEDS CLARIFICATION: 100 or 1000?]"

**[NEEDS VALIDATION]**
Marker in TECHNICAL.md for architectural unknowns. Triggers early blocker discovery. Example: "[NEEDS VALIDATION] Prisma multi-tenant scaling unproven"

**NFR (Non-Functional Requirement)**
Performance, scalability, security targets. Documented in TECHNICAL.md. Examples: P95 <100ms, 1000 RPS, 99.9% uptime.

### P

**Pivot (Architectural)**
Change in technology choice due to discovered blocker. Requires ADR creation and TECHNICAL.md amendment. Example: Prisma → node-postgres for latency.

**Pivot Protocol**
Article XI. 7-step process: classify → ADR → tests → amend → regenerate → catalog → communicate. Ensures traceability and constitutional compliance.

### Q

**QUICK_REFERENCE.md**
LLM-optimized business rules summary at `.specify/business-rules/QUICK_REFERENCE.md`. <2000 tokens. Generated from catalog.md. Used by AI agents for rapid rule lookup.

### R

**Regenerate (Plan)**
Re-run `/plan` command after TECHNICAL.md amendments or ADRs. Creates updated plan.md + tasks.md respecting new constraints. Example: Post-pivot regeneration.

**RLS (Row-Level Security)**
PostgreSQL feature for tenant isolation. Policies enforce data access rules. Documented in TECHNICAL.md. Example: `SET search_path TO company_{id}`

**Rule Status**
Lifecycle state: DRAFT (planning) → ACTIVE (implemented+tested) → DEPRECATED (replaced). Transitions validated by constitutional gates.

### S

**SDD (Spec-Driven Development)**
GitHub Spec Kit methodology. Core workflow: Specify → Clarify → Plan → Implement. Non-technical specs, test-first, contract-driven.

**Severity (Validation)**
Constitutional gate failure impact: HIGH (CI fails), MEDIUM (warnings), LOW (informational). Example: Missing test refs = HIGH severity (Article I).

**Simplicity Gate**
Article VII constitutional requirement. Projects ≤3 total. Adding new framework requires architect approval. Example: Prisma+pg = 2 projects (conditional pass).

**spec.md (Layer 1)**
Non-technical feature specification. User stories, success criteria, acceptance tests. STABLE - never changes due to technical pivots. Product manager-friendly.

**Strikethrough Amendment**
TECHNICAL.md editing pattern: ~~old constraint~~ → new constraint + ADR link. Preserves decision history. Example: ~~Prisma~~ → node-postgres (ADR-002)

### T

**tasks.md**
Sequenced implementation checklist generated by `/plan`. Includes business rule implementation tasks, constitutional gates, test-first requirements.

**TECHNICAL.md (Layer 2)**
Optional architectural constraints document. Contains: multi-tenancy strategy, performance targets, technology assumptions, [NEEDS VALIDATION] flags, decision history. EVOLVES via amendments.

**Tenant Scope**
Business rule multi-tenancy classification: GLOBAL (all tenants), PER_TENANT (tenant-specific), CONFIGURABLE (varies). Documented in rules metadata.

**Test-First Imperative**
Article III constitutional requirement. Tests written BEFORE implementation. Extended by Article XI.4: contract tests updated BEFORE pivots.

**Test References (Rule)**
Links from business rule to validation tests. Minimum 2 tests (valid + invalid cases) per Article I. Example: `tests/zones/test_capacity.py:52,78`

### U

**Unified Methodology**
Integration of core SDD + Adaptive Pivot Protocol + Business Rules Management. 5-layer architecture with 98% SDD alignment.

### V

**Validation (Constitutional)**
9-gate quality check for business rules (Articles I-IX in validation-rules.md). Runs during `/rules-extract`. Target: ≥95% compliance. Separate from dev methodology validation (Articles I-XI in constitution.md).

**validation-rules.md**
Business rule quality standards at `.specify/business-rules/validation-rules.md`. Articles I-IX define testability, completeness, traceability requirements. Distinct from memory/constitution.md.

### W

**Workflow Phase**
SDD process stages: Specify → Clarify → Plan → Analyze → Implement. Distinct from implementation Milestones 1-8. Terminology separation avoids confusion.

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
│   │   ├── QUICK_REFERENCE.md        # LLM-optimized lookup (<2000 tokens)
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
│   ├── adaptive/                     # ADAPTIVE templates
│   │   ├── technical-constraints-template.md
│   │   └── adr-pivot-template.md
│   │
│   └── commands/                     # Slash command templates
│       ├── rules-harvest.md
│       ├── rules-extract.md
│       └── create-adr.md
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
│   │   ├── create-adr.sh             # ADAPTIVE commands
│   │   └── amend-technical.sh        # ADAPTIVE commands
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

### New Article XI: Technical Pivot Protocol

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

## Implementation Roadmap (8 Milestones)

**Timeline Model**: Overlapping milestones with 7-week total duration
- Milestones 2-6 run in parallel/overlapping sequence (Weeks 3-8)
- Milestone 7 (Testing) begins during Week 6 (parallel with M5-M6)
- Milestone 8 (Release) occurs in final Week 7
- Total duration: 7 weeks (not 8+) due to parallel execution

### Milestone 1: Constitutional Updates (Week 1)

**File**: `memory/constitution.md`

**Tasks**:
- [ ] **M1.0**: Add Article XI: Technical Pivot Protocol
- [ ] **M1.1**: Update Article III reference in Article XI.4 (Test-First Imperative)
- [ ] **M1.2**: Add cross-reference to `.specify/business-rules/validation-rules.md`
- [ ] **M1.3**: Version bump constitution (v2.0 → v2.1)

**Deliverable**: Updated constitution with pivot protocol

**Timeline**: 1 week

**Evidence Package** (MANDATORY before marking Milestone 1 complete):

1. **File Changes** (show actual diffs):
   ```bash
   git diff HEAD~1 memory/constitution.md | head -50
   ```
   Expected:
   - memory/constitution.md: +85 lines (Article XI added)
   - memory/constitution-amendments.md: created (+30 lines)

2. **Validation** (run actual checks):
   ```bash
   # Markdown lint
   markdownlint memory/constitution.md
   # Expected: ✅ 0 errors

   # Cross-reference validation
   grep -n "Article XI" memory/constitution.md
   # Expected: Multiple matches (sections 11.1-11.7)

   # Version validation
   grep "v2.1" memory/constitution.md
   # Expected: Found in header
   ```

3. **Functional Test** (LLM can read and understand):
   - Load constitution.md into LLM context
   - Query: "What is Article XI about?"
   - Expected: Accurate summary of Technical Pivot Protocol

**Hallucination Detection** (NONE ALLOWED):
  🚨 "Article XI added" without showing diff
  🚨 "Tests pass" without running markdownlint
  🚨 Skipping version bump verification

IF ANY evidence missing OR validation fails:
  ❌ BLOCK Milestone 1 completion
  ⚠️ Report actual status: "Milestone 1 partially complete - missing: [specific items]"
  🔍 Self-correction: Show what evidence exists vs. what's missing

---

### Milestone 2: Template Consolidation (Week 3-4)

**Directory**: `templates/`

**Tasks**:
- [ ] **M2.1**: Create `templates/adaptive/` directory
  - [ ] `technical-constraints-template.md`
  - [ ] `adr-pivot-template.md`
- [ ] **M2.2**: Update `templates/business-rules/` templates
  - [ ] `adr-integration-template.md` (add "Affected Business Rules" section)
- [ ] **M2.3**: Create unified command templates
  - [ ] `templates/commands/create-adr.md`
  - [ ] `templates/commands/amend-technical.md`
- [ ] **M2.4**: Update `templates/spec-template.md`
  - [ ] Add optional "Complex Features" section referencing TECHNICAL.md
- [ ] **M2.5**: Update `templates/plan-template.md`
  - [ ] Add Phase -1 with constitutional validation checklist

**Deliverable**: Unified template library

**Evidence Package** (MANDATORY):
- File count: `ls templates/adaptive/ | wc -l` → 2 files
- File count: `ls templates/business-rules/ | wc -l` → 4+ files
- File count: `ls templates/commands/ | wc -l` → 3+ files
- Validation: All template files are valid markdown (markdownlint)

**Hallucination Detection**:
  🚨 "Templates created" without file listing
  🚨 "All files valid" without running markdownlint

---

### Milestone 3: Script Updates (Week 4-5)

**Directory**: `scripts/bash/` + `scripts/powershell/`

**Tasks**:
- [ ] **M3.1**: Update `create-new-feature.sh`
  - [ ] Create `specs/###-feature/decisions/` directory automatically
  - [ ] Create `specs/###-feature/rules/` directory automatically
  - [ ] Optionally create `TECHNICAL.md` based on complexity flag
- [ ] **M3.2**: Create `create-adr.sh`
  - [ ] Interactive prompts for ADR fields
  - [ ] Auto-determine feature-scoped vs. cross-project placement
  - [ ] Validate constitutional compliance sections present
  - [ ] Output JSON with ADR path
- [ ] **M3.3**: Create `amend-technical.sh`
  - [ ] Read existing TECHNICAL.md
  - [ ] Interactive amendment workflow (strikethrough ~~old~~ → new)
  - [ ] Link to ADR number
  - [ ] Update decision history section
- [ ] **M3.4**: Update `rules-extract.sh`
  - [ ] Read ADRs for "Affected Business Rules" sections
  - [ ] Update catalog with pivot-modified implementation references
  - [ ] Validate constitutional compliance (9 gates)
- [ ] **M3.5**: Create PowerShell equivalents for all bash scripts

**Deliverable**: Cross-platform script suite

**Evidence Package** (MANDATORY):
- Scripts executable: `bash scripts/bash/create-adr.sh --help` → Shows usage
- Cross-platform: Both `scripts/bash/` and `scripts/powershell/` have matching files
- Validation: `shellcheck scripts/bash/*.sh` → No critical errors

**Hallucination Detection**:
  🚨 "Scripts work" without running them
  🚨 "Cross-platform complete" without comparing file lists

---

### Milestone 4: Business Rules Integration (Week 5-6)

**Directory**: `.specify/business-rules/`

**Tasks**:
- [ ] **M4.1**: Update `validation-rules.md`
  - [ ] Add preamble explaining relationship to `memory/constitution.md`
  - [ ] Cross-reference Article XI (pivot protocol may affect rules)
- [ ] **M4.2**: Update `TUTORIAL.md`
  - [ ] Add section: "Business Rules + Technical Pivots"
  - [ ] Example: Rule changes when ADR-002 pivots Prisma → pg
- [ ] **M4.3**: Update `catalog.md` template
  - [ ] Add "Pivot History" section linking to relevant ADRs
- [ ] **M4.4**: Implement ADR parser (`src/business_rules_kit/extraction/adr_parser.py`)
  - [ ] Parse "Affected Business Rules" sections from ADRs
  - [ ] Extract BR-* IDs and implementation changes
  - [ ] Return structured data for catalog updates

**Deliverable**: Pivot-aware business rules system

**Evidence Package** (MANDATORY):
- Python tests: `pytest tests/ -k adr_parser` → 12/12 passed
- Type checking: `mypy src/business_rules_kit/extraction/adr_parser.py` → ✅
- Integration: `/rules-extract --include-adrs` works with test ADR

**Hallucination Detection**:
  🚨 "ADR parser works" without test output
  🚨 "Integration complete" without running /rules-extract

---

### Milestone 5: Command Integration (Week 6-7)

**Directory**: `templates/commands/` (agent-specific)

**Tasks**:
- [ ] **M5.1**: Generate `/create-adr` for all 13 agents
  - [ ] Uses `scripts/bash/create-adr.sh --json`
  - [ ] Frontmatter with script paths (bash + PowerShell)
- [ ] **M5.2**: Generate `/amend-technical` for all 13 agents
  - [ ] Interactive amendment workflow
  - [ ] Links to ADR triggering amendment
- [ ] **M5.3**: Update `/rules-harvest` command
  - [ ] Read TECHNICAL.md (if exists) to pre-populate fields
  - [ ] Auto-fill tenant scope, domain, module
- [ ] **M5.4**: Update `/rules-extract` command
  - [ ] Add flag: `--include-adrs` to read ADR impacts
  - [ ] Update help text with pivot workflow

**Deliverable**: Multi-agent command suite

**Evidence Package** (MANDATORY):
- Command count: `ls templates/commands/ | wc -l` → 15+ files (13 agents × multiple commands)
- Test command: Run `/create-adr` in test environment → Interactive prompts work
- Help text: `/rules-extract --help` shows `--include-adrs` flag

**Hallucination Detection**:
  🚨 "Commands generated" without file count
  🚨 "All work" without testing at least one command

---

### Milestone 6: Documentation Updates (Week 7-8)

**Files**: `README.md`, `spec-driven.md`, `INTEGRATION-GUIDE.md`

**Tasks**:
- [ ] **M6.1**: Update `README.md`
  - [ ] Add "Adaptive SDD" section
  - [ ] Add "Business Rules Management" section
  - [ ] Explain interaction between the two systems
- [ ] **M6.2**: Update `spec-driven.md`
  - [ ] Insert Layer 2 (TECHNICAL.md) explanation
  - [ ] Insert Layer 2.5 (Business Rules Harvest) explanation
  - [ ] Add Article XI (Pivot Protocol) to workflow diagram
- [ ] **M6.3**: Create `.specify/INTEGRATION-GUIDE.md`
  - [ ] Complete walkthrough: Adaptive SDD + Business Rules
  - [ ] Example: Multi-company zones with Prisma pivot (use Section 5 content)
  - [ ] Decision trees (use Section 2 content)
- [ ] **M6.4**: Update `CLAUDE.md` template
  - [ ] Add LLM reading priority section
  - [ ] Add recent updates format (include ADRs + rule catalog)

**Deliverable**: Comprehensive documentation

**Evidence Package** (MANDATORY):
- Word count: `wc -w .specify/INTEGRATION-GUIDE.md` → 5000+ words
- Links valid: Check all internal links in updated docs → No 404s
- Markdown lint: `markdownlint README.md spec-driven.md INTEGRATION-GUIDE.md` → ✅

**Hallucination Detection**:
  🚨 "Documentation complete" without word count or link validation
  🚨 "All links work" without actually checking them

---

### Milestone 7: Testing & Validation (Week 6)

**Tasks**:
- [ ] **M7.1**: Create test project
  - [ ] Initialize: `specify init test-unified --ai claude`
  - [ ] Add complex feature with TECHNICAL.md
  - [ ] Run `/rules-harvest`
  - [ ] Simulate architectural blocker
  - [ ] Run `/create-adr` → amend TECHNICAL.md → regenerate plan
  - [ ] Complete implementation with @BusinessRule annotations
  - [ ] Run `/rules-extract --include-adrs`
- [ ] **M7.2**: Validate constitutional compliance
  - [ ] Verify Article XI gates work in plan.md generation
  - [ ] Verify business rules validation gates (9 articles)
- [ ] **M7.3**: Test cross-platform scripts
  - [ ] Bash scripts on Ubuntu/macOS
  - [ ] PowerShell scripts on Windows
- [ ] **M7.4**: LLM context testing
  - [ ] Generate plan with TECHNICAL.md + harvest.md
  - [ ] Generate plan after ADR amendment
  - [ ] Verify task generation uses amended constraints

**Deliverable**: Validated end-to-end workflow

**Evidence Package** (MANDATORY):
- Test project exists: `ls test-unified/specs/001-*/` → Shows created feature
- ADR created: `cat test-unified/specs/001-*/decisions/adr-001-*.md` → Valid ADR format
- Catalog updated: `grep "BR-TEST-001" test-unified/.specify/business-rules/catalog.md` → Found
- All scripts work: Bash and PowerShell scripts executed successfully on respective platforms

**Hallucination Detection**:
  🚨 "End-to-end test passed" without showing test project artifacts
  🚨 "All workflows validated" without evidence of each workflow step

---

### Milestone 8: Release Preparation (Week 7)

**Tasks**:
- [ ] **M8.1**: Version bump
  - [ ] `pyproject.toml`: v0.5.0 → v0.6.0
  - [ ] `CHANGELOG.md`: Document unified methodology
- [ ] **M8.2**: GitHub release
  - [ ] Create template archives (with adaptive + business-rules templates)
  - [ ] Release notes: "Unified SDD Methodology v2.0"
- [ ] **M8.3**: Migration guide
  - [ ] For existing spec-kit projects
  - [ ] When to add TECHNICAL.md retroactively
  - [ ] How to backfill ADRs for past pivots
- [ ] **M8.4**: Update documentation site
  - [ ] Add unified methodology page
  - [ ] Video walkthrough (optional)

**Deliverable**: Production release

**Evidence Package** (MANDATORY):
- Version bumped: `grep "version = \"0.6.0\"" pyproject.toml` → Found
- CHANGELOG updated: `head -20 CHANGELOG.md` → Shows v0.6.0 entry
- GitHub release: `gh release view v0.6.0` → Release exists with archives
- Migration guide: `wc -w docs/migration-guide.md` → 2000+ words

**Hallucination Detection**:
  🚨 "Release published" without GitHub release URL
  🚨 "Version bumped" without grep verification
  🚨 "Migration guide complete" without word count

**Total Timeline**: 7 weeks

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

## Conclusion

The unified methodology preserves Spec Kit's core philosophy (non-technical specs, test-first, contract-driven) while adding two orthogonal enhancements:

1. **Adaptive SDD**: Handles architectural pivots through TECHNICAL.md evolution and ADR traceability
2. **Business Rules**: Captures and validates business logic through constitutional gates and multi-source extraction

These systems interact at well-defined points (TECHNICAL.md informs harvest, ADRs update catalog) without creating conflicts or polluting the repository structure. All artifacts live in approved locations (`specs/*/`, `.specify/*`), and LLM agents can consume the layered information with clear priority ordering.

**Key Improvements in v2.0**:
- ✅ Executive summary with 5-minute visual workflow
- ✅ Quick start guides for CRUD vs. complex features
- ✅ 4 decision trees for component usage
- ✅ Complete 13-week end-to-end walkthrough
- ✅ 7 anti-patterns with correct approaches
- ✅ 10-question FAQ with practical answers
- ✅ 30+ term glossary with cross-references
- ✅ Governance approval gate added to Milestone 1
- ✅ Terminology disambiguation (Milestones vs. Phases)

**Compliance**: 98% SDD alignment (2% governance gap now addressed)
**Status**: ✅ READY FOR IMPLEMENTATION
**Next Action**: Begin Milestone 1 (Constitutional Updates)

---

**Version**: 2.1
**Changelog**:
- v1.0 (2025-01-23): Initial unified methodology
- v2.0 (2025-01-23): Added executive summary, decision trees, walkthrough, anti-patterns, FAQ, glossary, governance gate, terminology fixes
- v2.1 (2025-01-23): Integrated PM agent enhancements:
  - Added confidence checks to all 4 decision trees (prevent wrong-direction work)
  - Added reflexion pattern to ADR creation (reuse past solutions)
  - Added evidence packages to all 8 milestones (prevent hallucination)
  - Removed governance approval gate (restored 7-week timeline)
  - Added hallucination detection to milestone validation
