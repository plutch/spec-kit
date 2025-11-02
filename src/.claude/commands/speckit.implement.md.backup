---
description: Execute the implementation plan by processing and executing all tasks defined in tasks.md
scripts:
  sh: .specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: .specify/scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
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

9. **Implementation Quality Gate** (Pre-Commit Validation):

   **Purpose**: Systematic validation before marking implementation complete to ensure production readiness, security compliance, and alignment with specifications.

   **Execute Multi-Layer Quality Checks**:

   **A. Code Quality Validation** 🟠 MAJOR

   - [ ] **Linting**: Run project linter (eslint, pylint, clippy, etc.)
     - Zero errors allowed for critical severity issues
     - Warnings documented or suppressed with justification
     - Command: `npm run lint` / `cargo clippy` / `pylint src/` (project-specific)

   - [ ] **Formatting**: Run code formatter check
     - All files formatted consistently (prettier, black, rustfmt, gofmt)
     - No mixed tabs/spaces, consistent indentation
     - Command: `npm run format:check` / `black --check .` / `cargo fmt --check`

   - [ ] **Type Safety** (if TypeScript/typed language):
     - Zero type errors
     - No `any` types in production code (or explicitly justified)
     - Command: `tsc --noEmit` / `mypy src/` / `flow check`

   - [ ] **Dead Code Detection**:
     - No unused imports or variables
     - No commented-out code blocks (remove or document why kept)
     - No unreachable code paths

   **B. Security Validation** 🔴 CRITICAL

   - [ ] **No Hardcoded Secrets**:
     - Scan for API keys, passwords, tokens in code
     - All secrets loaded from vault/env variables
     - Command: `git secrets --scan` / `truffleHog` / `gitleaks detect`

   - [ ] **Input Validation Present**:
     - All user inputs validated (API endpoints, form submissions)
     - Schema validation implemented (Zod, Joi, JSON Schema)
     - SQL queries parameterized (no string concatenation)
     - HTML output escaped (XSS prevention)

   - [ ] **Authentication/Authorization**:
     - Protected endpoints have auth middleware
     - Database queries filtered by user/tenant permissions
     - Admin functions extra-protected

   - [ ] **Sensitive Data Handling**:
     - PII/credentials not logged in plaintext
     - Encryption used where required (from Security Review in plan.md)
     - Credit cards never stored (tokenization used)

   - [ ] **Dependency Security**:
     - Run security audit (npm audit, cargo audit, pip-audit)
     - High/critical vulnerabilities addressed or documented
     - Command: `npm audit --audit-level=high` / `cargo audit` / `pip-audit`

   **C. Test Coverage Validation** 🟠 MAJOR

   - [ ] **Test Execution**:
     - All tests pass (unit, integration, E2E)
     - No skipped tests without documented justification
     - Command: `npm test` / `cargo test` / `pytest` / `go test ./...`

   - [ ] **Coverage Thresholds Met** (from Test Strategy in plan.md):
     - Critical paths: ≥ 90% coverage
     - Business logic: ≥ 80% coverage
     - Overall: ≥ 75% coverage
     - Command: `npm run test:coverage` / `cargo tarpaulin` / `pytest --cov`

   - [ ] **High-Risk Requirements Tested**:
     - All 🔴 CRITICAL risk requirements (Risk Score ≥ 8) have tests
     - Edge cases from Edge Case Detection covered
     - Error scenarios tested (timeout, invalid input, auth failure)

   **D. Specification Alignment** 🟠 MAJOR

   - [ ] **Requirements Traceability**:
     - All P1/P2 User Stories implemented
     - All FR-XXX Functional Requirements addressed
     - All BR-XXX Business Rules enforced in code
     - Spot-check: Pick 3 random FRs, verify implementation matches spec

   - [ ] **Acceptance Criteria Met**:
     - All Given/When/Then scenarios testable
     - Automated tests validate acceptance criteria
     - Manual testing performed for UI/UX scenarios (if applicable)

   - [ ] **Technical Plan Adherence**:
     - File structure matches plan.md architecture
     - Tech stack used as specified (no surprise dependencies)
     - API contracts match contracts/ specifications
     - Data model matches data-model.md

   - [ ] **Risk Mitigation Validated**:
     - All 🔴 HIGH RISK requirements have mitigation controls implemented
     - Security controls from Security Review present
     - Performance requirements met (if specified in plan.md)

   **E. Documentation Completeness** 🟡 MEDIUM

   - [ ] **Code Documentation**:
     - Public APIs documented (JSDoc, Rustdoc, docstrings)
     - Complex algorithms explained
     - Non-obvious decisions have comments

   - [ ] **README/Quick-start Updated**:
     - Installation steps current
     - Environment variables documented
     - Development setup instructions accurate

   - [ ] **API Documentation** (if applicable):
     - OpenAPI/Swagger spec updated
     - Endpoint descriptions current
     - Example requests/responses provided

   **F. Build & Deploy Readiness** 🟠 MAJOR

   - [ ] **Build Success**:
     - Production build succeeds
     - No build warnings in production mode
     - Command: `npm run build` / `cargo build --release` / `go build`

   - [ ] **Docker Build** (if using containers):
     - Dockerfile builds successfully
     - Image size reasonable (< 500MB if possible)
     - Multi-stage builds used (if applicable)
     - Command: `docker build -t feature-test .`

   - [ ] **Environment Configuration**:
     - .env.example updated with new variables
     - All required env vars documented
     - No production secrets in .env.example

   **Quality Gate Decision Matrix**:

   | Check Category | Status | Blocking? | Action Required |
   |----------------|--------|-----------|-----------------|
   | Code Quality | [🟢 PASS / 🟡 WARNINGS / 🔴 FAIL] | WARN | Fix or document suppressions |
   | Security | [🟢 PASS / 🔴 FAIL] | **BLOCK** | Must fix before commit |
   | Test Coverage | [🟢 PASS / 🟡 PARTIAL / 🔴 FAIL] | WARN | Meet thresholds or justify |
   | Spec Alignment | [🟢 PASS / 🟡 PARTIAL / 🔴 FAIL] | WARN | Verify FR/BR implementation |
   | Documentation | [🟢 PASS / 🟡 PARTIAL] | INFO | Update if incomplete |
   | Build Readiness | [🟢 PASS / 🔴 FAIL] | **BLOCK** | Must build successfully |

   **Blocking Criteria** (Implementation CANNOT proceed if):
   - 🔴 Security vulnerabilities detected (hardcoded secrets, SQL injection risk, missing auth)
   - 🔴 Build fails
   - 🔴 Critical tests failing
   - 🔴 P1 User Stories not implemented

   **Warning Criteria** (Can proceed but create backlog tickets):
   - 🟡 Code quality warnings (linting, formatting)
   - 🟡 Test coverage below threshold (but critical paths covered)
   - 🟡 Incomplete documentation

   **Generate Quality Gate Report**:

   ```markdown
   ## Implementation Quality Gate Report

   **Feature**: [FEATURE_NAME]
   **Date**: [DATE]
   **Status**: [🟢 READY TO COMMIT | 🟡 NEEDS ATTENTION | 🔴 BLOCKED]

   ### Quality Checks Summary

   | Category | Status | Details | Blocking? |
   |----------|--------|---------|-----------|
   | Code Quality | 🟢 PASS | 0 errors, 3 warnings (documented) | No |
   | Security | 🟢 PASS | No secrets, auth implemented, audit clean | **Yes** |
   | Test Coverage | 🟡 PARTIAL | 78% overall (target 80%), critical paths 92% | No |
   | Spec Alignment | 🟢 PASS | All P1/P2 stories implemented, FRs validated | No |
   | Documentation | 🟡 PARTIAL | API docs updated, README needs env vars section | No |
   | Build | 🟢 PASS | Production build successful, Docker image 320MB | **Yes** |

   ### Detailed Findings

   **🟢 PASSED CHECKS**:
   - ✅ All 12 unit tests passing
   - ✅ All 8 integration tests passing
   - ✅ No hardcoded secrets detected
   - ✅ SQL injection prevention (parameterized queries)
   - ✅ XSS prevention (HTML escaping)
   - ✅ All P1 User Stories implemented
   - ✅ Production build successful

   **🟡 WARNINGS** (Non-blocking, backlog tickets created):
   - ⚠️ Test coverage 78% (target 80%) - 2% short, but all critical paths covered at 92%
   - ⚠️ 3 ESLint warnings (prefer-const in non-critical utils, suppressed with justification)
   - ⚠️ README missing new environment variables section (created ticket #247)

   **🔴 BLOCKERS** (None - ready to commit)

   ### Risk Mitigation Status

   | High-Risk Requirement | Risk Score | Mitigation Status |
   |----------------------|------------|-------------------|
   | FR-001: Payment processing | 🔴 10 | ✅ Stripe test mode, idempotency keys, webhook validation |
   | FR-003: Tenant isolation | 🔴 9 | ✅ Row-level security, tenant_id filtering on all queries |
   | FR-007: External API integration | 🟠 6 | ✅ Timeout handling (30s), retry with backoff, circuit breaker |

   ### Compliance Status

   - **OWASP Top 10**: All critical vulnerabilities (A01, A02, A03, A07, A08) mitigated
   - **GDPR** (if applicable): Right to access/delete endpoints implemented
   - **PCI-DSS** (if applicable): No card storage, tokenization used

   ### Next Steps

   **Ready to Commit**:
   - [ ] Create commit with implementation
   - [ ] Create backlog tickets for warnings (test coverage gap, docs)
   - [ ] Proceed to `/speckit.reconcile` for post-implementation validation

   **NOT Ready (if blocked)**:
   - [ ] Fix blocking issues listed above
   - [ ] Re-run quality gate validation
   ```

   **Output**: Display Quality Gate Report and gate decision (READY / NEEDS ATTENTION / BLOCKED)

10. **Final Validation & Completion**:
   - If Quality Gate status is 🔴 BLOCKED: HALT and require fixes before proceeding
   - If Quality Gate status is 🟡 NEEDS ATTENTION: Create backlog tickets, allow commit
   - If Quality Gate status is 🟢 READY TO COMMIT: Proceed to completion
   - Verify all required tasks are marked [X] in tasks.md
   - Report final status with summary of completed work
   - Suggest running `/speckit.reconcile` for post-implementation gap closure

11. **Implementation Code Review Gate** (Evidence-Based Self-Check):

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

