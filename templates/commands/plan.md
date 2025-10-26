---
description: Execute the implementation planning workflow using the plan template to generate design artifacts.
scripts:
  sh: scripts/bash/setup-plan.sh --json
  ps: scripts/powershell/setup-plan.ps1 -Json
agent_scripts:
  sh: scripts/bash/update-agent-context.sh __AGENT__
  ps: scripts/powershell/update-agent-context.ps1 -AgentType __AGENT__
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_SPEC, IMPL_PLAN, SPECS_DIR, BRANCH. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load context**: Read FEATURE_SPEC and `/memory/constitution.md`. Load IMPL_PLAN template (already copied).

3. **Execute plan workflow**: Follow the structure in IMPL_PLAN template to:
   - Fill Technical Context (mark unknowns as "NEEDS CLARIFICATION")
   - Fill Constitution Check section from constitution
   - Evaluate gates (ERROR if violations unjustified)
   - Phase 0: Generate research.md (resolve all NEEDS CLARIFICATION)
   - Phase 1: Generate data-model.md, contracts/, quickstart.md
   - Phase 1: Update agent context by running the agent script
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

## Key rules

- Use absolute paths
- ERROR on gate failures or unresolved clarifications
