# EARS Requirements Quick Reference

**Easy Approach to Requirements Syntax (EARS)** - A structured format for writing unambiguous, testable requirements.

**Version**: 2.3.0
**Part of**: Spec-Kit for Claude Code

---

## Why EARS?

Traditional requirements use vague language:
- ❌ "The system should handle errors gracefully"
- ❌ "Users can efficiently search for products"
- ❌ "The app must respond quickly"

EARS requirements are **specific, measurable, and testable**:
- ✅ `WHEN an API call fails THEN the system SHALL retry up to 3 times with exponential backoff`
- ✅ `WHEN a user types 3+ characters in the search box THEN the system SHALL display results within 500ms`
- ✅ `IF user is idle for 15 minutes THEN the system SHALL terminate the session`

---

## The Four EARS Patterns

### Pattern 1: Event-Driven (WHEN...THEN)

**Format**: `WHEN [trigger event] THEN [system] SHALL [system response]`

**Use For**: User actions, external events, system triggers

**Examples**:
```
✅ WHEN a user submits valid credentials THEN the Authentication Service SHALL grant access within 2 seconds

✅ WHEN an API request fails with 503 status THEN the system SHALL retry up to 3 times with exponential backoff (1s, 2s, 4s)

✅ WHEN a payment is confirmed THEN the Order Service SHALL send a confirmation email within 30 seconds

✅ WHEN a user uploads a file >10MB THEN the system SHALL display a progress bar updating every 500ms
```

**Bad Examples (and why)**:
```
❌ WHEN a user logs in THEN show the dashboard
   → Missing: WHO performs action (system component), no performance criteria

❌ WHEN an error occurs THEN handle it gracefully
   → Missing: What error? What does "gracefully" mean? Specific response needed
```

---

### Pattern 2: State-Driven (IF...THEN)

**Format**: `IF [precondition/state] THEN [system] SHALL [system response]`

**Use For**: Conditional logic, state checks, authorization, validation

**Examples**:
```
✅ IF user is inactive for 15 minutes THEN the Session Manager SHALL terminate the session

✅ IF shopping cart total exceeds $100 THEN the Checkout Service SHALL apply free shipping discount

✅ IF uploaded file is not PDF or DOCX format THEN the system SHALL reject the file with error code 422

✅ IF user has admin role THEN the Dashboard SHALL display all organizations (not just assigned ones)
```

**Bad Examples**:
```
❌ IF user is logged in THEN show personalized content
   → Missing: System component, what content? What personalization?

❌ IF error happens THEN notify user
   → Missing: What error condition? How to notify (toast, modal, email)?
```

---

### Pattern 3: Continuous (WHILE...THE)

**Format**: `WHILE [ongoing condition] THE [system] SHALL [continuous behavior]`

**Use For**: Ongoing processes, monitoring, real-time updates

**Examples**:
```
✅ WHILE file upload is in progress THE UI SHALL display progress percentage updated every 500ms

✅ WHILE WebSocket connection is active THE system SHALL send heartbeat ping every 30 seconds

✅ WHILE user is typing in search box THE system SHALL throttle API calls to maximum 1 request per 300ms

✅ WHILE background job is running THE system SHALL write status updates to job_status table every 10 seconds
```

**Bad Examples**:
```
❌ WHILE loading THE system SHALL show a spinner
   → Missing: What's loading? How often to update? When to stop?

❌ WHILE processing THE system SHALL monitor progress
   → Missing: How to monitor? What intervals? Where stored?
```

---

### Pattern 4: Contextual (WHERE...THE)

**Format**: `WHERE [context/location] THE [system] SHALL [behavior in that context]`

**Use For**: Context-specific behavior, environmental conditions, platform variations

**Examples**:
```
✅ WHERE user has admin role THE Dashboard SHALL display all organizations with edit permissions

✅ WHERE screen width < 768px THE UI SHALL collapse navigation menu into hamburger icon

✅ WHERE environment is production THE system SHALL use encrypted database connections with TLS 1.3

✅ WHERE user timezone is UTC+8 THE system SHALL display timestamps in Asia/Shanghai timezone
```

**Bad Examples**:
```
❌ WHERE user is on mobile THE UI SHALL adapt
   → Missing: How to adapt? Specific responsive behaviors needed

❌ WHERE in production THE system SHALL be secure
   → Missing: What security measures? Specific controls needed
```

---

## Requirement IDs

Every EARS requirement gets a unique ID for traceability.

**Format**: `REQ-{CATEGORY}-{NUMBER}.{SUB}`

**Categories** (examples):
- `AUTH` - Authentication
- `DASH` - Dashboard
- `PAY` - Payments
- `NOTI` - Notifications
- `SRCH` - Search
- `ADMN` - Administration

**Numbering**:
- `{NUMBER}` - Major requirement (1, 2, 3...)
- `{SUB}` - Sub-requirement for related functionality (1, 2, 3...)

**Examples**:
```yaml
REQ-AUTH-1: User Authentication
  REQ-AUTH-1.1: WHEN user submits valid credentials THEN Authentication Service SHALL grant access within 2 seconds
  REQ-AUTH-1.2: IF user is inactive for 15 minutes THEN Session Manager SHALL terminate session
  REQ-AUTH-1.3: WHEN user enters invalid password 3 times THEN system SHALL lock account for 15 minutes

REQ-DASH-1: Dashboard Data Display
  REQ-DASH-1.1: WHERE user has admin role THE Dashboard SHALL display all organizations
  REQ-DASH-1.2: WHERE user has viewer role THE Dashboard SHALL display only assigned organizations
  REQ-DASH-1.3: WHEN dashboard loads THEN system SHALL fetch data within 1 second

REQ-PAY-1: Payment Processing
  REQ-PAY-1.1: WHEN payment is submitted THEN Payment Service SHALL process within 5 seconds
  REQ-PAY-1.2: IF payment fails THEN system SHALL retry up to 2 times with 3-second delay
  REQ-PAY-1.3: WHEN payment succeeds THEN system SHALL send confirmation email within 30 seconds
```

---

## Full Traceability Chain

EARS requirements flow through the entire development lifecycle:

### 1. Specification (`spec.md`)
```markdown
## Functional Requirements

### FR-001: User Authentication (REQ-AUTH-1)

**REQ-AUTH-1.1**: WHEN a user submits valid credentials THEN the Authentication Service SHALL grant access within 2 seconds

**REQ-AUTH-1.2**: IF user is inactive for 15 minutes THEN the Session Manager SHALL terminate the session

**REQ-AUTH-1.3**: WHEN user enters invalid password 3 times THEN the system SHALL lock the account for 15 minutes
```

### 2. Implementation Plan (`plan.md`)
```markdown
### Phase 2.1: Authentication Implementation (_Requirements: REQ-AUTH-1.1, REQ-AUTH-1.2, REQ-AUTH-1.3_)

1. **AuthService.login()** - Validate credentials and create session
   - Implements: REQ-AUTH-1.1
   - Performance: <2s response time

2. **SessionManager.checkInactivity()** - Monitor user activity
   - Implements: REQ-AUTH-1.2
   - Runs every 60s to check last activity timestamp

3. **AuthService.lockAccount()** - Implement account lockout
   - Implements: REQ-AUTH-1.3
   - Lock duration: 15 minutes
```

### 3. Task Breakdown (`tasks.md`)
```markdown
## Backend Tasks

### Task Group: Authentication (_Requirements: REQ-AUTH-1_)

- [ ] T001 [P] Implement AuthService.login() (_REQ-AUTH-1.1_)
  - Validate credentials against database
  - Create session token (JWT)
  - Return token within 2-second SLA

- [ ] T002 [P] Implement SessionManager inactivity check (_REQ-AUTH-1.2_)
  - Background job runs every 60 seconds
  - Check lastActivityTimestamp on all active sessions
  - Terminate sessions inactive >15 minutes

- [ ] T003 [P] Implement account lockout logic (_REQ-AUTH-1.3_)
  - Track failed login attempts in Redis (15-minute TTL)
  - Lock account after 3 failed attempts
  - Clear counter on successful login

## Test Tasks

- [ ] T020 [P] Unit test: AuthService.login() (_REQ-AUTH-1.1_)
  - Test valid credentials return token <2s
  - Test invalid credentials return 401

- [ ] T021 [P] Integration test: Session timeout (_REQ-AUTH-1.2_)
  - Simulate 15-minute inactivity
  - Verify session terminated

- [ ] T022 [P] Integration test: Account lockout (_REQ-AUTH-1.3_)
  - Submit 3 invalid passwords
  - Verify account locked for 15 minutes
  - Verify lock expires after 15 minutes
```

### 4. Test Coverage
```typescript
// tests/auth.service.spec.ts

describe('AuthService.login() [REQ-AUTH-1.1]', () => {
  it('should grant access within 2 seconds for valid credentials', async () => {
    const start = performance.now();
    const result = await authService.login('user@example.com', 'password123');
    const duration = performance.now() - start;

    expect(result.token).toBeDefined();
    expect(duration).toBeLessThan(2000); // REQ-AUTH-1.1 performance criteria
  });
});

describe('SessionManager.checkInactivity() [REQ-AUTH-1.2]', () => {
  it('should terminate session after 15 minutes of inactivity', async () => {
    const session = await createSession();

    // Fast-forward time by 15 minutes
    jest.advanceTimersByTime(15 * 60 * 1000);

    await sessionManager.checkInactivity();

    const sessionExists = await sessionManager.exists(session.id);
    expect(sessionExists).toBe(false); // REQ-AUTH-1.2 requirement
  });
});
```

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Vague System Response
```
❌ WHEN user clicks submit THEN validate the form
```
**Problem**: Who validates? What validation rules? What happens after?

**Fixed**:
```
✅ WHEN user clicks submit button THEN the Form Validator SHALL check all required fields and display inline error messages for any invalid fields within 100ms
```

---

### ❌ Mistake 2: Missing Performance Criteria
```
❌ WHEN search is executed THEN display results quickly
```
**Problem**: How quickly? What's acceptable?

**Fixed**:
```
✅ WHEN user submits search query THEN the Search Service SHALL return results within 500ms (p95 latency)
```

---

### ❌ Mistake 3: Ambiguous Conditions
```
❌ IF user is authorized THEN allow access
```
**Problem**: Authorized how? What does "allow access" mean?

**Fixed**:
```
✅ IF user JWT token contains "admin" role THEN the API Gateway SHALL allow access to /admin/* endpoints
```

---

### ❌ Mistake 4: Multiple Actions in One Requirement
```
❌ WHEN order is placed THEN send confirmation email, update inventory, charge payment, and create shipping label
```
**Problem**: Too many responsibilities. Hard to test individually.

**Fixed** (Split into separate requirements):
```
✅ REQ-ORDER-1.1: WHEN order is placed THEN the Order Service SHALL send confirmation email within 30 seconds
✅ REQ-ORDER-1.2: WHEN order is placed THEN the Inventory Service SHALL decrement stock count
✅ REQ-ORDER-1.3: WHEN order is placed THEN the Payment Service SHALL charge the stored payment method
✅ REQ-ORDER-1.4: WHEN order payment succeeds THEN the Shipping Service SHALL create shipping label
```

---

## Writing Effective EARS Requirements

### Checklist for Every Requirement

- [ ] **Specific System Component**: Which service/module performs the action?
- [ ] **Measurable Criteria**: Can you test if it's met? (time, count, size, etc.)
- [ ] **Testable**: Can you write an automated test for this?
- [ ] **Atomic**: Does it describe ONE behavior (not multiple)?
- [ ] **Clear Trigger/Condition**: What causes this to happen?
- [ ] **Unambiguous**: Could two engineers interpret this differently?

### Good vs. Bad Examples

| ❌ Bad | ✅ Good (EARS) |
|--------|----------------|
| System should load quickly | WHEN user navigates to homepage THEN the Page Loader SHALL render content within 2 seconds (p95) |
| Validate user input | WHEN user submits form THEN the Form Validator SHALL check email format matches regex `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$` |
| Handle errors gracefully | WHEN API call returns 5xx error THEN the Error Handler SHALL display user-friendly message and log full error details to Sentry |
| Support mobile devices | WHERE screen width < 768px THE UI SHALL stack form fields vertically with 100% width |
| Notify users | WHEN payment succeeds THEN the Notification Service SHALL send email within 30 seconds using SendGrid API |

---

## EARS in Spec-Kit Commands

### `/speckit.specify` - Generate Requirements
Automatically creates EARS-formatted requirements in `spec.md`:

```bash
/speckit.specify "User authentication with session management"

# Output: spec.md contains
REQ-AUTH-1.1: WHEN user submits valid credentials THEN Authentication Service SHALL grant access within 2 seconds
REQ-AUTH-1.2: IF user is inactive for 15 minutes THEN Session Manager SHALL terminate session
```

### `/speckit.plan` - Add Traceability
Links requirements to implementation phases:

```bash
/speckit.plan

# Output: plan.md contains
### Phase 2.1: Authentication Implementation (_Requirements: REQ-AUTH-1.1, REQ-AUTH-1.2_)
```

### `/speckit.tasks` - Tag Tasks
Tags tasks with requirement IDs for test coverage mapping:

```bash
/speckit.tasks

# Output: tasks.md contains
- [ ] T001 [P] Implement login endpoint (_REQ-AUTH-1.1_)
- [ ] T002 [P] Implement session timeout (_REQ-AUTH-1.2_)
```

---

## Resources

- **Full Template**: [src/.specify/templates/requirements-ears.md](../src/.specify/templates/requirements-ears.md)
- **Spec-Kit Documentation**: [README.md](../README.md)
- **Original EARS Methodology**: Mavin, Alistair. "EARS (Easy Approach to Requirements Syntax)." 2009

---

**Last Updated**: 2025-11-05
**Spec-Kit Version**: 2.3.0
