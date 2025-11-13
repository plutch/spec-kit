---
description: Lightweight implementation workflow for small, low-risk tasks with full constitutional enforcement and quality gates
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

You are implementing a SMALL, LOW-RISK task using Spec-Kit's lightweight workflow.

**⚠️ IMPORTANT: When to Use This Command**

✅ **Use /speckit.quick when**:
- Simple, isolated feature (form, component, bug fix)
- Clear requirements (no ambiguity)
- LOW risk (no sensitive data, simple logic)
- < 2 days effort
- Uses existing patterns (no architectural changes)

❌ **Use FULL workflow (/speckit.specify) instead when**:
- MEDIUM or HIGH risk (score ≥4)
- Complex logic or many edge cases
- Architectural changes needed (new ADRs)
- Ambiguous requirements (needs clarification)
- Multi-system integration
- 3+ days effort

---

## Workflow

### Phase 0: Context Loading (v2.7 Optimization)

**Purpose**: Load strategic context only (5-8KB budget) for quick task evaluation.

**Current Phase**: `quick_implementation`

**Context Loading Strategy**:

1. **Read Configuration** (if exists):
   - Check for `.specify/config.yml` or `.specify/config.example.yml`
   - Extract `context_budget.implementation` (default: 50KB, use 15KB for quick)
   - Extract `memory.strict_phase_loading` (default: true)

2. **Load Memory Files** (strategic context only):

   For each memory file in `.specify/memory/`:

   a. **Read YAML Frontmatter**:
      - Extract `inclusion_mode`, `context_level`, `load_phases`, `exclude_phases`
      - If metadata missing: Default to `inclusion_mode: always, context_level: strategic`

   b. **Determine if file should be loaded**:
      ```
      IF inclusion_mode = "always" (e.g., constitution.md):
        → Load STRATEGIC sections only (Core Principles, not Implementation Patterns)
        → Skip tactical sections (not needed for quick pre-flight)

      ELSE IF inclusion_mode = "conditional":
        IF "implementation" IN load_phases:
          → Load STRATEGIC sections only
        ELSE:
          → SKIP this file

      ELSE IF inclusion_mode = "manual":
        → SKIP (manual loading via @filename only)
      ```

   c. **Filter Sections** (if loading file):
      - Scan for `<!-- SECTION_META: context_level=X -->` comments
      - For quick workflow: Load ONLY sections where `context_level=strategic`
      - Skip tactical/reference sections (code examples not needed yet)

   d. **Track Budget**:
      - Target: 5-8KB (minimal strategic context)
      - If exceeds budget: Truncate lowest priority sections first

3. **Backward Compatibility**:
   - If `.specify/memory/` doesn't exist → Skip context loading (no error)
   - If memory files lack metadata → Load constitution.md only (prioritize core principles)
   - Never fail command due to missing memory files

---

### Phase 1: Quick Requirements Capture

**Prompt user** for task details:

```
Please provide the following information about this task:

1. **What** are you building? (1-2 sentences)
2. **Why** is this needed? (user value or bug impact)
3. **Acceptance Criteria** (bullet list of success conditions)
4. **Files/Components** affected (if known)

Example:
  What: User profile edit form with name, email, bio, avatar upload
  Why: Allow users to update their profile information
  Acceptance Criteria:
    - Form fields: name (text, required), email (email, required), bio (textarea), avatar (file upload)
    - Client-side validation (email format, required fields, file size < 5MB)
    - Save button disabled until form valid
    - Success toast on save, error toast on failure
    - Follows existing FormLayout component pattern
  Files: src/components/UserProfile/EditForm.tsx (new)
```

**User's response:**
[Wait for user to provide task details]

---

**Create task document** at `.specify/quick-tasks/quick-task-[timestamp].md`:

Use template from `.specify/templates/quick-task-template.md` if exists, otherwise create:

```markdown
# Quick Task: [Title from user's "What"]

**Status**: In Progress
**Created**: [ISO 8601 timestamp]
**Risk**: LOW (assumed for quick workflow)

## Description
[User's "What" response]

## User Value
[User's "Why" response]

## Acceptance Criteria
[User's criteria list - format as checklist]
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Implementation Scope
- **Files**: [List from user or "TBD"]
- **Estimated Effort**: < 2 days
- **Risk Level**: 🟢 LOW

---

## Status Log

### [Timestamp] - Task Created
Quick workflow initiated via `/speckit.quick`
```

**Output to user**:
```
✅ Task captured: [Title]

📋 Quick task document: .specify/quick-tasks/quick-task-[timestamp].md

Next: Constitutional pre-flight check...
```

---

### Phase 2: Constitutional Pre-Flight Check

**Purpose**: Validate task aligns with constitutional principles BEFORE implementation begins.

**Load constitution.md** (if exists):
- Read `.specify/memory/constitution.md`
- Extract relevant articles (Article I-X if present)
- Extract Prohibited Patterns section (if present)

**Validation Checklist**:

1. **Article I: Test-First Philosophy** (if applicable):
   - Does this task involve testable logic?
     - ✅ YES: TDD required (RED-GREEN-REFACTOR)
     - ⚠️ NO: Acceptable (e.g., pure UI styling, static content)
   - Document decision in task file

2. **Article V: Integration-First Testing** (if applicable):
   - Does task mention mocking auth/database/multi-tenancy?
     - ❌ YES: Violation - prohibited patterns
     - ✅ NO: Compliant
   - Check Prohibited Patterns section

3. **Simplicity & Anti-Abstraction**:
   - Does approach mention custom wrappers/frameworks?
     - ❌ YES: Requires justification
     - ✅ NO: Compliant
   - Verify usage of existing patterns/libraries

4. **Prohibited Pattern Scan**:
   - Scan task description for prohibited patterns from constitution
   - Common patterns to check:
     - `jest.mock('auth0')` or `mockAuth`
     - `MockDatabase()` or `jest.mock('db')`
     - `mockTenantContext()` or mocked multi-tenancy
     - `fs.readFileSync()` in production code
     - Custom ORM wrappers without justification

**Gate Logic**:

```
IF constitution.md NOT exists:
  ⚠️ WARN: "No constitution found. Proceeding with standard practices (TDD, simplicity, integration-first testing)."
  Status: ✅ PASS (default to best practices)

IF violations detected:
  ❌ FAIL: "Task violates constitution Article [X]: [Specific violation]"
  Example: "Task mentions mocking Auth0 (Article V violation: Integration-First Testing)"
  Action: "Please revise approach to use real Auth0 test tenant or Docker-based auth mock"
  → STOP workflow, ask user to revise

IF conditional concerns (e.g., new dependency, edge case):
  ⚠️ CONDITIONAL: "Potential constitutional concern: [Issue]"
  Example: "Task adds new npm package 'custom-form-lib' - violates simplicity unless justified"
  Action: "Confirm this approach is acceptable? (yes/no)"
  → Wait for user response
  → IF no: STOP workflow
  → IF yes: Continue with documented exception

ELSE:
  ✅ PASS: "Constitutional pre-flight complete"
  Status: All articles compliant
```

**Update task document**:

```markdown
---

## Constitutional Pre-Flight

**Status**: [✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL]
**Validated**: [Timestamp]

### Validation Results

| Article | Requirement | Status | Notes |
|---------|-------------|--------|-------|
| Article I (TDD) | Test-first approach | ✅ PASS | Testable logic - TDD required |
| Article V (Integration-First) | Real dependencies | ✅ PASS | No mocking of auth/db/tenancy |
| Simplicity | Use existing patterns | ✅ PASS | Uses FormLayout component |
| Prohibited Patterns | Pattern scan | ✅ PASS | No violations detected |

**Overall**: ✅ PASS - Approved to proceed

**Approval**: Pre-flight complete at [Timestamp]
```

**Output to user**:

```
✅ Constitutional Pre-Flight: PASS

All constitutional gates satisfied:
  ✅ TDD applicable (testable logic)
  ✅ No integration-first violations
  ✅ Uses existing patterns (no custom abstractions)
  ✅ No prohibited patterns detected

Proceeding to implementation...
```

**If FAIL or user declines CONDITIONAL**:
```
❌ Constitutional Pre-Flight: FAIL

Violations detected:
  ❌ [Article]: [Specific violation]
  ❌ [Article]: [Specific violation]

Recommendations:
  1. [How to fix violation 1]
  2. [How to fix violation 2]

Please revise task approach and run /speckit.quick again.
```

---

### Phase 3: TDD Implementation

**Load tactical context** (if not already loaded):
- Read constitution.md tactical sections (Implementation Patterns, TDD examples)
- Read testing-approach.md (if exists and applicable)
- Budget: Additional 20-30KB for implementation guidance

**Implementation Instructions**:

```
Follow RED-GREEN-REFACTOR for all testable logic:

### TDD Cycle

1. **RED Phase: Write Failing Tests First**

   Requirements:
   - Write tests BEFORE implementation code
   - Tests must fail initially (RED)
   - Cover all acceptance criteria
   - Use existing test patterns from codebase

   Test Types:
   - **Unit Tests**: Business logic, validation, calculations
   - **Integration Tests**: Component behavior, API calls (use real dependencies per Article V)
   - **E2E Tests**: User workflows (if applicable)

   Commit message example:
   ```
   RED: Add failing test for user profile form validation

   - Test email format validation
   - Test required fields (name, email)
   - Test file upload size limit (5MB)
   ```

2. **GREEN Phase: Implement Minimal Code to Pass Tests**

   Requirements:
   - Write ONLY enough code to make tests pass
   - Follow existing architectural patterns
   - Use framework/library features (no reinventing)
   - Keep it simple (Article I)

   Patterns to Follow:
   - **UI Components**: Use existing component library (MUI, Chakra, etc.)
   - **Forms**: Reuse existing form components (FormLayout, Button, Input)
   - **Validation**: Use existing validation library (Yup, Zod, joi)
   - **API Calls**: Use existing API client/hooks
   - **State Management**: Use existing state patterns (Redux, Context, Zustand)

   Commit message example:
   ```
   GREEN: Implement user profile form with validation

   - Add EditForm component using FormLayout
   - Integrate email validation with Yup schema
   - Add file upload with size validation
   - Wire up save handler with API hook
   ```

3. **REFACTOR Phase: Clean Up Implementation**

   Requirements:
   - Extract reusable logic (if >2 duplications)
   - Add inline comments for complex logic
   - Ensure type safety (TypeScript strict mode)
   - Remove duplication
   - Improve naming clarity

   Commit message example:
   ```
   REFACTOR: Extract form validation schema and file helpers

   - Move Yup schema to shared validations
   - Extract file size validation to utils
   - Add JSDoc comments for validation rules
   ```

### Component-Specific Guidance

**For UI Components**:
- Follow existing component library conventions (MUI, Chakra, etc.)
- Reuse existing components (Button, FormField, Layout, Modal, Toast)
- Add accessibility attributes (aria-label, role, aria-describedby)
- Responsive design (mobile-first if required)
- Error states and loading states
- Keyboard navigation support

**For API Changes**:
- Follow existing API conventions (REST, GraphQL, tRPC)
- Input validation using schema library
- Error handling (try-catch, error responses)
- Type safety (TypeScript interfaces)
- Authentication/authorization checks

**For Bug Fixes**:
- Add regression test first (RED)
- Fix bug (GREEN)
- Refactor if needed (REFACTOR)

### Documentation (Minimal)

Requirements:
- Inline comments for "why" (not "what")
- JSDoc/TSDoc for public APIs (if exposed)
- Update README only if public API changed
- No separate documentation files for quick tasks

### Execution

Implement following TDD cycle:
1. Write tests (RED)
2. Run tests → Verify failures
3. Implement code (GREEN)
4. Run tests → Verify passes
5. Refactor (REFACTOR)
6. Run tests → Verify still passes
7. Commit each phase separately

Ensure ALL acceptance criteria are met.
```

**During Implementation**:
- Execute code changes following TDD cycle
- Create/modify files as needed
- Run tests frequently
- Commit after each TDD phase (RED, GREEN, REFACTOR)

**Update task document** after implementation:

```markdown
---

## Implementation Summary

**Status**: Implementation complete, awaiting quality gate
**Completed**: [Timestamp]

### Files Changed
- [File 1]: [Brief description]
- [File 2]: [Brief description]
- [File 3]: [Brief description]

### Tests Added
- [Test file 1]: [Coverage description]
- [Test file 2]: [Coverage description]

### Commits
- [SHA]: RED - [Description]
- [SHA]: GREEN - [Description]
- [SHA]: REFACTOR - [Description]

### Acceptance Criteria Status
- [x] [Criterion 1] - Implemented in [file]
- [x] [Criterion 2] - Implemented in [file]
- [x] [Criterion 3] - Implemented in [file]
```

---

### Phase 4: Quality Gate (Streamlined)

**Purpose**: Validate implementation quality across 4 critical dimensions before commit.

Run four sequential reviewers. Each reviewer outputs status: ✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY

---

#### 4.1 Code Reviewer

**Automated Checks**:

```bash
# Run linting (detect project type and run appropriate linter)
npm run lint || yarn lint || pnpm lint || \
eslint . || \
pylint **/*.py || \
cargo clippy || \
dotnet format --verify-no-changes || \
go fmt -l . || \
rubocop || \
phpcs

# Check type safety (if applicable)
npm run type-check || tsc --noEmit || \
mypy . || \
cargo check || \
dotnet build --no-restore

# Review staged changes
git diff --cached
```

**Manual Checklist**:
- ✅ No linting errors
- ✅ No type errors
- ✅ Follows existing code style (indentation, naming, structure)
- ✅ All acceptance criteria met (reference task document)
- ✅ Inline comments present for complex logic
- ✅ No commented-out code (remove or justify)
- ✅ No debug statements (console.log, print, debugger)

**Status Decision**:
```
IF any automated check fails:
  → ❌ NOT READY

ELSE IF acceptance criteria not met:
  → ❌ NOT READY

ELSE IF code style issues or missing comments:
  → ⚠️ NEEDS REVIEW

ELSE:
  → ✅ READY
```

**Output**:
```markdown
### Code Reviewer: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**Linting**: [✅ PASS | ❌ FAIL - X errors]
**Type Safety**: [✅ PASS | ❌ FAIL - X errors]
**Code Style**: [✅ PASS | ⚠️ MINOR ISSUES]
**Acceptance Criteria**: [✅ ALL MET | ❌ X/Y met]
**Documentation**: [✅ ADEQUATE | ⚠️ MINIMAL]

**Issues**:
- [Issue 1 if any]
- [Issue 2 if any]

**Status**: [✅/⚠️/❌]
```

---

#### 4.2 Quality/Tests Reviewer

**Automated Checks**:

```bash
# Run all tests
npm test || yarn test || pnpm test || \
pytest || \
cargo test || \
dotnet test || \
go test ./... || \
bundle exec rspec || \
phpunit

# Check coverage (if configured)
npm run test:coverage || \
pytest --cov || \
cargo tarpaulin || \
dotnet test /p:CollectCoverage=true
```

**Manual Checklist**:
- ✅ All tests pass (0 failures)
- ✅ New tests added for new logic
- ✅ Coverage adequate for business logic (≥75% lines)
- ✅ Tests are readable (clear arrange-act-assert)
- ✅ Tests are maintainable (no brittle assertions)
- ✅ No skipped/disabled tests without justification
- ✅ Integration tests use real dependencies (per Article V)

**Status Decision**:
```
IF any test fails:
  → ❌ NOT READY

ELSE IF no tests added for new logic:
  → ❌ NOT READY

ELSE IF coverage < 75% for business logic:
  → ⚠️ NEEDS REVIEW (may be acceptable for simple UI)

ELSE:
  → ✅ READY
```

**Output**:
```markdown
### Quality/Tests Reviewer: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**Test Execution**: [✅ PASS (X/X) | ❌ FAIL (X failures)]
**New Tests Added**: [✅ YES (X tests) | ❌ NO]
**Coverage**: [✅ XX% (≥75%) | ⚠️ XX% (<75%)]
**Test Quality**: [✅ READABLE & MAINTAINABLE | ⚠️ ISSUES]

**Test Summary**:
- Unit tests: X passed
- Integration tests: X passed
- E2E tests: X passed (if applicable)

**Issues**:
- [Issue 1 if any]

**Status**: [✅/⚠️/❌]
```

---

#### 4.3 Security Reviewer

**Automated Checks**:

```bash
# Check for hardcoded secrets in changes
git diff --cached | grep -i -E '(password|secret|token|api[_-]?key|private[_-]?key|access[_-]?key|auth[_-]?token).*=.*["\047]'

# Check for security anti-patterns
git diff --cached | grep -i -E '(eval\(|innerHTML|dangerouslySetInnerHTML|exec\(|system\(|shell_exec)'

# Check for SQL injection risks (if database code)
git diff --cached | grep -i -E '(db\.query\(|execute\(|exec\().*\+|`\$\{.*\}`'

# Dependency audit (if package changes)
npm audit || yarn audit || pnpm audit || \
pip-audit || \
cargo audit || \
dotnet list package --vulnerable
```

**Manual Checklist**:
- ✅ No hardcoded secrets (API keys, passwords, tokens)
- ✅ No sensitive data in logs/errors
- ✅ Input validation present (if user input accepted)
- ✅ No SQL injection risk (use parameterized queries/ORM)
- ✅ No XSS risk (sanitize HTML, use framework escaping)
- ✅ Authentication checks present (if protected resource)
- ✅ Authorization checks present (if role-based access)
- ✅ No vulnerable dependencies (if package.json/requirements.txt changed)

**Status Decision**:
```
IF secrets detected OR SQL injection risk OR XSS risk:
  → ❌ NOT READY

ELSE IF missing auth/authz checks (when applicable):
  → ❌ NOT READY

ELSE IF vulnerable dependencies:
  → ⚠️ NEEDS REVIEW (may have acceptable risk)

ELSE IF input validation missing (when applicable):
  → ⚠️ NEEDS REVIEW

ELSE:
  → ✅ READY
```

**Output**:
```markdown
### Security Reviewer: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**Secrets Scan**: [✅ NONE DETECTED | ❌ X found]
**Input Validation**: [✅ PRESENT | ⚠️ MISSING | N/A]
**SQL Injection**: [✅ SAFE | ❌ RISK DETECTED | N/A]
**XSS Protection**: [✅ SAFE | ❌ RISK DETECTED | N/A]
**Authentication**: [✅ PRESENT | ⚠️ MISSING | N/A]
**Authorization**: [✅ PRESENT | ⚠️ MISSING | N/A]
**Dependency Audit**: [✅ NO VULNERABILITIES | ⚠️ X vulnerabilities | N/A]

**Issues**:
- [Issue 1 if any]

**Status**: [✅/⚠️/❌]
```

---

#### 4.4 Constitutional Reviewer

**Purpose**: Validate implementation adheres to constitutional principles via evidence-based checks.

**Load constitution** (if not already loaded):
- Read `.specify/memory/constitution.md`
- Focus on Prohibited Patterns section

---

**10.4.1: TDD Compliance Validation** (if testable logic)

**Automated Check**:
```bash
# Check git history for RED-GREEN-REFACTOR pattern
git log --oneline --reverse --since="1 day ago" | head -20
```

**Expected Pattern**:
```
abc1234 RED: Add failing test for [feature]
def5678 GREEN: Implement [feature] to pass test
ghi9012 REFACTOR: Clean up [implementation]
```

**Validation**:
- ✅ PASS: Git history shows RED → GREEN → REFACTOR commits
- ⚠️ PARTIAL: Tests added but not in TDD order (acceptable for bug fixes)
- ❌ FAIL: No test commits, or implementation before tests

**Output**:
```markdown
**10.4.1 TDD Compliance**: [✅ PASS | ⚠️ PARTIAL | ❌ FAIL]

Git history analysis:
- [Commit 1]: RED - [Message]
- [Commit 2]: GREEN - [Message]
- [Commit 3]: REFACTOR - [Message]

Status: [✅ TDD pattern followed | ❌ TDD not followed]
```

---

**10.4.2: Simplicity Validation**

**Automated Check**:
```bash
# Check for new dependencies
git diff --cached package.json requirements.txt Cargo.toml *.csproj go.mod Gemfile composer.json 2>/dev/null

# Check project file count
find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.jsx" -o -name "*.tsx" -o -name "*.py" -o -name "*.rs" -o -name "*.go" \) | wc -l
```

**Validation**:
- ✅ PASS: No new dependencies, uses existing frameworks
- ⚠️ REVIEW: New dependency added (ask user if justified)
- ❌ FAIL: Multiple new frameworks added without justification

**Output**:
```markdown
**10.4.2 Simplicity**: [✅ PASS | ⚠️ REVIEW | ❌ FAIL]

Dependencies:
- New dependencies: [List or "None"]
- Uses existing frameworks: [✅ YES | ❌ NO]

Status: [✅ Simple | ⚠️ Needs justification]
```

---

**10.4.3: Anti-Abstraction Validation**

**Automated Check**:
```bash
# Search for custom wrappers
git diff --cached | grep -i -E '(class.*Wrapper|function wrap|createWrapper|Abstract.*Factory)'
```

**Validation**:
- ✅ PASS: No custom wrappers detected
- ⚠️ REVIEW: Wrapper detected → Check if justified in constitution
- ❌ FAIL: Unjustified custom abstraction

**Output**:
```markdown
**10.4.3 Anti-Abstraction**: [✅ PASS | ⚠️ REVIEW | ❌ FAIL]

Custom wrappers: [None detected | Found: [list]]
Justification: [Present in constitution | Missing]

Status: [✅ No abstractions | ⚠️ Needs review]
```

---

**10.4.4: Integration-First Validation** (if external dependencies)

**Automated Check**:
```bash
# Search for prohibited mocks in tests
git diff --cached **/*.test.* **/*.spec.* | grep -i -E '(jest\.mock.*auth|mockAuth|MockDatabase|mockTenant|mock.*db\.)'
```

**Validation**:
- ✅ PASS: Tests use real dependencies (Docker, Testcontainers, test APIs)
- ⚠️ CONDITIONAL: Mocks used → Check if allowed by Article V (email, SMS with justification)
- ❌ FAIL: Prohibited mocks (auth, database, tenant context)

**Output**:
```markdown
**10.4.4 Integration-First**: [✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL]

Test dependencies:
- Authentication: [Real test tenant | ❌ Mocked]
- Database: [Docker container | ❌ Mocked]
- Multi-tenancy: [Real isolation | ❌ Mocked]

Status: [✅ Real dependencies | ❌ Prohibited mocks]
```

---

**10.4.5: Prohibited Pattern Detection**

**Automated Check**:
```bash
# Scan for prohibited patterns from constitution
# Example patterns (customize based on constitution):

# File system synchronous operations
git diff --cached | grep -E 'fs\.readFileSync|fs\.writeFileSync'

# Raw database queries (if ORM required)
git diff --cached | grep -E 'db\.query\(|connection\.execute\('

# Mocked auth/db/tenancy
git diff --cached | grep -i -E 'jest\.mock.*auth|MockDatabase|mockTenantContext'
```

**Validation**:
- ✅ PASS: No prohibited patterns detected
- ❌ FAIL: Prohibited pattern detected → Must remediate

**Pattern Remediation Table** (from constitution):
| Prohibited Pattern | Why Prohibited | Remediation |
|--------------------|----------------|-------------|
| `fs.readFileSync()` | Blocks event loop | Use `fs.promises.readFile()` |
| `db.query()` | SQL injection risk | Use ORM (Prisma, TypeORM) |
| `jest.mock('auth0')` | Violates Article V | Use real Auth0 test tenant |

**Output**:
```markdown
**10.4.5 Prohibited Patterns**: [✅ PASS | ❌ FAIL]

Scan results:
- [Pattern name]: [✅ Not detected | ❌ Found at [file:line]]

Violations:
- [Pattern]: Found at [location] → Remediation: [how to fix]

Status: [✅ No violations | ❌ X violations]
```

---

**10.4.6: Complexity Check** (quick scan)

**Automated Check**:
```bash
# Count lines per function (rough complexity estimate)
git diff --cached | grep -E '^\+.*function|^\+.*def |^\+.*fn ' -A 50 | wc -l

# Check for deeply nested logic
git diff --cached | grep -E '^\+[ \t]{8,}(if|for|while|switch)'
```

**Validation**:
- ✅ PASS: Functions < 50 lines, no excessive nesting
- ⚠️ REVIEW: Large function or complex nesting → Consider refactoring
- ❌ FAIL: Cyclomatic complexity > 15 (if measurable)

**Output**:
```markdown
**10.4.6 Complexity**: [✅ PASS | ⚠️ REVIEW | ❌ FAIL]

Complexity indicators:
- Function size: [< 50 lines | ⚠️ X lines]
- Nesting depth: [< 4 levels | ⚠️ X levels]

Status: [✅ Acceptable | ⚠️ Consider refactoring]
```

---

**Constitutional Reviewer Summary**:

**Aggregate Status Decision**:
```
IF any validation == ❌ FAIL:
  → Overall: ❌ NOT READY

ELSE IF any validation == ⚠️ REVIEW:
  → Overall: ⚠️ NEEDS REVIEW

ELSE:
  → Overall: ✅ READY
```

**Output**:
```markdown
### Constitutional Reviewer: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**Validation Summary**:
- 10.4.1 TDD Compliance: [Status]
- 10.4.2 Simplicity: [Status]
- 10.4.3 Anti-Abstraction: [Status]
- 10.4.4 Integration-First: [Status]
- 10.4.5 Prohibited Patterns: [Status]
- 10.4.6 Complexity: [Status]

**Critical Issues**: [X issues] (❌ FAIL items)
**Review Items**: [X items] (⚠️ REVIEW items)

**Status**: [✅/⚠️/❌]
```

---

### Quality Gate Report (Aggregated)

**Generate comprehensive summary**:

```markdown
## Quality Gate Report

**Generated**: [Timestamp]
**Task**: [Task title]

### Reviewer Results

| Reviewer | Status | Critical Issues |
|----------|--------|-----------------|
| Code Reviewer | [✅/⚠️/❌] | [X] |
| Quality/Tests | [✅/⚠️/❌] | [X] |
| Security | [✅/⚠️/❌] | [X] |
| Constitutional | [✅/⚠️/❌] | [X] |

### Overall Status

**Decision**: [✅ READY TO COMMIT | ⚠️ NEEDS ATTENTION | ❌ BLOCKED]

**Gate Logic**:
```
IF any reviewer == ❌ NOT READY:
  → Overall: ❌ BLOCKED
  → Action: Fix critical issues before commit

ELSE IF any reviewer == ⚠️ NEEDS REVIEW:
  → Overall: ⚠️ NEEDS ATTENTION
  → Action: Review warnings, consider fixes

ELSE:
  → Overall: ✅ READY TO COMMIT
  → Action: Proceed to commit
```

### Critical Issues (Must Fix)
[List all ❌ NOT READY findings]
1. [Reviewer]: [Issue description] → [How to fix]
2. [Reviewer]: [Issue description] → [How to fix]

### Warnings (Review Recommended)
[List all ⚠️ NEEDS REVIEW findings]
1. [Reviewer]: [Warning description] → [Consideration]
2. [Reviewer]: [Warning description] → [Consideration]

### Next Steps

**IF ❌ BLOCKED**:
1. Fix critical issues listed above
2. Re-run quality gate: Review changes manually
3. Proceed when all reviewers pass

**IF ⚠️ NEEDS ATTENTION**:
Option 1: Address warnings now (recommended)
Option 2: Document warnings and proceed (acceptable risk)

User decision required: Continue with warnings? (yes/no)

**IF ✅ READY TO COMMIT**:
Proceed to Phase 5: Commit & Complete
```

**Update task document**:

```markdown
---

## Quality Gate Results

[Insert Quality Gate Report here]

**Approved**: [Timestamp]
**Decision**: [✅/⚠️/❌]
```

**Interactive Gate Logic**:

```
IF overall_status == ❌ BLOCKED:
  Output: "❌ Quality Gate: BLOCKED"
  Output: Critical issues report
  Output: "Please fix critical issues and re-run /speckit.quick"
  → STOP workflow

ELSE IF overall_status == ⚠️ NEEDS ATTENTION:
  Output: "⚠️ Quality Gate: NEEDS ATTENTION"
  Output: Warnings report
  Ask: "Review warnings above. Continue with warnings? (yes/no)"
  → IF no: STOP workflow
  → IF yes: Continue to Phase 5 with documented exceptions

ELSE: (overall_status == ✅ READY)
  Output: "✅ Quality Gate: READY TO COMMIT"
  Output: Summary report
  → Continue to Phase 5
```

---

### Phase 5: Commit & Complete

**Purpose**: Commit implementation and mark task complete.

**Commit Changes**:

```bash
# Stage all changes
git add .

# Create commit with descriptive message
git commit -m "[Type]: [Brief description]

[Optional: More details if needed]

Acceptance Criteria Met:
- [Criterion 1]
- [Criterion 2]
- [Criterion 3]

Quality Gate: ✅ PASS
- Code: ✅ Linting passed, criteria met
- Tests: ✅ All tests passed (X/X), coverage XX%
- Security: ✅ No vulnerabilities detected
- Constitutional: ✅ TDD followed, Article V compliant"
```

**Commit Message Format**:
- **Type**: feat (new feature), fix (bug fix), refactor (code cleanup), test (add tests), docs (documentation)
- **Brief description**: 50 characters or less
- **Body**: Details if needed, reference acceptance criteria
- **Footer**: Quality gate summary

**Update Task Document** (final):

```markdown
---

## Status: ✅ COMPLETE

**Completed**: [Timestamp]
**Commit**: [SHA]
**Quality Gate**: ✅ PASS

### Implementation Summary

**Files Changed**: [X files]
- [File 1]: [Description]
- [File 2]: [Description]

**Tests**: [X tests added, all passing]
- Unit: [X passed]
- Integration: [X passed]

**Coverage**: [XX%]

**Constitutional Compliance**: ✅ Verified
- TDD: ✅ RED-GREEN-REFACTOR followed
- Simplicity: ✅ Uses existing patterns
- Integration-First: ✅ Real dependencies
- Prohibited Patterns: ✅ None detected

### Acceptance Criteria
- [x] [Criterion 1] - Implemented in [file]
- [x] [Criterion 2] - Implemented in [file]
- [x] [Criterion 3] - Implemented in [file]

**All criteria met**: ✅ YES

---

## Quick Workflow Summary

**Total Time**: [Duration from creation to completion]
**Token Usage**: [Estimated ~60-90K tokens]

**Phases Completed**:
1. ✅ Context Loading (strategic)
2. ✅ Requirements Capture
3. ✅ Constitutional Pre-Flight (PASS)
4. ✅ TDD Implementation (RED-GREEN-REFACTOR)
5. ✅ Quality Gate (4 reviewers, all PASS)
6. ✅ Commit & Complete

**Next Steps**: Feature ready for deployment/PR
```

**Output to User**:

```
✅ Quick Task Complete!

📋 Summary:
- Task: [Title]
- Files changed: [X]
- Tests: [X passed, X added]
- Coverage: [XX%]
- Quality Gate: ✅ PASS (Code ✅, Tests ✅, Security ✅, Constitutional ✅)
- Commit: [SHA] - [Message]

📊 Implementation Details:
- TDD: ✅ Followed RED-GREEN-REFACTOR
- Constitutional: ✅ All articles compliant
- Security: ✅ No vulnerabilities
- Documentation: ✅ Updated

📁 Task Document: .specify/quick-tasks/quick-task-[timestamp].md

🎯 All acceptance criteria met:
- ✅ [Criterion 1]
- ✅ [Criterion 2]
- ✅ [Criterion 3]

No reconciliation needed for quick workflow.
Feature is ready for deployment/PR.

---

💡 TIP: For more complex features with architectural changes or MEDIUM/HIGH risk,
      use the full workflow: /speckit.specify → /speckit.plan → /speckit.implement
```

---

## Graceful Degradation

**Missing Files/Context** (backward compatibility):

| Condition | Behavior | Impact |
|-----------|----------|--------|
| No constitution.md | Skip constitutional checks, warn user | ⚠️ Standard practices apply (TDD, simplicity) |
| No .specify/memory/ | Skip context loading | ℹ️ No project-specific context |
| No templates/ | Generate task document from inline template | ✅ No impact on workflow |
| No config.yml | Use default budgets (15KB context) | ✅ Works with defaults |

**Quality Gate Failures**:

| Scenario | Handling |
|----------|----------|
| Linting fails | ❌ BLOCKED - Must fix before commit |
| Tests fail | ❌ BLOCKED - Must fix before commit |
| Secrets detected | ❌ BLOCKED - Must remove before commit |
| TDD not followed | ❌ BLOCKED - Must follow RED-GREEN-REFACTOR |
| Warnings only | ⚠️ User chooses to fix or proceed with documentation |

---

## Token Budget Estimate

| Phase | Tokens | Cumulative |
|-------|--------|------------|
| Context Loading (strategic) | 5-8K | 5-8K |
| Requirements Capture | 2-3K | 7-11K |
| Constitutional Pre-Flight | 3-5K | 10-16K |
| TDD Implementation (tactical context) | 30-50K | 40-66K |
| Quality Gate (4 reviewers) | 15-25K | 55-91K |
| Commit & Complete | 2-3K | 57-94K |

**Total**: 57-94K tokens (~$1.10-$1.80)

**Comparison**:
- Full workflow (v2.7): ~90-146K tokens ($1.75-$2.80)
- Quick workflow: ~57-94K tokens ($1.10-$1.80)
- **Savings**: 35-40% reduction

---

## Success Criteria

**This command succeeds when**:
1. ✅ Task captured with clear acceptance criteria
2. ✅ Constitutional pre-flight passed (or graceful degradation)
3. ✅ Implementation follows TDD (RED-GREEN-REFACTOR)
4. ✅ All quality gates pass (Code, Tests, Security, Constitutional)
5. ✅ All acceptance criteria met
6. ✅ Changes committed with descriptive message
7. ✅ Task document complete

**This command fails when**:
- ❌ User provides ambiguous/incomplete requirements → Recommend /speckit.specify
- ❌ Task is HIGH-risk or complex → Block and recommend full workflow
- ❌ Constitutional violations detected → Stop and request revision
- ❌ Quality gate critical failures → Stop and require fixes
- ❌ Tests fail → Stop and require fixes

---

**END OF /speckit.quick WORKFLOW**
