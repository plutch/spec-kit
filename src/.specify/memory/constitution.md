# [PROJECT_NAME] Constitution
<!-- Example: Spec Constitution, TaskFlow Constitution, etc. -->

## Core Principles

### [PRINCIPLE_1_NAME]
<!-- Example: I. Library-First -->
[PRINCIPLE_1_DESCRIPTION]
<!-- Example: Every feature starts as a standalone library; Libraries must be self-contained, independently testable, documented; Clear purpose required - no organizational-only libraries -->

### [PRINCIPLE_2_NAME]
<!-- Example: II. CLI Interface -->
[PRINCIPLE_2_DESCRIPTION]
<!-- Example: Every library exposes functionality via CLI; Text in/out protocol: stdin/args → stdout, errors → stderr; Support JSON + human-readable formats -->

### [PRINCIPLE_3_NAME]
<!-- Example: III. Test-First (NON-NEGOTIABLE) -->
[PRINCIPLE_3_DESCRIPTION]
<!-- Example: TDD mandatory: Tests written → User approved → Tests fail → Then implement; Red-Green-Refactor cycle strictly enforced -->

### [PRINCIPLE_4_NAME]
<!-- Example: IV. Integration Testing -->
[PRINCIPLE_4_DESCRIPTION]
<!-- Example: Focus areas requiring integration tests: New library contract tests, Contract changes, Inter-service communication, Shared schemas -->

### [PRINCIPLE_5_NAME]
<!-- Example: V. Observability, VI. Versioning & Breaking Changes, VII. Simplicity -->
[PRINCIPLE_5_DESCRIPTION]
<!-- Example: Text I/O ensures debuggability; Structured logging required; Or: MAJOR.MINOR.BUILD format; Or: Start simple, YAGNI principles -->

## [SECTION_2_NAME]
<!-- Example: Additional Constraints, Security Requirements, Performance Standards, etc. -->

[SECTION_2_CONTENT]
<!-- Example: Technology stack requirements, compliance standards, deployment policies, etc. -->

## [SECTION_3_NAME]
<!-- Example: Development Workflow, Review Process, Quality Gates, etc. -->

[SECTION_3_CONTENT]
<!-- Example: Code review requirements, testing gates, deployment approval process, etc. -->

## Prohibited Patterns (Machine-Readable - v2.4)

**Purpose**: Enable automated constitutional validation during `/speckit.implement` (Step 10.4 Constitutional Reviewer) and `/speckit.reconcile` (Step 5 Question 5).

**Format**: Each pattern must include:
1. **Pattern** - Code pattern to detect (use regex-compatible syntax)
2. **Reason** - Why this pattern is prohibited
3. **Detection** - How to scan for it (grep/ripgrep command or tool)
4. **Remediation** - Recommended fix or alternative approach

---

### Pattern 1: [PATTERN_NAME]
<!-- Example: Synchronous File Operations -->

- **Pattern**: `[CODE_PATTERN]`
  <!-- Example: fs.readFileSync, fs.writeFileSync -->

- **Reason**: [WHY_PROHIBITED]
  <!-- Example: Blocks event loop, violates non-blocking principle (Article II: Async-First) -->

- **Detection**: `[DETECTION_COMMAND]`
  <!-- Example: grep -rn "fs\.\(read\|write\)FileSync" src/ -->

- **Remediation**: [ALTERNATIVE_APPROACH]
  <!-- Example: Use fs.promises.readFile() / fs.promises.writeFile() for async operations -->

---

### Pattern 2: [PATTERN_NAME]
<!-- Example: Direct Database Access -->

- **Pattern**: `[CODE_PATTERN]`
  <!-- Example: db.query(), db.execute() -->

- **Reason**: [WHY_PROHIBITED]
  <!-- Example: Bypasses repository layer, violates separation of concerns (Article VIII: Anti-Abstraction Gate - repositories are justified) -->

- **Detection**: `[DETECTION_COMMAND]`
  <!-- Example: grep -rn "db\.\(query\|execute\)" src/ --exclude-dir=repositories -->

- **Remediation**: [ALTERNATIVE_APPROACH]
  <!-- Example: Use repository methods: userRepo.findById(), orderRepo.create() -->

---

### Pattern 3: [PATTERN_NAME]
<!-- Example: Global State Mutations -->

- **Pattern**: `[CODE_PATTERN]`
  <!-- Example: window.globalState = {}, global.config = {} -->

- **Reason**: [WHY_PROHIBITED]
  <!-- Example: Creates hidden dependencies, makes testing difficult, violates functional principles -->

- **Detection**: `[DETECTION_COMMAND]`
  <!-- Example: grep -rn "\(window\|global\)\.[a-zA-Z]* =" src/ -->

- **Remediation**: [ALTERNATIVE_APPROACH]
  <!-- Example: Use dependency injection, pass state explicitly, or use context providers -->

---

### Pattern 4: Mocked Auth in Integration Tests (Article V - v2.6)

- **Pattern**: `jest.mock('auth0')`, `jest.mock('@auth0/auth0-react')`, `mockJWT()`, `jest.spyOn(authService, 'validate')`

- **Reason**: Violates Article V: Integration-First Testing - authentication logic MUST use real Auth0 test tenant in integration tests. Mocking auth hides JWT validation bugs, token expiry issues, and RBAC errors that only surface with real tokens.

- **Detection**: `grep -rn "jest\.mock.*auth\|mockJWT\|mock.*auth.*validate" tests/integration/ tests/e2e/`

- **Remediation**: Use real Auth0 test tenant with test credentials. Example:
  ```javascript
  // ❌ WRONG (integration test)
  jest.mock('@auth0/auth0-react');

  // ✅ CORRECT (integration test)
  const auth0 = new Auth0Client({
    domain: process.env.AUTH0_TEST_DOMAIN,
    clientId: process.env.AUTH0_TEST_CLIENT_ID
  });
  const token = await auth0.getTokenSilently();
  ```

---

### Pattern 5: Mocked Database in Integration Tests (Article V - v2.6)

- **Pattern**: `MockDatabase()`, `InMemoryDB()`, `createMockRepository()` in integration tests

- **Reason**: Violates Article V: Integration-First Testing - integration tests MUST use real database (Docker/Testcontainers) to catch SQL syntax errors, transaction issues, index performance problems, and migration failures that only surface with real databases.

- **Detection**: `grep -rn "InMemory\|MockDB\|MockRepository\|mock.*database" tests/integration/ tests/e2e/`

- **Remediation**: Use Docker/Testcontainers for integration tests. Example:
  ```javascript
  // ❌ WRONG (integration test)
  const db = new InMemoryDatabase();

  // ✅ CORRECT (integration test)
  const { GenericContainer } = require('testcontainers');
  const postgres = await new GenericContainer('postgres:15')
    .withExposedPorts(5432)
    .start();
  const db = new Database(postgres.getConnectionString());
  ```

---

### Pattern 6: Mocked Tenant Context (Multi-Tenant Violation - Article V - v2.6)

- **Pattern**: `mockTenantContext()`, `setCurrentTenant(fakeId)`, `jest.mock('tenant-middleware')`

- **Reason**: Violates Article V: Integration-First Testing - tenant isolation MUST be tested with real tenant filtering in database queries. Mocking tenant context hides data leakage bugs where queries miss `WHERE tenant_id = ?` filters, leading to cross-tenant data exposure in production.

- **Detection**: `grep -rn "mockTenant\|fakeTenant\|setCurrentTenant.*fake\|mock.*tenant" tests/integration/ tests/e2e/`

- **Remediation**: Use real tenant IDs in test database, verify tenant filtering in queries. Example:
  ```javascript
  // ❌ WRONG (integration test)
  mockTenantContext({ tenantId: 'fake-123' });
  const orders = await getOrders();

  // ✅ CORRECT (integration test)
  const tenantA = await createTestTenant({ name: 'Tenant A' });
  const tenantB = await createTestTenant({ name: 'Tenant B' });

  // Create orders for different tenants
  await createOrder({ tenantId: tenantA.id, total: 100 });
  await createOrder({ tenantId: tenantB.id, total: 200 });

  // Verify tenant isolation
  const tenantAOrders = await getOrders({ tenantId: tenantA.id });
  expect(tenantAOrders).toHaveLength(1); // Should NOT see Tenant B's orders
  ```

---

### Pattern 7: [PROJECT_SPECIFIC_PATTERN]
<!-- Add project-specific prohibited patterns below -->

- **Pattern**: `[CODE_PATTERN]`
- **Reason**: [WHY_PROHIBITED]
- **Detection**: `[DETECTION_COMMAND]`
- **Remediation**: [ALTERNATIVE_APPROACH]

---

**Usage in Commands**:
- **`/speckit.implement` Step 10.4.5**: Constitutional Reviewer scans for all patterns
- **`/speckit.reconcile` Step 5 Question 5**: Checks surgical edits for pattern violations
- **Violation Reporting**: Format: `file:line - Pattern: [pattern name] - Reason: [why prohibited]`

**Notes**:
- Patterns are project-specific - customize for your codebase
- Detection commands should be grep/ripgrep compatible
- Use `--exclude-dir` to avoid false positives in test/vendor code
- Document exceptions in plan.md Complexity Tracking if pattern is unavoidable

## Governance
<!-- Example: Constitution supersedes all other practices; Amendments require documentation, approval, migration plan -->

[GOVERNANCE_RULES]
<!-- Example: All PRs/reviews must verify compliance; Complexity must be justified; Use [GUIDANCE_FILE] for runtime development guidance -->

**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
<!-- Example: Version: 2.1.1 | Ratified: 2025-06-13 | Last Amended: 2025-07-16 -->
