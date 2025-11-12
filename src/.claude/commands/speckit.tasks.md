---
description: Generate an actionable, dependency-ordered tasks.md for the feature based on available design artifacts.
scripts:
  sh: .specify/scripts/bash/check-prerequisites.sh --json
  ps: .specify/scripts/powershell/check-prerequisites.ps1 -Json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

## Context Loading (v2.7 - Context Optimization)

**Purpose**: Load relevant memory content for task generation, optimized for token efficiency.

**Current Phase**: `tasks` (task breakdown phase)

**Context Loading Strategy**:

a. **Read Configuration** (if exists):
   - Check for `.specify/config.yml` or `.specify/config.example.yml`
   - Extract `context_budget.implementation` (default: 50KB - tasks shares budget with implementation)
   - Extract `memory.strict_phase_loading` (default: true)
   - Extract `budget_enforcement.mode` (default: strict)

b. **Load Memory Files** (phase-aware):

   For each memory file in `.specify/memory/`:

   i. **Read YAML Frontmatter**:
      - Extract `inclusion_mode`, `context_level`, `load_phases`, `exclude_phases`
      - If metadata missing: Default to `inclusion_mode: always, context_level: strategic, load_phases: all`

   ii. **Determine if file should be loaded**:
      ```yaml
      IF current_phase IN exclude_phases:
        → SKIP this file entirely

      ELSE IF inclusion_mode = "always":
        → Load tactical sections (task templates, implementation procedures)

      ELSE IF inclusion_mode = "conditional":
        IF current_phase IN load_phases OR "tasks" IN load_phases OR "implementation" IN load_phases:
          → Load tactical sections (task breakdown patterns, implementation procedures)
        ELSE:
          → SKIP (e.g., specification-only content excluded)

      ELSE IF inclusion_mode = "manual":
        → SKIP (manual loading via @filename only)
      ```

   iii. **Filter Sections** (if loading file):
      - Scan for `<!-- SECTION_META: context_level=X, load_phases=Y -->` comments
      - For tasks phase: Load sections where `tasks` OR `implementation` IN load_phases
      - Load tactical sections (task templates, implementation procedures, file structure patterns)
      - Skip strategic sections (high-level principles without actionable details)

   iv. **Track Budget**:
      - Sum loaded content size
      - If exceeds `context_budget.implementation` (50KB):
        - **strict mode**: Warn user, prioritize task templates + implementation procedures
        - **warn mode**: Continue loading, warn about budget exceeded
        - **adaptive mode**: Intelligently filter content to fit budget

c. **Context Loading Report** (verbose mode):

   IF `verbose_context_loading: true` in config:
   ```markdown
   📊 Context Loading Report

   Phase: tasks
   Budget: 50KB

   Loaded:
   - task-templates.md (tactical: task breakdown patterns): 7.2KB
   - testing-approach.md (tactical: test task patterns): 8.5KB
   - file-structure-patterns.md (tactical: project organization): 5.3KB
   - Skipped: constitution.md (strategic: principles only, not task details)
   - Skipped: deployment-runbook.md (manual only)

   Total Loaded: 21KB / 50KB (42% budget used)
   Remaining: 29KB available

   Status: ✅ Within budget
   ```

d. **Backward Compatibility**:
   - If `.specify/memory/` doesn't exist → Skip context loading (no error)
   - If memory files lack metadata → Load all content (v2.6 behavior)
   - Never fail command due to missing memory files

**Expected Token Savings**: ~15% (30-40KB → 25-35KB for tasks phase)

**Key Difference from Other Phases**:
- Tasks loads TACTICAL content (task templates, implementation procedures, file structure)
- Specification/Planning load STRATEGIC (principles, patterns, architecture)
- This ensures tasks have actionable implementation details, not just high-level principles

## Execution Steps

1. **Setup**: Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Load design documents**: Read from FEATURE_DIR:
   - **Required**: plan.md (tech stack, libraries, structure), spec.md (user stories with priorities)
   - **Optional**: data-model.md (entities), contracts/ (API endpoints), research.md (decisions), quickstart.md (test scenarios)
   - Note: Not all projects have all documents. Generate tasks based on what's available.

3. **Execute task generation workflow**:
   - Load plan.md and extract tech stack, libraries, project structure
   - Load spec.md and extract user stories with their priorities (P1, P2, P3, etc.)
   - If data-model.md exists: Extract entities and map to user stories
   - If contracts/ exists: Map endpoints to user stories
   - If research.md exists: Extract decisions for setup tasks
   - Generate tasks organized by user story (see Task Generation Rules below)
   - Generate dependency graph showing user story completion order
   - Create parallel execution examples per user story
   - Validate task completeness (each user story has all needed tasks, independently testable)

4. **Generate tasks.md**: Use `.specify/templates/tasks-template.md` as structure, fill with:
   - Correct feature name from plan.md
   - Phase 1: Setup tasks (project initialization)
   - Phase 2: Foundational tasks (blocking prerequisites for all user stories)
   - Phase 3+: One phase per user story (in priority order from spec.md)
   - Each phase includes: story goal, independent test criteria, tests (if requested), implementation tasks
   - Final Phase: Polish & cross-cutting concerns
   - All tasks must follow the strict checklist format (see Task Generation Rules below)
   - Clear file paths for each task
   - Dependencies section showing story completion order
   - Parallel execution examples per story
   - Implementation strategy section (MVP first, incremental delivery)

5. **Report**: Output path to generated tasks.md and summary:
   - Total task count
   - Task count per user story
   - Parallel opportunities identified
   - Independent test criteria for each story
   - Suggested MVP scope (typically just User Story 1)
   - Format validation: Confirm ALL tasks follow the checklist format (checkbox, ID, labels, file paths)

6. **Task List Review Gate** (Evidence-Based Self-Check):

   **Purpose**: Validate task list completeness and readiness to proceed to implementation phase.

   **MANDATORY: The Four Task List Questions** (MUST answer with ACTUAL evidence):

   ❓ **"Does each user story have complete task coverage?"**
      ```yaml
      Action Required:
        - Review tasks.md phase structure
        - For each user story phase, verify:
          - Models for entities in that story
          - Services for business logic
          - Endpoints/UI for user interactions
          - Tests (if TDD requested)
          - Integration tasks
        - Map user stories from spec.md to task phases
        - Show ACTUAL task counts per story, not summaries

      Expected Evidence Format:
        User Story Coverage:

        User Story 1 (P1): [Story name from spec.md]
        ✓ Models: [X tasks] - [list task IDs]
        ✓ Services: [Y tasks] - [list task IDs]
        ✓ Endpoints/UI: [Z tasks] - [list task IDs]
        ✓ Tests: [N tasks] - [list task IDs] (if TDD)
        ✓ Integration: [M tasks] - [list task IDs]
        Total: [X+Y+Z+N+M] tasks
        Independent test criteria: [Yes/No] - [criteria from tasks.md]

        User Story 2 (P2): [Story name from spec.md]
        [... same structure ...]

        [Repeat for all user stories]

      Completeness Check:
        - [ ] All user stories from spec.md have task phases
        - [ ] Each story has models, services, endpoints
        - [ ] Each story has independent test criteria
        - [ ] Setup/Foundational phases complete
        - [ ] Polish phase includes cross-cutting concerns

      Readiness Determination:
        IF all user stories covered AND all checklists pass:
          → ✅ Complete coverage
        IF ≥90% coverage AND minor gaps documented:
          → ⚠️ Minor gaps (can proceed with noted risks)
        IF <90% coverage OR major story gaps:
          → ❌ Incomplete (regenerate tasks)
      ```

   ❓ **"Are tasks properly formatted and executable?"**
      ```yaml
      Action Required:
        - Scan tasks.md for format violations
        - Verify EVERY task follows checklist format:
          - [ ] [TaskID] [P?] [Story?] Description with file path
        - Check task IDs are sequential (T001, T002, T003...)
        - Validate story labels match user stories from spec.md
        - Ensure file paths are specific and absolute

      Expected Evidence Format:
        Format Validation:
        ✓ Total tasks: [N]
        ✓ Tasks with checkboxes: [N]/[N] (100%)
        ✓ Tasks with IDs: [N]/[N] (100%)
        ✓ Tasks with file paths: [N]/[N] (100%)
        ✓ Parallel tasks marked [P]: [X] tasks
        ✓ Story-labeled tasks [USn]: [Y] tasks

        Format Violations (if any):
        - Line [N]: [Issue description]
        - Line [M]: [Issue description]

        Task ID Sequence Check:
        ✓ Sequential: [Yes/No]
        ✓ Range: T001 → T[max]
        ✓ Gaps/duplicates: [None/list them]

        Story Label Validation:
        ✓ User Stories in spec.md: [US1, US2, US3, ...]
        ✓ Story labels used in tasks: [US1, US2, US3, ...]
        ✓ Match: [Yes/No - if No, list mismatches]

      Executability Check:
        - [ ] Each task has specific file path
        - [ ] Each task has clear action verb
        - [ ] Tasks executable without additional context
        - [ ] Dependencies clear from phase structure

      IF format violations OR executability fails:
        → ❌ Tasks not properly formatted
        → Fix violations before implementation
      ```

   ❓ **"Are dependencies and parallelization correct?"**
      ```yaml
      Action Required:
        - Review dependency graph in tasks.md
        - Verify user story completion order makes sense
        - Check parallel markers [P] are safe:
          - Different files
          - No dependencies on incomplete tasks
        - Validate foundational phase blocks user stories
        - Ensure MVP scope is clear (typically US1)

      Expected Evidence Format:
        Dependency Structure:
        ✓ Phase 1 (Setup): [X tasks] - No dependencies
        ✓ Phase 2 (Foundational): [Y tasks] - Blocks user stories
        ✓ Phase 3+ (User Stories): [Z phases] in priority order

        User Story Dependencies:
        - US1 (P1): Independent (can implement first) ✅
        - US2 (P2): Depends on [US1/Foundational/Independent]
        - US3 (P3): Depends on [...]
        [... list all user story dependencies ...]

        Parallelization Opportunities:
        ✓ Parallel tasks per story: [N] tasks marked [P]
        ✓ Parallel tasks validation:
          - All [P] tasks operate on different files: [Yes/No]
          - No [P] tasks depend on incomplete work: [Yes/No]

        Parallel Execution Examples (from tasks.md):
        [Copy examples from tasks.md showing how to run parallel tasks]

        MVP Scope:
        ✓ Recommended: User Story [X] ([Name])
        ✓ Tasks in MVP: [N] tasks (T001-TXXX)
        ✓ MVP deliverable: [Description of what MVP enables]

      Safety Checks:
        - [ ] Foundational tasks complete before user stories
        - [ ] No circular dependencies between stories
        - [ ] Parallel tasks truly independent
        - [ ] MVP is smallest testable increment

      IF dependency errors OR unsafe parallelization:
        → ❌ Task ordering incorrect
        → Fix dependencies before implementation
      ```

   ❓ **"Is there evidence of task list quality?"**
      ```yaml
      Required Evidence (ALL must be provided):

      1. File Verification (MANDATORY):
         Run: ls -la [FEATURE_DIR]
         Output:
           ✓ tasks.md: [size] bytes, [timestamp]
           ✓ Exists: Yes/No

      2. Task Count Validation (MANDATORY):
         Run: grep -c "^- \[ \]" tasks.md
         Output:
           Total tasks: [N]

         Per-phase breakdown:
         - Setup: [X] tasks
         - Foundational: [Y] tasks
         - User Story phases: [Z] tasks across [P] phases
         - Polish: [W] tasks

      3. Format Compliance Check (MANDATORY):
         Run format validation grep commands:
         - Tasks with checkboxes: grep -c "^- \[ \]" tasks.md → [N]
         - Tasks with IDs: grep -c "\[T[0-9]\{3\}\]" tasks.md → [N]
         - Tasks with [P]: grep -c "\[P\]" tasks.md → [X]
         - Tasks with [US: grep -c "\[US[0-9]\]" tasks.md → [Y]

         Target: All counts should match or exceed expectations

      4. Spec-to-Tasks Traceability (MANDATORY):
         User Story Coverage:
         - Total user stories in spec.md: [N]
         - User stories with task phases: [M]
         - Coverage: [M/N * 100]%

         Target: 100% coverage

         Missing Coverage (if any):
         - [User Story X]: No task phase - [Why]

      5. Parallel Opportunity Analysis (MANDATORY):
         Run: grep -c "\[P\]" tasks.md
         Output:
           Parallelizable tasks: [X]
           Total tasks: [N]
           Parallelization ratio: [X/N * 100]%

         Expected: ≥30% for efficient implementation

      IF any evidence is MISSING:
        ❌ CANNOT report completion
        → Gather missing evidence first
        → Re-run this step with complete evidence
      ```

   **Hallucination Prevention (7 Red Flags for Task Generation):**
   ```yaml
   Detect and BLOCK these patterns:

   🚨 "All user stories covered" WITHOUT showing task mapping
      → Self-correction: "Wait, I need to map each story to tasks"

   🚨 "Tasks generated" WITHOUT format validation
      → Self-correction: "Let me verify all tasks follow checklist format"

   🚨 "Complete task list" WITH missing user stories
      → Self-correction: "Some user stories have no task phases"

   🚨 Claiming executability WITHOUT file paths
      → Self-correction: "Tasks need specific file paths"

   🚨 Marking tasks [P] WITHOUT checking dependencies
      → Self-correction: "I need to verify parallel tasks are safe"

   🚨 Skipping traceability validation
      → Self-correction: "I must map tasks back to requirements"

   🚨 "Ready for implementation" statements WITHOUT evidence
      → Self-correction: "Need to verify completeness criteria first"

   IF detected: STOP → Gather evidence → Report honestly
   ```

   **Determine Status**:

   ✅ **READY for Implementation**:
   ```yaml
   Criteria (ALL must be met):
     - 100% user story coverage (all stories have task phases)
     - All tasks follow checklist format (checkbox, ID, story label, file path)
     - Dependencies logical (foundational blocks stories, no cycles)
     - Parallelization safe (different files, no incomplete dependencies)
     - MVP scope clear and testable
     - Independent test criteria for each story

   IF ALL criteria met:
     → Proceed to Step 7 (present completion report)
   ```

   ⚠️ **NEEDS REVIEW** (with risks noted):
   ```yaml
   Criteria:
     - ≥90% user story coverage (minor gaps documented)
     - Most tasks properly formatted (≤5 format violations)
     - Dependencies mostly correct
     - MVP scope clear

   IF criteria met:
     → Present risks to user
     → Ask: "Minor gaps/format issues. Proceed to implementation or fix tasks?"
     → Wait for user decision before Step 7
   ```

   ❌ **NOT READY** (regeneration needed):
   ```yaml
   Criteria (ANY triggers NOT READY):
     - <90% user story coverage
     - Significant format violations (>5 tasks)
     - Circular dependencies or blocking issues
     - Unsafe parallelization
     - MVP scope unclear
     - Missing independent test criteria

   IF NOT READY:
     → Present issues to user with evidence
     → Recommend: "Regenerate tasks.md addressing [specific issues]"
     → STOP before Step 7 (do not proceed to implementation)
   ```

   **Output Format** (Present to User - ONLY if evidence provided):
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ✅ Task List Review Complete
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

   Branch: [branch-name]
   Tasks: [path/to/tasks.md]

   ## User Story Coverage

   Total User Stories: [N]
   Covered: [M]/[N] ([X]%)

   User Story 1 (P1): [Name]
   ✓ Tasks: [X] (T001-TXXX)
   ✓ Models: [Y], Services: [Z], Endpoints: [W]
   ✓ Independent test: [criteria]

   User Story 2 (P2): [Name]
   ✓ Tasks: [X] (TXXX-TYYY)
   [... same structure ...]

   [List all user stories with task breakdown]

   ## Task Format Validation

   ✓ Total tasks: [N]
   ✓ Checklist format: [N]/[N] (100%)
   ✓ Task IDs sequential: T001 → T[max]
   ✓ File paths specific: [N]/[N] (100%)
   ✓ Story labels match spec: [Yes/No]

   [IF format violations:]
   Format Issues:
   - Line [X]: [Issue]

   ## Dependency & Parallelization

   Phase Structure:
   ✓ Phase 1 (Setup): [X] tasks
   ✓ Phase 2 (Foundational): [Y] tasks - Blocks user stories
   ✓ Phase 3+ (Stories): [Z] phases in priority order
   ✓ Final Phase (Polish): [W] tasks

   User Story Dependencies:
   - US1: Independent ✅
   - US2: Depends on [US1/Foundational/etc.]
   [... list all ...]

   Parallelization:
   ✓ Parallel tasks: [X]/[N] ([Y]%)
   ✓ Safe parallelization: [Yes/No]
   ✓ Parallel examples provided: [Yes/No]

   MVP Scope:
   ✓ Recommended: [User Story name]
   ✓ Tasks: [N] tasks (T001-TXXX)
   ✓ Deliverable: [What MVP enables]

   ## Readiness Assessment

   [IF ✅ READY:]
   ✅ All user stories covered with complete task phases
   ✅ All tasks properly formatted and executable
   ✅ Dependencies logical and safe parallelization
   ✅ MVP scope clear and testable
   ✅ Ready to proceed to implementation (/speckit.implement)

   [IF ⚠️ NEEDS REVIEW:]
   ⚠️ Minor issues detected:
      - [Issue 1]: [Why low impact]
      - [Issue 2]: [Why low impact]

   Risk: May require task adjustments during implementation.
   Coverage: [X]% (target: 100%)

   [IF ❌ NOT READY:]
   ❌ Critical issues prevent implementation:
      - [Issue 1]: [HIGH impact] - [Why blocks implementation]
      - [Issue 2]: [HIGH impact] - [Why blocks implementation]

   Required Actions:
   1. [Specific fix - e.g., "Add task phases for User Stories 3-5"]
   2. [Specific fix - e.g., "Fix format violations in lines X-Y"]
   3. [Specific fix - e.g., "Resolve circular dependency between US2 and US4"]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Evidence: Task List Details
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   File Info:
   ```
   [Output of: ls -lah FEATURE_DIR/tasks.md]
   ```

   Task Count:
   ```
   [Output of: grep -c "^- \[ \]" tasks.md]
   Total: [N] tasks
   ```

   Format Compliance:
   ```
   Checkboxes: [N]
   Task IDs: [N]
   [P] markers: [X]
   [US] labels: [Y]
   ```

   Phase Breakdown (from tasks.md):
   - Setup: T001-TXXX ([X] tasks)
   - Foundational: TXXX-TYYY ([Y] tasks)
   - User Story phases: TYYY-TZZZ ([Z] tasks)
   - Polish: TZZZ-T[max] ([W] tasks)

   User Story Mapping:
   [Map from spec.md user stories to task phase headers]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Next Steps
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [IF ✅ READY:]
   1. Start implementation:
      /speckit.implement

   2. Or review task breakdown:
      - Tasks: [path/to/tasks.md]
      - Review specific phase: [which phase]

   3. Or start with MVP first:
      /speckit.implement [MVP scope: User Story 1]

   [IF ⚠️ NEEDS REVIEW:]
   Choose one:
   1. Proceed with noted risks:
      /speckit.implement

   2. Fix minor issues first:
      [Specific fixes for each issue]

   3. Review tasks:
      [path/to/tasks.md]

   [IF ❌ NOT READY:]
   REQUIRED: Fix critical issues before implementation:

   1. For coverage gaps:
      - Review spec.md user stories
      - Regenerate tasks for missing stories
      - Run /speckit.tasks again

   2. For format issues:
      - Fix task format violations
      - Ensure all tasks have: checkbox, ID, story label, file path
      - Validate with format grep commands

   3. For dependency issues:
      - Review phase structure
      - Fix circular dependencies
      - Ensure foundational phase blocks correctly

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Ready to proceed? (yes/review tasks/fix issues/start MVP)
   ```

   **User Interaction:**
   ```yaml
   IF user says "yes" or "proceed":
     IF status = READY:
       → Suggest: /speckit.implement
       → Or: /speckit.implement [MVP scope]
     IF status = NEEDS REVIEW:
       → Warn about risks, then suggest: /speckit.implement
     IF status = NOT READY:
       → Block: "Cannot proceed - critical issues in task list"
       → Suggest: /speckit.tasks (regenerate)

   IF user says "review" or "review tasks":
     → Show tasks.md path
     → Offer to display specific phase or user story tasks

   IF user says "fix" or "fix issues":
     → List specific issues with line numbers
     → Suggest fixes for each issue
     → Ask: "Fix automatically or manual edit?"

   IF user says "start MVP" or "MVP":
     → Identify MVP scope (usually User Story 1)
     → List MVP tasks (T001-TXXX)
     → Suggest: /speckit.implement [MVP scope]

   IF EVIDENCE MISSING:
     ❌ "Cannot complete task list review without evidence."
     → List missing evidence
     → Gather evidence automatically where possible
     → Re-run review gate
   ```

   **Benefits** (from PM Agent Reflexion pattern):
   - ✅ Prevents implementation start with incomplete task breakdown
   - ✅ Evidence-based task completeness validation
   - ✅ No false "ready" claims without traceability proof
   - ✅ Transparent validation of task list quality
   - ✅ User confidence in implementation plan

Context for task generation: {ARGS}

The tasks.md should be immediately executable - each task must be specific enough that an LLM can complete it without additional context.

## Task Generation Rules

**CRITICAL**: Tasks MUST be organized by user story to enable independent implementation and testing.

**Tests are OPTIONAL**: Only generate test tasks if explicitly requested in the feature specification or if user requests TDD approach.

### Checklist Format (REQUIRED)

Every task MUST strictly follow this format:

```text
- [ ] [TaskID] [P?] [Story?] Description with file path
```

**Format Components**:

1. **Checkbox**: ALWAYS start with `- [ ]` (markdown checkbox)
2. **Task ID**: Sequential number (T001, T002, T003...) in execution order
3. **[P] marker**: Include ONLY if task is parallelizable (different files, no dependencies on incomplete tasks)
4. **[Story] label**: REQUIRED for user story phase tasks only
   - Format: [US1], [US2], [US3], etc. (maps to user stories from spec.md)
   - Setup phase: NO story label
   - Foundational phase: NO story label
   - User Story phases: MUST have story label
   - Polish phase: NO story label
5. **Description**: Clear action with exact file path
6. **Requirement Reference** (NEW v2.3): Add `(_REQ-XX-Y.Z_)` for tasks implementing specific requirements
   - Link to requirement IDs from spec.md (REQ-AUTH-1.1, REQ-DASH-2.1, etc.)
   - Add at end of description when task directly implements a requirement
   - Multiple requirements: `(_REQ-AUTH-1.1, REQ-AUTH-1.2_)`

**Examples**:

- ✅ CORRECT: `- [ ] T001 Create project structure per implementation plan`
- ✅ CORRECT: `- [ ] T005 [P] Implement authentication middleware in src/middleware/auth.py (_REQ-AUTH-1.1_)`
- ✅ CORRECT: `- [ ] T012 [P] [US1] Create User model in src/models/user.py`
- ✅ CORRECT: `- [ ] T014 [US1] Implement UserService with 2-second response time in src/services/user_service.py (_REQ-AUTH-1.1, REQ-AUTH-1.2_)`
- ✅ CORRECT (v2.3): `- [ ] T018 [US1] Add login endpoint POST /api/auth/login in src/routes/auth.py (_REQ-AUTH-1.1_)`
- ❌ WRONG: `- [ ] Create User model` (missing ID and Story label)
- ❌ WRONG: `T001 [US1] Create model` (missing checkbox)
- ❌ WRONG: `- [ ] [US1] Create User model` (missing Task ID)
- ❌ WRONG: `- [ ] T001 [US1] Create model` (missing file path)

### Task Organization

1. **From User Stories (spec.md)** - PRIMARY ORGANIZATION:
   - Each user story (P1, P2, P3...) gets its own phase
   - Map all related components to their story:
     - Models needed for that story
     - Services needed for that story
     - Endpoints/UI needed for that story
     - If tests requested: Tests specific to that story
   - Mark story dependencies (most stories should be independent)

2. **From Requirements (spec.md + plan.md)** - NEW v2.3 TRACEABILITY:
   - Extract requirement IDs from spec.md (REQ-XX-Y.Z format)
   - Extract requirement references from plan.md phases (_Requirements: REQ-XX-Y.Z_)
   - For each task that implements a requirement:
     - Add requirement reference at end: `(_REQ-XX-Y.Z_)`
     - Multiple requirements: `(_REQ-XX-1.1, REQ-XX-1.2_)`
   - This enables full traceability: spec.md → plan.md → tasks.md → tests

3. **From Contracts**:
   - Map each contract/endpoint → to the user story it serves
   - If tests requested: Each contract → contract test task [P] before implementation in that story's phase

4. **From Data Model**:
   - Map each entity to the user story(ies) that need it
   - If entity serves multiple stories: Put in earliest story or Setup phase
   - Relationships → service layer tasks in appropriate story phase

5. **From Setup/Infrastructure**:
   - Shared infrastructure → Setup phase (Phase 1)
   - Foundational/blocking tasks → Foundational phase (Phase 2)
   - Story-specific setup → within that story's phase

### Phase Structure

- **Phase 1**: Setup (project initialization)
- **Phase 2**: Foundational (blocking prerequisites - MUST complete before user stories)
- **Phase 3+**: User Stories in priority order (P1, P2, P3...)
  - Within each story: Tests (if requested) → Models → Services → Endpoints → Integration
  - Each phase should be a complete, independently testable increment
- **Final Phase**: Polish & Cross-Cutting Concerns

---

## Final Step: Update Specification Metadata

After generating tasks.md, update `spec-metadata.json` in the spec directory:

```json
{
  "phase": "tasks",
  "approvals": {
    "tasks": {
      "generated": true,
      "approved": false,
      "timestamp": "{ISO 8601 timestamp}"
    }
  },
  "metadata": {
    "updated_at": "{ISO 8601 timestamp}"
  }
}
```

**Recommend Next Steps**:
- "✅ Tasks generated. Review tasks-N.md and run `/speckit.implement` to begin implementation"
- "Or run `/speckit.status` to check current workflow state"
- IF test-first workflow: "Run tests (they should fail), then implement to make them pass"
