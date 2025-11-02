---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
scripts:
  sh: .specify/scripts/bash/setup-plan.sh --json
  ps: .specify/scripts/powershell/setup-plan.ps1 -Json
agent_scripts:
  sh: .specify/scripts/bash/update-agent-context.sh __AGENT__
  ps: .specify/scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**: Read FEATURE_SPEC and `.specify/memory/constitution.md`. Load IMPL_PLAN template (already copied).

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate data-model.md, contracts/, quickstart.md
   - Phase 1: Update agent context by running the agent script
   - Phase 2: Execute Architecture Review Checklist
   - Phase 2.5: Execute Test Strategy Planning
   - Phase 3: Execute Security & Compliance Review
   - Re-evaluate Constitution Check post-design

4. **Stop and report**: Command ends after Phase 2 planning. Report branch, IMPL_PLAN path, and generated artifacts.

5. **Planning Review Gate** (Evidence-Based Self-Check):

   **Purpose**: Validate planning completeness and readiness to proceed to task generation phase.

   **MANDATORY: The Four Planning Questions** (MUST answer with ACTUAL evidence):

   ❓ **"Are all NEEDS CLARIFICATION items resolved?"**
      ```yaml
      Action Required:
        - Scan IMPL_PLAN (plan.md) for "NEEDS CLARIFICATION" markers
        - Check research.md exists and addresses all unknowns
        - Verify each NEEDS CLARIFICATION replaced with concrete decision
        - Show ACTUAL count, not summaries

      Expected Evidence Format:
        Clarification Status:
        ✓ NEEDS CLARIFICATION markers before: [N]
        ✓ NEEDS CLARIFICATION markers after: [M] (target: 0)
        ✓ Resolved items: [N-M]

        [IF M > 0:]
        Remaining NEEDS CLARIFICATION:
        - [Section]: [What's unclear] - [Line number in IMPL_PLAN]

        research.md Status:
        ✓ File exists: [Yes/No]
        ✓ Decisions documented: [X] decisions
        ✓ Each decision has: Decision + Rationale + Alternatives

      Readiness Determination:
        IF M = 0 AND research.md complete:
          → ✅ All unknowns resolved
        IF M ≤ 2 AND low-impact clarifications:
          → ⚠️ Minor gaps (can proceed with caution)
        IF M > 2 OR high-impact unknowns remain:
          → ❌ NOT READY (need more research/clarification)
      ```

   ❓ **"Are design artifacts complete and consistent?"**
      ```yaml
      Action Required:
        - Verify data-model.md exists with entities from spec
        - Check contracts/ directory has API specifications
        - Validate quickstart.md integration scenarios present
        - Cross-check entities in data-model match contracts

      Expected Evidence Format:
        Design Artifacts Status:
        ✓ data-model.md: [Present/Missing]
          - Entities defined: [X]
          - Relationships documented: [Y]
          - Validation rules: [Z]
          - State transitions: [if applicable]

        ✓ contracts/: [Present/Missing]
          - API specifications: [X files]
          - Endpoint count: [Y endpoints]
          - Schema format: [OpenAPI/GraphQL/other]
          - Coverage: [% of functional requirements]

        ✓ quickstart.md: [Present/Missing]
          - Integration scenarios: [X scenarios]
          - Setup steps: [documented]
          - Example usage: [present]

        ✓ research.md: [Present/Missing]
          - Technical decisions: [X decisions]
          - All justified: [Yes/No]

      Consistency Validation:
        - [ ] Entities in data-model match contract schemas
        - [ ] Functional requirements covered by endpoints
        - [ ] Quickstart scenarios align with contracts
        - [ ] Tech stack decisions from research reflected in plan
        - [ ] No contradictions between artifacts

      IF any artifact missing OR consistency check fails:
        → ❌ Design incomplete
        → List missing/inconsistent items
      ```

   ❓ **"Does planning satisfy constitution constraints?"**
      ```yaml
      Action Required:
        - Review Constitution Check section in IMPL_PLAN
        - Verify all gate evaluations completed
        - Check for ERROR conditions (gate violations)
        - Validate post-design re-evaluation done

      Expected Evidence Format:
        Constitution Check Status:
        ✓ Gates evaluated: [X]/[Y] gates
        ✓ Violations found: [N]
        ✓ Violations justified: [M]/[N]

        [IF unjustified violations exist:]
        Gate Violations (BLOCKING):
        - [Gate Name]: [What violated] - [Justification: Yes/No]

        Post-Design Re-evaluation:
        ✓ Completed: [Yes/No]
        ✓ New violations: [N]
        ✓ Resolution: [how addressed]

      Constitution Compliance:
        IF all gates pass OR all violations justified:
          → ✅ Constitution satisfied
        IF unjustified violations exist:
          → ❌ BLOCKING ERROR (must resolve violations)
          → Cannot proceed to task generation
      ```

   ❓ **"Is there evidence of planning completeness?"**
      ```yaml
      Required Evidence (ALL must be provided):

      1. File Existence Check (MANDATORY):
         Run: ls -la [FEATURE_DIR]
         Output:
           ✓ plan.md: [size] bytes, [timestamp]
           ✓ research.md: [size] bytes, [timestamp]
           ✓ data-model.md: [size] bytes, [timestamp]
           ✓ contracts/: [X files], [total size]
           ✓ quickstart.md: [size] bytes, [timestamp]

      2. Content Validation (MANDATORY):
         Run grep checks:
         - NEEDS CLARIFICATION count in plan.md: [N] (target: 0)
         - Entities in data-model.md: [list names]
         - Endpoints in contracts/: [list paths/operations]
         - Scenarios in quickstart.md: [list titles]

      3. Spec-to-Plan Traceability (MANDATORY):
         Functional Requirements Coverage:
         - Total requirements in spec.md: [N]
         - Requirements with endpoints: [M]
         - Coverage: [M/N * 100]%

         User Stories Coverage:
         - Total user stories: [X]
         - Stories in quickstart scenarios: [Y]
         - Coverage: [Y/X * 100]%

         Target: ≥90% coverage for both

      4. Agent Context Update (MANDATORY):
         Run: Check agent-specific context file updated
         Output:
           ✓ Agent detected: [pm-agent/other]
           ✓ Context file: [path]
           ✓ Technologies added: [list from research.md]
           ✓ Manual additions preserved: [Yes/No]

      IF any evidence is MISSING:
        ❌ CANNOT report completion
        → Gather missing evidence first
        → Re-run this step with complete evidence
      ```

   **Hallucination Prevention (7 Red Flags for Planning):**
   ```yaml
   Detect and BLOCK these patterns:

   🚨 "Planning complete" WITH remaining NEEDS CLARIFICATION
      → Self-correction: "Wait, unresolved items remain in plan"

   🚨 "All artifacts generated" WITHOUT showing file list
      → Self-correction: "Let me verify files actually exist"

   🚨 "Design complete" WITH missing data-model or contracts
      → Self-correction: "Core artifacts are missing"

   🚨 Claiming coverage WITHOUT traceability to spec
      → Self-correction: "Need to map requirements to design"

   🚨 Skipping constitution check evaluation
      → Self-correction: "I need to verify gate compliance"

   🚨 Hiding gate violations or errors
      → Self-correction: "Must report violations honestly"

   🚨 "Ready for tasks" statements WITHOUT evidence
      → Self-correction: "Need to verify completeness criteria first"

   IF detected: STOP → Gather evidence → Report honestly
   ```

   **Determine Status**:

   ✅ **READY for Task Generation**:
   ```yaml
   Criteria (ALL must be met):
     - NEEDS CLARIFICATION count = 0
     - All design artifacts present (data-model, contracts, quickstart, research)
     - Constitution gates pass (no unjustified violations)
     - Requirements coverage ≥ 90%
     - Artifact consistency validated
     - Agent context updated

   IF ALL criteria met:
     → Proceed to Step 6 (present completion report)
   ```

   ⚠️ **CAN PROCEED** (with risks noted):
   ```yaml
   Criteria:
     - NEEDS CLARIFICATION ≤ 2 (low impact items)
     - Core artifacts present (may have minor gaps)
     - Constitution gates mostly pass
     - Requirements coverage ≥ 75%

   IF criteria met:
     → Present risks to user
     → Ask: "Minor gaps remain. Proceed to task generation or address gaps?"
     → Wait for user decision before Step 6
   ```

   ❌ **NOT READY** (more planning needed):
   ```yaml
   Criteria (ANY triggers NOT READY):
     - NEEDS CLARIFICATION > 2
     - Critical artifact missing (data-model or contracts)
     - Constitution gate violations unjustified
     - Requirements coverage < 75%
     - Major consistency issues between artifacts

   IF NOT READY:
     → Present issues to user with evidence
     → Recommend specific actions:
        - Run /speckit.clarify for spec ambiguities
        - Research missing decisions
        - Generate missing artifacts
        - Resolve gate violations
     → STOP before Step 6 (do not proceed to tasks)
   ```

   **Output Format** (Present to User - ONLY if evidence provided):
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   📐 Planning Review Complete
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Status: [✅ READY | ⚠️ CAN PROCEED | ❌ NOT READY]

   Branch: [branch-name]
   Plan: [path/to/plan.md]

   ## Clarification Resolution

   ✓ NEEDS CLARIFICATION: [N] → [M] (target: 0)
   ✓ Resolved items: [N-M]

   [IF M > 0:]
   Remaining Clarifications:
   - [Section]: [What's unclear] - [plan.md:line]

   ## Design Artifacts

   ✓ research.md: [X decisions documented]
   ✓ data-model.md: [Y entities, Z relationships]
   ✓ contracts/: [N endpoints across M files]
   ✓ quickstart.md: [P integration scenarios]

   Artifact Sizes:
   - plan.md: [size] bytes
   - research.md: [size] bytes
   - data-model.md: [size] bytes
   - contracts/: [total size] ([N files])
   - quickstart.md: [size] bytes

   ## Requirements Coverage

   Functional Requirements: [M]/[N] covered ([X]%)
   User Stories in Scenarios: [Y]/[Z] covered ([Y]%)

   Target: ≥90% coverage
   Status: [✅ Met | ⚠️ Partial | ❌ Below target]

   ## Constitution Compliance

   Gates Evaluated: [X]/[Y]
   Violations: [N] found, [M] justified
   Status: [✅ Pass | ❌ Blocking violations]

   [IF violations exist:]
   Gate Violations:
   - [Gate]: [Violation] - [Justified: Yes/No]

   ## Consistency Check

   ✅ Entities match contract schemas
   ✅ Endpoints cover functional requirements
   ✅ Quickstart aligns with contracts
   ✅ Tech decisions reflected in plan
   ✅ No artifact contradictions

   ## Agent Context

   ✓ Agent: [pm-agent/other]
   ✓ Context file: [path]
   ✓ Technologies added: [list]
   ✓ Manual additions preserved: Yes

   ## Readiness Assessment

   [IF ✅ READY:]
   ✅ All unknowns resolved (NEEDS CLARIFICATION = 0)
   ✅ Complete design artifact set generated
   ✅ Constitution gates satisfied
   ✅ Requirements coverage ≥90%
   ✅ Ready to proceed to task generation

   [IF ⚠️ CAN PROCEED:]
   ⚠️ Minor gaps detected:
      - [Gap 1]: [Why low impact]
      - [Gap 2]: [Why low impact]

   Risk: May require clarification during task generation.
   Requirements coverage: [X]% (target: 90%)

   [IF ❌ NOT READY:]
   ❌ Critical gaps prevent task generation:
      - [Gap 1]: [HIGH impact] - [Why blocks tasks]
      - [Gap 2]: [HIGH impact] - [Why blocks tasks]

   Required Actions:
   1. [Specific action 1 - e.g., "Resolve NEEDS CLARIFICATION in Technical Context"]
   2. [Specific action 2 - e.g., "Generate missing data-model.md"]
   3. [Specific action 3 - e.g., "Justify gate violations or redesign"]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Evidence: Generated Artifacts
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   File Listing:
   ```
   [Output of: ls -lah FEATURE_DIR]
   ```

   NEEDS CLARIFICATION Check:
   ```
   [Output of: grep -n "NEEDS CLARIFICATION" plan.md]
   ```

   Entity List (data-model.md):
   - [Entity 1]: [fields count]
   - [Entity 2]: [fields count]

   Endpoint List (contracts/):
   - [Endpoint 1]: [method] [path]
   - [Endpoint 2]: [method] [path]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Next Steps
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [IF ✅ READY:]
   1. Proceed to task generation:
      /speckit.tasks

   2. Or review planning artifacts:
      - Plan: [path/to/plan.md]
      - Research: [path/to/research.md]
      - Data Model: [path/to/data-model.md]
      - Contracts: [path/to/contracts/]

   [IF ⚠️ CAN PROCEED:]
   Choose one:
   1. Proceed with noted risks:
      /speckit.tasks

   2. Address minor gaps:
      [Specific actions for each gap]

   3. Review artifacts:
      [List artifact paths]

   [IF ❌ NOT READY:]
   REQUIRED: Address critical gaps before task generation:

   1. For spec ambiguities:
      /speckit.clarify [focus areas]

   2. For missing research:
      - Research [specific topics]
      - Update research.md

   3. For missing artifacts:
      - Generate [specific artifact]
      - Ensure consistency with spec

   4. For gate violations:
      - Justify violations in Constitution Check, OR
      - Redesign to satisfy constraints

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Ready to proceed? (yes/review artifacts/address gaps/run clarify)
   ```

   **User Interaction:**
   ```yaml
   IF user says "yes" or "proceed":
     IF status = READY:
       → Suggest: /speckit.tasks
     IF status = CAN PROCEED:
       → Warn about risks, then suggest: /speckit.tasks
     IF status = NOT READY:
       → Block: "Cannot proceed - critical gaps remain"
       → Suggest specific remediation actions

   IF user says "review" or "review artifacts":
     → Show list of generated artifacts with paths
     → Offer to display specific artifact contents

   IF user says "address gaps":
     → List specific gaps with:
        - Why they're critical (impact)
        - Suggested resolution steps
        - Which artifacts need updating
     → Ask: "Which gap to address first?"

   IF user says "run clarify" or "clarify":
     → Suggest: /speckit.clarify [with focus on planning gaps]
     → List specific areas needing clarification

   IF EVIDENCE MISSING:
     ❌ "Cannot complete planning review without evidence."
     → List missing evidence
     → Gather evidence automatically where possible
     → Re-run review gate
   ```

   **Benefits** (from PM Agent Reflexion pattern):
   - ✅ Prevents premature task generation with unresolved planning gaps
   - ✅ Evidence-based design completeness validation
   - ✅ No false "ready" claims without artifact verification
   - ✅ Transparent validation of planning effectiveness
   - ✅ User confidence in design quality before implementation

## Phases

### Phase 0: Outline & Research

1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:

   ```text
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

### Phase 1: Design & Contracts

**Prerequisites:** `research.md` complete

1. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable

2. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

3. **Agent context update**:
   - Run `{AGENT_SCRIPT}`
   - These scripts detect which AI agent is in use
   - Update the appropriate agent-specific context file
   - Add only new technology from current plan
   - Preserve manual additions between markers

**Output**: data-model.md, /contracts/*, quickstart.md, agent-specific file

### Phase 1.5: Supplementary Spec Discovery & Integration (v2.1)

**Prerequisites:** Phase 1 complete

**Purpose**: Auto-discover hierarchical specifications and integrate into plan.md

1. **Discover supplementary specs**:
   ```bash
   # Find all *-SPEC.md files in feature directory
   SUPP_SPECS=$(find "${SPECS_DIR}/${SPEC_DIR}" -maxdepth 1 -name "*-SPEC.md" -type f)

   if [ -z "$SUPP_SPECS" ]; then
     echo "No supplementary specs found (standard workflow)"
   else
     echo "Discovered supplementary specs:"
     echo "$SUPP_SPECS" | while read -r spec; do
       echo "  - $(basename "$spec")"
     done
   fi
   ```

2. **Parse supplementary spec metadata**:
   - Extract frontmatter from each *-SPEC.md
   - Parse: scope, functional_requirements, target_agents
   - Validate: parent = "spec.md", required fields present

3. **Generate "Supplementary Specifications" section**:
   ```markdown
   ## Supplementary Specifications

   This feature includes detailed implementation guides beyond the core spec.md:

   ### {SCOPE_DISPLAY}
   - **[{SPEC_NAME}]({SPEC_NAME})**
     - **Scope**: {SCOPE}
     - **Coverage**: {FR_LIST}
     - **Size**: {SIZE_KB}KB, {LINES} lines
     - **Target Agents**: {TARGET_AGENTS}

   ### Agent Usage Requirements

   **CRITICAL**: All implementation tasks MUST reference relevant supplementary specs.

   **Token Efficiency**: Load supplementary specs ONCE at session start, reference sections during implementation.
   ```

4. **Append to plan.md**:
   - If supplementary specs exist, append "Supplementary Specifications" section
   - Include metadata for each spec (size, scope, target agents)
   - Add agent usage requirements

5. **Validation (optional)**:
   - Check frontmatter validity
   - Warn if supplementary spec missing required fields
   - Suggest `/speckit.validate-hierarchy` for strict validation

**Output**: plan.md enhanced with supplementary spec references (if any exist)

**Note**: This phase is optional and only runs if *-SPEC.md files are present in the feature directory.

### Phase 2: Architecture Review Checklist

**Prerequisites:** Phase 1 complete, design artifacts generated

**Purpose**: Validate architectural decisions against design principles and identify potential quality issues before task generation.

**Execute Multi-Perspective Architecture Analysis**:

1. **Design Coherence Review (Martin Fowler Lens)**:

   Check for separation of concerns violations:
   - [ ] Are responsibilities clearly separated across modules/components?
   - [ ] Does any single component handle too many concerns?
   - [ ] Are there unnecessary couplings between components?
   - [ ] Do similar behaviors have consistent implementations?

   Check for SOLID principle violations:
   - [ ] Single Responsibility: Each component has one reason to change?
   - [ ] Open/Closed: Design extensible without modification?
   - [ ] Liskov Substitution: Abstractions properly defined?
   - [ ] Interface Segregation: No fat interfaces forcing unused dependencies?
   - [ ] Dependency Inversion: Depend on abstractions, not concretions?

   Check for DRY violations:
   - [ ] Is there duplicated logic across components?
   - [ ] Are similar patterns handled inconsistently?
   - [ ] Could redundant requirements be consolidated?

   **Flag when:**
   - Components violate single responsibility
   - Tight coupling between unrelated concerns
   - Inconsistent handling of similar scenarios
   - Unnecessary complexity in design

2. **Goal Alignment Review (Alistair Cockburn Lens)**:

   Validate business goal alignment:
   - [ ] Is the primary stakeholder clearly identified?
   - [ ] Does each component directly support a business goal?
   - [ ] What breaks if we remove non-essential components?
   - [ ] Are there features without clear business justification?

   Check for overengineering:
   - [ ] Does solution complexity match problem complexity?
   - [ ] Are there simpler alternatives achieving 80% of value?
   - [ ] Are workflows unnecessarily complicated?
   - [ ] Are there unnecessary state transitions?

   **Flag when:**
   - Requirements lack clear business justification
   - Overcomplicated solutions for simple problems
   - Features that don't serve primary user goals
   - Complexity without proportional value

3. **ADR Consistency Validation**:

   Cross-check design artifacts against Architecture Decision Records:
   - [ ] Does data-model.md align with data storage ADRs?
   - [ ] Do contracts/ align with API design ADRs?
   - [ ] Do tech choices from research.md match ADR decisions?
   - [ ] Are there contradictions between plan and existing ADRs?

   **Flag when:**
   - Design contradicts established ADRs
   - Missing ADRs for significant architectural decisions
   - ADR rationale conflicts with current requirements
   - Pattern violations from established conventions

4. **Production Readiness Checklist**:

   Verify operational concerns are addressed:
   - [ ] Are performance characteristics specified?
   - [ ] Are failure modes and error handling documented?
   - [ ] Are monitoring/observability requirements defined?
   - [ ] Are security considerations documented?
   - [ ] Are scalability requirements addressed?

   **Flag when:**
   - Missing error handling for critical flows
   - No performance requirements for user-facing operations
   - Missing monitoring/alerting specifications
   - Security concerns not addressed in design

**Architecture Quality Report Format**:

```markdown
## Architecture Review

**Design Coherence**: [✅ Pass | ⚠️ Issues Found | ❌ Major Violations]
- Separation of Concerns: [assessment]
- SOLID Principles: [violations if any]
- DRY Compliance: [duplications if any]

**Goal Alignment**: [✅ Clear | ⚠️ Some Unclear | ❌ Misaligned]
- Business Justification: [all components justified]
- Complexity Match: [appropriate/overengineered/underdesigned]
- Simplification Opportunities: [list if any]

**ADR Consistency**: [✅ Aligned | ⚠️ Minor Conflicts | ❌ Major Contradictions]
- Contradictions: [list if any]
- Missing ADRs: [list decisions needing ADRs]

**Production Readiness**: [✅ Addressed | ⚠️ Gaps | ❌ Critical Missing]
- Performance: [specified/missing]
- Error Handling: [comprehensive/gaps/missing]
- Monitoring: [specified/missing]
- Security: [addressed/gaps/missing]

**Recommendations**:
1. [Immediate action 1 with section reference]
2. [Immediate action 2 with section reference]
3. [Optional improvement 1]

**Overall Assessment**: [✅ Ready | ⚠️ Proceed with noted issues | ❌ Address critical issues first]
```

**Integration with Planning Review Gate**:
- Architecture review findings feed into Planning Review Gate (Step 5)
- Critical architecture issues (❌) block progression to task generation
- Minor issues (⚠️) noted but don't block if other criteria met
- Architecture quality contributes to "design artifacts complete and consistent" check

**Output**: Append Architecture Review section to plan.md before Planning Review Gate execution

---

### Phase 2.5: Test Strategy Planning

**Prerequisites:** Phase 1 complete, design artifacts and architecture review complete

**Purpose**: Define comprehensive testing strategy covering all test levels (unit, integration, E2E, performance), identify high-risk areas requiring extra test coverage, and plan test data/mock strategies.

**Execute Systematic Test Planning**:

1. **Test Coverage Analysis by Layer**:

   **Unit Tests** (Component-level isolation):
   - [ ] Identify all business logic functions requiring unit tests
   - [ ] List all edge cases, boundary conditions, error paths per function
   - [ ] Plan mocking strategy for external dependencies (APIs, DB, services)
   - [ ] Identify pure functions (no side effects) for property-based testing
   - [ ] Define expected code coverage threshold (e.g., 80% for critical paths)

   **Integration Tests** (Component interaction):
   - [ ] Identify all API endpoints and their request/response contracts
   - [ ] List all database operations (CRUD, transactions, migrations)
   - [ ] Identify all external service integrations (Stripe, Auth0, webhooks)
   - [ ] Plan test database seeding/cleanup strategy
   - [ ] Define integration test scope (which services to mock vs. real)

   **End-to-End Tests** (User workflows):
   - [ ] Map all critical user journeys from spec User Stories (P1/P2 priority)
   - [ ] Identify happy paths requiring E2E coverage
   - [ ] Identify error recovery flows requiring E2E validation
   - [ ] Plan browser/device matrix for UI testing (if applicable)
   - [ ] Define E2E test data setup and teardown strategy

   **Performance/Load Tests** (if applicable):
   - [ ] Identify high-traffic endpoints or batch operations
   - [ ] Define expected throughput (requests/second, transactions/minute)
   - [ ] Identify database queries requiring optimization/indexing
   - [ ] Plan load test scenarios (normal load, peak load, stress test)
   - [ ] Define performance acceptance criteria (latency, throughput)

2. **Risk-Based Test Prioritization**:

   For each requirement (FR-XXX, User Story), assign test priority based on risk:

   **🔴 HIGH PRIORITY** (Must have comprehensive test coverage):
   - Risk Score ≥ 8 from Risk Assessment (payment, auth, data integrity)
   - User-blocking defects (login, checkout, data entry)
   - Financial calculations (billing, invoicing, tax)
   - Security-sensitive operations (authentication, authorization, data access)
   - Compliance-related features (audit trails, GDII reporting)
   - Multi-tenant isolation (data filtering, tenant context)

   **🟠 MEDIUM PRIORITY** (Should have good test coverage):
   - Risk Score 4-7 from Risk Assessment
   - Important but non-blocking workflows
   - Data transformations and business logic
   - External integrations (non-payment)
   - Background jobs and scheduled tasks

   **🟡 LOW PRIORITY** (Basic test coverage acceptable):
   - Risk Score 0-3 from Risk Assessment
   - UI polish, cosmetic features
   - Admin-only features with low usage
   - Read-only operations on public data

3. **Edge Case & Error Scenario Test Planning**:

   For each high-risk requirement, identify test cases for:

   **Boundary Conditions**:
   - Empty/null/zero values
   - Maximum limits (string length, array size, file size)
   - Minimum limits (negative numbers, below threshold)
   - Edge of range (exactly 0, exactly max, one above/below)

   **Error Conditions**:
   - Network failures (timeout, connection refused)
   - Service unavailability (API down, database unreachable)
   - Authorization failures (expired token, insufficient permissions)
   - Validation failures (invalid format, constraint violation)
   - Resource exhaustion (rate limit, disk full, memory exhausted)

   **State Transitions**:
   - Invalid state transitions (A → C without B)
   - Recovery from error/cancelled/failed states
   - Crash recovery and resume-from-checkpoint
   - Idempotency (same operation retried safely)

   **Concurrency & Race Conditions** (if applicable):
   - Two users updating same record simultaneously
   - Webhook delivered twice (duplicate processing)
   - TOCTOU (time-of-check-time-of-use) races
   - Distributed lock acquisition and release

4. **Mock & Stub Strategy**:

   For each external dependency, define mocking approach:

   **External APIs** (Stripe, Auth0, tax services):
   - [ ] Use official SDK test mode (if available)
   - [ ] Use HTTP mocking library (nock, MSW, WireMock)
   - [ ] Create shared mock factories for common scenarios
   - [ ] Plan error scenario mocks (timeout, 500 error, rate limit)

   **Database**:
   - [ ] Use in-memory test DB for unit tests (SQLite, H2)
   - [ ] Use Docker containers for integration tests (PostgreSQL, MySQL)
   - [ ] Plan schema migration testing strategy
   - [ ] Define test data seeding and cleanup procedures

   **Time/Clock**:
   - [ ] Mock system clock for time-dependent logic (jest.useFakeTimers, freezegun)
   - [ ] Test date/time edge cases (timezone, DST, leap year)

   **File System / Storage**:
   - [ ] Mock file operations for unit tests (in-memory fs)
   - [ ] Use test buckets for integration tests (S3, R2)

5. **Test Data Requirements**:

   Define test data strategy:

   **Fixtures & Factories**:
   - [ ] Create reusable test data factories (Factory Bot, Faker)
   - [ ] Define fixture files for complex scenarios
   - [ ] Plan anonymized production data sampling (if needed)

   **Data Isolation**:
   - [ ] Each test creates/cleans up own data (no shared state)
   - [ ] Use transaction rollback for DB tests (if possible)
   - [ ] Plan parallel test execution strategy (isolated tenants/users)

   **Sensitive Data Handling**:
   - [ ] Never use real PII, credentials, or payment info in tests
   - [ ] Use synthetic test data (fake names, emails, cards)
   - [ ] Plan vault/secrets mocking for credential tests

6. **Test Automation & CI/CD Integration**:

   Plan test execution strategy:

   - [ ] Unit tests: Run on every commit (fast, < 30 seconds)
   - [ ] Integration tests: Run on PR (moderate, < 5 minutes)
   - [ ] E2E tests: Run on pre-merge + nightly (slow, < 30 minutes)
   - [ ] Performance tests: Run weekly or on demand

   **Quality Gates**:
   - [ ] Block merge if unit tests fail
   - [ ] Block merge if code coverage drops below threshold
   - [ ] Block merge if critical integration tests fail
   - [ ] Alert if E2E tests fail (but don't block if flaky)

7. **Generate Test Strategy Summary**:

   Create structured test plan section in plan.md:

   ```markdown
   ## Test Strategy

   **Test Coverage Summary**:

   | Test Level | Scope | Count | Priority Distribution | Execution |
   |------------|-------|-------|----------------------|-----------|
   | Unit Tests | Business logic, utilities | ~XX tests | 🔴 X, 🟠 X, 🟡 X | Every commit |
   | Integration Tests | APIs, DB, external services | ~XX tests | 🔴 X, 🟠 X, 🟡 X | Every PR |
   | E2E Tests | Critical user workflows | ~XX tests | 🔴 X, 🟠 X | Pre-merge + nightly |
   | Performance Tests | High-traffic endpoints | ~XX tests | 🔴 X, 🟠 X | Weekly/on-demand |

   **High-Risk Requirements Requiring Extra Test Coverage**:

   | Requirement | Risk Score | Why High-Risk | Test Strategy |
   |-------------|------------|---------------|---------------|
   | FR-001 | 🔴 10 | Payment processing, financial integrity | Unit + integration + E2E + error scenarios + idempotency |
   | FR-003 | 🔴 9 | Multi-tenant isolation, data leakage risk | Unit + integration with tenant filtering tests |
   | FR-007 | 🟠 6 | External API integration, timeout handling | Integration with mock API errors + retry logic |

   **Edge Case Test Scenarios** (from Edge Case Detection):

   - **Boundary Conditions**: [List specific test cases - e.g., "Cart with 1000 items (max limit)"]
   - **Error Conditions**: [List error scenarios - e.g., "Stripe timeout after 30s"]
   - **State Transitions**: [List state edge cases - e.g., "Resume from cancelled payment"]
   - **Concurrency**: [List race conditions - e.g., "Two users update same invoice"]

   **Mock & Stub Strategy**:

   - **Stripe API**: Use Stripe test mode + mock webhook deliveries
   - **Auth0**: Mock JWT token validation with jose library
   - **Database**: PostgreSQL Docker container for integration tests, in-memory for unit
   - **Clock**: Use jest.useFakeTimers for time-dependent logic
   - **File Storage**: Mock S3 with LocalStack for integration tests

   **Test Data Strategy**:

   - **Factories**: Use [factory library] for generating test users, tenants, invoices
   - **Fixtures**: Predefined complex scenarios in fixtures/ directory
   - **Isolation**: Each test creates/destroys own data, no shared state
   - **Sensitive Data**: Never use real PII, use Faker.js for synthetic data

   **CI/CD Integration**:

   - ✅ Unit tests: Run on commit (< 30s), block merge on failure
   - ✅ Integration tests: Run on PR (< 5min), block merge on failure
   - ✅ E2E tests: Run pre-merge + nightly (< 30min), alert on failure
   - ℹ️ Performance tests: Run weekly or on demand

   **Code Coverage Thresholds**:

   - Critical paths (auth, payment, billing): ≥ 90% coverage
   - Business logic: ≥ 80% coverage
   - UI components: ≥ 70% coverage
   - Overall project: ≥ 75% coverage
   ```

**Output**: Append Test Strategy section to plan.md after Architecture Review

**Integration with Planning Review Gate**:
- Test strategy completeness feeds into Planning Review Gate (Step 5)
- Missing test coverage for high-risk requirements (🔴) blocks progression
- Test strategy quality contributes to "design artifacts complete" check

---

### Phase 3: Security & Compliance Review

**Prerequisites:** Phase 2.5 complete, test strategy defined, risk assessment available from spec

**Purpose**: Systematically validate security controls, compliance requirements, and data protection measures to prevent vulnerabilities and meet regulatory obligations.

**Execute Multi-Layer Security Analysis**:

1. **OWASP Top 10 Vulnerability Prevention**:

   Check for common web application vulnerabilities:

   **A01: Broken Access Control** 🔴 CRITICAL
   - [ ] Are all endpoints protected with authentication/authorization?
   - [ ] Is authorization checked at both route AND data level (not just UI hiding)?
   - [ ] Can users access resources they don't own (IDOR - Insecure Direct Object Reference)?
   - [ ] Are admin functions protected from regular users?
   - [ ] Is multi-tenant isolation enforced at database query level (tenant_id filtering)?
   - [ ] Are API rate limits enforced per user/tenant?

   **A02: Cryptographic Failures** 🔴 CRITICAL
   - [ ] Is sensitive data encrypted at rest (PII, passwords, API keys, financial data)?
   - [ ] Is data encrypted in transit (HTTPS/TLS for all connections)?
   - [ ] Are passwords hashed with strong algorithms (bcrypt, Argon2, PBKDF2)?
   - [ ] Are API keys/secrets stored in vault (not in code or env files committed to git)?
   - [ ] Is database connection encrypted (TLS/SSL)?
   - [ ] Are sensitive fields masked in logs and error messages?

   **A03: Injection** 🔴 CRITICAL
   - [ ] Are all database queries parameterized (no string concatenation)?
   - [ ] Is user input sanitized before database operations (SQL injection prevention)?
   - [ ] Is user input escaped before rendering in HTML (XSS prevention)?
   - [ ] Are JSON payloads validated against schema before processing?
   - [ ] Is command execution avoided or strictly sandboxed (no shell injection)?
   - [ ] Are file paths validated to prevent directory traversal attacks?

   **A04: Insecure Design** 🟠 MAJOR
   - [ ] Are security controls designed into the architecture (not bolted on later)?
   - [ ] Is threat modeling performed for high-risk features?
   - [ ] Are security requirements defined in spec (not implementation detail)?
   - [ ] Is defense-in-depth applied (multiple layers of security)?
   - [ ] Are failure modes secure (fail closed, not open)?

   **A05: Security Misconfiguration** 🟠 MAJOR
   - [ ] Are default passwords/keys changed in production?
   - [ ] Are unnecessary features/endpoints disabled?
   - [ ] Are security headers configured (CSP, HSTS, X-Frame-Options)?
   - [ ] Are verbose error messages disabled in production?
   - [ ] Is directory listing disabled?
   - [ ] Are CORS policies restrictive (not allow all origins)?

   **A06: Vulnerable Components** 🟠 MAJOR
   - [ ] Are dependencies scanned for known vulnerabilities (Snyk, Dependabot)?
   - [ ] Are outdated libraries identified and upgrade plan defined?
   - [ ] Are unmaintained dependencies flagged for replacement?
   - [ ] Is a process defined for security patch updates?

   **A07: Identification & Authentication Failures** 🔴 CRITICAL
   - [ ] Is multi-factor authentication (MFA) required for admin accounts?
   - [ ] Are passwords validated against minimum strength requirements?
   - [ ] Is brute-force protection implemented (rate limiting, account lockout)?
   - [ ] Are sessions invalidated on logout and timeout?
   - [ ] Are session tokens cryptographically random and unpredictable?
   - [ ] Is "remember me" functionality secure (long-lived tokens properly stored)?

   **A08: Software & Data Integrity Failures** 🔴 CRITICAL
   - [ ] Are CI/CD pipelines secured against tampering?
   - [ ] Are dependencies verified with checksums/signatures?
   - [ ] Is auto-update disabled or verified with signatures?
   - [ ] Are unsigned/unverified objects from untrusted sources rejected?
   - [ ] Is webhook payload authenticity verified (HMAC signatures)?

   **A09: Security Logging & Monitoring Failures** 🟠 MAJOR
   - [ ] Are authentication failures logged (failed logins, invalid tokens)?
   - [ ] Are authorization failures logged (access denied events)?
   - [ ] Are critical operations logged (payment, data export, admin actions)?
   - [ ] Are logs protected from tampering (append-only, external storage)?
   - [ ] Are alerts configured for suspicious activity patterns?
   - [ ] Is log retention defined (compliance requirements)?

   **A10: Server-Side Request Forgery (SSRF)** 🟠 MAJOR
   - [ ] Are user-supplied URLs validated before fetching (whitelist approach)?
   - [ ] Are internal network requests blocked from user-initiated actions?
   - [ ] Are webhook destinations validated against allowed domains?
   - [ ] Is metadata service access blocked (169.254.169.254 for cloud)?

2. **Authentication & Authorization Review**:

   **Authentication Mechanisms**:
   - [ ] What authentication method is used? (OAuth, JWT, session cookies, API keys)
   - [ ] Are tokens properly validated on every request?
   - [ ] Is token expiration/refresh implemented?
   - [ ] Are refresh tokens securely stored (httpOnly cookies, encrypted storage)?
   - [ ] Is logout implemented securely (token invalidation, not just client-side deletion)?

   **Authorization Model**:
   - [ ] What authorization model is used? (RBAC, ABAC, ACL, custom)
   - [ ] Are permissions checked at API layer (not just UI)?
   - [ ] Are database queries filtered by user/tenant permissions?
   - [ ] Is principle of least privilege applied (users get minimum necessary access)?
   - [ ] Are admin functions separated and extra-protected?

3. **Data Protection & Privacy**:

   **Sensitive Data Inventory**:
   - [ ] List all PII fields (name, email, address, phone, SSN, DOB)
   - [ ] List all financial data (credit cards, bank accounts, payment info)
   - [ ] List all authentication credentials (passwords, API keys, tokens)
   - [ ] List all health information (if applicable - HIPAA)
   - [ ] List all proprietary business data (contracts, revenue, algorithms)

   **Data Protection Measures**:
   - [ ] Is PII encrypted at rest in database?
   - [ ] Are credit card numbers never stored (use tokenization - Stripe, PayPal)?
   - [ ] Are passwords never stored in plaintext (bcrypt/Argon2 hashing)?
   - [ ] Are API keys stored in vault (AWS Secrets Manager, HashiCorp Vault)?
   - [ ] Is sensitive data masked in logs (credit card → **** **** **** 1234)?
   - [ ] Is data anonymized/pseudonymized when possible?

   **Data Retention & Deletion**:
   - [ ] Is data retention policy defined (how long to keep data)?
   - [ ] Is data deletion process defined (hard delete vs soft delete)?
   - [ ] Is "right to be forgotten" implemented (GDPR Article 17)?
   - [ ] Are backups included in deletion process?
   - [ ] Is audit trail preserved after data deletion?

4. **Input Validation & Sanitization**:

   **Validation Strategy**:
   - [ ] Is server-side validation enforced (never trust client-side only)?
   - [ ] Are all API inputs validated against schema (JSON Schema, Zod, Joi)?
   - [ ] Are maximum lengths enforced for strings (prevent DoS via huge payloads)?
   - [ ] Are numeric inputs range-checked (min/max values)?
   - [ ] Are email/URL/date formats validated?
   - [ ] Are file uploads validated (type, size, content, not just extension)?

   **Sanitization Strategy**:
   - [ ] Is HTML escaped before rendering user content (XSS prevention)?
   - [ ] Are SQL queries parameterized (SQL injection prevention)?
   - [ ] Are file paths sanitized (directory traversal prevention)?
   - [ ] Are special characters handled in search queries?

5. **Rate Limiting & DDoS Protection**:

   **Rate Limiting**:
   - [ ] Are rate limits defined for all public endpoints?
   - [ ] Are stricter limits applied to expensive operations (search, export, reporting)?
   - [ ] Are rate limits per-user or per-IP (or both)?
   - [ ] Is rate limit response appropriate (429 status, Retry-After header)?
   - [ ] Are authenticated users given higher limits than anonymous?

   **DDoS Protection**:
   - [ ] Is CDN/WAF configured (Cloudflare, AWS Shield)?
   - [ ] Are connection limits set on load balancer?
   - [ ] Is request size limited (prevent large payload attacks)?
   - [ ] Are slow-client attacks mitigated (timeouts on requests)?

6. **Secrets Management**:

   **Secret Storage**:
   - [ ] Are secrets stored in vault (not in code, not in .env committed to git)?
   - [ ] Are production secrets separate from dev/staging?
   - [ ] Is secret rotation process defined?
   - [ ] Are secrets encrypted in transit when retrieved?
   - [ ] Is access to secrets logged and audited?

   **Secret Types to Protect**:
   - [ ] Database credentials
   - [ ] API keys (Stripe, Auth0, external services)
   - [ ] Encryption keys
   - [ ] JWT signing keys
   - [ ] OAuth client secrets
   - [ ] Webhook signing secrets

7. **Audit Logging Requirements**:

   **What to Log**:
   - [ ] Authentication events (login, logout, MFA, password reset)
   - [ ] Authorization failures (access denied, insufficient permissions)
   - [ ] Data access (viewing PII, exporting data, admin queries)
   - [ ] Data modifications (create, update, delete critical records)
   - [ ] Financial transactions (payments, refunds, subscription changes)
   - [ ] Admin actions (user management, role changes, config updates)
   - [ ] Security events (failed auth, suspicious patterns, rate limit exceeded)

   **Log Format & Storage**:
   - [ ] Are logs structured (JSON) for easy parsing?
   - [ ] Do logs include: timestamp, user, action, resource, IP, result?
   - [ ] Are logs tamper-proof (append-only, external storage)?
   - [ ] Are logs retained per compliance requirements (e.g., 7 years for financial)?
   - [ ] Are logs accessible for incident response and forensics?
   - [ ] Are PII in logs minimized or encrypted?

8. **Compliance Checks**:

   **GDPR (EU)** (if applicable):
   - [ ] Is user consent obtained for data collection?
   - [ ] Is privacy policy clear and accessible?
   - [ ] Is "right to access" implemented (users can download their data)?
   - [ ] Is "right to be forgotten" implemented (users can delete account)?
   - [ ] Is "right to rectification" implemented (users can correct data)?
   - [ ] Is data breach notification process defined (72-hour deadline)?
   - [ ] Is data processing documented (GDPR Article 30 records)?

   **CCPA (California)** (if applicable):
   - [ ] Is "do not sell my personal information" implemented?
   - [ ] Is data disclosure notice provided?
   - [ ] Is opt-out mechanism available?

   **PCI-DSS (Payment Card Industry)** (if handling credit cards):
   - [ ] Are credit card numbers never stored (use tokenization)?
   - [ ] Is PCI-DSS SAQ (Self-Assessment Questionnaire) completed?
   - [ ] Is network segmentation implemented (cardholder data isolated)?
   - [ ] Are quarterly vulnerability scans performed?

   **HIPAA (Healthcare)** (if handling health information):
   - [ ] Are PHI (Protected Health Information) fields encrypted?
   - [ ] Is BAA (Business Associate Agreement) required for vendors?
   - [ ] Is access logging implemented for all PHI access?
   - [ ] Is minimum necessary access enforced?

   **SOX (Financial Reporting)** (if public company):
   - [ ] Are financial data changes audited?
   - [ ] Is segregation of duties enforced?
   - [ ] Are access controls documented and reviewed?

   **Industry-Specific** (Dominican Republic DGII, etc.):
   - [ ] Are tax reporting requirements documented?
   - [ ] Is fiscal receipt generation compliant with local regulations?
   - [ ] Are e-invoice formats validated against government schemas?

9. **Generate Security Review Summary**:

   Create structured security review section in plan.md:

   ```markdown
   ## Security & Compliance Review

   **OWASP Top 10 Assessment**:

   | Vulnerability | Status | Mitigation | Priority |
   |---------------|--------|------------|----------|
   | A01: Broken Access Control | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🔴 CRITICAL |
   | A02: Cryptographic Failures | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🔴 CRITICAL |
   | A03: Injection | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🔴 CRITICAL |
   | A04: Insecure Design | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🟠 MAJOR |
   | A05: Security Misconfiguration | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🟠 MAJOR |
   | A06: Vulnerable Components | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🟠 MAJOR |
   | A07: Auth Failures | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🔴 CRITICAL |
   | A08: Integrity Failures | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🔴 CRITICAL |
   | A09: Logging Failures | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🟠 MAJOR |
   | A10: SSRF | [🟢 Addressed / 🟡 Partial / 🔴 At Risk] | [Mitigation strategy] | 🟠 MAJOR |

   **Authentication & Authorization**:
   - **Method**: [OAuth 2.0 / JWT / Session Cookies / API Keys]
   - **Token Validation**: [How tokens are validated on each request]
   - **Authorization Model**: [RBAC / ABAC / Custom]
   - **Multi-Tenant Isolation**: [How tenant_id filtering is enforced]

   **Data Protection**:
   - **PII Encrypted**: [Yes/No - which fields]
   - **Credentials Storage**: [Bcrypt/Argon2 for passwords, Vault for API keys]
   - **Data Retention**: [How long data is kept, deletion process]
   - **Sensitive Data in Logs**: [Masked/Excluded]

   **Input Validation & Sanitization**:
   - **Validation Library**: [Zod / Joi / JSON Schema / Custom]
   - **SQL Injection Prevention**: [Parameterized queries, ORM]
   - **XSS Prevention**: [HTML escaping, CSP headers]
   - **File Upload Validation**: [Type, size, content validation]

   **Rate Limiting & DDoS**:
   - **Rate Limits**: [X requests/minute per user, Y per IP]
   - **Expensive Operations**: [Export: Z requests/hour, Search: W requests/minute]
   - **DDoS Protection**: [Cloudflare WAF / AWS Shield / Custom]

   **Secrets Management**:
   - **Vault**: [AWS Secrets Manager / HashiCorp Vault / Azure Key Vault]
   - **Secret Rotation**: [Automated / Manual / Frequency]
   - **Secrets Logged**: [No / Only in encrypted audit trail]

   **Audit Logging**:
   - **Log Format**: [JSON structured logs]
   - **Log Storage**: [CloudWatch / ELK / Datadog / External SIEM]
   - **Retention Period**: [X days/months/years per compliance requirement]
   - **Events Logged**: [Auth, authz failures, data access, financial transactions, admin actions]

   **Compliance Requirements**:
   - **GDPR**: [✅ Applicable - Right to access/delete implemented | N/A]
   - **CCPA**: [✅ Applicable - Do not sell implemented | N/A]
   - **PCI-DSS**: [✅ Applicable - Tokenization used, no card storage | N/A]
   - **HIPAA**: [✅ Applicable - PHI encrypted, BAA required | N/A]
   - **SOX**: [✅ Applicable - Audit trail, segregation of duties | N/A]
   - **DGII (Dominican Republic)**: [✅ Applicable - Fiscal receipts, e-invoice schema | N/A]

   **Security Risks Identified**:

   | Risk | Severity | Description | Mitigation Required |
   |------|----------|-------------|---------------------|
   | [Risk 1] | 🔴 CRITICAL | [Description] | [Action item with deadline] |
   | [Risk 2] | 🟠 MAJOR | [Description] | [Action item with deadline] |

   **Security Sign-Off**:
   - [ ] All 🔴 CRITICAL vulnerabilities addressed or explicitly accepted with justification
   - [ ] All 🟠 MAJOR vulnerabilities mitigated or backlog tickets created
   - [ ] Secrets management strategy defined and vault configured
   - [ ] Audit logging implemented for all critical operations
   - [ ] Compliance requirements identified and mapped to implementation
   ```

**Output**: Append Security & Compliance Review section to plan.md after Test Strategy

**Integration with Planning Review Gate**:
- Security review findings feed into Planning Review Gate (Step 5)
- Critical security vulnerabilities (🔴) block progression to task generation
- Security controls for high-risk features must be explicitly defined
- Compliance requirements must be acknowledged and planned

**Security Review Blocking Criteria**:
- 🔴 BLOCK: Any OWASP Top 10 critical vulnerability (A01, A02, A03, A07, A08) unaddressed
- 🔴 BLOCK: PII/financial data stored without encryption
- 🔴 BLOCK: No authentication/authorization for sensitive endpoints
- 🔴 BLOCK: SQL injection or XSS vulnerabilities possible
- 🟠 WARN: Missing rate limiting, incomplete logging, unrotated secrets

---

## Key rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications
