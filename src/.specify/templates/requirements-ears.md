# EARS Requirements Format Guide

**EARS** = **E**asy **A**pproach to **R**equirements **S**yntax

## Purpose

EARS is a structured natural language format for writing unambiguous, testable requirements. It eliminates vague language and ensures every requirement can be verified through testing.

---

## Core Patterns

### 1. Event-Driven (WHEN-THEN)

**Pattern**: `WHEN [event/condition] THEN [system/subject] SHALL [response]`

**Use When**: System reacts to user actions, external events, or state changes

**Examples**:
```
WHEN a user submits valid login credentials THEN the Authentication Service SHALL grant access within 2 seconds

WHEN a payment transaction fails THEN the Payment System SHALL retry the transaction up to 3 times with exponential backoff

WHEN disk space falls below 10% THEN the Monitoring Service SHALL trigger a critical alert
```

**Key Elements**:
- **WHEN**: Describes the trigger event or condition
- **THEN**: Describes the expected system behavior
- **SHALL**: Indicates mandatory behavior (use MAY for optional, SHOULD for recommended)

### 2. State-Driven (IF-THEN)

**Pattern**: `IF [precondition/state] THEN [system/subject] SHALL [response]`

**Use When**: Behavior depends on existing state or conditions before action

**Examples**:
```
IF a user is not authenticated THEN the API Gateway SHALL reject the request with HTTP 401

IF an email address is invalid THEN the Registration Service SHALL return a validation error

IF the shopping cart is empty THEN the Checkout Button SHALL be disabled
```

**Key Difference from WHEN**:
- **IF**: State that must be true before action
- **WHEN**: Event that triggers action

### 3. Continuous (WHILE-SHALL)

**Pattern**: `WHILE [ongoing condition] THE [system/subject] SHALL [continuous behavior]`

**Use When**: System must maintain behavior throughout a duration or state

**Examples**:
```
WHILE a file upload is in progress THE Upload Manager SHALL display a progress indicator

WHILE a user session is active THE Session Manager SHALL validate the authentication token on every API request

WHILE a video is streaming THE Player SHALL adjust quality based on available bandwidth
```

### 4. Contextual (WHERE-SHALL)

**Pattern**: `WHERE [location/context/trigger] THE [system/subject] SHALL [contextual behavior]`

**Use When**: Behavior applies in specific locations, contexts, or scopes

**Examples**:
```
WHERE a user navigates to the profile page THE UI SHALL display the user's full account information

WHERE an API response contains sensitive data THE Data Layer SHALL encrypt the payload before transmission

WHERE a database transaction fails THE Transaction Manager SHALL rollback all changes
```

---

## Choosing the Right Pattern

| Pattern | Trigger Type | Example Scenario |
|---------|--------------|------------------|
| **WHEN-THEN** | Event/Action | User clicks button, API receives request, timer expires |
| **IF-THEN** | State/Condition | User is admin, file exists, value is null |
| **WHILE-SHALL** | Duration/Loop | During loading, throughout session, until completion |
| **WHERE-SHALL** | Location/Context | On specific page, in specific module, for specific role |

**Decision Flow**:
1. Is there a triggering event? → Use **WHEN-THEN**
2. Is it a precondition check? → Use **IF-THEN**
3. Is it ongoing behavior? → Use **WHILE-SHALL**
4. Is it location/context-specific? → Use **WHERE-SHALL**

---

## Requirements Quality Checklist

### ✅ Good Requirements (EARS)

**Testable**:
```
✅ WHEN a user submits a form with missing required fields THEN the Form Validator SHALL display inline error messages for each missing field
```
→ Clear test: Submit form without required fields, verify error messages appear

**Measurable**:
```
✅ WHEN a search query is submitted THEN the Search Service SHALL return results within 500 milliseconds for 95% of requests
```
→ Clear metric: p95 response time < 500ms

**Unambiguous**:
```
✅ IF a password is less than 8 characters THEN the Password Validator SHALL reject it with error code PWD_TOO_SHORT
```
→ Clear validation rule and error code

### ❌ Poor Requirements (Vague)

**Untestable**:
```
❌ The system should be fast
✅ WHEN a user loads the dashboard THEN the UI SHALL render within 1 second for 99% of requests
```

**Ambiguous**:
```
❌ The application must handle errors gracefully
✅ WHEN an API request fails THEN the Error Handler SHALL log the error, display a user-friendly message, and provide a retry option
```

**Immeasurable**:
```
❌ Users should be able to easily find products
✅ WHEN a user performs a search THEN the Search Results Page SHALL display relevant products ranked by relevance score within 2 seconds
```

---

## Advanced Patterns

### Combining Patterns (Complex Requirements)

**Pattern**: Use multiple EARS clauses with logical connectors

**Example**:
```
WHEN a user initiates a password reset THEN the Authentication Service SHALL:
  1. IF the email exists in the database THEN send a reset link to the user's email address
  2. IF the email does not exist THEN display a generic success message (to prevent email enumeration attacks)
  3. WHILE the reset link is valid (24 hours) THE system SHALL allow password changes
  4. WHEN the user clicks the reset link THEN the Password Reset Page SHALL load with a token validation
```

### Quantified Requirements (Performance, Scale)

**Pattern**: Include specific metrics in the response clause

**Examples**:
```
WHEN the API receives a bulk import request THEN the Import Service SHALL process at least 1000 records per second

IF concurrent user count exceeds 10,000 THEN the Load Balancer SHALL distribute traffic across at least 5 instances

WHILE a report is generating THE Progress Indicator SHALL update at least every 2 seconds
```

### Security Requirements

**Pattern**: Explicitly state security behavior

**Examples**:
```
WHEN a user attempts to access a protected resource THEN the Authorization Service SHALL verify the JWT token signature and expiration before granting access

IF a login attempt fails 3 times within 10 minutes THEN the Account Security System SHALL temporarily lock the account for 15 minutes

WHERE sensitive data is transmitted THE Communication Layer SHALL use TLS 1.3 or higher encryption
```

---

## Requirements Traceability

### Linking Requirements to Design & Tests

**In spec.md:**
```markdown
## Requirements

### 1. User Authentication
**REQ-AUTH-1.1**: WHEN a user submits valid credentials THEN the Authentication Service SHALL grant access within 2 seconds

**REQ-AUTH-1.2**: IF credentials are invalid THEN the Authentication Service SHALL reject access and log the attempt with timestamp and IP address
```

**In plan-N.md:**
```markdown
## Phase 2: Authentication Implementation

### 2.1: Credential Validation (_Requirements: REQ-AUTH-1.1, REQ-AUTH-1.2_)
- Implement bcrypt password hashing
- Add JWT token generation
- Create login endpoint `/api/auth/login`
```

**In tasks-N.md:**
```markdown
### Task 2.1.1: Implement login endpoint (_REQ-AUTH-1.1_)
- [ ] Create POST `/api/auth/login` endpoint
- [ ] Validate credentials against database
- [ ] Generate JWT token on success
- [ ] Return token within 2 seconds (performance test)
```

**In test files:**
```javascript
// REQ-AUTH-1.1
test('should authenticate user with valid credentials within 2 seconds', async () => {
  const start = Date.now();
  const response = await request(app)
    .post('/api/auth/login')
    .send({ email: 'test@example.com', password: 'ValidPassword123' });
  
  const duration = Date.now() - start;
  
  expect(response.status).toBe(200);
  expect(response.body.token).toBeDefined();
  expect(duration).toBeLessThan(2000); // REQ-AUTH-1.1: < 2 seconds
});
```

---

## EARS vs Traditional Requirements

| Traditional (Vague) | EARS (Structured) |
|---------------------|-------------------|
| "The system should be secure" | "WHEN a user attempts unauthorized access THEN the Authorization Service SHALL return HTTP 403 and log the attempt" |
| "Users can search for products" | "WHEN a user enters a search query THEN the Search Service SHALL return relevant products within 500ms" |
| "Handle errors appropriately" | "IF an API request fails THEN the Error Handler SHALL log the error with stack trace and return a user-friendly message with error code" |
| "Support multiple languages" | "WHERE a user selects a language preference THE UI SHALL display all text content in the selected language from the i18n resource bundle" |

---

## Tips for Writing EARS Requirements

### 1. Be Specific
❌ "The system should validate inputs"
✅ "WHEN a user submits a form THEN the Input Validator SHALL check all fields against schema definitions and reject invalid inputs with field-level error messages"

### 2. Use Measurable Metrics
❌ "The application should be fast"
✅ "WHEN a user loads the homepage THEN the Page Renderer SHALL display first contentful paint within 1.5 seconds"

### 3. Avoid Implementation Details (in high-level requirements)
❌ "The system shall use Redis for caching"
✅ "WHEN a user requests frequently accessed data THEN the Data Layer SHALL retrieve it from cache within 50ms"
→ Implementation detail (Redis) goes in design doc, not requirements

### 4. One Behavior Per Requirement
❌ "WHEN a user logs in THEN the system SHALL authenticate them, log the event, update last login timestamp, and send a notification email"
✅ Split into 4 separate requirements (REQ-AUTH-1.1, REQ-AUTH-1.2, REQ-AUTH-1.3, REQ-AUTH-1.4)

### 5. Use SHALL for Mandatory, SHOULD for Recommended, MAY for Optional
- **SHALL**: Non-negotiable behavior (functional requirements)
- **SHOULD**: Recommended but not mandatory (nice-to-have)
- **MAY**: Optional behavior (future consideration)

**Example**:
```
WHEN a user deletes an account THEN the System SHALL permanently remove all personal data (GDPR compliance)

WHEN a user uploads a large file THEN the Upload Manager SHOULD compress it before storage (optimization)

WHERE a user hovers over a tooltip icon THE UI MAY display additional context (UX enhancement)
```

---

## Common Mistakes to Avoid

### 1. Using "and" to Chain Multiple Behaviors
❌ "WHEN a user logs in THEN the system SHALL authenticate them and log the event and update the timestamp"
✅ Break into separate requirements:
  - REQ-1.1: WHEN a user logs in THEN the Authentication Service SHALL verify credentials
  - REQ-1.2: WHEN authentication succeeds THEN the Audit Logger SHALL record the login event
  - REQ-1.3: WHEN authentication succeeds THEN the User Service SHALL update the last_login timestamp

### 2. Vague Subjects
❌ "WHEN a button is clicked THEN something SHALL happen"
✅ "WHEN a user clicks the Submit button THEN the Form Handler SHALL validate inputs and submit data to the API"

### 3. Missing Measurable Criteria
❌ "WHEN a user searches THEN results SHALL be returned quickly"
✅ "WHEN a user submits a search query THEN the Search Service SHALL return results within 500ms for 95% of requests"

### 4. Implementation Leakage
❌ "WHEN a user logs in THEN the LoginController SHALL call AuthService.authenticate() with bcrypt hashing"
✅ "WHEN a user logs in THEN the Authentication Service SHALL verify credentials using secure password hashing"

---

## EARS in Different Contexts

### User Stories + EARS Acceptance Criteria

```markdown
**User Story**: As a registered user, I want to reset my password if I forget it, so that I can regain access to my account.

**Acceptance Criteria (EARS)**:
1. WHEN a user requests a password reset THEN the System SHALL send a reset link to the registered email address within 30 seconds
2. IF the email is not found in the system THEN the System SHALL display a generic success message (prevent email enumeration)
3. WHEN a user clicks the reset link THEN the Password Reset Page SHALL load with a token validation check
4. IF the reset token is expired (>24 hours) THEN the System SHALL display an error message and prompt the user to request a new link
5. WHEN a user submits a new password THEN the Password Validator SHALL enforce complexity rules (8+ characters, 1 uppercase, 1 lowercase, 1 number, 1 special character)
```

### API Contract Requirements

```markdown
**Endpoint**: POST `/api/auth/login`

**Requirements**:
1. WHEN a valid email and password are provided THEN the API SHALL return HTTP 200 with a JWT token in the response body
2. IF the email or password is missing THEN the API SHALL return HTTP 400 with error code "MISSING_CREDENTIALS"
3. IF credentials are invalid THEN the API SHALL return HTTP 401 with error code "INVALID_CREDENTIALS"
4. WHEN authentication succeeds THEN the JWT token SHALL be valid for 24 hours
5. WHILE the token is valid THE API SHALL accept it for authenticated requests via the Authorization header
```

---

## Further Reading

- **EARS Paper (Mavin et al., 2009)**: "Easy Approach to Requirements Syntax (EARS)"
- **IEEE 830**: IEEE Recommended Practice for Software Requirements Specifications
- **INCOSE**: International Council on Systems Engineering - Requirements Best Practices

---

**Last Updated**: 2025-11-05
**Version**: 1.0.0
