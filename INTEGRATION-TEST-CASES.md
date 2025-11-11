# Discovery Interview Gates - Test Cases
# Comprehensive testing guide for v2.7.0 integration

---

## Test Suite 1: Pre-Spec Discovery (Enhanced Step 1)

### Test 1.1: Vagueness Detection - Generic Description

**Input:**
```bash
/speckit.specify "make the app better"
```

**Expected Behavior:**
1. Vagueness detection triggers (< 30 words, generic verb "make better")
2. Shows A/B option:
   ```
   🤔 Your description seems high-level.

   A) Generate spec now (fast, ~1 min, may include assumptions)
   B) Ask 3-5 generic-specific questions first (better spec, ~2-3 min)

   Recommendation: Choose B (better spec with discovery)
   ```
3. Wait for user response

**Test Validation:**
- ✅ Vagueness detected correctly
- ✅ Generic domain detected (no specific keywords)
- ✅ A/B option presented
- ✅ Recommendation shown

---

### Test 1.2: Domain Detection - Authentication

**Input:**
```bash
/speckit.specify "add OAuth2 authentication with JWT tokens"
```

**Expected Behavior:**
1. Domain detection identifies "authentication" (keywords: "OAuth2", "authentication", "JWT")
2. Shows A/B option with domain context:
   ```
   🤔 Your description seems high-level and relates to authentication.

   A) Generate spec now
   B) Ask 3-5 authentication-specific questions first

   Recommendation: Choose B (auth features benefit from specific flow/storage decisions)
   ```
3. IF user chooses B:
   - Show "🔐 Authentication Feature Discovery"
   - Ask 4 auth-specific questions (method, storage, duration, multi-tenancy)
4. Store discovery insights

**Test Validation:**
- ✅ Domain detected as "authentication"
- ✅ Domain-specific recommendation shown
- ✅ Auth template loaded (not generic)
- ✅ 4 auth questions asked
- ✅ Insights stored for spec generation

---

### Test 1.3: Domain Detection - Multiple Domains

**Input:**
```bash
/speckit.specify "create REST API dashboard with authentication and database caching"
```

**Expected Behavior:**
1. Multiple domains detected: "api", "ui", "authentication", "data", "performance"
2. Primary domain: First detected ("api" appears first)
3. Shows A/B option with primary domain:
   ```
   🤔 Your description seems high-level and relates to api.

   A) Generate spec now
   B) Ask 3-5 api-specific questions first
   ```
4. IF user chooses B:
   - Use API template (primary domain)
   - Questions about API style, versioning, failure handling

**Test Validation:**
- ✅ Multiple domains detected
- ✅ Primary domain selected correctly
- ✅ API template used (not generic or other domain)

---

### Test 1.4: Discovery Skip - User Chooses Fast Path

**Input:**
```bash
/speckit.specify "add user authentication"
# User chooses: A
```

**Expected Behavior:**
1. Vagueness + domain detected → Show A/B option
2. User types "A" (or "fast" or "skip")
3. Skip discovery, proceed to Step 2 (short name generation)
4. No discovery insights stored
5. Spec generated with informed assumptions

**Test Validation:**
- ✅ Discovery skipped when user chooses A
- ✅ No questions asked
- ✅ Proceeds to spec generation
- ✅ spec-metadata.json shows discovery_mode: "skipped"

---

### Test 1.5: Discovery Insights Integration - EARS Requirements

**Input:**
```bash
/speckit.specify "add authentication"
# User chooses: B (discovery)
# Q1: OAuth 2.0
# Q2: httpOnly cookies
# Q3: Short-lived + refresh (15 min / 7 days)
# Q4: Multi-tenant (shared DB)
```

**Expected Behavior:**
1. Discovery completes, insights stored
2. Spec generation uses insights:
   - REQ-AUTH-1.1: "WHEN user initiates login THEN system SHALL redirect to OAuth provider within 200ms"
   - REQ-AUTH-1.2: "WHEN OAuth callback received THEN system SHALL store access token in httpOnly cookie"
   - REQ-AUTH-1.3: "WHEN access token expires (15 min) THEN system SHALL use refresh token (7 days)"
   - REQ-AUTH-2.1: "WHERE user belongs to multiple tenants THE system SHALL require explicit tenant selection"
3. [NEEDS CLARIFICATION] markers reduced (0-1 instead of 3+)

**Test Validation:**
- ✅ EARS requirements match discovery answers
- ✅ httpOnly cookies mentioned in requirements
- ✅ 15 min / 7 days durations in requirements
- ✅ Multi-tenant requirements present
- ✅ Markers count ≤ 1

---

### Test 1.6: Domain Detection - Performance

**Input:**
```bash
/speckit.specify "optimize API response time, currently takes 5 seconds"
```

**Expected Behavior:**
1. Domain detected: "performance" (keywords: "optimize", "response time")
2. Performance template loaded
3. Questions:
   - Q1: Bottleneck identification → User: "Database queries"
   - Q2: Target metric → User: "<500ms"
   - Q3: Caching strategy → User: "Redis for query cache"
   - Q4: Scale considerations → User: "1000-10000 concurrent users"

**Test Validation:**
- ✅ Performance domain detected
- ✅ Performance template used
- ✅ 4 performance questions asked
- ✅ Target metrics stored in insights

---

### Test 1.7: No Vagueness - Direct to Spec Generation

**Input:**
```bash
/speckit.specify "Implement OAuth 2.0 authentication using Auth0 provider with PKCE flow, storing access tokens in httpOnly cookies with 15-minute expiration and 7-day refresh tokens, supporting multi-tenant architecture with row-level security via tenant_id in JWT claims"
```

**Expected Behavior:**
1. Vagueness detection: PASS (detailed, >30 words, specific)
2. Skip discovery detection entirely
3. Proceed directly to Step 2 (short name generation)
4. No A/B option shown

**Test Validation:**
- ✅ No vagueness detected
- ✅ No A/B option shown
- ✅ Proceeds directly to spec generation
- ✅ spec-metadata.json shows discovery_mode: "not_required"

---

## Test Suite 2: Specification Review Gate (New Step 8)

### Test 2.1: Basic Approval Flow

**Setup:**
```bash
/speckit.specify "add OAuth authentication"
# Discovery skipped or completed
# Spec generated
```

**Expected Behavior:**
1. Specification Review Gate appears
2. Shows summary:
   - Requirements count
   - Top 5 EARS requirements
   - Risk assessment
   - Architecture impact
   - Quality scores
3. Shows approval prompt:
   ```
   ⏸️ WAITING_FOR_SPECIFICATION_APPROVAL

   Options:
   1. "approved" → Proceed
   2. "revise [aspect]" → Modify
   3. "clarify [question]" → Ask details
   4. "show full spec" → View complete spec
   ```
4. User types: "approved"
5. spec-metadata.json updated:
   ```json
   {
     "approvals": {
       "specification": {
         "approved": true,
         "timestamp": "...",
         "revisions_requested": 0
       }
     }
   }
   ```

**Test Validation:**
- ✅ Gate appears after spec generation
- ✅ Summary shows all sections
- ✅ Approval prompt shown
- ✅ "approved" response accepted
- ✅ metadata updated correctly
- ✅ Proceeds to Step 9 (completion message)

---

### Test 2.2: Revision Flow

**Setup:**
Same as Test 2.1, but:

**User Response:**
```
revise token storage to use Redis instead of cookies
```

**Expected Behavior:**
1. Extract revision aspect: "token storage to use Redis"
2. Ask clarifying question:
   ```
   🔄 Revision Request: token storage to use Redis

   To make this change accurately, I need to clarify:
   - Which Redis configuration? (Default, Cluster, Sentinel)
   - What TTL should tokens have? (15 min access token TTL)
   - Should refresh tokens also be in Redis?

   Your clarification:
   ```
3. User: "Default Redis, 15 min TTL, yes refresh tokens in Redis"
4. Regenerate spec.md:
   - Update auth requirements (REQ-AUTH-1.2: Redis storage)
   - Update Data Model (remove cookie storage, add Redis key structure)
   - Update NFRs (Redis availability requirement)
5. revision_count++
6. Return to review gate with updated summary
7. User types: "approved"

**Test Validation:**
- ✅ "revise" command recognized
- ✅ Aspect extracted correctly
- ✅ Clarifying question asked
- ✅ Spec regenerated with changes
- ✅ metadata shows revisions_requested: 1
- ✅ revision_history recorded
- ✅ Can approve after revision

---

### Test 2.3: Clarify Flow

**User Response:**
```
clarify what multi-tenant support means technically
```

**Expected Behavior:**
1. Extract question: "what multi-tenant support means technically"
2. Read relevant spec.md sections (multi-tenant requirements, data model)
3. Show answer:
   ```
   💡 Clarification: what multi-tenant support means technically

   Multi-tenant support in this specification means:

   1. **Row-Level Security (RLS)**: All database tables have a tenant_id column
      that filters data per tenant automatically.

   2. **JWT Claims**: Access tokens include tenant_id in JWT payload, verified
      on every request.

   3. **Tenant Context**: Middleware sets tenant context from JWT, all queries
      filter by tenant_id.

   [Quote from spec.md Requirements:]
   "REQ-TENANT-1.1: WHERE user belongs to tenant THE system SHALL filter all
   data queries by tenant_id"

   Does this answer your question?

   You can:
   - Ask another clarification question
   - Request a revision if something should change
   - Approve the spec if satisfied
   ```
4. User types: "approved"

**Test Validation:**
- ✅ "clarify" command recognized
- ✅ Question extracted
- ✅ Relevant spec sections read
- ✅ Answer provided with quotes
- ✅ Returns to approval loop
- ✅ Can approve after clarification

---

### Test 2.4: Show Full Spec

**User Response:**
```
show full spec
```

**Expected Behavior:**
1. Read complete spec.md file
2. Display with line numbers (optional)
3. Show entire content
4. Return to approval prompt:
   ```
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   End of specification.

   Please review and choose:
   - Type 'approved' to approve
   - Type 'revise [aspect]' to make changes
   - Type 'clarify [question]' for more details
   ```

**Test Validation:**
- ✅ "show full spec" recognized
- ✅ Complete spec.md displayed
- ✅ Returns to approval loop
- ✅ Can approve after viewing

---

### Test 2.5: Skip with Warning

**User Response:**
```
skip
```

**Expected Behavior:**
1. Show warning:
   ```
   ⚠️ WARNING: Skipping Approval Gate

   Skipping specification approval is NOT RECOMMENDED because:
   - Spec may contain assumptions that don't match your intent
   - Misunderstandings discovered late cost 10K-20K tokens in rework
   - Implementation may proceed in wrong direction

   Are you sure you want to skip approval? (yes/no)
   ```
2. User types: "yes"
3. metadata updated:
   ```json
   {
     "approvals": {
       "specification": {
         "approved": true,
         "approval_skipped": true,
         "skip_reason": "user requested"
       }
     }
   }
   ```
4. Warning logged, proceeds to Step 9

**Test Validation:**
- ✅ "skip" recognized
- ✅ Warning shown
- ✅ Confirmation required
- ✅ metadata shows approval_skipped: true
- ✅ Proceeds after confirmation

---

### Test 2.6: Auto-Approve via Config

**Setup:**
```yaml
# .specify/config.yml
discovery_gates:
  specification_review_gate:
    auto_approve: true
```

**Expected Behavior:**
1. Spec generated
2. Gate logic checks config
3. Skips approval loop
4. Shows warning:
   ```
   ⚠️ Auto-approval enabled via config (non-interactive mode)

   Specification auto-approved.
   ```
5. metadata updated:
   ```json
   {
     "approvals": {
       "specification": {
         "approved": true,
         "auto_approved": true
       }
     }
   }
   ```

**Test Validation:**
- ✅ Config setting honored
- ✅ Approval loop skipped
- ✅ Warning shown
- ✅ metadata shows auto_approved: true

---

### Test 2.7: Auto-Approve via Environment Variable

**Setup:**
```bash
export SPECKIT_AUTO_APPROVE=true
/speckit.specify "add authentication"
```

**Expected Behavior:**
Same as Test 2.6, but triggered by environment variable instead of config

**Test Validation:**
- ✅ Environment variable overrides config
- ✅ Auto-approved
- ✅ metadata shows auto_approved: true

---

## Test Suite 3: Constitutional Approval Gate (Planning Phase)

### Test 3.1: PASS Status - Clean Approval

**Setup:**
```bash
/speckit.plan
# Phase -1 validation completes
# Result: ✅ PASS (all articles passed)
```

**Expected Behavior:**
1. Constitutional gate appears
2. Shows validation summary:
   ```
   Overall Status: ✅ PASS

   Article I (Simplicity): ✅ PASS - No new frameworks
   Article V (Integration-First): ✅ PASS - Real dependencies in tests
   Article IX (Prohibited Patterns): ✅ PASS - No violations
   [... other articles ...]

   ✅ READY TO PROCEED - Plan complies with all constitutional principles.

   Type "approved" to proceed to tasks/implementation.
   ```
3. User types: "approved"
4. metadata updated:
   ```json
   {
     "approvals": {
       "planning": {
         "approved": true,
         "constitutional_status": "PASS"
       }
     }
   }
   ```

**Test Validation:**
- ✅ Gate appears after Phase -1
- ✅ PASS status shown correctly
- ✅ Approval allowed
- ✅ metadata updated
- ✅ Proceeds to finalize plan

---

### Test 3.2: CONDITIONAL Status - Justified Complexity

**Setup:**
```bash
/speckit.plan
# Plan adds Redis (new framework)
# Phase -1 validation: ⚠️ CONDITIONAL
```

**Expected Behavior:**
1. Gate shows CONDITIONAL status:
   ```
   Overall Status: ⚠️ CONDITIONAL

   Article I (Simplicity): ⚠️ CONDITIONAL
   - Condition: Plan adds Redis as new framework
   - Justification: "Reduces API latency by 80% (2s → 400ms)"
   - Impact: Adds Redis dependency, increases infrastructure complexity

   ⚠️ CONDITIONAL APPROVAL NEEDED

   Review conditional items and decide:
   1. "approved" - Accept conditional items
   2. "fix simplicity" - Remove Redis, find simpler solution
   3. "explain article i" - Show simplicity principle details
   ```
2. User types: "approved"
3. metadata records:
   ```json
   {
     "approvals": {
       "planning": {
         "approved": true,
         "constitutional_status": "CONDITIONAL",
         "conditional_items": [
           "Article I: Redis added (justified by 80% latency reduction)"
         ]
       }
     }
   }
   ```

**Test Validation:**
- ✅ CONDITIONAL status shown
- ✅ Justification displayed
- ✅ Can approve with acknowledgment
- ✅ Conditional items recorded

---

### Test 3.3: FAIL Status - Cannot Approve

**Setup:**
```bash
/speckit.plan
# Plan uses jest.mock('auth0') in integration tests
# Phase -1 validation: ❌ FAIL (Article V violation)
```

**Expected Behavior:**
1. Gate shows FAIL status:
   ```
   Overall Status: ❌ FAIL

   Article V (Integration-First Testing): ❌ FAIL
   - Violation: Phase 2.5 uses jest.mock('auth0') in integration tests
   - Location: Phase 2.5 → Mock & Stub Strategy
   - Why This Matters: Violates Integration-First - auth must use real Auth0
   - Required Fix: Change to real Auth0 test tenant

   ⛔ CANNOT PROCEED - Plan violates constitutional principles.

   You must either:
   1. Fix violations and re-run /speckit.plan
   2. Update constitution.md if principles need revision

   Type "fix test strategy" to revise the plan.
   ```
2. User types: "approved" → ⛔ ERROR: "Cannot approve FAILED validation"
3. User types: "fix test strategy"
4. Revision applied, Phase -1 re-run → ✅ PASS
5. Gate shows updated result
6. User types: "approved"

**Test Validation:**
- ✅ FAIL status blocks approval
- ✅ "approved" rejected with error
- ✅ "fix" command works
- ✅ Re-validation after fix
- ✅ Can approve after fix

---

### Test 3.4: Explain Article

**User Response:**
```
explain article v
```

**Expected Behavior:**
1. Read constitution.md Article V section
2. Extract plan.md Phase -1 validation for Article V
3. Show explanation:
   ```
   📖 Constitutional Article Explanation

   **Article V: Integration-First Testing**

   [Full article text from constitution.md]

   Integration tests MUST use real dependencies (not mocks) for:
   - Authentication/Authorization logic
   - Database queries
   - Multi-tenant isolation

   Prohibited:
   - jest.mock('auth0') in integration tests
   - MockDatabase() for integration tests
   - mockTenantContext() for tenant isolation tests

   Acceptable Test Helpers:
   - Time mocking (Date.now())
   - Test data generation (Faker.js)
   - Email/SMS mocks (external services)

   ---

   **How This Plan Relates to Article V:**

   Status: ❌ FAIL

   Validation Details:
   - Phase 2.5 Mock & Stub Strategy contains: `jest.mock('auth0')`
   - This violates Article V prohibition on mocking auth in integration tests

   Why This Failed:
   Integration tests must validate real Auth0 integration, not mocked behavior

   How to Fix:
   - Change Phase 2.5 to use real Auth0 test tenant
   - Update Mock & Stub Strategy: "Auth0: ✅ Real test tenant in integration tests"
   ```

**Test Validation:**
- ✅ "explain" command recognized
- ✅ Constitution.md article displayed
- ✅ Plan validation result shown
- ✅ Fix suggestions provided

---

### Test 3.5: Show Constitution

**User Response:**
```
show constitution
```

**Expected Behavior:**
1. Check if constitution.md exists
2. IF found: Display full file
3. IF not found: Show graceful degradation message:
   ```
   ℹ️ No constitution.md found

   This project doesn't have a constitution file.
   Constitutional validation is running in PARTIAL mode:
   - TDD compliance checking
   - Simplicity validation
   - Integration-first testing (basic)
   - NO prohibited pattern detection (requires constitution)

   To enable full validation, create constitution via:
   /speckit.constitution

   Type 'approved' to proceed with PARTIAL validation level.
   ```

**Test Validation:**
- ✅ Constitution displayed if exists
- ✅ Graceful degradation message if missing
- ✅ Can approve with PARTIAL status

---

### Test 3.6: Implementation Pre-Flight Check

**Setup:**
```bash
# After approving plan
/speckit.implement
```

**Expected Behavior (Step 2.5):**
1. Read spec-metadata.json
2. Check planning.approved
3. Check constitutional_status
4. Show pre-flight summary:
   ```
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ✅ Pre-Flight Check: PASSED
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Specification: ✅ Approved (2025-11-11T14:30:00Z)
   Planning: ✅ Approved (2025-11-11T16:00:00Z)
   Constitutional Status: PASS

   Implementation can proceed.
   Constitutional re-validation will occur at Step 10.4.
   ```
5. Proceed to implementation phases

**Test Validation:**
- ✅ metadata read correctly
- ✅ Approval status validated
- ✅ Pre-flight summary shown
- ✅ Proceeds to implementation

---

### Test 3.7: Implementation Blocked (No Planning Approval)

**Setup:**
```bash
# User skips /speckit.plan or doesn't approve
/speckit.implement
```

**Expected Behavior (Step 2.5):**
1. Read spec-metadata.json
2. planning.approved == false
3. ⛔ ERROR:
   ```
   ⛔ Pre-Flight Check: FAILED

   Planning not approved.

   Last planning phase: not generated

   You must run /speckit.plan and approve before implementing.

   Cannot proceed to implementation.
   ```
4. Exit with error

**Test Validation:**
- ✅ Missing approval detected
- ✅ Error message shown
- ✅ Implementation blocked
- ✅ Exit without proceeding

---

## Test Suite 4: Configuration & Edge Cases

### Test 4.1: Legacy Mode

**Setup:**
```yaml
# .specify/config.yml
legacy_mode:
  enabled: true
```

**Expected Behavior:**
1. All discovery gates disabled
2. No A/B option in discovery
3. No specification review gate
4. No constitutional approval gate
5. Exact v2.6 behavior

**Test Validation:**
- ✅ /speckit.specify proceeds without gates
- ✅ /speckit.plan proceeds without approval
- ✅ metadata shows legacy_mode: true

---

### Test 4.2: Command-Line Override

**Setup:**
```bash
/speckit.specify --skip-gates "add authentication"
```

**Expected Behavior:**
1. Config ignored
2. All gates skipped
3. Fast execution
4. metadata shows skip_reason: "command flag"

**Test Validation:**
- ✅ Flag overrides config
- ✅ Gates skipped
- ✅ Metadata records flag usage

---

### Test 4.3: Empty Response Handling

**Setup:**
```bash
/speckit.specify "add auth"
# Review gate appears
# User presses Enter (empty response)
```

**Expected Behavior:**
1. Empty response detected
2. Show clarification:
   ```
   ❓ I didn't understand that response.

   Please respond with one of:
   - 'approved' - Accept specification
   - 'revise [aspect]' - Make changes
   - 'clarify [question]' - Ask details
   - 'show full spec' - View complete spec

   Your response:
   ```
3. Wait for valid response

**Test Validation:**
- ✅ Empty response handled gracefully
- ✅ Clarification shown
- ✅ Returns to approval loop

---

### Test 4.4: Timeout in Non-Interactive Mode

**Setup:**
```yaml
# .specify/config.yml
discovery_gates:
  approval_timeout_minutes: 1
```

```bash
/speckit.specify "add auth"
# Wait 61 seconds without responding
```

**Expected Behavior:**
1. Approval gate waits for 60 seconds
2. After timeout:
   ```
   ⛔ ERROR: Approval timeout (non-interactive mode)

   No response received within 1 minute(s).

   For non-interactive environments, use:
   - SPECKIT_AUTO_APPROVE=true
   - --auto-approve flag
   - config: auto_approve: true
   ```
3. Exit with error code 1

**Test Validation:**
- ✅ Timeout enforced
- ✅ Error message shown
- ✅ Exit code 1

---

### Test 4.5: Multiple Revisions

**Setup:**
```bash
/speckit.specify "add auth"
# Review gate
# User: "revise token storage to Redis"
# Gate returns with update
# User: "revise session duration to 1 hour"
# Gate returns with update
# User: "approved"
```

**Expected Behavior:**
1. First revision applied, revision_count = 1
2. Second revision applied, revision_count = 2
3. metadata shows:
   ```json
   {
     "approvals": {
       "specification": {
         "revisions_requested": 2,
         "revision_history": [
           {"aspect": "token storage", "change": "..."},
           {"aspect": "session duration", "change": "..."}
         ]
       }
     }
   }
   ```

**Test Validation:**
- ✅ Multiple revisions allowed
- ✅ Counter increments correctly
- ✅ History tracked
- ✅ Can approve after multiple revisions

---

## Test Suite 5: Integration Tests (End-to-End)

### Test 5.1: Full Workflow with All Gates

**Scenario:** User goes through complete workflow with discovery and approvals

**Steps:**
```bash
# 1. Specify with discovery
/speckit.specify "add OAuth2 authentication"
→ Domain detected: authentication
→ Choose B (discovery)
→ Answer 4 auth questions
→ Spec generated
→ Review gate: "approved"

# 2. Validate gap (optional)
/speckit.validate-gap
→ Gap analysis complete

# 3. Plan with constitutional approval
/speckit.plan
→ Phase -1: ⚠️ CONDITIONAL (adds Redis)
→ Constitutional gate: "approved" (accept Redis)

# 4. Implement with pre-flight check
/speckit.implement
→ Pre-flight: ✅ PASS (all approvals present)
→ Implementation proceeds
```

**Test Validation:**
- ✅ Discovery insights flow to spec
- ✅ Spec approval recorded
- ✅ Planning approval with conditional
- ✅ Pre-flight validates approvals
- ✅ Complete audit trail in metadata

---

### Test 5.2: Blocked Workflow (Missing Approvals)

**Scenario:** User tries to skip steps

**Steps:**
```bash
# 1. Specify without approval
/speckit.specify "add auth"
→ Review gate: "skip" → "yes" (forced skip)

# 2. Try to plan
/speckit.plan
→ ⚠️ WARNING: "Specification not properly approved"
→ Continue anyway

# 3. Try to implement
/speckit.implement
→ ⛔ ERROR: "Planning not approved. Cannot implement."
→ EXIT

# 4. Go back and approve
/speckit.plan
→ Constitutional gate: "approved"

# 5. Try implement again
/speckit.implement
→ ✅ Pre-flight: PASS
→ Implementation proceeds
```

**Test Validation:**
- ✅ Warnings shown for skipped approvals
- ✅ Implementation blocked without planning approval
- ✅ Can recover by going back and approving
- ✅ Pre-flight validates correctly after approval

---

### Test 5.3: CI/CD Automation

**Scenario:** Fully automated pipeline

**Setup:**
```bash
export SPECKIT_AUTO_APPROVE=true
export SPECKIT_DISCOVERY_MODE=disabled
```

**Steps:**
```bash
/speckit.specify "add authentication from ticket-123"
→ Discovery skipped
→ Spec generated
→ Review gate auto-approved

/speckit.plan
→ Planning complete
→ Constitutional gate auto-approved

/speckit.implement
→ Pre-flight: PASS
→ Implementation proceeds
```

**Test Validation:**
- ✅ All gates auto-approved
- ✅ No user interaction required
- ✅ Warnings logged
- ✅ metadata shows auto_approved flags

---

## Test Execution Checklist

### Pre-Test Setup
- [ ] Install spec-kit v2.7
- [ ] Create test project directory
- [ ] Initialize git repository
- [ ] Copy spec-kit commands to test project
- [ ] Create .specify/config.yml with test settings

### Test Categories
- [ ] Test Suite 1: Pre-Spec Discovery (Tests 1.1-1.7)
- [ ] Test Suite 2: Specification Review Gate (Tests 2.1-2.7)
- [ ] Test Suite 3: Constitutional Approval Gate (Tests 3.1-3.7)
- [ ] Test Suite 4: Configuration & Edge Cases (Tests 4.1-4.5)
- [ ] Test Suite 5: Integration Tests (Tests 5.1-5.3)

### Validation Criteria
- [ ] All approval gates block when expected
- [ ] User responses handled correctly
- [ ] metadata.json updated accurately
- [ ] Configuration overrides work
- [ ] Environment variables honored
- [ ] Command-line flags override config
- [ ] Token costs within expected range (+500-900)
- [ ] Backward compatibility preserved (legacy mode)

### Performance Tests
- [ ] Discovery adds <400 tokens per feature
- [ ] Review gate adds <500 tokens per feature
- [ ] Constitutional gate adds <200 tokens per feature
- [ ] Total overhead: 500-900 tokens (5-10%)

### User Experience Tests
- [ ] Gate prompts are clear and actionable
- [ ] Error messages are helpful
- [ ] Revision flow is intuitive
- [ ] Approval process feels smooth
- [ ] Timeout messages are informative

---

## Test Report Template

```markdown
# Discovery Interview Gates Test Report
# Date: [YYYY-MM-DD]
# Tester: [Name]
# Version: v2.7.0

## Summary
- Tests Executed: [N]
- Tests Passed: [N]
- Tests Failed: [N]
- Pass Rate: [X]%

## Detailed Results

### Test Suite 1: Pre-Spec Discovery
- Test 1.1: [✅ PASS | ❌ FAIL] - [Notes]
- Test 1.2: [✅ PASS | ❌ FAIL] - [Notes]
[...]

### Test Suite 2: Specification Review Gate
[...]

### Test Suite 3: Constitutional Approval Gate
[...]

### Test Suite 4: Configuration & Edge Cases
[...]

### Test Suite 5: Integration Tests
[...]

## Issues Found
1. [Issue description]
   - Severity: [HIGH/MEDIUM/LOW]
   - Steps to reproduce: [...]
   - Expected: [...]
   - Actual: [...]

2. [...]

## Recommendations
- [Improvement suggestion 1]
- [Improvement suggestion 2]

## Overall Assessment
[PASS | FAIL | NEEDS WORK]

[Summary paragraph]
```

---

**END OF TEST CASES**

These test cases validate all three integration points and edge cases.
Run these tests before releasing v2.7.0 to ensure quality and correctness.
