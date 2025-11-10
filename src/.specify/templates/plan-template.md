# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `../../.claude/commands/speckit.plan.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]  
**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]  
**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]  
**Testing**: [e.g., pytest, XCTest, cargo test or NEEDS CLARIFICATION]  
**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]
**Project Type**: [single/web/mobile - determines source structure]  
**Performance Goals**: [domain-specific, e.g., 1000 req/s, 10k lines/sec, 60 fps or NEEDS CLARIFICATION]  
**Constraints**: [domain-specific, e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]  
**Scale/Scope**: [domain-specific, e.g., 10k users, 1M LOC, 50 screens or NEEDS CLARIFICATION]

## Phase -1: Constitutional Validation

*GATE: Must pass before technical planning begins. Re-validate after design decisions.*

### Purpose
Ensure this feature aligns with project principles and constitutional requirements BEFORE investing time in technical research and planning.

### Constitutional Gates

Read `memory/constitution.md` and validate this feature against each article:

**Common Articles** (adapt based on your project's constitution):

#### Article I-III: Core Principles
- [ ] **Test-First Philosophy**: Can this feature be developed with tests written BEFORE implementation?
  - If NO: Justify why TDD cannot apply
- [ ] **Simplicity**: Does this feature add minimal complexity?
  - If NO: Document complexity justification in Complexity Tracking section
- [ ] **Clarity**: Are requirements clear enough to start planning?
  - If NO: Run `/speckit.clarify` before proceeding

#### Article VII: Simplicity Gate (if your constitution has this)
- [ ] **Framework Addition**: Does this require adding a new framework/library?
  - If YES: List in Complexity Tracking with justification
- [ ] **Project Addition**: Does this require a new project/service?
  - If YES: Verify this doesn't exceed constitutional limits (e.g., max 3 projects)

#### Article VIII: Anti-Abstraction Gate (if your constitution has this)
- [ ] **Custom Wrappers**: Will this introduce abstraction layers?
  - If YES: Justify why built-in APIs insufficient
- [ ] **Design Patterns**: Will this use Repository, Factory, or other patterns?
  - If YES: Document specific problem patterns solve

#### Article IX: Integration-First Gate (if your constitution has this)
- [ ] **Contract Tests**: Can contract tests be defined BEFORE implementation?
  - If YES: Contracts will be created in Phase 1
  - If NO: Explain why (e.g., prototype feature)
- [ ] **Real Dependencies**: Will tests use real databases/services (not mocks)?
  - If NO: Justify why mocks necessary
- [ ] **Phase 2.5 Test Strategy Compliance** (validate AFTER Phase 2.5 Mock & Stub Strategy):
  - Auth/Database/Business Logic: Real dependencies in integration tests? (NOT mocked)
  - Mock & Stub Strategy section: Does it violate Article V prohibitions?
    - ❌ Prohibited: Mocking auth/JWT validation in integration tests
    - ❌ Prohibited: Mocking database queries in integration tests
    - ❌ Prohibited: Mocking multi-tenancy logic in integration tests
    - ✅ Allowed: Time mocking, test data generation (Faker.js), email/SMS mocks (with justification)
  - If violations found: Document justification in Complexity Tracking OR revise test strategy

#### Article XI: Technical Pivot Protocol (if complex feature)
- [ ] **Complexity Assessment**: Is this feature complex? (Multi-tenant, performance-critical, integration-heavy)
  - If YES: TECHNICAL.md will be created during planning
  - If NO: Skip TECHNICAL.md (CRUD features)
- [ ] **Risk Flags Identified**: Are there known technical uncertainties?
  - If YES: Document [NEEDS VALIDATION] flags in TECHNICAL.md
  - If NO: Proceed with standard planning

#### [Add Project-Specific Articles]
- [ ] **[Article Name]**: [Validation question]
  - Assessment: [PASS | CONDITIONAL | FAIL]
  - Justification: [If CONDITIONAL or FAIL, explain]

### Validation Result

**Status**: [ ] ✅ PASS | [ ] ⚠️ CONDITIONAL | [ ] ❌ FAIL

**If PASS**: Proceed to technical research and planning

**If CONDITIONAL**: Document required approvals/justifications below:
- [Approval 1]: [What is conditional, who must approve, by when]
- [Approval 2]: [...]

**If FAIL**: STOP - Address constitutional violations before planning:
- [Violation 1]: [What failed, how to fix]
- [Violation 2]: [...]

### Notes
- Constitutional validation is **mandatory** before `/speckit.plan`
- Violations require either:
  1. Amend feature to comply with constitution, OR
  2. Amend constitution to reflect new reality (requires approval), OR
  3. Document exception with architect approval

### Post-Implementation Checkpoint (NEW v2.4)

**Purpose**: This constitutional validation will be **RE-CHECKED during implementation** (`/speckit.implement` Step 2.5 and Step 10.4) to prevent constitutional drift.

**Automated Enforcement Points**:

1. **Step 2.5: Constitutional Pre-Flight Check**
   - Reads this Phase -1 validation result
   - Blocks implementation if ❌ FAIL status
   - Warns if ⚠️ CONDITIONAL status

2. **Step 10.4: Constitutional Reviewer** (Quality Gate)
   - Validates TDD compliance via git history analysis
   - Scans for framework/project count violations
   - Detects prohibited patterns from constitution.md
   - Checks for unjustified abstractions

**Enforcement Criteria** (will be validated during implementation):

- [ ] **TDD Evidence**: Test commits MUST precede feature commits in git history
- [ ] **Framework Count**: New frameworks MUST NOT exceed constitution limits (scanned from package.json/requirements.txt)
- [ ] **Project Count**: New projects MUST NOT exceed constitution limits (directory structure validation)
- [ ] **Abstractions**: Custom wrappers MUST be justified in Complexity Tracking section (below)
- [ ] **Prohibited Patterns**: Code MUST NOT contain patterns from constitution.md "Prohibited Patterns" section
- [ ] **Integration Tests**: Contract/integration tests MUST exist for cross-boundary features

**Justifications Required for Implementation**:

IF you checked YES for any of these during validation above:
- **Framework Addition**: Document in Complexity Tracking section → Will be verified against package.json during Step 10.4
- **Project Addition**: Document in Complexity Tracking section → Will be verified against directory structure during Step 10.4
- **Custom Wrappers**: Document in Complexity Tracking section → Will be verified via grep patterns during Step 10.4
- **Mocks in Integration Tests**: Document in Complexity Tracking section → Will be verified via test file analysis during Step 10.4

**Constitutional Compliance Promise**:

By proceeding to implementation, the development team commits to:
✅ Following TDD (tests before implementation)
✅ Staying within framework/project limits
✅ Justifying all abstractions
✅ Avoiding prohibited patterns
✅ Using real dependencies in integration tests

**IF Implementation Violates Constitution**:
- Step 2.5: BLOCKS execution if planning validation = ❌ FAIL
- Step 10.4: Reports violations with file:line locations
- Quality Gate: Overall status = ❌ NOT READY if constitutional violations found
- Remediation: Fix violations before commit allowed

**Documentation Reference**:
- Constitution: `.specify/memory/constitution.md`
- Prohibited Patterns: `.specify/memory/constitution.md` → "Prohibited Patterns" section
- Implementation Enforcement: `/speckit.implement` Step 2.5 and Step 10.4

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
