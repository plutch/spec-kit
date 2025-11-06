# Testing Approach & Standards

**Inclusion Mode**: Conditional
**Trigger Patterns**: `**/*.test.*`, `**/*.spec.*`, `**/tests/**/*`, `**/test/**/*`, `**/__tests__/**/*`
**Purpose**: Testing strategy, conventions, and quality standards

---

## Testing Philosophy

### Test-First Mindset
<!-- When TDD is required vs optional -->
- **TDD Required**: New features, bug fixes, refactoring, critical paths
- **TDD Optional**: Exploratory prototypes, documentation, configuration
- **Red-Green-Refactor Cycle**: Write failing test → Make it pass → Improve code

### Testing Pyramid
```
        /\
       /E2E\         Few (5-10% coverage)
      /------\
     /  Integ  \     Some (20-30% coverage)
    /----------\
   /    Unit     \   Many (60-75% coverage)
  /--------------\
```

## Test Coverage Requirements

### Coverage Thresholds
<!-- Adjust based on project risk level -->
- **Overall**: ≥ 75%
- **Critical Paths**: ≥ 90% (auth, payments, data integrity)
- **Business Logic**: ≥ 80%
- **UI Components**: ≥ 70%
- **Configuration/Utils**: ≥ 60%

### Coverage Metrics
- **Line Coverage**: % of lines executed
- **Branch Coverage**: % of conditional branches executed
- **Function Coverage**: % of functions called
- **Statement Coverage**: % of statements executed

**Priority**: Branch coverage > Line coverage

## Unit Testing

### Scope
- Test single function, method, or component in isolation
- Mock external dependencies (APIs, databases, file systems, time)
- Fast execution (< 10ms per test typical)
- No network I/O, no file system access (except temp/in-memory)

### Structure (AAA Pattern)
```javascript
describe('UserService.createUser', () => {
  it('should create user with hashed password', () => {
    // Arrange
    const userData = { email: 'test@example.com', password: 'secret123' };
    const mockHasher = jest.fn().mockReturnValue('hashed_secret123');

    // Act
    const user = UserService.createUser(userData, mockHasher);

    // Assert
    expect(mockHasher).toHaveBeenCalledWith('secret123');
    expect(user.password).toBe('hashed_secret123');
    expect(user.email).toBe('test@example.com');
  });
});
```

### Best Practices
- **One assertion concept per test** (can have multiple `expect()` for same concept)
- **Descriptive test names**: `should [expected behavior] when [condition]`
- **Test edge cases**: Null, undefined, empty, boundary values
- **Avoid test interdependence**: Each test can run independently
- **Mock external dependencies**: APIs, databases, clocks, random number generators

## Integration Testing

### Scope
- Test interaction between multiple modules/services
- Use real dependencies where practical (in-memory DB, test containers)
- Slower than unit tests (< 1s per test typical)
- May involve network I/O, database access

### When Required
- New API endpoints (contract testing)
- Database schema changes
- Service-to-service communication
- Authentication/authorization flows
- File uploads/processing

### Structure
```javascript
describe('POST /api/users (Integration)', () => {
  beforeEach(async () => {
    await setupTestDatabase(); // Real or in-memory DB
  });

  afterEach(async () => {
    await cleanupTestDatabase();
  });

  it('should create user and return 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', password: 'secret123' })
      .expect(201);

    expect(response.body.data).toHaveProperty('id');

    // Verify in database
    const user = await db.users.findById(response.body.data.id);
    expect(user.email).toBe('test@example.com');
  });
});
```

### Best Practices
- **Use test containers** (Docker) for real databases when possible
- **Test contracts**: Verify request/response schemas
- **Test error paths**: Network failures, timeout, invalid responses
- **Isolate tests**: Each test has clean state (setup/teardown)

## End-to-End (E2E) Testing

### Scope
- Test complete user workflows from UI to database
- Real browser, real network, real database (test environment)
- Slowest tests (5-30s per test typical)
- Focus on critical user journeys

### When Required
- User authentication flows
- Critical business transactions (checkout, payments, orders)
- Multi-step workflows
- Cross-browser compatibility checks

### Structure (Playwright Example)
```javascript
test('User can sign up and log in', async ({ page }) => {
  // Sign up
  await page.goto('/signup');
  await page.fill('#email', 'test@example.com');
  await page.fill('#password', 'secret123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');

  // Log out
  await page.click('[data-testid="logout"]');

  // Log back in
  await page.goto('/login');
  await page.fill('#email', 'test@example.com');
  await page.fill('#password', 'secret123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
});
```

### Best Practices
- **Test user journeys**, not individual pages
- **Use data attributes** for selectors (`data-testid="submit-button"`)
- **Avoid brittle selectors**: No classes, avoid IDs unless semantic
- **Parallel execution**: Run E2E tests concurrently when possible
- **Video/screenshot on failure**: Debugging aid

## Test Organization

### File Structure
```
src/
├── components/
│   ├── Button.tsx
│   └── Button.test.tsx           # Co-located unit tests
├── services/
│   ├── UserService.ts
│   └── UserService.test.ts
└── __tests__/
    ├── integration/
    │   └── api-users.test.ts     # Integration tests
    └── e2e/
        └── auth-flow.spec.ts     # E2E tests
```

### Naming Conventions
- **Unit/Integration**: `*.test.{js,ts}` or `*.spec.{js,ts}`
- **E2E**: `*.e2e.{js,ts}` or `*.spec.{js,ts}` in `e2e/` folder
- **Test Suites**: Match source file name (`Button.tsx` → `Button.test.tsx`)

## Mocking & Stubbing

### When to Mock
- **External APIs**: Third-party services, microservices
- **Database**: For unit tests (use real/in-memory for integration)
- **File System**: Read/write operations
- **Time**: `Date.now()`, `setTimeout`, `setInterval`
- **Random**: `Math.random()`, UUID generation
- **Network**: HTTP requests, WebSocket connections

### Mocking Libraries
<!-- Choose based on project tech stack -->
- **JavaScript/TypeScript**: Jest (built-in), Sinon, MSW (API mocking)
- **Python**: unittest.mock, pytest-mock
- **Java**: Mockito, PowerMock
- **C#**: Moq, NSubstitute

### Example (Jest)
```javascript
// Mock external API
jest.mock('../services/ExternalAPI');
import ExternalAPI from '../services/ExternalAPI';

ExternalAPI.fetchUser.mockResolvedValue({ id: 1, name: 'John' });

// Mock timer
jest.useFakeTimers();
setTimeout(callback, 1000);
jest.advanceTimersByTime(1000);
```

## Test Data Management

### Fixtures
- **Location**: `tests/fixtures/` or `__fixtures__/`
- **Format**: JSON, YAML, or code-based factories
- **Usage**: Consistent test data across tests

### Factories (Recommended)
```javascript
// tests/factories/user.factory.js
const createUser = (overrides = {}) => ({
  id: faker.datatype.uuid(),
  email: faker.internet.email(),
  name: faker.name.findName(),
  createdAt: new Date().toISOString(),
  ...overrides
});

// Usage in tests
const user = createUser({ email: 'specific@example.com' });
```

### Database Seeding
- **Development**: Seed with realistic, anonymized data
- **Testing**: Minimal seed data, clean state per test
- **Production**: Never seed in production

## Performance Testing

### Load Testing
- **Tool**: k6, Artillery, JMeter, Gatling
- **Scenarios**: Gradual ramp-up, spike, sustained load
- **Metrics**: Response time (p50, p95, p99), throughput, error rate

### Benchmarking
```javascript
// Performance regression tests
test('processLargeDataset should complete in < 1s', () => {
  const start = performance.now();
  processLargeDataset(largeSample);
  const duration = performance.now() - start;
  expect(duration).toBeLessThan(1000);
});
```

## Security Testing

### Input Validation
- Test SQL injection attempts
- Test XSS payloads
- Test CSRF attacks
- Test path traversal

### Authentication/Authorization
- Test missing auth tokens
- Test expired tokens
- Test insufficient permissions
- Test privilege escalation attempts

### Example
```javascript
test('should reject unauthenticated requests', async () => {
  const response = await request(app)
    .get('/api/users/me')
    .expect(401);

  expect(response.body.error.code).toBe('AUTHENTICATION_REQUIRED');
});
```

## Accessibility Testing

### Automated Tools
- **axe-core**: Accessibility rule engine
- **jest-axe**: Jest integration
- **Playwright**: Built-in accessibility assertions

### Example
```javascript
import { axe } from 'jest-axe';

test('Button should be accessible', async () => {
  const { container } = render(<Button>Click Me</Button>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## Continuous Integration

### CI Pipeline Requirements
1. **Run all tests**: Unit, integration, E2E
2. **Coverage report**: Fail if below threshold
3. **Lint/Format check**: Enforce code style
4. **Build verification**: Ensure app builds successfully
5. **Parallel execution**: Speed up test runs

### Test Execution Strategy
```yaml
# GitHub Actions example
- name: Unit & Integration Tests
  run: npm test -- --coverage --maxWorkers=4

- name: E2E Tests
  run: npm run test:e2e -- --workers=2

- name: Upload Coverage
  uses: codecov/codecov-action@v3
```

## Test Maintenance

### Flaky Tests
- **Identify**: Track test failures over time
- **Fix immediately**: Flaky tests erode confidence
- **Common causes**: Race conditions, timing issues, external dependencies

### Test Debt
- **Regular review**: Quarterly review of test suite
- **Remove obsolete tests**: Delete tests for removed features
- **Update brittle tests**: Refactor tests with implementation changes

### Test Speed
- **Monitor**: Track test execution time
- **Optimize**: Parallelize, reduce setup/teardown, use faster assertions
- **Target**: < 5min for full test suite

---

## Usage Instructions

This file is **conditionally loaded** when working in test files or test directories. It provides AI assistants with project-specific testing conventions to ensure consistent test quality.

**Last Updated**: [YYYY-MM-DD]
**Next Review**: [YYYY-MM-DD]
