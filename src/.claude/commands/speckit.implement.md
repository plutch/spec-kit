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

2. **Pre-Implementation Approval Gate Check** (NEW v2.3):

   **Check if previous phases are approved before proceeding:**

   - Read `spec-metadata.json` from FEATURE_DIR (if exists)
   - Check approval status and current phase

   **Approval Gate Logic**:
   ```
   IF spec-metadata.json exists:
     IF approvals.planning.approved == false:
       → 🔴 ERROR: "Planning not approved. Review plan.md and approve before implementation."
       → STOP execution

     IF approvals.tasks.approved == false AND tasks.md exists:
       → 🟠 WARN: "Tasks not approved. Recommend reviewing tasks.md before proceeding."
       → Ask: "Tasks not approved. Proceed anyway? (yes/no/-y for auto-approve)"
       → IF no: STOP
       → IF yes or -y flag: Continue (update tasks.approved = true)

     IF approvals.gap_analysis.approved == false AND metadata.risk_level == "HIGH":
       → 🟠 WARN: "HIGH RISK feature without gap analysis approval."
       → Recommend: "Run /speckit.status to review workflow state"

   ELSE:
     → ℹ️ INFO: "No spec-metadata.json found - proceeding without approval gates"
     → Recommend: "Consider running /speckit.memory to set up project memory for better workflow tracking"
   ```

   **Auto-Approval Flag**: If user provides `-y` flag, automatically approve current phase and proceed.

2.5. **Constitutional Pre-Flight Check** (NEW v2.4):

   **Purpose**: Validate that the implementation plan complies with the project constitution BEFORE code execution begins.

   **Check constitutional validation from planning phase:**

   - Read `.specify/memory/constitution.md` from repo root (if exists)
   - Read `plan.md` from FEATURE_DIR
   - Locate "Phase -1: Constitutional Validation" section in plan.md
   - Extract validation result: ✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL

   **Gate Logic**:
   ```
   IF plan.md contains "Phase -1: Constitutional Validation":

     IF validation result == ❌ FAIL:
       → 🔴 ERROR: "Constitutional violations from planning phase not resolved."
       → Show violations from plan.md
       → Message: "Address constitutional violations before implementation. Review plan.md Phase -1."
       → STOP execution

     IF validation result == ⚠️ CONDITIONAL:
       → 🟠 WARN: "Conditional constitutional approval detected."
       → Show conditional items from plan.md (e.g., "Repository pattern justified in Complexity Tracking")
       → Ask: "Constitutional approval is conditional. Verify justifications are valid. Proceed? (yes/no)"
       → IF no: STOP execution
       → IF yes: Continue with note: "Proceeding with conditional constitutional approval. Ensure justifications documented."

     IF validation result == ✅ PASS:
       → ✅ INFO: "Constitutional validation passed during planning phase."
       → Continue to next step

   ELSE (no constitutional validation in plan.md):
     → ℹ️ INFO: "No constitutional validation found in plan.md. This may be a legacy plan (pre-v2.4)."
     → ℹ️ INFO: "Constitutional validation will still run at Step 10.4 (Constitutional Reviewer)."
     → Message: "Consider updating plan with /speckit.plan to include Phase -1 constitutional validation."

     **Fallback Behavior** (addresses Finding #3):
     - Step 10.4 Constitutional Reviewer will run regardless of Phase -1 presence
     - TDD, simplicity, and integration checks still enforced
     - IF constitution.md exists: Pattern scanning runs at Step 10.4.5
     - IF constitution.md missing: Partial validation (see Graceful Degradation Matrix in Step 10.4)

     → Ask: "No Phase -1 constitutional validation in plan. Proceed with Step 10.4 validation only? (yes/no)"
     → IF no: STOP execution (user should update plan with Phase -1)
     → IF yes: Continue with note: "Proceeding without Phase -1 validation. Constitutional checks will run at Step 10.4."
   ```

   **Phase 2.6 Test Strategy Validation Check** (NEW v2.6):

   After Phase -1 check completes, check for Phase 2.6 test strategy validation:

   ```
   IF plan.md contains "Phase 2.6: Constitutional Test Strategy Validation" OR "Constitutional Test Strategy Validation (Article V)":

     - Locate validation section in plan.md
     - Extract validation result: ✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL

     IF validation result == ❌ FAIL:
       → 🔴 ERROR: "Test strategy violates Article V: Integration-First Testing"
       → Show violations from plan.md Phase 2.6 (e.g., "Violation 1: Mock Auth0 with jose library → Should use real Auth0 test tenant")
       → Message: "Revise Phase 2.5 Mock & Stub Strategy to comply with Article V. Run /speckit.plan to update."
       → STOP execution

     IF validation result == ⚠️ CONDITIONAL:
       → 🟠 WARN: "Test strategy has conditional Article V approval"
       → Show conditional items from plan.md (e.g., "Email mocks justified: No test mode available")
       → Ask: "Test strategy has conditional approval (e.g., email/SMS mocks). Proceed? (yes/no)"
       → IF no: STOP execution
       → IF yes: Continue with note: "Proceeding with conditional test strategy approval. Ensure justifications documented in Complexity Tracking."

     IF validation result == ✅ PASS:
       → ✅ INFO: "Test strategy complies with Article V: Integration-First Testing."
       → Continue to next step

   ELSE (no Phase 2.6 validation in plan.md):
     → ℹ️ INFO: "No Phase 2.6 test strategy validation found in plan.md. This may be a legacy plan (pre-v2.6)."
     → ℹ️ INFO: "Article V compliance will still be validated at Step 10.4.4 (Integration-First Validation)."
     → Message: "Consider updating plan with /speckit.plan to include Phase 2.6 test strategy validation."
     → Continue to next step (graceful degradation - Step 10.4 will validate)
   ```

   **Benefits**:
   - Prevents implementation of unconstitutional plans (Phase -1)
   - Prevents implementation with non-compliant test strategies (Phase 2.6)
   - Blocks test strategy violations BEFORE code execution (not at Step 10.4)
   - Enforces architectural discipline early
   - Catches violations before code changes occur
   - Zero overhead if planning passed (quick verification)

3. **Check checklists status** (if FEATURE_DIR/checklists/ exists):
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

4. Load and analyze the implementation context:
   - **REQUIRED**: Read tasks.md for the complete task list and execution plan
   - **REQUIRED**: Read plan.md for tech stack, architecture, and file structure
   - **IF EXISTS**: Read data-model.md for entities and relationships
   - **IF EXISTS**: Read contracts/ for API specifications and test requirements
   - **IF EXISTS**: Read research.md for technical decisions and constraints
   - **IF EXISTS**: Read quickstart.md for integration scenarios

5. **Project Setup Verification**:
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

6. Parse tasks.md structure and extract:
   - **Task phases**: Setup, Tests, Core, Integration, Polish
   - **Task dependencies**: Sequential vs parallel execution rules
   - **Task details**: ID, description, file paths, parallel markers [P]
   - **Execution flow**: Order and dependency requirements

7. Execute implementation following the task plan:
   - **Continuous execution**: Execute ALL tasks across all phases without interruption until complete
   - **Respect dependencies**: Run sequential tasks in order, parallel tasks [P] can run together
   - **Follow TDD approach**: Execute test tasks before their corresponding implementation tasks
   - **File-based coordination**: Tasks affecting the same files must run sequentially
   - **No phase checkpoints**: Validation occurs AFTER all tasks complete via sequential reviewers

8. Implementation execution rules:
   - **Setup first**: Initialize project structure, dependencies, configuration
   - **Tests before code**: If you need to write tests for contracts, entities, and integration scenarios
   - **Core development**: Implement models, services, CLI commands, endpoints
   - **Integration work**: Database connections, middleware, logging, external services
   - **Polish and validation**: Unit tests, performance optimization, documentation

9. Progress tracking and error handling:
   - Report progress after each completed task
   - Halt execution if any non-parallel task fails
   - For parallel tasks [P], continue with successful tasks, report failed ones
   - Provide clear error messages with context for debugging
   - Suggest next steps if implementation cannot proceed
   - **IMPORTANT** For completed tasks, make sure to mark the task off as [X] in the tasks file.

10. **Quality Gate Validation** (Parallel Execution):

   **Purpose**: Execute all validation checks in parallel to minimize latency while maintaining comprehensive quality assurance. All four reviewers (Code, Quality/Tests, Security, Constitutional) run concurrently, then results are aggregated.

   **Execution Model**: Run Code, Quality/Tests, Security, and Constitutional reviewers simultaneously (not sequentially), then aggregate results.

   **Performance Benefit**: ~53% faster than sequential execution (95s → 45s for typical projects)

   ---

   ### Parallel Validation Block

   Execute all reviewers simultaneously (not sequentially):

   #### 10.1 Code Quality Reviewer (Parallel Track 1)

   **Validation Tasks** (Run in parallel with other reviewers):

   **10.1.1 Code Quality Validation** 🟠 MAJOR

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

   **10.1.2 Specification Alignment** 🟠 MAJOR

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

   **10.1.3 Documentation Completeness** 🟡 MEDIUM

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

   **Code Reviewer Status** (determined after parallel execution):
   - ✅ **READY**: All checks pass, <5 linting warnings, all requirements traced, docs complete
   - ⚠️ **NEEDS REVIEW**: 5-20 linting warnings, minor spec gaps (<10% requirements), incomplete docs
   - ❌ **NOT READY**: Linting errors, type errors, P1 requirements missing, acceptance criteria unmet

   ---

   #### 10.2 Quality/Tests Reviewer (Parallel Track 2)

   **Validation Tasks** (Run in parallel with other reviewers):

   **10.2.1 Test Execution** 🔴 CRITICAL

   - [ ] **All Tests Pass**:
     - Unit tests passing (Jest, pytest, cargo test, go test)
     - Integration tests passing
     - E2E tests passing (if applicable)
     - No skipped tests without documented justification
     - Command: `npm test` / `cargo test` / `pytest` / `go test ./...`

   **10.2.2 Coverage Thresholds** 🟠 MAJOR

   - [ ] **Coverage Requirements Met** (from Test Strategy in plan.md):
     - **Critical paths**: ≥ 90% coverage (high-risk features, security, payments)
     - **Business logic**: ≥ 80% coverage (user stories, core workflows)
     - **Overall codebase**: ≥ 75% coverage
     - Command: `npm run test:coverage` / `cargo tarpaulin` / `pytest --cov`

   - [ ] **Coverage by Risk Level**:
     - Map coverage to Risk Assessment from spec.md
     - All 🔴 HIGH-RISK requirements (Risk Score ≥ 8) must have ≥ 90% coverage
     - 🟠 MEDIUM-RISK requirements should have ≥ 80% coverage

   **10.2.3 High-Risk Requirements Testing** 🟠 MAJOR

   For features with Risk Score ≥ 8 (from spec.md Risk Assessment):

   - [ ] **Edge Case Coverage**:
     - Boundary conditions tested (min/max values, empty inputs, limits)
     - Error conditions tested (timeouts, invalid input, auth failures)
     - State transitions tested (order of operations, concurrent access)
     - Race conditions tested (if multi-threaded/async)

   - [ ] **Production Resilience Tests**:
     - Timeout handling tested (external APIs, database queries)
     - Retry logic tested (exponential backoff, max retries)
     - Idempotency tested (can replay requests safely)
     - Crash recovery tested (graceful degradation)

   **10.2.4 Build Readiness** 🔴 CRITICAL

   - [ ] **Production Build Success**:
     - Build succeeds with zero errors
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

   **Quality/Tests Reviewer Status** (determined after parallel execution):
   - ✅ **READY**: All tests pass, coverage thresholds met, high-risk requirements tested, build succeeds
   - ⚠️ **NEEDS REVIEW**: 70-75% overall coverage (but critical paths ≥90%), minor edge cases missing, non-blocking build warnings
   - ❌ **NOT READY**: ANY test failures, coverage <70%, critical paths <90%, build fails

   ---

   #### 10.3 Security Reviewer (Parallel Track 3)

   **Validation Tasks** (Run in parallel with other reviewers):

   **10.3.1 Secrets Scanning** 🔴 CRITICAL

   - [ ] **No Hardcoded Secrets**:
     - Scan for API keys, passwords, tokens, private keys in code
     - All secrets loaded from environment variables or vault
     - No credentials in config files, logs, or comments
     - Commands:
       ```bash
       git secrets --scan
       truffleHog --regex --entropy=False .
       gitleaks detect
       grep -rE "(api_key|password|secret|token|private_key)" src/
       ```

   - [ ] **Secret Management Verification**:
     - .env.example contains placeholder values only (NO real secrets)
     - All required secrets documented in .env.example
     - Secrets loading mechanism implemented (dotenv, vault client)

   **10.3.2 Authentication & Authorization** 🔴 CRITICAL

   - [ ] **Authentication Implementation**:
     - Token validation present (JWT, OAuth, session)
     - Token expiration enforced
     - Refresh token rotation (if applicable)
     - Password hashing (bcrypt, argon2, scrypt - NOT md5/sha1)

   - [ ] **Authorization Controls**:
     - Protected endpoints have auth middleware
     - RBAC/ABAC enforced (role-based / attribute-based access control)
     - Principle of least privilege applied
     - Admin functions have extra protection (2FA, IP whitelist, audit log)

   - [ ] **Database Query Filtering**:
     - All queries filtered by user/tenant ID
     - Row-level security (RLS) enabled (if using Postgres)
     - No data leakage between tenants/users

   **10.3.3 Input Validation & Injection Prevention** 🟠 MAJOR

   - [ ] **Input Validation**:
     - All user inputs validated (API endpoints, forms, URL params)
     - Schema validation implemented (Zod, Joi, JSON Schema, class-validator)
     - Whitelist validation (reject unexpected fields)
     - Length limits enforced (prevent DoS)

   - [ ] **SQL Injection Prevention**:
     - All SQL queries parameterized (prepared statements, ORM)
     - NO string concatenation in SQL queries
     - Verify: `grep -r "SELECT.*+" src/` (should return nothing suspicious)

   - [ ] **XSS Prevention**:
     - HTML output escaped (template engines auto-escape by default)
     - Content Security Policy (CSP) headers set
     - User-generated content sanitized (DOMPurify, sanitize-html)

   - [ ] **Path Traversal Prevention**:
     - File paths sanitized (no `../` in user inputs)
     - File uploads validated (extension, MIME type, size)

   **10.3.4 OWASP Top 10 Validation** 🟠 MAJOR

   Comprehensive checklist from plan.md Security Review (Phase 3):

   - [ ] **A01: Broken Access Control**
     - Authorization checks on all protected resources
     - No horizontal privilege escalation (user A can't access user B's data)
     - No vertical privilege escalation (regular user can't access admin functions)

   - [ ] **A02: Cryptographic Failures**
     - Sensitive data encrypted at rest (PII, credentials)
     - Sensitive data encrypted in transit (HTTPS, TLS 1.2+)
     - No weak encryption (AES-256, RSA-2048+, NOT DES/RC4)

   - [ ] **A03: Injection**
     - SQL injection prevented (parameterized queries)
     - NoSQL injection prevented (sanitized queries)
     - Command injection prevented (no shell execution from user input)
     - LDAP/XML injection prevented (if applicable)

   - [ ] **A04: Insecure Design**
     - Threat modeling performed (from Risk Assessment in spec.md)
     - Rate limiting implemented (prevent brute force, DoS)
     - Business logic abuse prevented (cart manipulation, discount stacking)

   - [ ] **A05: Security Misconfiguration**
     - Default credentials changed
     - Unnecessary features disabled
     - Error messages don't leak sensitive info (stack traces hidden in production)
     - Security headers set (X-Frame-Options, X-Content-Type-Options, HSTS)

   - [ ] **A06: Vulnerable and Outdated Components**
     - Dependencies up-to-date (or documented exceptions)
     - Security audit passed (see 9.3.5 below)

   - [ ] **A07: Identification and Authentication Failures**
     - Weak password policy NOT allowed (minimum length, complexity)
     - Brute force protection (rate limiting, account lockout)
     - Session management secure (HttpOnly, Secure, SameSite cookies)
     - Multi-factor authentication (if required by spec.md)

   - [ ] **A08: Software and Data Integrity Failures**
     - Software updates signed/verified
     - CI/CD pipeline secured (no code injection possible)
     - Deserialization secure (no unsafe pickle/yaml.load)

   - [ ] **A09: Security Logging and Monitoring Failures**
     - Authentication events logged (login, logout, failed attempts)
     - Authorization failures logged (access denied)
     - Data access logged (sensitive operations, admin actions)
     - Logs stored securely (no PII in logs, access controlled)

   - [ ] **A10: Server-Side Request Forgery (SSRF)**
     - URL inputs validated (whitelist allowed domains)
     - Internal services not accessible from user input
     - Redirect validation (no open redirects)

   **10.3.5 Dependency Security Audits** 🟠 MAJOR

   - [ ] **Run Security Audits**:
     - Command: `npm audit --audit-level=high` (Node.js)
     - Command: `cargo audit` (Rust)
     - Command: `pip-audit` (Python)
     - Command: `go list -json -m all | nancy sleuth` (Go)

   - [ ] **Vulnerability Assessment**:
     - **HIGH/CRITICAL**: MUST fix or provide mitigation plan
     - **MEDIUM**: Document and create backlog ticket
     - **LOW**: Document for future fix

   **10.3.6 Risk Mitigation Validation** 🔴 CRITICAL

   For HIGH-risk features (Risk Score 8-12 from spec.md Risk Assessment):

   - [ ] **Risk Mitigation Controls Implemented**:
     - Verify all mitigation strategies from Risk Assessment table
     - Security controls from Security Review (plan.md Phase 3) present
     - Performance requirements met (if specified)

   - [ ] **Compliance Requirements Met** (if applicable):
     - **GDPR**: Right to access, right to delete, consent management
     - **CCPA**: Data disclosure, opt-out mechanisms
     - **PCI-DSS**: No card storage, tokenization, secure transmission
     - **HIPAA**: PHI encryption, access controls, audit logging
     - **SOC 2**: Access controls, logging, data protection

   **Security Reviewer Status** (determined after parallel execution):
   - ✅ **READY**: No secrets, auth implemented, OWASP mitigated, no HIGH/CRITICAL vulnerabilities, risk controls present
   - ⚠️ **NEEDS REVIEW**: LOW/MEDIUM vulnerabilities documented, partial OWASP coverage (non-critical categories)
   - ❌ **NOT READY**: Hardcoded secrets, missing auth, SQL injection risk, HIGH/CRITICAL vulnerabilities, missing risk controls

   ---

   #### 10.4 Constitutional Reviewer (Parallel Track 4) - NEW v2.4

   **Validation Tasks** (Run in parallel with other reviewers):

   **10.4.1 TDD Compliance Validation** 🔴 CRITICAL

   - [ ] **Test-First Evidence** (Article III: Test-First Development):
     - Check git history: Tests committed BEFORE implementation code?
       ```bash
       # For each feature file, check if corresponding test file was committed first
       git log --oneline --follow -- path/to/feature.ts
       git log --oneline --follow -- path/to/feature.test.ts
       # Verify test commits precede feature commits
       ```
     - Check task order in tasks.md: Test tasks completed before implementation tasks?
     - Verify Red-Green-Refactor cycle: Can git log timestamps show TDD flow?
     - **Evidence Required**: At least 3 examples showing test-first commits

   - [ ] **Test Coverage for Critical Paths**:
     - Verify critical business logic has test coverage (as defined in plan.md)
     - Check for missing tests on error handling paths
     - Validate edge cases have corresponding test cases

   **10.4.2 Simplicity Validation** 🟠 MAJOR

   - [ ] **Framework Count** (Article VII: Simplicity Gate):
     - Scan package.json / requirements.txt / Cargo.toml / go.mod for dependencies
     - Cross-check with plan.md Complexity Tracking section (if exists)
     - Count frameworks: Web framework, ORM, validation, state management, etc.
     - **Violation**: New frameworks added WITHOUT justification in plan.md
     - **Expected**: If constitution limits frameworks to N, verify count ≤ N

   - [ ] **Project Count** (Article VII: Simplicity Gate):
     - Check directory structure: Count services/projects in src/
     - Compare with plan.md Project Structure section
     - **Violation**: New projects/services created without planning approval
     - **Expected**: If constitution limits projects to N, verify count ≤ N

   **10.4.3 Anti-Abstraction Validation** 🟡 MEDIUM

   - [ ] **Custom Wrapper Detection** (Article VIII: Anti-Abstraction Gate):
     - Scan for abstraction patterns:
       ```bash
       # Search for common abstraction patterns
       grep -r "class.*Repository" src/
       grep -r "class.*Factory" src/
       grep -r "class.*Adapter" src/
       grep -r "interface.*Provider" src/
       grep -r "abstract class" src/
       ```
     - For each pattern found, verify justification in plan.md Complexity Tracking
     - **Violation**: Custom wrappers NOT documented in plan
     - **Allowed**: Abstractions with documented problem they solve

   - [ ] **Indirection Layers**:
     - Check for unnecessary middleware, decorators, proxies
     - Verify each layer serves documented purpose
     - Flag abstractions that don't solve concrete problems

   **10.4.4 Integration-First Validation** 🟠 MAJOR

   - [ ] **Contract Tests Present** (Article IX: Integration-First Gate):
     - Search for contract/integration test files:
       ```bash
       find . -name "*.integration.spec.ts" -o -name "*.contract.test.*"
       ```
     - For API features: Verify contracts/ directory populated
     - For database features: Verify integration tests use real DB (Docker/Testcontainers)
     - **Violation**: Integration-heavy feature with only unit tests
     - **Expected**: Integration tests exist for cross-boundary features

   - [ ] **Real Dependencies in Tests**:
     - Check integration tests use real services (not mocks):
       - Database: Docker containers, in-memory DBs, real connections
       - APIs: Real HTTP calls (test mode), contract tests
       - External services: Test/sandbox environments
     - **Violation**: Integration tests mocking database/auth/business logic
     - **Allowed**: Mocks for email/SMS with justification

   **10.4.5 Prohibited Pattern Detection** 🔴 CRITICAL

   - [ ] **Constitution-Specific Rules**:
     - Read `.specify/memory/constitution.md` (if exists)
     - Look for "Prohibited Patterns" section (added in v2.4)
     - For each prohibited pattern:
       ```markdown
       Example from constitution:
       - Pattern: `fs.readFileSync`, `fs.writeFileSync`
       - Reason: Blocks event loop
       - Detection: Regex scan for `fs\.(read|write)FileSync`
       ```
     - Scan codebase using provided detection patterns
     - **Violation**: Any prohibited pattern found without documented exception
     - **Report Format**: `file:line - Pattern: [pattern name] - Reason: [why prohibited]`

   - [ ] **Project-Specific Anti-Patterns**:
     - Common examples:
       - Direct database access (bypassing repository layer)
       - Synchronous blocking operations (in async contexts)
       - Global state mutations (in functional code)
       - Circular dependencies
     - Scan for patterns defined in project constitution
     - Flag violations with file location and remediation suggestion

   **⚠️ Graceful Degradation Matrix** (NEW - addresses Finding #3):

   How Constitutional Reviewer behaves under different conditions:

   | Condition | Behavior | Reviewer Status | Quality Gate Impact |
   |-----------|----------|----------------|---------------------|
   | **constitution.md present + Prohibited Patterns section present** | Full validation (all 6 checks) | ✅ READY / ⚠️ NEEDS REVIEW / ❌ NOT READY | Normal blocking power |
   | **constitution.md missing** | Skip pattern scanning (10.4.5), run TDD + simplicity + integration checks (10.4.1-4, 10.4.6) | ℹ️ PARTIAL (no constitution defined) | ⚠️ NEEDS REVIEW (reduced validation) |
   | **constitution.md present, Prohibited Patterns section missing** | Run TDD + simplicity + integration checks, skip pattern scanning | ⚠️ PARTIAL (patterns undefined) | ⚠️ NEEDS REVIEW (reduced validation) |
   | **Phase -1 missing in plan.md** | Pre-flight check (Step 2.5) prompts user, Step 10.4 runs with reduced confidence | ⚠️ LEGACY PLAN | User decision at Step 2.5 |
   | **Phase -1 present but no result (❌/⚠️/✅)** | Pre-flight check treats as incomplete | 🟠 INCOMPLETE PLANNING | Prompt user to complete Phase -1 |

   **When constitution.md is missing**:
   ```yaml
   ℹ️ INFO: "No constitution.md found - constitutional pattern scanning skipped"
   ℹ️ INFO: "Constitutional Reviewer running in PARTIAL mode:"
   ℹ️ INFO: "  ✅ TDD compliance validation (10.4.1)"
   ℹ️ INFO: "  ✅ Simplicity validation (10.4.2)"
   ℹ️ INFO: "  ✅ Anti-abstraction validation (10.4.3)"
   ℹ️ INFO: "  ✅ Integration-first validation (10.4.4)"
   ℹ️ INFO: "  ⚠️ Prohibited pattern detection skipped (10.4.5 - no patterns defined)"
   ℹ️ INFO: "  ✅ Technical debt validation (10.4.6)"

   Constitutional Reviewer Status: ℹ️ PARTIAL (constitution optional)
   Overall Quality Gate: ⚠️ NEEDS REVIEW (reduced constitutional validation)

   Recommendation: Consider creating constitution.md via /speckit.constitution
   ```

   **When Prohibited Patterns section is missing**:
   ```yaml
   ⚠️ WARN: "constitution.md found but no 'Prohibited Patterns' section"
   ⚠️ WARN: "Pattern scanning skipped - recommend adding Prohibited Patterns section"

   Constitutional Reviewer Status: ⚠️ PARTIAL (patterns undefined)

   Recommendation: Add "## Prohibited Patterns (Machine-Readable - v2.4)" section to constitution.md
   ```

   **Rationale**: Constitution is OPTIONAL for Spec-Kit users. Projects without constitution.md still get partial constitutional validation (TDD, simplicity, integration-first) but skip project-specific pattern enforcement.

   **10.4.6 Technical Debt & Complexity Drift** 🟡 MEDIUM

   - [ ] **Complexity Escalation**:
     - Compare current complexity with plan.md baseline
     - Check cyclomatic complexity of new functions (tools: eslint-plugin-complexity)
     - Flag functions exceeding project complexity thresholds
     - **Violation**: Functions with complexity > threshold without justification

   - [ ] **Technical Pivot Protocol** (Article XI, if applicable):
     - For HIGH complexity features: Verify TECHNICAL.md exists (if required by constitution)
     - Check TECHNICAL.md addresses architectural decisions
     - Validate implementation matches TECHNICAL.md approach
     - **Violation**: Complex feature without required TECHNICAL.md

   **Constitutional Reviewer Status** (determined after parallel execution):
   - ✅ **READY**: TDD followed, simplicity maintained, no abstractions without justification, integration tests present, no prohibited patterns
   - ⚠️ **NEEDS REVIEW**: Minor simplicity concerns (near limits), abstractions justified, conditional TDD compliance
   - ❌ **NOT READY**: TDD skipped (no test-first evidence), unjustified complexity, prohibited patterns found, missing integration tests

   ---

   ### Aggregated Quality Gate Report

   **Purpose**: Combine results from all 4 reviewers into a single comprehensive report.

   **Wait for ALL parallel reviewers to complete**, then aggregate:

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   📋 Quality Gate Summary (Parallel Validation)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   **Feature**: [FEATURE_NAME]
   **Date**: [DATE]
   **Execution Time**: [N]s (parallel) vs ~95s (sequential) → [X]% faster

   | Reviewer | Status | Critical Issues | Warnings | Details |
   |----------|--------|----------------|----------|---------|
   | Code Reviewer | [✅/⚠️/❌] | [N] errors | [M] warnings | Linting, type safety, spec alignment, docs |
   | Quality/Tests | [✅/⚠️/❌] | [N] failures | [M] warnings | Test execution, coverage, build readiness |
   | Security | [✅/⚠️/❌] | [N] vulns | [M] low-severity | Secrets, auth, OWASP, dependency audit |
   | Constitutional | [✅/⚠️/❌] | [N] violations | [M] concerns | TDD compliance, simplicity, prohibited patterns |

   **Overall Status**: [✅ READY / ⚠️ NEEDS REVIEW / ❌ NOT READY]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Code Reviewer Findings
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   | Validation Area | Status | Details |
   |----------------|--------|---------|
   | Linting | [🟢/🟡/🔴] | [N] errors, [N] warnings |
   | Formatting | [🟢/🔴] | All files consistent / [N] format violations |
   | Type Safety | [🟢/🔴] | Zero type errors / [N] type errors |
   | Dead Code | [🟢/🟡] | Clean / [N] unused imports |
   | Requirements Traceability | [🟢/🟡/🔴] | [N]/[N] requirements traced |
   | Acceptance Criteria | [🟢/🔴] | All met / [N] unmet |
   | Technical Plan | [🟢/🔴] | Matches plan / deviations found |
   | Code Documentation | [🟢/🟡] | Complete / [N] functions missing docs |
   | README/API Docs | [🟢/🟡] | Updated / needs sections |

   **Findings**:
   - ✅ [List passed checks - ONLY if exists]
   - ⚠️  [List warnings - ONLY if exists]
   - 🔴 [List failures - ONLY if exists]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Quality/Tests Reviewer Findings
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   | Validation Area | Status | Details |
   |----------------|--------|---------|
   | Test Execution | [🟢/🔴] | [N] tests passed, [N] failed |
   | Unit Tests | [🟢/🔴] | [N]/[N] passed |
   | Integration Tests | [🟢/🔴] | [N]/[N] passed |
   | E2E Tests | [🟢/🔴/N/A] | [N]/[N] passed |
   | Coverage - Critical | [🟢/🟡/🔴] | [N]% (target ≥90%) |
   | Coverage - Business | [🟢/🟡/🔴] | [N]% (target ≥80%) |
   | Coverage - Overall | [🟢/🟡/🔴] | [N]% (target ≥75%) |
   | High-Risk Testing | [🟢/🟡/🔴] | [N]/[N] requirements tested |
   | Edge Cases | [🟢/🟡] | Complete / [N] missing |
   | Production Resilience | [🟢/🟡] | Tested / partial |
   | Build Success | [🟢/🔴] | Success / [N] errors |
   | Docker Build | [🟢/🔴/N/A] | Success / failed |
   | Env Configuration | [🟢/🟡] | Complete / needs update |

   **Test Summary**:
   - Total Tests: [N]
   - Passed: [N]
   - Failed: [N]
   - Skipped: [N] (with justification)

   **Coverage Breakdown**:
   - Critical Paths: [N]% ([N]/[N] lines)
   - Business Logic: [N]% ([N]/[N] lines)
   - Overall: [N]% ([N]/[N] lines)

   **High-Risk Requirements** (Risk Score ≥ 8):
   | Requirement | Risk Score | Tests | Coverage |
   |-------------|------------|-------|----------|
   | [FR-XXX] | [N] | [N] tests | [N]% |

   **Findings**:
   - ✅ [List passed checks - ONLY if exists]
   - ⚠️  [List warnings - ONLY if exists]
   - 🔴 [List failures - ONLY if exists]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Security Reviewer Findings
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   | Validation Area | Status | Details |
   |----------------|--------|---------|
   | Secrets Scanning | [🟢/🔴] | No secrets / [N] secrets found |
   | Authentication | [🟢/🔴] | Implemented / missing |
   | Authorization | [🟢/🔴] | RBAC enforced / gaps found |
   | Input Validation | [🟢/🟡/🔴] | Complete / partial / missing |
   | SQL Injection Prevention | [🟢/🔴] | Parameterized queries / vulnerable |
   | XSS Prevention | [🟢/🟡/🔴] | Escaped output / vulnerable |
   | OWASP A01 (Access Control) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | OWASP A02 (Cryptography) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | OWASP A03 (Injection) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | OWASP A04 (Insecure Design) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | OWASP A05 (Misconfiguration) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | OWASP A06 (Vulnerable Components) | [🟢/🟡/🔴] | Audited / vulnerabilities found |
   | OWASP A07 (Auth Failures) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | OWASP A08 (Integrity Failures) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | OWASP A09 (Logging Failures) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | OWASP A10 (SSRF) | [🟢/🟡/🔴] | Mitigated / partial / vulnerable |
   | Dependency Audit | [🟢/🟡/🔴] | Clean / [N] vulnerabilities |
   | Risk Mitigation | [🟢/🔴] | Controls implemented / missing |
   | Compliance | [🟢/🟡/N/A] | Requirements met / partial |

   **OWASP Top 10 Summary**:
   - Mitigated: [N]/10 categories
   - Partial: [N]/10 categories
   - Vulnerable: [N]/10 categories

   **Dependency Vulnerabilities**:
   - HIGH/CRITICAL: [N]
   - MEDIUM: [N]
   - LOW: [N]

   **High-Risk Requirements** (Risk Score ≥ 8):
   | Requirement | Risk Score | Mitigation Status |
   |-------------|------------|-------------------|
   | [FR-XXX] | [N] | [✅ Implemented / ⚠️ Partial / 🔴 Missing] |

   **Findings**:
   - ✅ [List passed checks - ONLY if exists]
   - ⚠️  [List warnings - ONLY if exists]
   - 🔴 [List failures - ONLY if exists]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Constitutional Reviewer Findings (v2.4)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   | Validation Area | Status | Details |
   |----------------|--------|---------|
   | TDD Evidence | [🟢/🟡/🔴] | [N] test-first commits verified / missing |
   | Test Coverage (Critical) | [🟢/🟡/🔴] | [N]% critical paths tested |
   | Framework Count | [🟢/🟡/🔴] | [N]/[M] allowed (within limits / near limit / exceeded) |
   | Project Count | [🟢/🟡/🔴] | [N]/[M] allowed |
   | Custom Abstractions | [🟢/🟡/🔴] | [N] found, [M] justified |
   | Integration Tests | [🟢/🟡/🔴] | Present / partial / missing |
   | Real Dependencies | [🟢/🟡/🔴] | Used / some mocks / excessive mocking |
   | Prohibited Patterns | [🟢/🔴] | None found / [N] violations |
   | Complexity Drift | [🟢/🟡/🔴] | Within thresholds / near limit / exceeded |
   | Technical Debt | [🟢/🟡] | Controlled / [N] concerns |

   **TDD Compliance Evidence**:
   - Test-First Commits: [N]/[M] features (target: all features)
   - Examples:
     - ✅ `feature.test.ts` committed at [commit-hash] before `feature.ts` at [commit-hash]
     - ✅ tasks.md shows test tasks completed before implementation tasks
     - 🔴 `another-feature.ts` committed WITHOUT test-first evidence

   **Simplicity Metrics**:
   - Frameworks: [N]/[M] allowed (Constitution limit: [M])
     - [List frameworks: Express, Prisma, Zod, etc.]
   - Projects: [N]/[M] allowed
     - [List projects: api-service, worker-service]
   - Complexity Status: [Within limits / Near threshold / Exceeded]

   **Anti-Abstraction Validation**:
   - Custom Wrappers Found: [N]
     - ✅ [Pattern] - Justified in plan.md: [reason]
     - ⚠️  [Pattern] - Justification weak/missing
     - 🔴 [Pattern] - Not documented, remove or justify

   **Integration-First Validation**:
   - Contract/Integration Tests: [N] files found
     - Files: [list .integration.spec.ts, .contract.test.ts files]
   - Real Dependencies:
     - ✅ Database: Docker Postgres, real connections
     - ✅ APIs: Real HTTP calls in test mode
     - ⚠️  Email: Mocked (justification: external service)
     - 🔴 Auth: Mocked (VIOLATION - should use real auth in integration tests)

   **Prohibited Patterns Scan**:
   - Constitution Patterns Checked: [N] patterns
     - ✅ No `fs.readFileSync` (synchronous blocking)
     - ✅ No direct `db.query()` calls (bypasses repository)
     - 🔴 Found: `src/utils/sync-reader.ts:42` - `fs.readFileSync` (CRITICAL violation)

   **Technical Pivot Protocol** (if HIGH complexity):
   - TECHNICAL.md Required: [Yes/No]
   - TECHNICAL.md Present: [Yes/No]
   - Status: [✅ Complete / 🔴 Missing]

   **Findings**:
   - ✅ [List passed checks - ONLY if exists]
   - ⚠️  [List warnings/concerns - ONLY if exists]
   - 🔴 [List violations - ONLY if exists]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Overall Decision
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   **Aggregation Logic Applied**:

   IF Code = ❌ NOT READY OR Quality/Tests = ❌ NOT READY OR Security = ❌ NOT READY OR Constitutional = ❌ NOT READY:
     → Overall: ❌ NOT READY
     → Message: "Critical issues found. MUST fix before proceeding."
     → Action: STOP workflow, present all failures to user

   ELSE IF Code = ⚠️ NEEDS REVIEW OR Quality/Tests = ⚠️ NEEDS REVIEW OR Security = ⚠️ NEEDS REVIEW OR Constitutional = ⚠️ NEEDS REVIEW:
     → Overall: ⚠️ NEEDS REVIEW
     → Message: "Minor issues present. Review warnings and decide to proceed or fix."
     → Action: Ask user: "Proceed with warnings or fix issues first?"

   ELSE:
     → Overall: ✅ READY
     → Message: "All quality gates passed. Ready for final validation."
     → Action: Proceed to Step 11 (Final Validation & Completion)

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Next Action
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [IF ✅ READY:]
   ✅ All quality gates passed. Proceeding to Final Validation (Step 11).

   [IF ⚠️ NEEDS REVIEW:]
   ⚠️  Minor issues present. Options:
   1. Fix warnings now (recommended for critical features)
   2. Document warnings and proceed
   3. Create backlog tickets for warnings

   User, do you want to proceed or fix warnings first?

   [IF ❌ NOT READY:]
   ❌ CRITICAL ISSUES FOUND - Implementation NOT ready for commit.

   **Required Actions**:
   [List all blocking failures from all 4 reviewers]

   Fix these critical issues, then re-run quality gate validation.

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ```

   ---

   ### Overall Status Determination

   **Aggregation Logic** (applied after all parallel reviewers complete):

   ```yaml
   COLLECT reviewer statuses:
     code_status = [Code Reviewer Status: ✅/⚠️/❌]
     quality_status = [Quality/Tests Reviewer Status: ✅/⚠️/❌]
     security_status = [Security Reviewer Status: ✅/⚠️/❌]

   DETERMINE overall status:
     IF code_status = ❌ NOT READY OR quality_status = ❌ NOT READY OR security_status = ❌ NOT READY:
       → Overall: ❌ NOT READY
       → Message: "Critical issues found in [failing reviewers]. MUST fix before proceeding."
       → Action: STOP workflow, present aggregated report with all failures

     ELSE IF code_status = ⚠️ NEEDS REVIEW OR quality_status = ⚠️ NEEDS REVIEW OR security_status = ⚠️ NEEDS REVIEW:
       → Overall: ⚠️ NEEDS REVIEW
       → Message: "Minor issues present. Review warnings and decide to proceed or fix."
       → Action: Ask user: "Proceed with warnings or fix issues first?"

     ELSE:
       → Overall: ✅ READY
       → Message: "All quality gates passed. Ready for final validation."
       → Action: Proceed to Step 10 (Final Validation & Completion)

   PRESENT aggregated quality gate report to user with overall status.
   ```

   ---

   ### Blocking Criteria (Aggregated from All Reviewers)

   **CANNOT proceed to Step 10 if ANY of the following**:

   **Code Reviewer Blockers**:
   - 🔴 Linting/type errors present
   - 🔴 P1 User Stories not implemented
   - 🔴 Acceptance criteria unmet

   **Quality/Tests Reviewer Blockers**:
   - 🔴 ANY test failures
   - 🔴 Coverage <70% overall
   - 🔴 Critical paths <90% coverage
   - 🔴 High-risk requirements (Score ≥8) without tests
   - 🔴 Build fails

   **Security Reviewer Blockers**:
   - 🔴 Hardcoded secrets detected
   - 🔴 Missing authentication on protected endpoints
   - 🔴 SQL injection / XSS vulnerabilities
   - 🔴 HIGH/CRITICAL dependency vulnerabilities
   - 🔴 High-risk requirements (Score ≥8) missing mitigation controls
   - 🔴 CRITICAL OWASP categories (A01, A02, A03, A07, A08) vulnerable

   ---

   ### Warning Criteria (Can proceed with documentation)

   **Quality gates can proceed with ⚠️ NEEDS REVIEW if ONLY warnings**:

   **Code Reviewer Warnings**:
   - 🟡 Linting warnings (5-20, documented)
   - 🟡 Minor spec gaps (<10% requirements untraceable)
   - 🟡 Incomplete documentation (non-blocking)

   **Quality/Tests Reviewer Warnings**:
   - 🟡 Coverage 70-75% overall (but critical paths ≥90%)
   - 🟡 Minor edge cases missing tests
   - 🟡 Non-blocking build warnings

   **Security Reviewer Warnings**:
   - 🟡 LOW/MEDIUM dependency vulnerabilities (documented)
   - 🟡 Partial OWASP coverage (non-critical categories like A09 Logging)
   - 🟡 Compliance requirements partially met (document gaps)

   ---

   ### Benefits of Parallel Execution

   1. **Performance**: ~53% faster execution (95s → 45s for typical projects)
   2. **Better User Experience**: All failures shown at once (not iterative "fix → rerun → fail again")
   3. **Maintains Safety**: Still blocks if ANY reviewer fails (✅ preserves sequential safety guarantees)
   4. **Comprehensive Feedback**: All validation dimensions checked before any blocking
   5. **Time Savings**: Linting (30s), testing (45s), security scans (20s) run simultaneously

   **Example Performance Comparison**:
   - Sequential: 30s (lint) → 45s (test) → 20s (security) = **95s total**
   - Parallel: max(30s, 45s, 20s) = **45s total** → **53% faster**

   ---

11. **Final Validation & Completion**:
   - If Quality Gate status is ❌ NOT READY: HALT and require fixes before proceeding
   - If Quality Gate status is ⚠️ NEEDS REVIEW: Create backlog tickets, allow commit
   - If Quality Gate status is ✅ READY: Proceed to completion
   - Verify all required tasks are marked [X] in tasks.md
   - Report final status with summary of completed work
   - Suggest running `/speckit.reconcile` for post-implementation gap closure

12. **Implementation Code Review Gate** (Evidence-Based Self-Check):

   **Purpose**: Prevent hallucination and ensure evidence-based completion claims. This gate runs AFTER Step 10 (Parallel Quality Gate Validation) has completed.

   **MANDATORY: The Four Questions** (MUST answer with ACTUAL evidence):

   ❓ **"Did all 4 reviewers execute in parallel?"**
      ```yaml
      Action Required:
        - Verify Code, Quality/Tests, Security, and Constitutional reviewers all ran concurrently
        - Show ACTUAL status for each reviewer from parallel execution
        - Report: Aggregated Quality Gate Summary presented

      Expected Evidence:
        Quality Gate Summary:
        ✓ Code Reviewer: [✅/⚠️/❌] - [N] errors, [M] warnings
        ✓ Quality/Tests: [✅/⚠️/❌] - [N] failures, [M] warnings
        ✓ Security: [✅/⚠️/❌] - [N] vulnerabilities, [M] low-severity
        Overall Status: [✅ READY / ⚠️ NEEDS REVIEW / ❌ NOT READY]
        Execution Time: [N]s (parallel) vs ~95s (sequential) → [X]% faster

      Hallucination Detection:
        🚨 "All reviewers passed" WITHOUT showing aggregated report → ❌ BLOCK completion
        🚨 Claiming parallel execution without timing evidence → ❌ BLOCK completion
        🚨 Hiding failures from any reviewer → ❌ BLOCK completion
      ```

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

---

## Final Step: Update Specification Metadata

After completing implementation and passing quality gate, update `spec-metadata.json`:

```json
{
  "phase": "implementation",
  "approvals": {
    "implementation": {
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
- IF quality gate passed: "✅ Implementation complete. Run `/speckit.reconcile` to verify all requirements met"
- IF quality gate had issues: "⚠️ Address [N] issues before marking complete"
- "Or run `/speckit.status` to check current workflow state"

---

Note: This command assumes a complete task breakdown exists in tasks.md. If tasks are incomplete or missing, suggest running `/speckit.tasks` first to regenerate the task list.

