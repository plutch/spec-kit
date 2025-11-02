# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

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
