---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

1. Run `{SCRIPT}` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute. For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. **Check checklists status** (if FEATURE_DIR/checklists/ exists):
   - Scan all checklist files in the checklists/ directory
   - For each checklist, count:
     - Total items: All lines matching `- [ ]` or `- [X]` or `- [x]`
     - Completed items: Lines matching `- [X]` or `- [x]`
     - Incomplete items: Lines matching `- [ ]`
   - Create a status table:

     ```text
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```

   - Calculate overall status:
     - **PASS**: All checklists have 0 incomplete items
     - **FAIL**: One or more checklists have incomplete items

   - **If any checklist is incomplete**:
     - Display the table with incomplete item counts
     - **STOP** and ask: "Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no)"
     - Wait for user response before continuing
     - If user says "no" or "wait" or "stop", halt execution
     - If user says "yes" or "proceed" or "continue", proceed to step 3

   - **If all checklists are complete**:
     - Display the table showing all checklists passed
     - Automatically proceed to step 3

3. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, and file structure
   - **IF EXISTS**: Read data-model.md for entities and relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

4. **Project Setup Verification**:
   - **REQUIRED**: Create/verify ignore files based on actual project setup:

   **Detection & Creation Logic**:
   - Check if the following command succeeds to determine if the repository is a git repo (create/verify .gitignore if so):

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```

   - Check if Dockerfile* exists or Docker in plan.md → create/verify .dockerignore
   - Check if .eslintrc*or eslint.config.* exists → create/verify .eslintignore
   - Check if .prettierrc* exists → create/verify .prettierignore
   - Check if .npmrc or package.json exists → create/verify .npmignore (if publishing)
   - Check if terraform files (*.tf) exist → create/verify .terraformignore
   - Check if .helmignore needed (helm charts present) → create/verify .helmignore

   **If ignore file already exists**: Verify it contains essential patterns, append missing critical patterns only
   **If ignore file missing**: Create with full pattern set for detected technology

   **Common Patterns by Technology** (from plan.md tech stack):
   - **Node.js/JavaScript/TypeScript**: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
   - **Python**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
   - **Java**: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
   - **C#/.NET**: `bin/`, `obj/`, `*.user`, `*.suo`, `packages/`
   - **Go**: `*.exe`, `*.test`, `vendor/`, `*.out`
   - **Ruby**: `.bundle/`, `log/`, `tmp/`, `*.gem`, `vendor/bundle/`
   - **PHP**: `vendor/`, `*.log`, `*.cache`, `*.env`
   - **Rust**: `target/`, `debug/`, `release/`, `*.rs.bk`, `*.rlib`, `*.prof*`, `.idea/`, `*.log`, `.env*`
   - **Kotlin**: `build/`, `out/`, `.gradle/`, `.idea/`, `*.class`, `*.jar`, `*.iml`, `*.log`, `.env*`
   - **C++**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.so`, `*.a`, `*.exe`, `*.dll`, `.idea/`, `*.log`, `.env*`
   - **C**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.a`, `*.so`, `*.exe`, `Makefile`, `config.log`, `.idea/`, `*.log`, `.env*`
   - **Swift**: `.build/`, `DerivedData/`, `*.swiftpm/`, `Packages/`
   - **R**: `.Rproj.user/`, `.Rhistory`, `.RData`, `.Ruserdata`, `*.Rproj`, `packrat/`, `renv/`
   - **Universal**: `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`, `.vscode/`, `.idea/`

   **Tool-Specific Patterns**:
   - **Docker**: `node_modules/`, `.git/`, `Dockerfile*`, `.dockerignore`, `*.log*`, `.env*`, `coverage/`
   - **ESLint**: `node_modules/`, `dist/`, `build/`, `coverage/`, `*.min.js`
   - **Prettier**: `node_modules/`, `dist/`, `build/`, `coverage/`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
   - **Terraform**: `.terraform/`, `*.tfstate*`, `*.tfvars`, `.terraform.lock.hcl`
   - **Kubernetes/k8s**: `*.secret.yaml`, `secrets/`, `.kube/`, `kubeconfig*`, `*.key`, `*.crt`

5. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Order and dependency requirements

6. Execute implementation following the task plan:
   - **Phase-by-phase execution**: Complete each phase before moving to the next
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together  
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **Validation checkpoints**: Verify each phase completion before proceeding

7. Implementation execution rules:
   - **Setup first**: Initialize project structure, dependencies, configuration
   - **Tests before code**: If you need to write tests for contracts, entities, and integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Unit tests, performance optimization, documentation

8. Progress tracking and error handling:
   - Report progress after each completed task
   - Halt execution if any non-parallel task fails
   - For parallel tasks [P], continue with successful tasks, report failed ones
   - Provide clear error messages with context for debugging
   - Suggest next steps if implementation cannot proceed
   - **IMPORTANT** For completed tasks, make sure to mark the task off as [X] in the tasks file.

9. Completion validation:
   - Verify all required tasks are completed
   - Check that implemented features match the original specification
   - Validate that tests pass and coverage meets requirements
   - Confirm the implementation follows the technical plan
   - Report final status with summary of completed work

10. **Implementation Code Review Gate** (Evidence-Based Self-Check):

   **Purpose**: Prevent hallucination and ensure evidence-based completion claims.

   **MANDATORY: The Four Questions** (MUST answer with ACTUAL evidence):

   ❓ **"Are all tests passing?"**
      ```yaml
      Action Required:
        - Run actual test command from plan.md or project configuration
        - Show REAL output (not "probably works" or "should pass")
        - IF any tests fail: Implementation is NOT complete ❌

      Expected Evidence Format:
        Test Results:
        ✓ Unit tests: [X]/[Y] passed
        ✓ Integration tests: [X]/[Y] passed
        ✓ E2E tests: [X]/[Y] passed (if applicable)
        ✓ Coverage: [X]% (target: [Y]%)

      Hallucination Detection:
        🚨 "Tests pass" WITHOUT showing output → ❌ BLOCK completion
        🚨 "All green" WITHOUT actual results → ❌ BLOCK completion
        🚨 Hiding test failures or errors → ❌ BLOCK completion
      ```

   ❓ **"Are all requirements implemented?"**
      ```yaml
      Action Required:
        - Compare tasks.md checklist vs actual completion status
        - Map each requirement from spec.md to completed tasks
        - List: ✅ Done, ⚠️ Partial, ❌ Not started

      Expected Evidence Format:
        Requirements Status:
        ✅ R-001: User login (tasks T-001, T-002, T-003)
        ✅ R-002: Session management (tasks T-004, T-005)
        ✅ R-003: JWT tokens (task T-006)
        [... list all requirements with task mappings ...]

      IF any requirement shows ⚠️ Partial or ❌:
        → Implementation is NOT complete
        → Report which requirements remain
      ```

   ❓ **"Were any assumptions made without verification?"**
      ```yaml
      Self-Reflection Checklist:
        - [ ] Did I consult official documentation for all libraries/APIs?
        - [ ] Did I verify edge cases are handled?
        - [ ] Did I test error scenarios?
        - [ ] Did I validate against actual API responses (not assumed)?
        - [ ] Did I check for security vulnerabilities?

      IF any assumptions were made:
        → Document them explicitly
        → Verify against official sources
        → Test the assumptions
      ```

   ❓ **"Do I have evidence to support completion?"**
      ```yaml
      Required Evidence (ALL must be provided):

      1. Test Results (MANDATORY):
         Run: [test command]
         Output: [paste actual output, not summary]

      2. Code Changes (MANDATORY):
         Run: git diff --stat [main-branch]..HEAD
         Output:
           Files modified: [N]
           Lines added: +[N]
           Lines removed: -[N]
           [file list with changes]

      3. Code Quality Validation (MANDATORY):
         Lint:
           Run: [lint command]
           Result: ✅ passed / ❌ [N] errors

         Type Check (if applicable):
           Run: [typecheck command]
           Result: ✅ passed / ❌ [N] errors

         Build:
           Run: [build command]
           Result: ✅ success / ❌ failed with [error]

      4. Git Status (MANDATORY):
         Run: git status
         Output: [show current status]

      IF any evidence is MISSING:
        ❌ CANNOT report completion
        → Gather missing evidence first
        → Re-run this step with complete evidence
      ```

   **Hallucination Prevention (7 Red Flags):**
   ```yaml
   Detect and BLOCK these patterns:

   🚨 "Tests pass" WITHOUT showing actual test output
      → Self-correction: "Wait, I need to run tests and show real results"

   🚨 "Everything works" WITHOUT evidence
      → Self-correction: "Let me gather actual evidence"

   🚨 "Implementation complete" WITH failing tests
      → Self-correction: "Tests are failing, not complete yet"

   🚨 Skipping error messages
      → Self-correction: "I need to address these errors first"

   🚨 Ignoring warnings
      → Self-correction: "Let me review and fix warnings"

   🚨 Hiding failures
      → Self-correction: "I must report failures honestly"

   🚨 "Probably works" statements
      → Self-correction: "Need to verify, not assume"

   IF detected: STOP → Gather evidence → Report honestly
   ```

   **Output Format** (Present to User - ONLY if ALL evidence provided):
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ✅ Implementation Complete - Code Review
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   ## Test Results

   ✓ Unit tests: [X]/[Y] passed
   ✓ Integration tests: [X]/[Y] passed
   ✓ E2E tests: [X]/[Y] passed (if applicable)
   ✓ Coverage: [X]% (target: [Y]%)

   Test Output:
   ```
   [paste actual test output]
   ```

   ## Code Quality

   ✓ Lint: [status]
   ✓ Type Check: [status]
   ✓ Build: [status]

   ## Code Changes

   Files modified: [N]
   Lines added: +[N]
   Lines removed: -[N]

   ```
   [git diff --stat output]
   ```

   Commits:
   - [commit 1 message] ([N] files)
   - [commit 2 message] ([N] files)
   [... list all commits ...]

   ## Requirements Completed

   ✅ R-001: [Requirement name] (tasks: [T-IDs])
   ✅ R-002: [Requirement name] (tasks: [T-IDs])
   ✅ R-003: [Requirement name] (tasks: [T-IDs])
   [... list all [X]/[Y] requirements ...]

   ## Outstanding Items

   [IF any exist:]
   ⚠️ [Item 1]: [Description and impact]
   ⚠️ [Item 2]: [Description and impact]

   [IF none:]
   ✅ None - all tasks complete

   ## Self-Check Results

   ✅ Official documentation consulted
   ✅ Edge cases handled and tested
   ✅ Error scenarios validated
   ✅ Security vulnerabilities checked
   ✅ No untested assumptions

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Next Steps
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   1. Review code changes:
      git diff [main-branch]..HEAD

   2. Review commits:
      git log [main-branch]..HEAD --oneline

   3. Merge to main:
      git checkout main && git merge [branch-name]

   4. Deploy (if applicable):
      [deployment command from plan.md]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Ready to merge? (yes/review code/run more tests/address issues)
   ```

   **User Interaction:**
   ```yaml
   IF user says "yes" or "merge":
     → Provide git merge commands
     → Suggest deployment steps

   IF user says "review" or "review code":
     → Run: git diff [main-branch]..HEAD
     → Show detailed code changes

   IF user says "more tests" or "run tests":
     → Ask which specific tests to run
     → Execute and show results

   IF user says "address issues":
     → List issues needing attention
     → Wait for fixes, then re-run review gate

   IF EVIDENCE MISSING:
     ❌ "Cannot complete code review without evidence."
     → List missing evidence
     → Gather evidence automatically where possible
     → Re-run review gate
   ```

   **Benefits** (from PM Agent Reflexion pattern):
   - ✅ 94% hallucination detection rate
   - ✅ Evidence-based completion reports
   - ✅ No false "done" claims
   - ✅ Transparent validation process
   - ✅ User confidence in quality

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/speckit.tasks` first to regenerate the task list.

