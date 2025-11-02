---
description: End-to-end workflow orchestration that coordinates multiple personas and commands for complex feature development.
---

## Overview

`/speckit.workflow` orchestrates complete Spec-Driven Development workflows by coordinating multiple personas (architect, analyzer, etc.) and automating the specify → clarify → plan → tasks sequence with intelligent decision gates.

**Adapted from**: [workflow.md](../../SuperClaude_Framework/superclaude/commands/workflow.md)

## Usage

```bash
# Generate complete workflow from PRD
/speckit.workflow @PRD.md --strategy systematic --depth deep

# Quick workflow (exploratory)
/speckit.workflow "Build user authentication" --strategy fast

# Resume existing workflow
/speckit.workflow --resume feature/000001-auth
```

## Personas

### Available Personas

| Persona | Role | Responsibilities |
|---------|------|------------------|
| **Architect** | System Design | Architecture patterns, tech stack, scalability |
| **Analyzer** | Requirements | Requirement consistency, completeness, traceability |
| **Frontend** | UI/UX | User experience, interface design, accessibility |
| **Backend** | API/Data | Data architecture, API design, integration |
| **Security** | Security | Threat modeling, auth/authz, compliance |
| **DevOps** | Operations | Deployment, monitoring, infrastructure |
| **Tester** | Quality | Test strategy, quality gates, risk assessment |

### Persona Coordination

Personas collaborate at key decision points:

```yaml
Specification Phase:
  Lead: Analyzer
  Support: Frontend, Backend (domain modeling)

Clarification Phase:
  Lead: Analyzer
  Support: Security (for security questions), Architect (for technical scope)

Planning Phase:
  Lead: Architect
  Support: Backend, Frontend, DevOps (deployment concerns)

Task Breakdown:
  Lead: Architect
  Support: All personas (identify parallel work)

Implementation:
  Rotate: Based on task type (UI → Frontend, API → Backend, etc.)
```

## Strategies

### Systematic Strategy (Default)

**Thoroughness**: High
**Speed**: Moderate
**Best for**: Production features, complex systems

**Flow**:
1. Comprehensive specification with discovery dialogue
2. Multi-round clarification (up to 5 questions)
3. Detailed planning with constitutional gates
4. Granular task breakdown with dependencies
5. Phased implementation with reviews

### Fast Strategy

**Thoroughness**: Moderate
**Speed**: Fast
**Best for**: Prototypes, spikes, simple features

**Flow**:
1. Basic specification (skip discovery if clear)
2. Single-round clarification (top 3 issues only)
3. Lightweight plan (skip some gates)
4. Coarse-grained tasks
5. Rapid implementation

### Deep Strategy

**Thoroughness**: Very High
**Speed**: Slow
**Best for**: Critical features, security-sensitive, compliance

**Flow**:
1. Comprehensive specification with extended discovery
2. Multi-phase clarification with expert review
3. Detailed planning with all constitutional gates
4. Security and compliance review
5. Fine-grained tasks with risk mitigation
6. Incremental implementation with continuous review

## Execution Flow

### Phase 0: Initialization

```yaml
Wave 1 - Context Load (Parallel):
  1. Bash: git rev-parse --abbrev-ref HEAD
  2. Read: .specify/memory/pm_context.md
  3. Read: .specify/memory/constitution.md
  4. Parse: $ARGUMENTS for PRD file or feature description

Checkpoint:
  ❓ Input valid? (PRD file or description provided)
  ❓ Constitution loaded?
  ❓ Strategy specified? (default: systematic)

Decision:
  IF --resume flag:
    → Load existing feature state
    → Continue from last phase
  ELSE:
    → Start new workflow
```

### Phase 1: Specification

```yaml
Persona: Analyzer (Lead)

Actions:
  1. IF PRD file provided:
       → Read and parse PRD
       → Extract requirements
     ELSE:
       → Use feature description from args

  2. Activate: /speckit.specify [description]
     → Let specify command handle vagueness detection
     → Let specify command create branch and spec.md

  3. Analyzer Review:
     → Check for missing requirements
     → Verify traceability to business goals
     → Flag any obvious gaps

Checkpoint:
  ❓ Spec created successfully?
  ❓ All requirements captured?
  ❓ Ready for clarification?

Output: specs/[FEATURE]/spec.md
```

### Phase 2: Clarification

```yaml
Persona: Analyzer (Lead), Security (Support)

Actions:
  1. Scan spec for ambiguities
  2. IF ambiguities found:
       → Activate: /speckit.clarify
       → Follow systematic clarification flow
  3. IF security-related feature:
       → Security Persona: Additional security questions
       → Threat modeling considerations

Decision Gates:
  - IF strategy == "fast" AND clarifications <= 3:
      → Skip additional rounds, proceed
  - IF strategy == "deep":
      → Activate: /speckit.review --focus requirements
      → Address review findings
  - ELSE:
      → Standard clarification flow

Output: Updated specs/[FEATURE]/spec.md with clarifications
```

### Phase 3: Planning

```yaml
Persona: Architect (Lead), Backend/Frontend (Support)

Actions:
  1. Architect: Design system architecture
     → Consider scalability, maintainability
     → Choose patterns and frameworks
  2. Backend: Data architecture design
     → Entities, relationships, storage
  3. Frontend: UI/UX considerations (if applicable)
  4. DevOps: Deployment and infrastructure planning

  5. Activate: /speckit.plan [tech stack details]

  6. Constitutional Gate Validation:
     - Library-first (Article I)
     - CLI interface (Article II)
     - Test-first (Article III)
     - Simplicity (Article VII)
     - Anti-abstraction (Article VIII)
     - Integration-first testing (Article IX)

Decision Gates:
  - IF any gate fails:
      → Architect: Revise plan
      → Re-run /speckit.plan
  - IF strategy == "deep":
      → Activate: /speckit.review --focus architecture --mode discussion
      → Incorporate expert recommendations

Output: specs/[FEATURE]/plan.md
```

### Phase 4: Task Breakdown

```yaml
Persona: Architect (Lead), All (Support)

Actions:
  1. Activate: /speckit.tasks

  2. Dependency Analysis:
     → Identify task dependencies
     → Mark parallelizable tasks

  3. Persona Assignment:
     → Frontend tasks → Frontend persona
     → Backend tasks → Backend persona
     → Security tasks → Security persona
     → Infrastructure → DevOps persona

  4. Risk Mitigation:
     → Tester: Identify high-risk areas
     → Prioritize testing for risky components

Output: specs/[FEATURE]/tasks.md with [P] markers and persona assignments
```

### Phase 5: Implementation Orchestration

```yaml
Persona: Rotating (based on task type)

Actions:
  1. Load tasks from tasks.md
  2. Identify first batch (parallel-safe tasks)
  3. Assign to appropriate personas:
     - UI components → Frontend
     - API endpoints → Backend
     - Auth logic → Security
     - Infrastructure → DevOps

  4. For each task:
     IF implements interface/contract:
       → Contract tests first (Test-first)
     THEN:
       → Implementation
     THEN:
       → Integration tests

  5. Progress tracking:
     → Update state.json after each task
     → Mark tasks complete in tasks.md

Coordination:
  - Frontend and Backend work in parallel on independent tasks
  - Integration points require synchronization
  - DevOps prepares infrastructure while code is being written

Output: Implemented feature across all layers
```

### Phase 6: Validation & Review

```yaml
Persona: Tester (Lead), Security (Support)

Actions:
  1. Tester: Execute test suite
     → Run unit tests
     → Run integration tests
     → Run end-to-end tests

  2. Security: Security review (if sensitive feature)
     → OWASP Top 10 check
     → Threat model validation

  3. IF strategy == "deep":
       → Full expert review panel
       → /speckit.review --mode critique

  4. Quality Gates:
     - All tests passing?
     - Code coverage >= target?
     - Security vulnerabilities addressed?
     - Performance metrics met?

Output: Test results, security report, final review
```

## Workflow State Management

State stored in `.specify/memory/workflows/[FEATURE].json`:

```json
{
  "feature": "000001-auth-system",
  "workflow_id": "wf-001-20250126",
  "strategy": "systematic",
  "depth": "normal",
  "current_phase": "planning",
  "phases_completed": ["specification", "clarification"],
  "personas_active": {
    "architect": "active",
    "analyzer": "completed",
    "backend": "pending"
  },
  "decision_gates": {
    "simplicity_gate": "passed",
    "anti_abstraction_gate": "passed"
  },
  "blockers": [],
  "next_actions": [
    "Complete plan.md",
    "Run constitutional gate validation"
  ],
  "started_at": "2025-01-26T10:00:00Z",
  "updated_at": "2025-01-26T14:30:00Z"
}
```

## Resume Workflow

```bash
/speckit.workflow --resume feature/000001-auth

# Output:
📍 Resuming Workflow: 000001-auth-system

Current Phase: PLANNING (60%)
Last Action: Architect completed initial plan

Next Steps:
1. Validate constitutional gates
2. Generate tasks.md
3. Begin implementation

Continue? (yes/no)
```

## Parallel Execution

Workflow identifies parallelizable work:

```yaml
Parallel Wave 1:
  - Frontend: Build login UI component
  - Backend: Implement auth API endpoints
  - DevOps: Set up auth service infrastructure

Synchronization Point: Integration testing

Parallel Wave 2:
  - Frontend: Connect UI to API
  - Backend: Add logging and monitoring
  - Tester: Prepare test scenarios
```

## Example: Complete Workflow

```bash
/speckit.workflow "Add OAuth2 authentication" --strategy systematic

# Phase 1: Specification
📝 Analyzer: Creating specification...
✅ Spec created: specs/000003-oauth2-auth/spec.md

# Phase 2: Clarification
💭 Analyzer: Found 4 ambiguities
Q1: Which OAuth2 providers? (Google, GitHub, Microsoft?)
[User answers...]

✅ Clarifications recorded

# Phase 3: Planning
🏗️ Architect: Designing system architecture...
🔐 Security: Adding threat model...
📋 Plan created: specs/000003-oauth2-auth/plan.md

🚦 Gate Check: All constitutional gates passed ✅

# Phase 4: Tasks
✅ Tasks generated: 12 tasks, 4 parallelizable
   [P] Frontend: OAuth button UI
   [P] Backend: OAuth flow handler
   [P] DevOps: Configure OAuth apps

# Phase 5: Implementation
🔨 Frontend Persona: Building login UI...
🔨 Backend Persona: Implementing OAuth flow...
🔨 DevOps Persona: Setting up infrastructure...

# Phase 6: Validation
✅ Tests: 45/45 passing
✅ Security: No vulnerabilities found
✅ Review: 8.5/10 overall

🎉 Workflow Complete!
   Branch: feature/000003-oauth2-auth
   Ready for: Pull request
```

## Command Options

```bash
# Strategy
--strategy systematic   # Default: comprehensive
--strategy fast        # Quick for prototypes
--strategy deep        # Extra thorough

# Depth
--depth shallow        # Minimal detail
--depth normal         # Default
--depth deep           # Maximum detail

# Personas
--personas frontend,backend   # Only these personas
--personas all                # All personas (default)

# Resume
--resume feature/000001-auth  # Continue existing workflow

# Auto-execute
--auto                 # Automatically execute all phases
--interactive          # Pause at each checkpoint (default)
```

## Success Criteria

- ✅ End-to-end workflow automation
- ✅ Multi-persona coordination
- ✅ Dependency management
- ✅ Quality gates enforced
- ✅ State persistence and resume capability
- ✅ Parallel execution where possible
- ✅ Strategy selection affects thoroughness vs speed

---

**Implementation**: Phase 3.7
**Dependencies**: All previous phases (1.1-3.6)
**Pattern**: Multi-persona orchestration with workflow state
**Complexity**: High (coordinates all other commands)
**Token Budget**: 5000-10000 tokens (full workflow)
