# Integration Point 1: Enhanced Pre-Spec Discovery
# File: src/.claude/commands/speckit.specify.md
# Location: Replace lines 22-167 (existing Step 1)

---

1. **Vagueness Detection & Pre-Spec Discovery** (Adaptive Dialogue + Domain-Specific Templates):

   **Purpose**: If the feature description is too high-level or vague, engage in brief discovery dialogue BEFORE generating the spec to build better understanding and prevent assumption-heavy specifications.

   **Step 1.1: Vagueness Detection Heuristics**

   Evaluate the user's feature description against these criteria:

   - **Length**: < 30 words (insufficient detail)
   - **Uncertainty markers**: Contains "maybe", "thinking about", "not sure", "possibly", "could", "might"
   - **Generic terms without specifics**:
     - "dashboard" (without saying what data/metrics)
     - "authentication" (without specifying type: OAuth, JWT, sessions)
     - "API" (without endpoints or purpose)
     - "admin panel" (without specific admin functions)
     - "performance" (without target metrics or bottleneck identification)
     - "caching" (without layer specification)
     - "search" (without scope or technology)
   - **Lack of concrete verbs**: "add something", "make it better", "improve"
   - **Missing context**: Who/what/why/when are unclear

   **Step 1.2: Domain Detection**

   Analyze feature description to identify domain category:

   ```yaml
   # Domain Detection Logic (run before vagueness check)

   DOMAIN_KEYWORDS = {
     "authentication": ["auth", "login", "logout", "signin", "signup", "password", "credential",
                       "oauth", "saml", "jwt", "token", "session", "sso", "2fa", "mfa"],

     "data": ["database", "model", "entity", "crud", "create", "read", "update", "delete",
              "schema", "migration", "table", "record", "data", "storage"],

     "ui": ["dashboard", "interface", "ui", "ux", "page", "view", "component", "form",
            "button", "modal", "dialog", "layout", "navigation", "menu", "widget"],

     "api": ["api", "endpoint", "rest", "graphql", "grpc", "webhook", "integration",
             "service", "http", "request", "response", "middleware"],

     "performance": ["performance", "optimize", "speed", "slow", "cache", "caching",
                     "latency", "throughput", "bottleneck", "scaling"],
   }

   # Check each domain
   detected_domains = []
   description_lower = feature_description.lower()

   for domain, keywords in DOMAIN_KEYWORDS:
     for keyword in keywords:
       if keyword in description_lower:
         if domain not in detected_domains:
           detected_domains.append(domain)
         break

   # Primary domain is the first detected (or "generic" if none)
   primary_domain = detected_domains[0] if detected_domains else "generic"
   ```

   **Step 1.3: Decision Flow**

   ```yaml
   IF description meets ANY vagueness criteria OR primary_domain != "generic":

     # Show discovery option with domain context
     IF primary_domain == "authentication":
       recommendation = "B (auth features benefit from specific flow/storage decisions)"
     ELSE IF primary_domain == "data":
       recommendation = "B (data models need relationship and retention clarity)"
     ELSE IF primary_domain == "ui":
       recommendation = "B (UI features need UX priority and accessibility decisions)"
     ELSE IF primary_domain == "api":
       recommendation = "B (API features need style and versioning decisions)"
     ELSE IF primary_domain == "performance":
       recommendation = "B (performance features need target metrics and bottleneck identification)"
     ELSE:
       recommendation = "B (better spec with discovery)"

     Present discovery option to user:

     "🤔 Your description seems high-level [IF domain detected: and relates to {primary_domain}].

     To create a better spec, I can:

     **A) Generate spec now** (fast, ~1 min, may include assumptions)
     **B) Ask 3-5 {primary_domain}-specific questions first** (better spec, ~2-3 min)

     Type 'A' for fast generation, 'B' for discovery, or just 'proceed' to skip.

     Recommendation: Choose {recommendation}"

     Wait for user response.

     IF user responds with "A", "fast", "skip", or "proceed":
       → Skip discovery, proceed to Step 2 (short name generation)
       → Note: Will make informed assumptions based on best practices

     ELSE IF user responds with "B", "discovery", "questions", or "yes":
       → Run domain-specific discovery dialogue (Step 1.4)

     ELSE (unclear response):
       → "I didn't understand. Type 'A' for fast generation or 'B' for discovery questions."
       → Wait for clearer response

   ELSE (description is detailed and specific):
     → Skip detection entirely, proceed directly to Step 2 (short name generation)
   ```

   **Step 1.4: Domain-Specific Discovery Dialogue**

   Run discovery based on detected primary domain:

   ---

   ### 🔐 AUTHENTICATION DOMAIN TEMPLATE

   **Triggered by:** "auth", "login", "oauth", "jwt", "session", etc.

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   🔐 Authentication Feature Discovery
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   I'll ask 4-5 questions to understand your authentication requirements.
   ```

   **Question 1 (MANDATORY): Authentication Method**

   ```markdown
   ❓ Question 1 of 4: Authentication Method

   Which authentication approach should the system use?

   **Recommended:** Option A (OAuth 2.0) for modern user-facing apps, Option B (JWT) for API-first apps

   | Option | Method | Best For | Security Level |
   |--------|--------|----------|----------------|
   | A | OAuth 2.0 with external provider (Auth0, Okta) | Modern apps with SSO | High |
   | B | JWT with custom auth service | API-first apps, microservices | Medium-High |
   | C | Session-based (traditional cookies) | Server-side rendered apps | Medium |
   | D | API keys | Service-to-service communication | Low-Medium |
   | Short | Provide different method (<=10 words) | | |

   You can reply with the option letter (e.g., "A"), accept the recommendation ("yes" or "recommended"), or provide your own answer.

   Your choice:
   ```

   **Wait for answer, validate, then proceed to Question 2**

   **Question 2 (MANDATORY): Token/Session Storage**

   ```markdown
   ❓ Question 2 of 4: Token/Session Storage

   [IF user chose OAuth or JWT:]
   Where should authentication tokens be stored on the client?

   [IF user chose Session-based:]
   Where should session identifiers be stored?

   **Recommended:** Option A (httpOnly cookies) - most secure against XSS attacks

   | Option | Storage Location | Security | Best For |
   |--------|------------------|----------|----------|
   | A | httpOnly cookies | Highest (XSS-proof) | Web applications |
   | B | localStorage | Medium (XSS vulnerable) | SPAs with caution |
   | C | sessionStorage | Medium (tab-scoped) | Temporary sessions |
   | D | In-memory only | Highest (no persistence) | High-security apps |
   | Short | Specify other storage (<=10 words) | | |

   Your choice:
   ```

   **Wait for answer, validate, then proceed to Question 3**

   **Question 3 (MANDATORY): Session Duration & Refresh**

   ```markdown
   ❓ Question 3 of 4: Session Duration & Refresh

   [IF OAuth or JWT:]
   What token lifespan and refresh strategy should be used?

   [IF Session-based:]
   What session timeout and renewal strategy?

   **Recommended:** Option A (short-lived with refresh) - best security/UX balance

   | Option | Strategy | Access Token/Session | Refresh Token | Use Case |
   |--------|----------|---------------------|---------------|----------|
   | A | Short-lived + refresh | 15 minutes | 7 days | Modern apps (best practice) |
   | B | Medium-lived | 24 hours | Optional | Moderate security |
   | C | Long-lived | 30 days | None | Low-security, convenience |
   | D | Custom | Specify: ___ | Specify: ___ | Specific requirements |

   Your choice:
   ```

   **Wait for answer, validate, then proceed to Question 4**

   **Question 4 (CONDITIONAL): Multi-Tenancy**

   ```markdown
   ❓ Question 4 of 4: Multi-Tenancy Support

   Does this authentication system need to support multiple organizations/tenants?

   **Context:** Multi-tenancy affects token structure, user isolation, and database design.

   | Option | Tenancy Model | Isolation | Complexity |
   |--------|---------------|-----------|------------|
   | A | Single-tenant | N/A (one org only) | Simple |
   | B | Multi-tenant (shared database) | Row-level security via tenant_id | Medium |
   | C | Multi-tenant (separate databases) | Database per tenant | High |
   | D | Hybrid | Shared for small, dedicated for large | Very High |
   | Short | Describe tenancy needs (<=10 words) | | |

   **Recommended:** Option A for new apps (can evolve to B later)

   Your choice:
   ```

   **Wait for answer, validate, then OPTIONAL Question 5**

   **Question 5 (OPTIONAL): Security Requirements**

   Only ask if user chose OAuth or described "high security" in description:

   ```markdown
   ❓ Question 5 of 5: Additional Security Requirements (Optional)

   Are any of these security features required?

   | Option | Feature | Impact |
   |--------|---------|--------|
   | A | Multi-factor authentication (MFA/2FA) | Higher security, adds setup flow |
   | B | Role-based access control (RBAC) | Permissions per role |
   | C | Single sign-on (SSO) across apps | Unified login experience |
   | D | None (basic auth only) | Simpler implementation |
   | E | Multiple (specify which) | __ |

   Your choice (or type "skip" to skip):
   ```

   **After all answers collected:**

   ```markdown
   ✅ Discovery complete! Generating authentication spec with:
   - Method: [User's choice from Q1]
   - Storage: [User's choice from Q2]
   - Duration: [User's choice from Q3]
   - Tenancy: [User's choice from Q4]
   [IF Q5 answered:] - Security: [User's choice from Q5]

   These decisions will inform the EARS requirements and risk assessment...
   ```

   Store answers in memory for spec generation:

   ```yaml
   discovery_insights:
     domain: "authentication"
     auth_method: "[answer to Q1]"
     token_storage: "[answer to Q2]"
     session_duration: "[answer to Q3]"
     multi_tenancy: "[answer to Q4]"
     security_features: "[answer to Q5 if provided]"
   ```

   ---

   ### 💾 DATA/CRUD DOMAIN TEMPLATE

   **Triggered by:** "database", "crud", "model", "entity", "schema", etc.

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   💾 Data Model Feature Discovery
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   I'll ask 3-5 questions to understand your data modeling requirements.
   ```

   **Question 1 (MANDATORY): Primary Entities**

   ```markdown
   ❓ Question 1 of 4: Primary Data Entities

   What are the main entities (objects/tables) this feature involves?

   **Format:** Short answer (<=20 words)

   **Examples:**
   - "User, Post, Comment"
   - "Order, OrderItem, Product, Customer"
   - "Project, Task, Assignment, User"

   Your answer:
   ```

   **Question 2 (MANDATORY): Data Relationships**

   ```markdown
   ❓ Question 2 of 4: Data Relationships

   What are the primary relationships between entities?

   [IF user provided entities in Q1, tailor options:]
   How do [Entity A] and [Entity B] relate?

   | Option | Relationship Type | Example |
   |--------|-------------------|---------|
   | A | One-to-many | One User has many Posts |
   | B | Many-to-many | Students have many Courses, Courses have many Students |
   | C | One-to-one | One User has one Profile |
   | D | Complex/Mixed | Combination of the above |
   | Short | Describe relationships (<=30 words) | |

   Your choice:
   ```

   **Question 3 (MANDATORY): Data Retention & Deletion**

   ```markdown
   ❓ Question 3 of 4: Data Retention Policy

   How should data be retained and deleted?

   **Recommended:** Option C (soft delete) for most apps - allows recovery

   | Option | Policy | Implementation | Recovery |
   |--------|--------|----------------|----------|
   | A | Indefinite | No automatic deletion | N/A |
   | B | Time-limited | Delete after X days/months | None |
   | C | Soft delete (user-controlled) | Mark as deleted, user can restore | Easy |
   | D | Compliance-driven | GDPR right to be forgotten, immediate purge | None |
   | Short | Describe retention policy (<=20 words) | | |

   Your choice:
   ```

   **Question 4 (CONDITIONAL): Data Migration**

   Only ask if feature description mentions "existing" or "migrate":

   ```markdown
   ❓ Question 4 of 4: Existing Data Migration (Optional)

   You mentioned existing data. How should migration work?

   | Option | Strategy | Risk Level | Downtime |
   |--------|----------|------------|----------|
   | A | Migrate all existing data upfront | Low | Yes (1-time) |
   | B | New data only, legacy system side-by-side | Medium | No |
   | C | Dual-write period, then migrate | High | No (complex) |
   | D | N/A (no existing data) | None | No |

   Your choice (or type "skip" if not applicable):
   ```

   **After all answers:**

   ```markdown
   ✅ Discovery complete! Generating data model spec with:
   - Entities: [Q1 answer]
   - Relationships: [Q2 answer]
   - Retention: [Q3 answer]
   [IF Q4 answered:] - Migration: [Q4 answer]
   ```

   Store in memory:

   ```yaml
   discovery_insights:
     domain: "data"
     entities: "[answer to Q1]"
     relationships: "[answer to Q2]"
     retention_policy: "[answer to Q3]"
     migration_strategy: "[answer to Q4 if provided]"
   ```

   ---

   ### 🎨 UI/UX DOMAIN TEMPLATE

   **Triggered by:** "dashboard", "ui", "interface", "form", "page", "component", etc.

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   🎨 UI/UX Feature Discovery
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   I'll ask 3-4 questions to understand your UI/UX requirements.
   ```

   **Question 1 (MANDATORY): User Flow Priority**

   ```markdown
   ❓ Question 1 of 3: User Experience Priority

   What's the most important UX aspect for this feature?

   **Recommended:** Option A for consumer apps, Option B for admin/financial apps

   | Option | Priority | Optimizes For | Trade-off |
   |--------|----------|---------------|-----------|
   | A | Speed | Minimal clicks, fast response | Less validation |
   | B | Security | Confirmation dialogs, validation steps | More clicks |
   | C | Simplicity | Clean UI, progressive disclosure | Limited power-user features |
   | D | Power users | Shortcuts, bulk actions, advanced features | Steeper learning curve |
   | Short | Describe priority (<=15 words) | | |

   Your choice:
   ```

   **Question 2 (MANDATORY): Error Recovery Strategy**

   ```markdown
   ❓ Question 2 of 3: Error Recovery

   How should users recover from errors or mistakes?

   | Option | Strategy | User Experience | Use Case |
   |--------|----------|-----------------|----------|
   | A | Automatic retry | System handles errors silently | Transient failures |
   | B | User intervention | Show error, let user decide | Complex operations |
   | C | Fail fast | Block action, require fix | Data validation |
   | D | Graceful degradation | Partial functionality maintained | Non-critical features |
   | Short | Describe error handling (<=20 words) | | |

   **Recommended:** Option B (user intervention) for most apps

   Your choice:
   ```

   **Question 3 (MANDATORY): Accessibility Requirements**

   ```markdown
   ❓ Question 3 of 3: Accessibility Level

   What accessibility standards must this feature meet?

   | Option | Standard | Requirements | Effort |
   |--------|----------|--------------|--------|
   | A | WCAG 2.1 AA | Keyboard nav, screen reader, 4.5:1 contrast | Medium (industry standard) |
   | B | WCAG 2.1 AAA | Enhanced contrast (7:1), more alternatives | High |
   | C | Basic only | Best-effort accessibility | Low |
   | D | Custom | Specify requirements: ___ | Varies |

   **Recommended:** Option A (WCAG 2.1 AA) - required by many regulations

   Your choice:
   ```

   **After all answers:**

   ```markdown
   ✅ Discovery complete! Generating UI/UX spec with:
   - Priority: [Q1 answer]
   - Error Handling: [Q2 answer]
   - Accessibility: [Q3 answer]
   ```

   Store in memory:

   ```yaml
   discovery_insights:
     domain: "ui"
     ux_priority: "[answer to Q1]"
     error_recovery: "[answer to Q2]"
     accessibility: "[answer to Q3]"
   ```

   ---

   ### 🔌 API DOMAIN TEMPLATE

   **Triggered by:** "api", "endpoint", "rest", "graphql", "integration", "webhook", etc.

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   🔌 API Feature Discovery
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   I'll ask 3-5 questions to understand your API requirements.
   ```

   **Question 1 (MANDATORY): API Style**

   ```markdown
   ❓ Question 1 of 4: API Architecture Style

   What API style should this feature use?

   | Option | Style | Best For | Learning Curve |
   |--------|-------|----------|----------------|
   | A | REST | CRUD operations, standard HTTP | Low |
   | B | GraphQL | Complex queries, flexible clients | Medium |
   | C | gRPC | High-performance, service-to-service | High |
   | D | Webhooks | Event-driven, push notifications | Low |
   | Short | Specify API style (<=10 words) | | |

   **Recommended:** Option A (REST) for most web apps

   Your choice:
   ```

   **Question 2 (MANDATORY): External Service Failures**

   ```markdown
   ❓ Question 2 of 4: External Service Failure Handling

   [IF feature involves external API calls:]
   How should external service failures (timeouts, errors) be handled?

   | Option | Strategy | Implementation | Use Case |
   |--------|----------|----------------|----------|
   | A | Retry with exponential backoff | 3 retries: 2s, 4s, 8s delays | Transient failures |
   | B | Circuit breaker | Stop calling after 5 failures in 1 min | Cascading failures |
   | C | Fallback to cached data | Serve stale data temporarily | Read operations |
   | D | Fail fast | Return error immediately | Critical operations |
   | E | Combination | Specify: ___ | Complex scenarios |

   **Recommended:** Option B + C (circuit breaker with cache fallback)

   Your choice:
   ```

   **Question 3 (MANDATORY): API Versioning**

   ```markdown
   ❓ Question 3 of 4: API Versioning Strategy

   How should API versions be managed over time?

   | Option | Strategy | Format | Example | Visibility |
   |--------|----------|--------|---------|------------|
   | A | URL path | /v1/users, /v2/users | /v1/users | High (easy to see) |
   | B | Header | Accept: application/vnd.api.v1+json | GET /users | Medium |
   | C | Query parameter | /users?version=1 | /users?v=1 | High (but less clean) |
   | D | No versioning | Breaking changes not allowed | /users | N/A (additive only) |
   | Short | Describe versioning (<=15 words) | | | |

   **Recommended:** Option A (URL path) - most visible and cache-friendly

   Your choice:
   ```

   **Question 4 (CONDITIONAL): Rate Limiting**

   Only ask if feature is public API or high-traffic:

   ```markdown
   ❓ Question 4 of 4: Rate Limiting (Optional)

   Should this API have rate limiting to prevent abuse?

   | Option | Strategy | Limit | Use Case |
   |--------|----------|-------|----------|
   | A | Per-user rate limit | 100 requests/minute per user | Authenticated APIs |
   | B | Per-IP rate limit | 1000 requests/hour per IP | Public endpoints |
   | C | Tiered rate limits | Free: 100/min, Premium: 1000/min | SaaS APIs |
   | D | No rate limiting | Unlimited | Internal APIs only |
   | Short | Describe rate limiting (<=15 words) | | |

   Your choice (or type "skip"):
   ```

   **After all answers:**

   ```markdown
   ✅ Discovery complete! Generating API spec with:
   - Style: [Q1 answer]
   - Failure Handling: [Q2 answer]
   - Versioning: [Q3 answer]
   [IF Q4 answered:] - Rate Limiting: [Q4 answer]
   ```

   Store in memory:

   ```yaml
   discovery_insights:
     domain: "api"
     api_style: "[answer to Q1]"
     failure_handling: "[answer to Q2]"
     versioning_strategy: "[answer to Q3]"
     rate_limiting: "[answer to Q4 if provided]"
   ```

   ---

   ### ⚡ PERFORMANCE DOMAIN TEMPLATE

   **Triggered by:** "performance", "optimize", "speed", "cache", "slow", etc.

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ⚡ Performance Feature Discovery
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   I'll ask 3-4 questions to understand your performance requirements.
   ```

   **Question 1 (MANDATORY): Performance Bottleneck**

   ```markdown
   ❓ Question 1 of 4: Performance Bottleneck Identification

   What specific performance problem needs solving?

   **Format:** Short answer (<=30 words)

   **Examples:**
   - "API endpoints take 5+ seconds to respond"
   - "Database queries are slow with large datasets"
   - "Page load time is 10 seconds"
   - "Search results take too long"

   Your answer:
   ```

   **Question 2 (MANDATORY): Target Performance Metric**

   ```markdown
   ❓ Question 2 of 4: Target Performance Metric

   What's the target performance after optimization?

   [IF user mentioned API/endpoint in Q1:]
   Target response time for API endpoints?

   [IF user mentioned page load in Q1:]
   Target page load time?

   [IF user mentioned database in Q1:]
   Target query execution time?

   | Option | Target | Improvement | Difficulty |
   |--------|--------|-------------|------------|
   | A | < 100ms | Very fast (excellent UX) | High |
   | B | < 500ms | Fast (good UX) | Medium |
   | C | < 2s | Acceptable | Low |
   | D | Custom | Specify: ___ | Varies |

   **Recommended:** Option B (<500ms) for interactive operations

   Your choice:
   ```

   **Question 3 (MANDATORY): Caching Strategy**

   ```markdown
   ❓ Question 3 of 4: Caching Strategy

   Where should caching be applied to improve performance?

   | Option | Caching Layer | Data Type | TTL |
   |--------|---------------|-----------|-----|
   | A | Database query cache (Redis) | Frequently-accessed data | 5-15 min |
   | B | CDN edge cache | Static assets, API responses | 1 hour - 1 day |
   | C | Application-level cache (in-memory) | Session data, temp results | 1-5 min |
   | D | Multiple layers | Combination of above | Varies |
   | E | No caching | Real-time data only | N/A |
   | Short | Describe caching approach (<=20 words) | | |

   **Recommended:** Option A (Redis) for database-heavy apps

   Your choice:
   ```

   **Question 4 (CONDITIONAL): Scale Considerations**

   Only ask if user mentioned high traffic or scale:

   ```markdown
   ❓ Question 4 of 4: Scale Considerations (Optional)

   What's the expected traffic scale for this feature?

   | Option | Scale | Concurrent Users | Infrastructure |
   |--------|-------|------------------|----------------|
   | A | Small | <100 concurrent users | Single server OK |
   | B | Medium | 100-1000 concurrent users | Load balancer + cache |
   | C | Large | 1000-10000 concurrent users | Auto-scaling + CDN + cache |
   | D | Very large | >10000 concurrent users | Distributed system |
   | Short | Describe expected scale (<=15 words) | | |

   Your choice (or type "skip"):
   ```

   **After all answers:**

   ```markdown
   ✅ Discovery complete! Generating performance spec with:
   - Bottleneck: [Q1 answer]
   - Target: [Q2 answer]
   - Caching: [Q3 answer]
   [IF Q4 answered:] - Scale: [Q4 answer]
   ```

   Store in memory:

   ```yaml
   discovery_insights:
     domain: "performance"
     bottleneck: "[answer to Q1]"
     target_metric: "[answer to Q2]"
     caching_strategy: "[answer to Q3]"
     scale_requirements: "[answer to Q4 if provided]"
   ```

   ---

   ### 🔧 GENERIC DOMAIN TEMPLATE (Fallback)

   **Triggered by:** No specific domain detected or multiple domains detected

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   🔧 Feature Discovery (Generic)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   I'll ask 3-5 general questions to understand your requirements.
   ```

   **Keep existing generic questions from lines 72-141 of current speckit.specify.md:**

   - Question 1: Outcome & Problem
   - Question 2: Scope & Priorities
   - Question 3: Scale & Risk Considerations
   - Question 4 (optional): Out of Scope
   - Question 5 (optional): Constraints

   ---

   **Step 1.5: Use Discovery Insights During Spec Generation**

   After discovery completes, store insights for use during spec generation:

   ```yaml
   # Pass to spec generation steps (5-7)

   USE discovery_insights WHEN:

   1. Generating Functional Requirements (Step 5):
      - Map domain-specific answers to EARS requirements
      - Example (auth domain):
        IF auth_method == "OAuth 2.0":
          → Generate: "REQ-AUTH-1.1: WHEN user initiates login THEN system SHALL redirect to OAuth provider within 200ms"
        IF token_storage == "httpOnly cookies":
          → Generate: "REQ-AUTH-1.2: WHEN OAuth callback received THEN system SHALL store access token in httpOnly cookie"

   2. Populating Non-Functional Requirements (Step 5):
      - Use target metrics from discovery
      - Example (performance domain):
        IF target_metric == "<500ms":
          → Add NFR: "NFR-PERF-1: API response time SHALL be <500ms for 95th percentile"

   3. Filling User Scenarios (Step 4):
      - Use outcome/problem from Q1 (generic template)
      - Use ux_priority (UI domain) to shape user stories

   4. Reducing [NEEDS CLARIFICATION] Markers:
      - Discovery answers eliminate ambiguities
      - Only add markers for aspects NOT covered by discovery
      - Target: 0-2 markers instead of 3+ without discovery

   5. Risk Assessment (Step 7c):
      - Domain influences risk scores
      - Auth domain → Higher "Access Control Risk"
      - Data domain → Higher "Data Sensitivity Risk"
      - API domain → Higher "External Integration Risk"
   ```

   **Step 1.6: Discovery Completion Message**

   After storing insights:

   ```markdown
   ✅ Discovery complete! Generating specification with enriched understanding...

   [Show brief summary of insights:]
   - Domain: {primary_domain}
   - Key decisions: [2-3 bullet points from answers]
   - [NEEDS CLARIFICATION] markers reduced significantly

   Proceeding to specification generation...
   ```

   Proceed to Step 2 (short name generation).

---

**END OF ENHANCED STEP 1**

This replaces lines 22-167 in src/.claude/commands/speckit.specify.md
