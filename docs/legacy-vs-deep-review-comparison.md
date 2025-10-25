# Legacy vs Deep Review: Side-by-Side Comparison

This document shows concrete examples of how the legacy clarify command compares to the deep review mode for identical specifications.

---

## Example 1: Simple CRUD Feature

### Spec: User Profile Management

**Complexity:** Low
- 8 functional requirements
- No complex features
- Standard CRUD operations

---

### Legacy Clarify Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Taxonomy Scan Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Functional Scope: Clear
Domain & Data Model: Partial
Interaction & UX: Clear
Non-Functional: Missing
Integration: Clear
Edge Cases: Missing
Constraints: Clear

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 1 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What fields should the user profile include?

Format: Short answer (<=5 words)
>
```

**Questions Asked:**
1. What fields should user profile include?
2. Can users delete their profiles?
3. What validation is required for profile updates?
4. Should profile changes be audited?
5. What happens when profile data is missing?

**Time:** 45 seconds
**Tokens:** ~2,100
**Issues Caught:** None explicitly identified

---

### Deep Review Output (Fast Mode)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Deep Review Mode: FAST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Rationale: Simple CRUD feature, <10 requirements
Analysis: 10 thoughts (~5,000 tokens)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Key Findings
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Logic Issues: None
Completeness Gaps: 2 (profile picture storage, validation rules)
Optimization: 1 (profile deletion vs soft-delete)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 1 of 3 (reduced from 5 due to simplicity)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Question**: Should user profile deletion be permanent or soft-delete?

**Recommended:** Option A - Soft delete (mark as deleted, retain data)

Reasoning: Allows account recovery, supports audit compliance, prevents
           accidental data loss. Standard pattern for user-facing features.

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A** | Soft delete (mark deleted, retain data) | **Pros:** Recoverable, audit trail, compliance-friendly<br>**Cons:** Database growth, cleanup needed |
| **B** | Permanent delete (remove immediately) | **Pros:** Clean database, true privacy<br>**Cons:** Irreversible, no audit trail |
| **C** | Delayed delete (30-day grace period) | **Pros:** User-friendly, recoverable, clean eventual<br>**Cons:** Most complex, requires scheduled job |

>
```

**Questions Asked:**
1. Should profile deletion be permanent or soft-delete? (with trade-offs)
2. Where should profile pictures be stored and what size limits apply?
3. What validation strategy for email/phone uniqueness?

**Time:** 55 seconds
**Tokens:** ~5,200
**Issues Caught:**
- Soft-delete consideration (optimization)
- Storage strategy for images (completeness)
- Validation approach (completeness)

**Verdict:** For simple CRUD, deep review adds marginal value. Fast mode acceptable, but legacy is sufficient.

---

## Example 2: Authentication Feature

### Spec: Multi-Factor Authentication

**Complexity:** Medium-High
- 16 functional requirements
- Security-critical domain
- Multiple auth methods

---

### Legacy Clarify Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 1 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What MFA methods should be supported?

Format: Short answer (<=5 words)
> SMS and authenticator app

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 2 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

How should MFA codes be delivered?

Format: Short answer (<=5 words)
> SMS via Twilio

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 3 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Is MFA required or optional?

Format: Short answer (<=5 words)
> Required for all users

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 4 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

How long should MFA codes remain valid?

Format: Short answer (<=5 words)
> 5 minutes

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 5 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What happens if user loses MFA device?

Format: Short answer (<=5 words)
> Admin reset required
```

**Questions Asked:** 5 basic questions
**Time:** 90 seconds
**Tokens:** ~2,300
**Issues Caught:** None explicitly

**Problems NOT Caught:**
- SMS security concerns (NIST deprecation)
- Rate limiting for MFA attempts
- Backup codes for device loss
- Session handling after MFA
- MFA enrollment flow unclear

---

### Deep Review Output (Balanced Mode)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Deep Review Mode: BALANCED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Rationale: Authentication domain detected + 16 requirements
Analysis: 18 thoughts (~9,800 tokens)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Key Findings
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Logic Issues: 1
- Requirement R-AUTH-007 (MFA required) conflicts with R-AUTH-012
  (user can disable MFA). Which takes precedence?

Completeness Gaps: 5
- Rate limiting for MFA attempts not specified (security risk)
- Backup codes/recovery mechanism missing (usability risk)
- Session handling after MFA verification unclear
- MFA enrollment flow undefined
- Security logging requirements not specified

Optimization Opportunities: 2
- SMS-based MFA flagged (NIST deprecated due to SIM-swap attacks)
  → Recommend TOTP authenticator apps + WebAuthn as alternatives
- Custom MFA implementation suggested
  → Consider Auth0/Okta with built-in MFA (faster, more secure)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 1 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category: Security Architecture
Impact: Security, Implementation Effort, Maintenance
Rationale: Spec mentions SMS MFA, but NIST SP 800-63B deprecates SMS due to
           SIM-swap vulnerability. Deep review analysis flagged this as security risk.

**Question**: Which MFA methods should be supported?

**Recommended:** Option C - TOTP + WebAuthn (no SMS)

Reasoning: TOTP (authenticator apps) is NIST-approved and widely adopted.
           WebAuthn (hardware keys) provides strongest security. SMS is vulnerable
           to SIM-swap attacks. Modern pattern: Auth0, GitHub, AWS all moving
           away from SMS-only MFA.

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A** | SMS only (via Twilio) | **Pros:** Easiest for users, no app install<br>**Cons:** NIST deprecated, SIM-swap risk, SMS costs, phishing vulnerable |
| **B** | SMS + TOTP (authenticator apps) | **Pros:** User choice, backward compatible<br>**Cons:** SMS security risk remains, more complex |
| **C** | TOTP + WebAuthn (no SMS) | **Pros:** NIST compliant, strongest security, no SMS costs<br>**Cons:** Requires user education, hardware key optional |
| **D** | Managed provider (Auth0/Okta MFA) | **Pros:** All methods included, proven security, fast implementation<br>**Cons:** Monthly cost, vendor dependency |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 2 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category: Policy Enforcement
Impact: User Experience, Security Posture
Rationale: Spec contains contradiction - R-AUTH-007 says "MFA required for all users"
           but R-AUTH-012 says "Users can disable MFA in settings". This blocks
           implementation decisions.

**Question**: Should MFA be mandatory or optional?

**Recommended:** Option B - Mandatory for privileged users, optional for standard users

Reasoning: Balances security and usability. Forces admins/elevated accounts to use MFA
           (reduces attack surface), while allowing flexibility for standard users.
           Industry pattern: Salesforce, Slack, GitHub all use role-based MFA.

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A** | Mandatory for all users | **Pros:** Highest security, consistent policy<br>**Cons:** User resistance, support burden, onboarding friction |
| **B** | Role-based (mandatory for admins/privileged) | **Pros:** Balances security + usability, reduces attack surface<br>**Cons:** More complex policy enforcement |
| **C** | Optional for all users | **Pros:** User-friendly, low friction<br>**Cons:** Weakest security, vulnerable accounts |
| **D** | Conditional (based on risk signals) | **Pros:** Smart security, minimal friction<br>**Cons:** Complex implementation, requires risk engine |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 3 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category: Account Recovery
Impact: User Experience, Support Burden, Security
Rationale: Deep analysis identified missing recovery mechanism. If user loses
           MFA device, how do they regain access? Current spec says "admin reset"
           but this doesn't scale.

**Question**: How should users recover access if they lose their MFA device?

**Recommended:** Option C - Backup codes + admin reset fallback

Reasoning: Backup codes (10-digit one-time codes) are industry standard recovery
           mechanism. Allows self-service recovery while maintaining security.
           Admin reset as fallback for lost backup codes. Used by Google, GitHub,
           Microsoft, Dropbox.

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A** | Admin reset only | **Pros:** Simple, centralized control<br>**Cons:** Support burden, slow recovery, doesn't scale, user frustration |
| **B** | Email-based recovery (send temporary code) | **Pros:** Self-service, fast recovery<br>**Cons:** Defeats MFA purpose (email account compromise), security risk |
| **C** | Backup codes + admin reset fallback | **Pros:** Self-service, secure, industry standard, scales<br>**Cons:** Users must store codes securely, education needed |
| **D** | SMS recovery code to alternate number | **Pros:** Self-service, familiar pattern<br>**Cons:** Still vulnerable to SIM-swap, requires storing alternate number |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 4 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category: Security Policy
Impact: Security, User Experience, Attack Prevention
Rationale: Rate limiting for MFA attempts not specified in spec. This is critical
           for preventing brute-force attacks on 6-digit TOTP codes (1M combinations).

**Question**: What rate limiting should apply to MFA verification attempts?

**Recommended:** Option B - 5 attempts per 15 minutes, then temporary lockout

Reasoning: Balances security and usability. Prevents brute-force while allowing
           for user typos. OWASP recommendation: 3-5 attempts before lockout.
           15-minute lockout is standard (Auth0, Okta, AWS).

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A** | 3 attempts per hour, then account lockout | **Pros:** Strongest security, prevents brute-force<br>**Cons:** Users frequently locked out, support burden |
| **B** | 5 attempts per 15 minutes, temporary lockout | **Pros:** Balanced, OWASP-aligned, user-friendly<br>**Cons:** Slightly higher brute-force risk than option A |
| **C** | 10 attempts per hour, progressive delays | **Pros:** Most user-friendly, self-service recovery<br>**Cons:** Weaker security, longer brute-force window |
| Short | Provide different answer (<=5 words) | N/A |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Question 5 of 5
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Category: User Onboarding
Impact: User Experience, Security Adoption
Rationale: MFA enrollment flow not defined in spec. When/how do users set up MFA?
           Forced during first login? Optional? This affects implementation and UX.

**Question**: When should users be prompted to enroll in MFA?

**Recommended:** Option C - Forced enrollment for privileged users, optional prompt for others

Reasoning: Ensures critical accounts protected immediately, while allowing
           flexibility for standard users. Soft prompt encourages adoption without
           forcing. Aligns with role-based enforcement from Q2.

| Option | Description | Trade-offs |
|--------|-------------|------------|
| **A** | Forced enrollment during first login (all users) | **Pros:** Immediate security, high adoption<br>**Cons:** Onboarding friction, support burden, user resistance |
| **B** | Optional prompt (can skip) | **Pros:** Low friction, user choice<br>**Cons:** Low adoption, users skip indefinitely |
| **C** | Role-based (forced for privileged, prompt for others) | **Pros:** Balanced, protects critical accounts, encourages adoption<br>**Cons:** More complex enrollment logic |
| **D** | Incentivized (offer benefits for enrollment) | **Pros:** Positive framing, voluntary adoption<br>**Cons:** Lower adoption, delayed security, requires incentive design |

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Deep Review Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Questions Asked: 5/5
Token Usage: ~9,800 tokens
Time: 95 seconds

Issues Resolved:
✓ MFA method security concern (SMS deprecated)
✓ Policy contradiction (mandatory vs optional)
✓ Missing recovery mechanism
✓ Missing rate limiting (security gap)
✓ Missing enrollment flow

Optimization Implemented:
✓ TOTP + WebAuthn recommended over SMS
✓ Role-based enforcement suggested (better than binary)
✓ Backup codes standard recovery pattern

Remaining Considerations:
- Security logging requirements (defer to planning)
- Session management details (defer to planning)
- MFA remember-me device option (defer to planning)
```

**Questions Asked:** 5 high-impact questions
**Time:** 95 seconds
**Tokens:** ~9,800 tokens
**Issues Caught:**
- SMS security vulnerability (optimization)
- Policy contradiction (logic issue)
- Missing recovery mechanism (completeness)
- Missing rate limiting (security gap)
- Missing enrollment flow (completeness)

**Verdict:** **Deep review adds significant value** for authentication features. Catches security issues legacy missed.

---

## Example 3: Payment Integration

### Spec: Subscription Billing

**Complexity:** High
- 22 functional requirements
- Payment domain (critical)
- External integrations

---

### Legacy Clarify (Abbreviated)

**Questions:**
1. What payment gateway should be used? → Stripe
2. What subscription tiers are available? → Basic, Pro, Enterprise
3. How should failed payments be handled? → Retry 3 times
4. Are refunds supported? → Yes
5. What currency should be supported? → USD only

**Issues NOT Caught:**
- Idempotency for payment operations
- Webhook security (signature validation)
- PCI compliance requirements
- Subscription proration on upgrades
- Dunning management for failed payments
- Tax calculation requirements

**Time:** 75 seconds
**Tokens:** ~2,400

---

### Deep Review (Thorough Mode)

**Key Findings:**
- **Logic Issues (2)**:
  - Spec says "Stripe" but also mentions "payment gateway abstraction" - which is it?
  - Requirement R-BILL-009 (immediate charge) conflicts with R-BILL-014 (trial period)

- **Completeness Gaps (7)**:
  - Idempotency requirements missing (critical for payments)
  - Webhook security not specified
  - PCI compliance scope unclear
  - Subscription proration strategy undefined
  - Dunning management (failed payment retries) vague
  - Tax calculation requirements missing (sales tax, VAT)
  - Invoice generation requirements not specified

- **Optimization Opportunities (3)**:
  - Custom payment implementation suggested → Recommend Stripe Billing (handles subscriptions, invoices, dunning, tax)
  - Manual retry logic mentioned → Stripe Smart Retries handle this automatically
  - Building invoice system → Stripe generates invoices automatically

**Questions:**
1. **Should payment implementation be custom or use Stripe Billing?**
   - Recommended: Stripe Billing (handles subscriptions, invoices, tax, dunning automatically)
   - Saves 4-6 weeks development, PCI compliant, battle-tested

2. **How should payment idempotency be guaranteed?**
   - Recommended: Idempotency keys (Stripe native)
   - Prevents double-charging, required for reliable payments

3. **How should webhook security be verified?**
   - Recommended: Signature validation (Stripe webhook signatures)
   - Prevents webhook spoofing attacks

4. **What subscription proration strategy on upgrades/downgrades?**
   - Recommended: Automatic proration (Stripe default)
   - Industry standard, user-friendly

5. **How should failed payment dunning be managed?**
   - Recommended: Stripe Smart Retries + email notifications
   - Automated retry timing, reduces churn

**Token Usage:** ~14,500 tokens
**Time:** 155 seconds

**Issues Caught:**
- Logic contradiction (trial vs immediate charge)
- Gateway abstraction confusion
- All 7 completeness gaps identified
- Optimization: Stripe Billing saves 4-6 weeks dev time
- Security: Idempotency + webhook validation required

**Verdict:** **Deep review critical** for payment features. Caught security gaps and major optimization (Stripe Billing saves massive effort).

---

## Summary Comparison Table

| Feature Complexity | Legacy Clarify | Deep Review | Recommendation |
|-------------------|----------------|-------------|----------------|
| **Simple CRUD** (Example 1) | ✅ Adequate | ⚠️ Slight improvement | Use legacy or fast mode (cost-effective) |
| **Auth/Security** (Example 2) | ⚠️ Misses security issues | ✅ Catches critical gaps | Use balanced or thorough mode |
| **Payment/Critical** (Example 3) | ❌ Misses many gaps | ✅ Comprehensive analysis | Use thorough mode (essential) |

---

## ROI Analysis by Complexity

### Simple CRUD (Example 1)

| Metric | Legacy | Deep Review (Fast) | Delta |
|--------|--------|-------------------|-------|
| Token Cost | 2,100 | 5,200 | +3,100 |
| Time | 45s | 55s | +10s |
| Issues Caught | 0 | 3 (minor) | +3 |
| Estimated Rework Saved | 0 tokens | ~2,000 tokens | +2,000 |
| **Net ROI** | N/A | **-1,100 tokens (loss)** | ❌ Not worth it |

**Verdict:** For simple CRUD, deep review costs more than it saves. Use legacy.

---

### Authentication (Example 2)

| Metric | Legacy | Deep Review (Balanced) | Delta |
|--------|--------|------------------------|-------|
| Token Cost | 2,300 | 9,800 | +7,500 |
| Time | 90s | 95s | +5s |
| Issues Caught | 0 | 5 (critical) | +5 |
| Estimated Rework Saved | 0 tokens | ~25,000 tokens | +25,000 |
| **Net ROI** | N/A | **+17,500 tokens (3.3x)** | ✅ Strong ROI |

**Breakdown of Rework Saved:**
- SMS security refactor avoided: ~8,000 tokens
- Policy contradiction fixed early: ~5,000 tokens
- Recovery mechanism designed upfront: ~6,000 tokens
- Rate limiting designed vs retrofitted: ~3,000 tokens
- Enrollment flow designed upfront: ~3,000 tokens

**Verdict:** For auth features, deep review saves 3x its cost. Essential.

---

### Payment Integration (Example 3)

| Metric | Legacy | Deep Review (Thorough) | Delta |
|--------|--------|------------------------|-------|
| Token Cost | 2,400 | 14,500 | +12,100 |
| Time | 75s | 155s | +80s |
| Issues Caught | 0 | 12 (9 critical) | +12 |
| Estimated Rework Saved | 0 tokens | ~45,000 tokens | +45,000 |
| Development Time Saved | 0 | 4-6 weeks (Stripe Billing) | Massive |
| **Net ROI** | N/A | **+32,900 tokens (3.7x) + weeks saved** | ✅ Exceptional ROI |

**Breakdown of Rework Saved:**
- Idempotency design vs retrofit: ~8,000 tokens
- Webhook security added late: ~6,000 tokens
- Proration logic missing: ~7,000 tokens
- Dunning management undefined: ~8,000 tokens
- Tax calculation missing: ~6,000 tokens
- Invoice system avoided (Stripe Billing): ~10,000 tokens

**Additional Value:**
- Stripe Billing recommendation saves 4-6 weeks custom development
- PCI compliance simpler (Stripe-hosted)
- Ongoing maintenance reduced

**Verdict:** For payment features, deep review is **mission-critical**. Saves months of work.

---

## Decision Matrix

Use this to decide which mode for your feature:

```
┌─────────────────────────────────────────────────────────────────┐
│ Feature Complexity Decision Tree                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ START: What type of feature are you building?                  │
│                                                                 │
│ ┌─ Simple CRUD (read/write data, no business logic)           │
│ │  Examples: User profiles, settings, basic lists             │
│ │  → Use: LEGACY or FAST mode                                 │
│ │  → Why: Deep review adds minimal value                      │
│ │                                                              │
│ ┌─ Standard Features (moderate logic, standard patterns)      │
│ │  Examples: Search, filtering, notifications                 │
│ │  → Use: FAST or BALANCED mode                               │
│ │  → Why: Some optimization value, moderate cost              │
│ │                                                              │
│ ┌─ Authentication / Authorization                             │
│ │  Examples: Login, SSO, MFA, permissions                     │
│ │  → Use: BALANCED or THOROUGH mode                           │
│ │  → Why: Security issues expensive, deep review essential    │
│ │                                                              │
│ ┌─ Payment / Billing                                          │
│ │  Examples: Checkout, subscriptions, invoices                │
│ │  → Use: THOROUGH mode                                       │
│ │  → Why: Mistakes costly, compliance critical, optimization  │
│ │          suggestions (like Stripe Billing) save weeks       │
│ │                                                              │
│ ┌─ Multi-Tenant / Data Isolation                              │
│ │  Examples: Tenant management, resource scoping              │
│ │  → Use: BALANCED or THOROUGH mode                           │
│ │  → Why: Isolation bugs catastrophic, architecture critical  │
│ │                                                              │
│ ┌─ Real-Time / Distributed Systems                            │
│ │  Examples: WebSocket, event streaming, sync                 │
│ │  → Use: BALANCED or THOROUGH mode                           │
│ │  → Why: Edge cases complex, failure modes critical          │
│ │                                                              │
│ ┌─ External Integrations                                      │
│ │  Examples: APIs, webhooks, third-party services             │
│ │  → Use: BALANCED or THOROUGH mode                           │
│ │  → Why: Failure modes, rate limits, security critical       │
│ │                                                              │
│ ┌─ Compliance / Audit                                         │
│ │  Examples: GDPR, HIPAA, SOC 2, audit logs                  │
│ │  → Use: THOROUGH mode                                       │
│ │  → Why: Compliance mistakes expensive, requirements complex │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

### When Legacy is Sufficient

✅ **Use legacy clarify** when:
- Simple CRUD operations
- <10 requirements
- Standard patterns with no security/compliance concerns
- Prototypes or spikes (throw-away code)
- Time-critical (need results in <1 minute)
- Token budget very constrained

---

### When Deep Review Pays Off

✅ **Use deep review (balanced/thorough)** when:
- Authentication, authorization, payment, multi-tenant domains
- >15 requirements
- Security-critical or compliance-regulated
- Novel approaches or first-time implementations
- External integrations with failure modes
- Optimization suggestions could save significant effort

---

### The ROI Sweet Spot

**Deep review delivers best ROI for:**
1. **Security-critical features** (auth, payment)
   - ROI: 3-4x tokens saved
   - Catches vulnerabilities early

2. **Complex integrations** (external APIs, multi-tenant)
   - ROI: 2-3x tokens saved
   - Identifies missing edge cases

3. **Features with optimization potential** (custom vs managed services)
   - ROI: Can save weeks of development (e.g., Stripe Billing)
   - Suggests alternatives you may not have considered

---

## Conclusion

The comparison shows that **deep review value scales with feature complexity**:

- **Simple features**: Marginal value, use legacy
- **Medium complexity**: Moderate value, use balanced mode
- **High complexity/critical domains**: Exceptional value, use thorough mode

The key insight: **Deep review isn't about asking more questions—it's about asking better questions** that catch issues legacy scanning misses.

---

**Last Updated:** 2025-10-24
**Version:** 1.0.0
