# Spec-Kit for Claude Code - Framework Distribution Project

## Project Purpose

This is a **framework distribution project**, not a regular software project. It contains template files and Claude Code commands that are meant to be copied into OTHER projects to enable Spec-Driven Development (SDD).

## Important: Do NOT Execute Commands from src/

The `src/.claude/commands/` directory contains TEMPLATE commands that will be copied to other projects. These are NOT active commands for this project.

**Why?** If `.claude/` were at the root of this repository, Claude Code would interpret them as commands for THIS project (the spec-kit project itself), not as distributable templates for other projects.

## Working on This Project

When making changes to this framework:

1. **Edit template commands** in `src/.claude/commands/`
   - Each command file is self-contained (no @include references)
   - Files: speckit.specify.md, speckit.clarify.md, speckit.analyze.md, speckit.analyze-ux.md, speckit.plan.md, speckit.tasks.md, speckit.implement.md, speckit.constitution.md, speckit.supplement.md, speckit.reconcile.md, speckit.validate-hierarchy.md, speckit.amend-technical.md, speckit.pm.md, speckit.next.md, speckit.memory.md, speckit.status.md, speckit.validate-gap.md

2. **Update templates** in `src/.specify/templates/`
   - Specification templates
   - Planning templates
   - Task templates
   - Command templates (original sources before copying to .claude/)

3. **Modify documentation** in `src/.specify/docs/`
   - spec-driven.md - Core methodology
   - HIERARCHICAL-SPECS-GUIDE.md
   - RECONCILE-GUIDE.md
   - VALIDATION-GUIDE.md

4. **Review framework quality** with workflow-reviewer agent
   - Agent location: `src/.claude/agents/workflow-reviewer.md`
   - Purpose: Analyze workflow coherence, DX, quality framework integration
   - Use for: Pre-release reviews, post-feature reviews, quarterly health checks
   - See: `src/.claude/agents/README.md` for usage guide

5. **Test changes** by copying to a test project and running there
   ```bash
   cp -r src/{.claude,.specify} /path/to/test-project/
   cd /path/to/test-project/
   # Now test the commands
   ```

## Project Structure

```
spec-kit-claude/
├── CLAUDE.md                  # This file - explains project purpose
├── README.md                  # Framework overview and usage
├── INSTALL.md                 # Installation instructions
├── CODE_OF_CONDUCT.md         # Community guidelines
├── LICENSE                    # MIT License
└── src/                       # Distribution source (users copy this)
    ├── .claude/
    │   ├── agents/            # Framework review agents
    │   └── commands/          # 17 self-contained command files
    └── .specify/
        ├── config.example.yml     # Configuration template
        ├── business-rules/    # Business rule validation templates
        ├── memory/            # Project constitution templates
        ├── templates/         # All SDD templates
        └── docs/              # Methodology guides
```

## Development Guidelines

### Commands Are Self-Contained
All commands in `src/.claude/commands/` are self-contained markdown files. They do NOT use `@include` references. This ensures:
- Easy distribution (just copy files)
- No dependency resolution needed
- Immediate availability in target projects

### Templates Are Reference Material
The `src/.specify/templates/` directory contains:
- Original template sources (before copying to commands)
- Additional templates users can customize
- Reference documentation

### Maintain Claude Code-Specific Focus
- All commands designed for Claude Code
- No multi-LLM compatibility layers
- Claude-optimized workflows and patterns

### Installation Is Copy/Paste Only
No scripts, no CLI, no package managers required. Users simply:
1. Copy `src/.claude/` to their project
2. Copy `src/.specify/` to their project
3. Start using commands immediately

## Available Commands

Once users copy the framework to their project, they get access to:

**Core Workflow (6 commands)**:
- `/speckit.specify` - Create feature specifications with EARS format support (v2.3+)
- `/speckit.clarify` - Clarify ambiguities in specs
- `/speckit.validate-gap` - Analyze implementation feasibility (v2.3+)
- `/speckit.plan` - Generate implementation plans with requirements traceability (v2.3+)
- `/speckit.tasks` - Break down plans into tasks with EARS references (v2.3+)
- `/speckit.implement` - Execute implementation

**Advanced Workflow (5 commands)**:
- `/speckit.constitution` - Create project constitution
- `/speckit.supplement` - Create hierarchical supplementary specs
- `/speckit.reconcile` - Post-implementation gap closure
- `/speckit.validate-hierarchy` - Validate spec relationships
- `/speckit.amend-technical` - Amend architecture decisions via ADR

**Quality & Session Management (6 commands)**:
- `/speckit.analyze` - Fast quality analysis with Socratic Validation and Quick Wins (v2.2+)
- `/speckit.analyze-ux` - Fast UX quality analysis with Component System Audit (v2.2+)
- `/speckit.memory` - Create/update project memory files (v2.3+)
- `/speckit.status` - View workflow status and phase progression (v2.3+)
- `/speckit.pm` - Session context and state management
- `/speckit.next` - Smart workflow navigator with quality/risk-aware recommendations (v2.2+)

## Constitutional Enforcement Bridge (v2.4.0)

Spec-Kit v2.4.0 closes the **"Validated at Planning, Violated at Implementation"** governance gap by adding constitutional validation checkpoints throughout the workflow.

### Root Cause Addressed

**Problem**: Constitutional validation occurred during planning (`/speckit.plan` Phase -1) but was **NOT re-checked** during implementation (`/speckit.implement`). This created a 6-phase vulnerability window where violations could be introduced without detection.

**Solution**: Three enforcement points ensure constitutional compliance from planning through reconciliation.

---

### Enforcement Point 1: Constitutional Pre-Flight Check (`/speckit.implement` Step 2.5)

**Purpose**: Validate planning phase constitutional approval BEFORE code execution begins.

**Gate Logic**:
- ❌ FAIL → STOP execution, show violations, require fixes
- ⚠️ CONDITIONAL → WARN, show conditional items, ask user to confirm
- ✅ PASS → Proceed to implementation

**Token Cost**: +20-50 tokens per implementation

**Benefit**: Blocks unconstitutional implementations before code changes occur

---

### Enforcement Point 2: Constitutional Reviewer (`/speckit.implement` Step 10.4)

**Purpose**: Validate implementation adheres to constitutional principles during Quality Gate.

**Parallel Execution**: Runs alongside Code Quality (10.1), Quality/Tests (10.2), and Security (10.3) reviewers.

**Validation Tasks**:

**10.4.1 TDD Compliance Validation** 🔴 CRITICAL
- Git history analysis: Tests committed BEFORE implementation?
- Task order validation: Test tasks completed before implementation tasks?
- Red-Green-Refactor evidence: Git timestamps show TDD flow?

**10.4.2 Simplicity Validation** 🟠 MAJOR
- Framework count: Scan package.json/requirements.txt vs constitution limits
- Project count: Directory structure vs constitution limits
- Complexity tracking: New frameworks/projects justified in plan.md?

**10.4.3 Anti-Abstraction Validation** 🟡 MEDIUM
- Custom wrapper detection: Repository, Factory, Adapter patterns
- Justification check: Patterns documented in plan.md Complexity Tracking?

**10.4.4 Integration-First Validation** 🟠 MAJOR
- Contract test presence: *.integration.spec.ts, *.contract.test.* files exist?
- Real dependencies: Integration tests use Docker/Testcontainers (not mocks)?

**10.4.5 Prohibited Pattern Detection** 🔴 CRITICAL
- Read constitution.md "Prohibited Patterns" section
- Scan codebase for violations (fs.readFileSync, db.query(), etc.)
- Report: `file:line - Pattern: [name] - Reason: [why prohibited]`

**10.4.6 Technical Debt & Complexity Drift** 🟡 MEDIUM
- Cyclomatic complexity check
- TECHNICAL.md validation (for HIGH complexity features)

**Reviewer Status**:
- ✅ READY: TDD followed, simplicity maintained, no violations
- ⚠️ NEEDS REVIEW: Minor concerns, abstractions justified
- ❌ NOT READY: TDD skipped, unjustified complexity, prohibited patterns found

**Token Cost**: +200-500 tokens (parallel execution, no latency penalty)

**Benefit**: Catches violations DURING implementation via evidence-based validation (git history, file scans, pattern detection)

---

### Enforcement Point 3: Constitutional Re-Validation (`/speckit.reconcile` Step 5 Question 5)

**Purpose**: Prevent constitutional drift during surgical edits/gap closure.

**Required Evidence**:
1. **Complexity Analysis**: New frameworks, projects, abstractions introduced?
2. **TDD Compliance**: If new functionality added, were tests added first?
3. **Prohibited Patterns Scan**: Surgical edits introduce constitution violations?
4. **Simplicity Preservation**: Append-only edits maintain simplicity constraints?

**Status**:
- ✅ COMPLIANT: No new complexity, TDD followed, no violations
- ⚠️ CONDITIONAL: New complexity justified, TDD partial
- ❌ NON-COMPLIANT: Unjustified complexity, TDD skipped, violations found → STOP

**Token Cost**: +50-100 tokens per reconciliation

**Benefit**: Maintains constitutional integrity during post-implementation gap closure

---

### Template Enhancements (v2.4.0)

**1. Constitution Template** (`.specify/memory/constitution.md`)

Added **Prohibited Patterns (Machine-Readable)** section:
```markdown
### Pattern 1: Synchronous File Operations
- Pattern: fs.readFileSync, fs.writeFileSync
- Reason: Blocks event loop, violates non-blocking principle
- Detection: grep -rn "fs\.\(read\|write\)FileSync" src/
- Remediation: Use fs.promises.readFile() / fs.promises.writeFile()
```

**Purpose**: Enable automated pattern scanning during Step 10.4.5 and Step 5 Question 5

**2. Plan Template** (`.specify/templates/plan-template.md`)

Added **Post-Implementation Checkpoint** to Phase -1:
- Documents automated enforcement points (Step 2.5, Step 10.4)
- Lists enforcement criteria (TDD, framework count, prohibited patterns, etc.)
- Explains constitutional compliance promise
- References constitution.md and prohibited patterns

**Purpose**: Informs developers that constitutional validation will be re-checked during implementation

---

### Quality Gate Integration

**Aggregated Quality Gate Report** now includes 4th reviewer:

| Reviewer | Status | Critical Issues | Warnings | Details |
|----------|--------|----------------|----------|---------|
| Code Reviewer | [✅/⚠️/❌] | [N] errors | [M] warnings | Linting, type safety, spec alignment, docs |
| Quality/Tests | [✅/⚠️/❌] | [N] failures | [M] warnings | Test execution, coverage, build readiness |
| Security | [✅/⚠️/❌] | [N] vulns | [M] low-severity | Secrets, auth, OWASP, dependency audit |
| **Constitutional** | [✅/⚠️/❌] | [N] violations | [M] concerns | **TDD compliance, simplicity, prohibited patterns** |

**Overall Decision Logic** (updated):
```
IF Code = ❌ OR Quality/Tests = ❌ OR Security = ❌ OR Constitutional = ❌:
  → Overall: ❌ NOT READY (STOP workflow)

ELSE IF Code = ⚠️ OR Quality/Tests = ⚠️ OR Security = ⚠️ OR Constitutional = ⚠️:
  → Overall: ⚠️ NEEDS REVIEW (ask user to proceed or fix)

ELSE:
  → Overall: ✅ READY (proceed to Final Validation)
```

---

### Token Economy Impact (v2.4.0)

| Phase | Current (v2.3) | Proposed (v2.4) | Increase | ROI |
|-------|----------------|-----------------|----------|-----|
| Planning | 10K-15K | 10K-15K (unchanged) | 0% | N/A |
| Implementation | 20K-30K | 20.5K-31K | **+2-3%** | 300-500% |
| Reconciliation | 5K-8K | 5.05K-8.1K | **+1-2%** | 300-500% |

**Cost Increase**: +500-1,100 tokens per feature (~2-3% overall)

**ROI Justification**: Catching 1 constitutional violation prevents 10K-20K tokens in rework cycles (late-stage architectural pivots, TDD retrofitting, prohibited pattern removal).

**Break-Even**: Detecting 1 violation per 20 features = net positive ROI.

---

### Benefits for Framework Users (v2.4.0)

✅ **Pre-Flight Gate**: Blocks unconstitutional implementations before code execution
✅ **Live Validation**: Pattern detection, git history analysis during implementation
✅ **Evidence-Based**: Automated scanning (no user discipline required)
✅ **Automated Enforcement**: Constitutional violations caught at commit time
✅ **Reduced Rework**: Violations discovered during implementation, not post-merge
✅ **Architectural Discipline**: Enforces principles throughout workflow
✅ **Complete Coverage**: Planning → Implementation → Reconciliation enforcement bridge

**Workflow Visualization**:
```
/speckit.plan → Phase -1: Constitutional Validation (EXISTING ✅)
                Result: ✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL

/speckit.implement → Step 2.5: Pre-Flight Check (NEW v2.4 🔴)
                     Verify planning result → BLOCK if ❌ FAIL

                  → Step 10.4: Constitutional Reviewer (NEW v2.4 🔴)
                     TDD evidence, simplicity, prohibited patterns
                     Status: ✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY

/speckit.reconcile → Step 5 Question 5: Re-Validation (NEW v2.4 🟠)
                     Surgical edits maintain compliance?
                     Status: ✅ COMPLIANT | ⚠️ CONDITIONAL | ❌ NON-COMPLIANT
```

**Result**: **ZERO constitutional violations** reach production.

---

### Graceful Degradation & Backward Compatibility (v2.4.0)

Spec-Kit v2.4.0 is designed to work with partial constitutional infrastructure. Projects can adopt constitutional enforcement incrementally without breaking existing workflows.

#### Degradation Behavior Matrix

| Condition | Behavior | Impact | Recommendation |
|-----------|----------|--------|----------------|
| **No constitution.md** | Constitutional Reviewer runs in PARTIAL mode (TDD + simplicity + integration checks, no pattern scanning) | ⚠️ NEEDS REVIEW (reduced validation) | Optional: Create via `/speckit.constitution` |
| **constitution.md without Prohibited Patterns section** | Pattern scanning skipped (10.4.5), other checks run normally | ⚠️ PARTIAL (patterns undefined) | Add "Prohibited Patterns" section to constitution.md |
| **Plans without Phase -1** | Step 2.5 prompts user, Step 10.4 validates anyway | User decision required | Answer "yes" to proceed, or update plan via `/speckit.plan` |
| **Phase -1 incomplete** | Step 2.5 treats as incomplete planning | 🟠 INCOMPLETE | Complete Phase -1 gates in plan.md |

#### Key Principles

1. **Constitution is Optional**: Projects without constitution.md still get TDD, simplicity, and integration-first validation
2. **Never Breaks Workflows**: Graceful degradation ensures v2.4 commands work with v2.3 projects
3. **Incremental Adoption**: Create constitution when ready, update plans incrementally
4. **Clear Feedback**: Degraded modes show informative messages (ℹ️ INFO, ⚠️ WARN)

#### Migration Path

**For projects upgrading from v2.3**:
- Existing plans without Phase -1 → Step 2.5 prompts, Step 10.4 validates (no breaking change)
- Existing constitutions without Prohibited Patterns → Add section for pattern scanning
- No constitution → Works immediately with PARTIAL mode validation

See [MIGRATION.md](MIGRATION.md) for detailed upgrade instructions.

---

### Constitutional Enforcement Enhancements (v2.5.0 + v2.6.0)

**v2.5.0 (Patch)** and **v2.6.0 (Minor)** address the **"Validated at Planning, Violated at Implementation"** gap by fixing guidance in Phase 2.5 and adding test strategy validation checkpoints.

#### Root Cause: Phase 2.5 Guidance Violations (Discovered via Root Cause Analysis)

**Problem**: Phase 2.5 (Test Strategy Planning) in `/speckit.plan` **actively recommended** mocking auth/database/multi-tenancy dependencies in integration tests, directly contradicting Article V: Integration-First Testing. This created guidance that CAUSED violations rather than preventing them.

**Solution (v2.5.0 + v2.6.0)**: Three-layer fix addressing planning guidance, validation checkpoints, and automated enforcement.

---

#### Fix 1: Phase 2.5 Mock & Stub Strategy Corrections (v2.5.0 PATCH)

**Changed**: [src/.claude/commands/speckit.plan.md](src/.claude/commands/speckit.plan.md) lines 821-941

**Before (v2.4)**:
```markdown
**External APIs** (Stripe, Auth0, tax services):
- [ ] Use official SDK test mode (if available)
- [ ] Use HTTP mocking library (nock, MSW, WireMock)  ← 🔴 VIOLATION
- [ ] Create shared mock factories for common scenarios  ← 🔴 VIOLATION

**Database**:
- [ ] Use in-memory test DB for unit tests (SQLite, H2)
- [ ] Use Docker containers for integration tests (PostgreSQL, MySQL)

**Mock & Stub Strategy** (example):
- **Auth0**: Mock JWT token validation with jose library  ← 🔴 VIOLATION
```

**After (v2.5.0)**:
```markdown
⚠️ **CONSTITUTIONAL ARTICLE V COMPLIANCE REQUIRED**

**Specific Prohibitions** (Integration-First Testing):
- ❌ NEVER mock authentication logic (Auth0, JWT validation) in integration tests
- ❌ NEVER mock database queries in integration tests
- ❌ NEVER mock multi-tenancy isolation logic in integration tests

**Acceptable Test Helpers**:
- ✅ Time mocking (Date.now(), jest.useFakeTimers)
- ✅ Test data generation (Faker.js)
- ✅ Email/SMS mocks (external services with justification)

**External APIs** (Stripe, Auth0, tax services):
- [ ] Use official SDK test mode (Auth0 test tenant, Stripe test keys)
- [ ] ⚠️ CONSTITUTIONAL CONSTRAINT: Auth/business logic APIs MUST use real services in integration tests
- [ ] HTTP mocking ONLY allowed for:
  - Email/SMS services (justification: external, no test mode available)
  - Transient failures (timeout simulation for error handling tests)
  - Unit tests isolating business logic (NOT integration tests)

**Database**:
- [ ] Use in-memory test DB for unit tests ONLY (SQLite, H2) - isolated business logic
- [ ] Use Docker containers for integration tests (PostgreSQL, MySQL, Testcontainers)
- [ ] ⚠️ CONSTITUTIONAL CONSTRAINT: Integration tests MUST use real database (NOT in-memory substitutes)

**Mock & Stub Strategy** (example):
- **Auth0**: ✅ Real Auth0 test tenant + real JWT validation in integration tests, ❌ NO jose mocks
- **Database**: ✅ PostgreSQL Docker container for integration tests
```

**Benefit**: Stops recommending prohibited mocks at the source (Phase 2.5 guidance)

---

#### Fix 2: Phase -1 Article IX Enhanced Gate (v2.5.0 PATCH)

**Changed**: [src/.specify/templates/plan-template.md](src/.specify/templates/plan-template.md) lines 69-76

**Added**:
```markdown
- [ ] **Phase 2.5 Test Strategy Compliance** (validate AFTER Phase 2.5 Mock & Stub Strategy):
  - Auth/Database/Business Logic: Real dependencies in integration tests? (NOT mocked)
  - Mock & Stub Strategy section: Does it violate Article V prohibitions?
    - ❌ Prohibited: Mocking auth/JWT validation in integration tests
    - ❌ Prohibited: Mocking database queries in integration tests
    - ❌ Prohibited: Mocking multi-tenancy logic in integration tests
    - ✅ Allowed: Time mocking, test data generation, email/SMS mocks (with justification)
  - If violations found: Document justification in Complexity Tracking OR revise test strategy
```

**Benefit**: Early warning during Phase -1 that Phase 2.5 test strategy must comply with Article V

---

#### Fix 3: Phase 2.6 Constitutional Test Strategy Validation (v2.6.0 MINOR)

**Added**: [src/.claude/commands/speckit.plan.md](src/.claude/commands/speckit.plan.md) lines 974-1070 (new phase after Phase 2.5)

**Purpose**: Validate that the planned test strategy complies with Article V BEFORE implementation begins. Closes the gap between Phase 2.5 (test strategy planning) and Step 10.4 (post-implementation validation).

**Validation Steps**:
1. **Read Constitution Article V** (if exists): Extract prohibited patterns, allowed exceptions
2. **Scan Phase 2.5 Mock & Stub Strategy**: Check for prohibited patterns (mocked auth, mocked DB, mocked tenant context)
3. **Generate Validation Report**: Table with dependency → approach → requirement → status
4. **Update Phase -1 Result**: If violations found, update Phase -1 to ⚠️ CONDITIONAL or ❌ FAIL

**Validation Report Format**:
```markdown
## Constitutional Test Strategy Validation (Article V)

**Article V Compliance**: [✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL]

| Dependency | Planned Approach | Article V Requirement | Status |
|------------|------------------|----------------------|--------|
| Auth0/Auth | [from Phase 2.5] | Real test tenant in integration tests | [✅/⚠️/❌] |
| Database | [from Phase 2.5] | Docker containers for integration tests | [✅/⚠️/❌] |
| Multi-tenancy | [from Phase 2.5] | Real tenant filtering (NOT mocked) | [✅/⚠️/❌] |

**Violations Found**: [N]
- Violation 1: Mock Auth0 with jose library → Should use real Auth0 test tenant
```

**Token Cost**: +200-300 tokens per planning session

**Benefit**: Blocks non-compliant test strategies BEFORE implementation (not at Step 10.4)

---

#### Fix 4: Article V Mock Patterns in Constitution Template (v2.6.0 MINOR)

**Added**: [src/.specify/memory/constitution.md](src/.specify/memory/constitution.md) lines 106-177 (Prohibited Patterns section)

**New Patterns**:
- **Pattern 4: Mocked Auth in Integration Tests** (jest.mock('auth0'), mockJWT())
- **Pattern 5: Mocked Database in Integration Tests** (MockDatabase(), InMemoryDB())
- **Pattern 6: Mocked Tenant Context** (mockTenantContext(), setCurrentTenant(fakeId))

**Example**:
```markdown
### Pattern 4: Mocked Auth in Integration Tests (Article V - v2.6)

- **Pattern**: `jest.mock('auth0')`, `mockJWT()`, `jest.spyOn(authService, 'validate')`
- **Reason**: Violates Article V - auth MUST use real Auth0 test tenant in integration tests
- **Detection**: `grep -rn "jest\.mock.*auth\|mockJWT" tests/integration/`
- **Remediation**: Use real Auth0 test tenant with test credentials
```

**Benefit**: Enables automated pattern scanning at Step 10.4.5 (Prohibited Pattern Detection)

---

#### Fix 5: Step 2.5 Enhanced Pre-Flight Check (v2.6.0 MINOR)

**Changed**: [src/.claude/commands/speckit.implement.md](src/.claude/commands/speckit.implement.md) lines 99-131

**Added**: Phase 2.6 validation check after Phase -1 check

**Gate Logic**:
```
IF plan.md contains "Phase 2.6: Constitutional Test Strategy Validation":

  IF validation result == ❌ FAIL:
    → 🔴 ERROR: "Test strategy violates Article V: Integration-First Testing"
    → Show violations from plan.md Phase 2.6
    → STOP execution

  IF validation result == ⚠️ CONDITIONAL:
    → 🟠 WARN: "Test strategy has conditional Article V approval"
    → Ask: "Proceed with conditional test strategy? (yes/no)"

  IF validation result == ✅ PASS:
    → ✅ INFO: "Test strategy complies with Article V"
    → Continue to next step
```

**Benefit**: Defense-in-depth - blocks implementation if Phase 2.6 found violations

---

#### Token Economy Impact (v2.5.0 + v2.6.0)

| Phase | v2.4.0 | v2.6.0 | Increase | ROI |
|-------|--------|--------|----------|-----|
| Planning | 10K-15K | 10.2K-15.3K | **+200-300** | Phase 2.6 validation |
| Implementation | 20.5K-31K | 20.5K-31K | **0** | Pre-flight check reuses Phase 2.6 result |
| Reconciliation | 5.05K-8.1K | 5.05K-8.1K | **0** | No change |

**Total Cost Increase**: +200-300 tokens per feature (~1.5-2% overall)

**ROI**: 400-600% (prevents 10K-20K token rework cycles from late-stage test strategy violations)

---

#### Benefits for Framework Users (v2.5.0 + v2.6.0)

✅ **Corrected Guidance**: Phase 2.5 no longer recommends prohibited mocks
✅ **Early Validation**: Phase -1 checks test strategy compliance
✅ **Mid-Planning Checkpoint**: Phase 2.6 validates before implementation begins
✅ **Automated Enforcement**: Step 10.4.5 scans for Article V mock patterns
✅ **Defense-in-Depth**: Step 2.5 pre-flight blocks if Phase 2.6 failed
✅ **Complete Coverage**: Planning → Mid-Planning → Pre-Implementation → Implementation enforcement

**Workflow Visualization (Updated)**:
```
/speckit.plan → Phase -1: Constitutional Validation (v2.4 ✅)
                  Article IX: Integration-First Gate (v2.5 🟠 ENHANCED)
                    ↓ validates Phase 2.5 test strategy
                Result: ✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL

              → Phase 2.5: Test Strategy Planning (v2.5 🟠 FIXED GUIDANCE)
                  Mock & Stub Strategy with Article V compliance warnings
                    ↓
              → Phase 2.6: Test Strategy Validation (v2.6 🔴 NEW)
                  Validates Phase 2.5 against Article V prohibitions
                  Updates Phase -1 result if violations found
                Result: ✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL

/speckit.implement → Step 2.5: Pre-Flight Check (v2.4 + v2.6 🟠 ENHANCED)
                     Verify Phase -1 result → BLOCK if ❌ FAIL
                     Verify Phase 2.6 result → BLOCK if ❌ FAIL (v2.6 NEW)

                  → Step 10.4: Constitutional Reviewer (v2.4 + v2.6 ✅)
                     Step 10.4.4: Integration-First Validation
                     Step 10.4.5: Prohibited Pattern Detection (uses Article V patterns v2.6)
                     Status: ✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY

/speckit.reconcile → Step 5 Question 5: Re-Validation (v2.4 ✅)
                     Surgical edits maintain compliance?
```

**Result**: **ZERO test strategy violations** reach implementation.

---

## Memory System & Phase Tracking (v2.3.0)

Spec-Kit v2.3.0 introduces persistent project memory and workflow phase tracking inspired by [cc-sdd-Superclaude](https://github.com/symbioquine/cc-sdd-Superclaude):

### Project Memory System (`/speckit.memory`)
- **Persistent AI Context**: Memory files survive session resets and provide long-term project knowledge
- **Three Inclusion Modes**:
  - **Always**: Core project context loaded in every command ([constitution.md](../src/.specify/memory/constitution.md))
  - **Conditional**: Pattern-based loading (e.g., [api-standards.md](../src/.specify/memory/api-standards.md) when in `src/api/**/*`)
  - **Manual**: On-demand via `@filename` syntax ([deployment-runbook.md](../src/.specify/memory/deployment-runbook.md))
- **Memory Files Created**:
  1. `constitution.md` - Core project principles, tech stack, file organization, standards
  2. `api-standards.md` - REST API conventions, request/response formats, versioning
  3. `testing-approach.md` - Testing strategy, coverage requirements, TDD practices
  4. `deployment-runbook.md` - Deployment procedures, infrastructure, incident response
- **Git-Aware Updates**: Preserves user customizations during updates
- **Benefit**: Eliminates repetitive context gathering, maintains consistency across sessions

### Phase Tracking & Approval Gates (`spec-metadata.json`)
- **Workflow State Machine**: Tracks progression through phases (specification → gap_analysis → planning → tasks → implementation → reconciliation → complete)
- **Approval Gates**: Commands check if previous phases are approved before proceeding
- **Auto-Approval Support**: Use `-y` flag to skip approval prompts (trusted workflows)
- **Metadata Tracking**:
  ```json
  {
    "version": "2.3.0",
    "feature_name": "oauth-integration",
    "phase": "planning",
    "approvals": {
      "specification": { "generated": true, "approved": true, "timestamp": "2025-11-05T10:30:00Z" },
      "gap_analysis": { "generated": true, "approved": true, "timestamp": "2025-11-05T11:00:00Z" },
      "planning": { "generated": true, "approved": false, "timestamp": "2025-11-05T12:00:00Z" }
    },
    "metadata": {
      "risk_level": "MEDIUM",
      "overall_quality": 82
    }
  }
  ```
- **Status Visibility**: Use `/speckit.status` to view current phase and approval status
- **Benefit**: Prevents phase skipping, enforces workflow discipline, tracks progress

### Gap Analysis Command (`/speckit.validate-gap`)
- **Feasibility Assessment**: Analyzes existing codebase vs new requirements before planning
- **Reusable Component Identification**: Maps what can be leveraged vs what needs creation
- **Implementation Options**: Presents 3 approaches (Extend, Create, Hybrid) with trade-offs
- **Technical Research Requirements**: Identifies unknowns that need investigation
- **Complexity Estimation**: T-shirt sizing (S/M/L/XL) with effort estimates
- **Workflow Position**: Between specification and planning (critical for HIGH-risk features)
- **Benefit**: Reduces surprises during implementation, better planning accuracy

### EARS Requirements Format
- **Structured Acceptance Criteria**: Easy Approach to Requirements Syntax
- **Four Core Patterns**:
  - `WHEN [event] THEN [system] SHALL [response]` - Event-driven
  - `IF [condition] THEN [system] SHALL [response]` - State-driven
  - `WHILE [duration] THE [system] SHALL [behavior]` - Continuous
  - `WHERE [context] THE [system] SHALL [behavior]` - Contextual
- **Benefits**: Testable, unambiguous, measurable requirements
- **Requirements Traceability**: Link requirements (REQ-XX-1.1) through spec → plan → tasks → tests
- **Guide**: See [requirements-ears.md](../src/.specify/templates/requirements-ears.md)
- **Example**:
  ```markdown
  REQ-AUTH-1.1: WHEN a user submits valid credentials THEN the Authentication Service SHALL grant access within 2 seconds
  REQ-AUTH-1.2: IF credentials are invalid THEN the Authentication Service SHALL reject access and log the attempt
  ```

### Status Dashboard (`/speckit.status`)
- **Phase Progression Visualization**: See where you are in the workflow with emoji indicators (✅/⏳/⬜/🔴)
- **Approval Status**: Track which phases are complete and approved
- **Quality Metrics**: View overall quality score and critical issues count
- **Workflow Readiness**: Determine if you can proceed to next phase
- **File Detection**: Lists all spec files with status and modification dates
- **Memory Files Active**: Shows which memory files are loaded
- **Smart Recommendations**: Suggests next command based on current state and quality
- **Benefit**: Complete visibility into workflow status at a glance

### Enhanced Workflow (v2.3)
```
/speckit.memory              # ← NEW: Create project memory (one-time setup)
    ↓
/speckit.specify              # Create specification → spec-metadata.json created
    ↓
/speckit.status               # ← NEW: Check workflow status
    ↓
/speckit.clarify              # Clarify ambiguities (optional)
    ↓
/speckit.validate-gap         # ← NEW: Assess feasibility (critical for HIGH-risk)
    ↓
/speckit.plan                 # Generate implementation plan with req traceability
    ↓
/speckit.status               # Check approval status before tasks
    ↓
/speckit.tasks                # Break down into tasks (optional) with EARS refs
    ↓
/speckit.implement            # Execute implementation (checks approval gates)
    ↓
/speckit.reconcile            # Post-implementation gap closure
    ↓
/speckit.status               # Verify feature complete
```

### Key Integration Points (v2.3)
1. **Memory System** → **All Commands**: Provides persistent context across sessions
2. **Gap Analysis** → **Planning**: Informs implementation approach and complexity
3. **EARS Requirements** → **Tasks** → **Tests**: Full traceability from spec to code
4. **Phase Tracking** → **Approval Gates**: Prevents proceeding with incomplete work
5. **Status Dashboard** → **Smart Recommendations**: Context-aware next steps

### Token Economy Impact (v2.3.0)
- **Cost Increase**: 10-15% for memory loading and metadata tracking
- **ROI**: 300-500% from persistent context (eliminates repetitive codebase analysis)
- **Justification**: Gap analysis prevents 1-2 week implementation delays from surprises

### Benefits for Framework Users (v2.3)
- ✅ **Persistent Knowledge**: AI "remembers" project context across sessions
- ✅ **Workflow Discipline**: Phase gates prevent incomplete implementations
- ✅ **Better Planning**: Gap analysis reduces surprises and improves estimates
- ✅ **Requirements Clarity**: EARS format ensures testable, unambiguous specs
- ✅ **Full Visibility**: Status dashboard shows exactly where you are in workflow
- ✅ **Smart Navigation**: Context-aware recommendations for next steps

## Quality Framework Enhancements (v2.2.0)

Spec-Kit v2.2.0 introduces evidence-based validation and component system auditing:

### Socratic Validation Questions (All Quality Commands)
- **4-Question Filter**: Every finding must answer "What user goal blocked?", "How many users affected?", "What is cost?", "Why this severity?"
- **Applied To**: `/speckit.analyze`, `/speckit.analyze-ux`, `/speckit.clarify --deep`
- **Benefit**: 40-60% reduction in false positives - only report issues with clear user impact

### Quick Wins Identification (High-Impact, Low-Effort Improvements)
- **Criteria**: High impact + 1-2 days effort + actionable + independent
- **Applied To**: `/speckit.analyze`, `/speckit.analyze-ux`, `/speckit.reconcile`
- **Output**: 3-5 quick wins per command with impact/effort estimates
- **Benefit**: Builds momentum, prioritizes by ROI, demonstrates immediate value

### Component System Audit (`/speckit.analyze-ux`)
- **Framework Support**: React/MUI, React/Chakra, Angular/Kendo (with MCP), Angular/Material, Vue/Vuetify, Generic
- **Analyzes**: Component consistency, design token adherence, reuse score, framework best practices
- **Templates**: 7 framework-specific audit templates in `.specify/templates/component-audit-templates/`
- **MCP Integration**: Kendo UI queries `kendo-angular-assistant` for official documentation (graceful fallback)
- **UI Design Quality**: 5 new dimensions (Visual Clarity, Component Consistency, Visual Hierarchy, Responsive Design, Design System Adherence)
- **Benefit**: Prevents custom component proliferation, enforces design system consistency

### Cross-Lens Conflicts Detection (`/speckit.clarify --deep`)
- **Detects**: Conflicts where expert recommendations contradict (Requirements vs Architecture, UX vs Performance, etc.)
- **Output**: Conflict description + tension + resolution options + recommended resolution
- **Benefit**: Reveals critical tradeoffs early, prevents late-stage architectural pivots

### Smart Recommendations (`/speckit.next`)
- **Quality-Driven**: Checks quality.json for overall_quality, critical_issues count
- **Risk-Aware**: Parses Risk Assessment from spec.md (HIGH/MEDIUM/LOW)
- **Context-Aware**: Detects UI-SPEC.md, hierarchical specs, gap reports
- **Quality-Gated Transitions**: Blocks/warns/suggests based on quality scores and risk levels
- **Benefit**: Prevents proceeding with low-quality or incomplete specs

### Token Economy Impact (v2.2.0)
- **Cost Increase**: 15-25% across enhanced commands
- **ROI**: 200-400% from prevented false positives, avoided rework, and architectural pivots
- **Justification**: Socratic validation prevents 5K-10K token waste per false positive, component audit saves 20K-50K in maintenance

## Workflow Integration & Phase Tracking (v2.3.0)

Spec-Kit v2.3.0 introduces unified state management, EARS requirements format, and comprehensive phase tracking:

### Memory System & State Management (`/speckit.memory`, `/speckit.status`, `/speckit.pm`, `/speckit.next`)
- **Three Inclusion Modes**: Always (constitution.md), Conditional (api-standards.md, testing-approach.md), Manual (deployment-runbook.md)
- **Persistent AI Context**: Project memory files loaded across sessions
- **Unified State Management**: spec-metadata.json as primary source, state.json as fallback
- **Phase Mapping**: Automatic conversion between lowercase (v2.3) and uppercase (v2.1) phase formats
- **Dual State Tracking**: Maintains both formats during v2.3 migration period

### EARS Requirements Format (`/speckit.specify`, `/speckit.plan`, `/speckit.tasks`)
- **Four Core Patterns**: WHEN [event] THEN [system] SHALL [response], IF [condition], WHILE [duration], WHERE [context]
- **Requirement IDs**: REQ-{CATEGORY}-{NUMBER}.{SUB} format (e.g., REQ-AUTH-1.1, REQ-DASH-2.1)
- **Full Traceability**: spec.md → plan.md → tasks.md → tests
- **Unambiguous Requirements**: Structured patterns eliminate interpretation ambiguity

### Phase Tracking & Approval Gates (`/speckit.specify`, `/speckit.plan`, `/speckit.tasks`, `/speckit.implement`, `/speckit.reconcile`)
- **Six Workflow Phases**: specification → gap_analysis → planning → tasks → implementation → reconciliation
- **Approval Gates**: Commands check spec-metadata.json before proceeding to prevent skipping steps
- **Phase Metadata**: Timestamps, approval status, quality scores tracked per phase
- **Workflow Validation**: /speckit.next validates prerequisites before recommending next command

### Gap Analysis & Feasibility (`/speckit.validate-gap`)
- **Feasibility Analysis**: Analyzes codebase before planning for reusable components
- **Three Implementation Options**: Recommended, Alternative, Minimal (with T-shirt sizing: S/M/L/XL)
- **Risk-Based Activation**: HIGH-risk features (score ≥8) automatically trigger gap analysis
- **Codebase Context**: Identifies existing patterns, services, utilities before planning

### Live Context Gathering (`/speckit.reconcile`, `/speckit.validate-hierarchy`)
- **Git Diff Execution**: Bash tool used for live context (not static examples)
- **Actual Implementation Evidence**: Shows what was ACTUALLY implemented vs specified
- **Silent Gap Detection**: Identifies gaps not in user's gap report via git diff
- **Auto-Detection**: Feature directory auto-detected from git branch

### Token Economy Impact (v2.3.0)
- **Cost Increase**: 10-20% across workflow commands (metadata operations, state management)
- **ROI**: 300-500% from prevented workflow skips, improved traceability, reduced rework
- **Justification**: Phase tracking prevents 10K-20K token waste from incorrect workflow sequencing, EARS format eliminates 5K-10K ambiguity clarification cycles

## Quality Framework Enhancements (v2.1)

Spec-Kit v2.1 introduced comprehensive quality, security, and testing features throughout the workflow:

### Risk Assessment Framework (`/speckit.specify`)
- **6-Criteria Risk Scoring** (0-12 scale): Data sensitivity, access control, external integration, performance, complexity, business impact
- **Risk Classifications**: 🔴 HIGH (8-12), 🟠 MEDIUM (4-7), 🟢 LOW (0-3)
- **Output**: Risk assessment table in spec.md with mitigation requirements

### Enhanced Severity System (All Commands)
- **Standardized Color-Coded Severity**: 🔴 CRITICAL, 🟠 MAJOR, 🟡 MEDIUM, 🟢 LOW, ℹ️ INFO
- **Applied Across**: analyze, analyze-ux, clarify (all modes), reconcile
- **Severity Discipline**: Reserve CRITICAL for actual data/security risks

### Edge Case Detection Mode (`/speckit.clarify --edge-cases`)
- **6 Detection Categories**: Boundary conditions, error conditions, state transitions, data quality, race conditions, multi-tenant isolation
- **Production Resilience**: Timeout handling, retry logic, idempotency, crash recovery
- **Output**: Edge case questions + Production Readiness Assessment

### UX Quality Framework (`/speckit.specify`, `/speckit.clarify --ux`, `/speckit.analyze-ux`)
- **5-Dimensional UX Scoring**: Usability, Efficiency, Accessibility, Error Prevention, Delight (0-10 scale each)
- **Expert Methodologies**: Nielsen, Norman, Cooper, Wroblewski, Krug, WCAG 2.1 AA
- **UX Lens Mode**: 4-pass multi-perspective review (Usability, Task Efficiency, Interaction Patterns, Accessibility)
- **Quick Wins**: High-impact, low-effort improvement recommendations

### Test Strategy Planning (`/speckit.plan` - Phase 2.5)
- **Test Coverage by Layer**: Unit, integration, E2E, performance tests
- **Risk-Based Prioritization**: Maps to Risk Assessment scores (🔴/🟠/🟡)
- **Edge Case Test Planning**: Boundary, error, state, concurrency test scenarios
- **Mock/Stub Strategy**: External APIs, database, clock, file system
- **Coverage Thresholds**: Critical paths ≥90%, business logic ≥80%, overall ≥75%

### Security & Compliance Review (`/speckit.plan` - Phase 3)
- **OWASP Top 10 Validation**: Comprehensive checklist for all 10 vulnerability categories
- **Authentication/Authorization**: Token validation, RBAC/ABAC, principle of least privilege
- **Data Protection**: Encryption, PII handling, data retention, GDPR/CCPA/PCI-DSS compliance
- **Input Validation**: Schema validation, SQL injection prevention, XSS protection
- **Audit Logging**: Authentication, authorization, data access, financial transactions
- **Blocking Criteria**: Critical OWASP vulnerabilities, missing encryption, auth gaps

### Implementation Quality Gate (`/speckit.implement` - Step 9)
- **6-Layer Validation**: Code quality, security, test coverage, spec alignment, documentation, build readiness
- **Security Checks**: No hardcoded secrets, input validation, auth/authz, dependency audit
- **Test Coverage**: All tests pass, coverage thresholds met, high-risk requirements tested
- **Gate Statuses**: 🟢 READY TO COMMIT, 🟡 NEEDS ATTENTION, 🔴 BLOCKED
- **Output**: Quality Gate Report with findings and next steps

### Key Integration Points
1. **Risk Assessment (specify)** → **Test Priority (plan)** → **Test Coverage Validation (implement)**
2. **Edge Case Detection (clarify)** → **Edge Case Tests (plan)** → **Test Execution (implement)**
3. **Security Review (plan)** → **Security Validation (implement)**
4. **UX Scoring (specify)** → **UX Lens Mode (clarify)** → **UX Analysis (analyze-ux)**

### Benefits for Framework Users
- ✅ **Production Readiness**: Systematic quality gates prevent incomplete implementations
- ✅ **Security by Design**: OWASP Top 10 validation built into workflow
- ✅ **Risk-Driven Development**: High-risk features get extra scrutiny automatically
- ✅ **Comprehensive Testing**: Risk-based test prioritization from specification phase
- ✅ **Compliance Assurance**: GDPR, CCPA, PCI-DSS, HIPAA requirements mapped
- ✅ **UX Excellence**: Evidence-based usability validation with expert methodologies

## Testing Changes

### Manual Testing Workflow

1. **Make changes** to commands or templates
2. **Copy to test project**:
   ```bash
   # From spec-kit-claude directory
   rm -rf /path/to/test-project/.claude
   rm -rf /path/to/test-project/.specify
   cp -r src/.claude /path/to/test-project/
   cp -r src/.specify /path/to/test-project/
   ```
3. **Test in Claude Code**:
   - Open test project in Claude Code
   - Run commands to verify functionality
   - Check for errors or unexpected behavior
4. **Iterate** until satisfied

### What to Test

- **Command execution**: Do commands run without errors?
- **Output quality**: Are generated files correct and complete?
- **Workflow integration**: Do commands chain together properly?
- **Documentation clarity**: Are instructions clear and actionable?

## Contributing

When contributing to this framework:

1. **Maintain self-contained commands** - No @include references
2. **Test thoroughly** - Copy to test project and verify
3. **Update README.md** - Document any new features or changes
4. **Keep it Claude-only** - No multi-LLM references
5. **Preserve structure** - Don't break the src/ distribution pattern

## Distribution

Users receive this framework as:
- **GitHub repository** - Clone and copy `src/` contents
- **Zip/tarball** - Download and extract `src/` contents
- **Direct copy** - Copy `src/{.claude,.specify}` from existing project

No installation process, no dependencies, no configuration required.

## FAQ

### Q: Why is src/ at the root instead of distribution in root?

**A:** Because Claude Code interprets `.claude/` at the root as active commands for THIS project. By keeping distribution files in `src/`, we avoid confusion and make it clear these are TEMPLATES to be copied elsewhere.

### Q: Can I use these commands in the spec-kit project itself?

**A:** No, these commands are designed for target projects (where you copy them). This project doesn't need spec-driven development because it's a template distribution project.

### Q: How do I update commands after copying to my project?

**A:** Either:
1. **Manual edits**: Edit commands directly in your project's `.claude/` directory
2. **Re-copy**: Delete `.claude/` and `.specify/`, copy fresh from spec-kit updates

### Q: Why no @include references?

**A:** Self-contained commands are easier to:
- Distribute (no dependency resolution)
- Customize (users can edit without breaking includes)
- Understand (everything in one file)
- Version (clear what changed in each command)

### Q: What is the workflow-reviewer agent for?

**A:** The workflow-reviewer agent analyzes the spec-kit framework itself (not user projects) for:
- **Workflow coherence**: State machine integrity, review gate consistency, command chaining
- **Developer experience**: Token economy, session continuity, error handling
- **Quality framework integration**: Risk → Test → Security flow validation
- **Process clarity**: Documentation alignment, simplification opportunities
- **Resilience**: Error recovery, graceful degradation, operational robustness

Use it for:
- Pre-release reviews (before major/minor versions)
- Post-feature reviews (after adding new commands or enhancements)
- Quarterly health checks (continuous improvement)

See `src/.claude/agents/README.md` for detailed usage instructions.

---

**Spec-Kit Version**: 2.3.0
**Target**: Claude Code
**License**: MIT
**Compatibility**: Claude Code 0.7.0+
