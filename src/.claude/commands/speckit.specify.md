---
description: Create or update the feature specification from a natural language feature description.
scripts:
  sh: .specify/scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: .specify/scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

The text the user typed after `/speckit.specify` in the triggering message **is** the feature description. Assume you always have it available in this conversation even if `{ARGS}` appears literally below. Do not ask the user to repeat it unless they provided an empty command.

Given that feature description, do this:

### Phase 0: Context Loading (v2.7 - Context Optimization)

**Purpose**: Load only relevant memory content for the specification phase to optimize token usage.

**Current Phase**: `specification`

**Context Loading Strategy**:

1. **Read Configuration** (if exists):
   - Check for `.specify/config.yml` or `.specify/config.example.yml`
   - Extract `context_budget.specification` (default: 15KB if not found)
   - Extract `memory.strict_phase_loading` (default: true)
   - Extract `budget_enforcement.mode` (default: strict)

2. **Load Memory Files** (phase-aware):

   **Load ALL memory files, but filter by phase**:

   For each memory file in `.specify/memory/`:

   a. **Read YAML Frontmatter**:
      - Extract `inclusion_mode`, `context_level`, `load_phases`, `exclude_phases`
      - If metadata missing: Default to `inclusion_mode: always, context_level: strategic, load_phases: all`

   b. **Determine if file should be loaded**:
      ```yaml
      IF current_phase IN exclude_phases:
        → SKIP this file entirely

      ELSE IF inclusion_mode = "always":
        → Load strategic sections only (filter by SECTION_META comments)

      ELSE IF inclusion_mode = "conditional":
        IF current_phase IN load_phases:
          → Load strategic sections (planning) OR tactical sections (implementation)
        ELSE:
          → SKIP this file

      ELSE IF inclusion_mode = "manual":
        → SKIP (manual loading via @filename only)
      ```

   c. **Filter Sections** (if loading file):
      - Scan for `<!-- SECTION_META: context_level=X, load_phases=Y -->` comments
      - For specification phase: Load ONLY sections where `context_level=strategic`
      - Skip sections marked `context_level=tactical` or `context_level=reference`

   d. **Track Budget**:
      - Sum loaded content size
      - If exceeds `context_budget.specification` (15KB):
        - **strict mode**: Warn user, truncate reference sections first
        - **warn mode**: Continue loading, warn about budget exceeded
        - **adaptive mode**: Intelligently summarize content to fit budget

3. **Context Loading Report** (verbose mode):

   IF `verbose_context_loading: true` in config:
   ```markdown
   📊 Context Loading Report

   Phase: specification
   Budget: 15KB

   Loaded:
   - constitution.md (strategic sections): 3.2KB
   - Skipped: api-standards.md (excluded phase: specification)
   - Skipped: testing-approach.md (excluded phase: specification)
   - Skipped: deployment-runbook.md (manual only)

   Total Loaded: 3.2KB / 15KB (21% budget used)
   Remaining: 11.8KB available

   Status: ✅ Within budget
   ```

4. **Backward Compatibility**:
   - If `.specify/memory/` doesn't exist → Skip context loading (no error)
   - If memory files lack metadata → Load all content (v2.6 behavior)
   - Never fail command due to missing memory files

**Expected Token Savings**: ~70% (25-35KB → 5-8KB for specification phase)

---

### Step 0.5: Workflow Recommendation (v2.9 - Quick Workflow)

**Purpose**: Analyze feature complexity and recommend appropriate workflow (quick vs full).

**Quick Risk Assessment** (Heuristic):

Before complexity analysis, perform a quick risk assessment based on feature description keywords:

**HIGH-RISK Indicators** (score ≥4, requires full workflow):
- Contains keywords: "payment", "billing", "stripe", "auth", "authentication", "authorization", "security", "encryption", "password", "multi-tenant", "tenant", "database migration", "data migration", "production data"
- External integrations: "API integration", "third-party", "OAuth", "SSO"
- Compliance: "GDPR", "HIPAA", "PCI", "SOC2", "compliance"
- Scale: "million users", "high traffic", "performance critical"

**MEDIUM-RISK Indicators** (score 4-7, full workflow recommended):
- Contains keywords: "database", "schema change", "API endpoint", "real-time", "WebSocket", "cron job", "background job"
- Data operations: "delete", "bulk update", "import", "export"

**Decision**:
```
IF feature_description contains ANY HIGH-RISK indicator:
  → Risk Level: 🔴 HIGH (score ≥8)
  → Output: "⚠️ HIGH-RISK feature detected. Full workflow required for proper risk assessment and security review."
  → Skip workflow recommendation entirely
  → Proceed directly to Step 1 (full workflow)

ELSE IF feature_description contains ≥2 MEDIUM-RISK indicators:
  → Risk Level: 🟠 MEDIUM (score 4-7)
  → Output: "⚠️ MEDIUM-RISK feature detected. Full workflow recommended for comprehensive planning."
  → Skip workflow recommendation entirely
  → Proceed directly to Step 1 (full workflow)

ELSE:
  → Risk Level: 🟢 LOW (score 0-3) - Assumed safe for complexity analysis
  → Continue to Complexity Analysis below
```

---

**Complexity Analysis** (if risk ≤3):

Evaluate the feature description against these criteria:

**SIMPLE Task Indicators** (consider `/speckit.quick`):
- ✅ Simple, isolated component (form, button, card, modal)
- ✅ Bug fix with clear root cause
- ✅ Minor enhancement to existing feature
- ✅ Clear requirements (no ambiguity)
- ✅ Estimated < 2 days effort
- ✅ No architectural changes needed
- ✅ LOW risk (no sensitive data, simple logic)
- ✅ Uses existing patterns/libraries

**COMPLEX Task Indicators** (use full workflow):
- ❌ Multi-component system or feature
- ❌ New architectural patterns or ADRs needed
- ❌ MEDIUM or HIGH risk (score ≥4)
- ❌ External system integration (auth, payments, APIs)
- ❌ Ambiguous or incomplete requirements
- ❌ 3+ days effort estimated
- ❌ Multiple edge cases or complex business logic

**Decision Logic**:

```yaml
IF feature_description meets ≥5 SIMPLE indicators AND 0 COMPLEX indicators:

  Present workflow recommendation to user:

  "💡 WORKFLOW RECOMMENDATION

  This appears to be a SIMPLE, LOW-RISK task that could use the quick workflow:

  **Option A: Quick Workflow** (`/speckit.quick`) ⚡
  - ✅ Single command (1-2 hours)
  - ✅ Minimal documentation overhead
  - ✅ Full constitutional enforcement & quality gates
  - ✅ TDD still required (RED-GREEN-REFACTOR)
  - ❌ No formal spec/plan (less documentation)
  - Best for: Forms, components, bug fixes, minor enhancements

  **Option B: Full Workflow** (`/speckit.specify` → `/speckit.plan` → `/speckit.implement`) 📋
  - ✅ Comprehensive specification & planning
  - ✅ Formal approval gates
  - ✅ Detailed risk assessment & test strategy
  - ✅ Better for team review & complex features
  - ❌ More time (4-8+ hours)
  - Best for: Complex features, architectural changes, high-risk work

  **Recommendation**: `/speckit.quick` (saves 35-40% time/tokens)

  Continue with full workflow anyway? (yes/no)
  Type 'yes' to proceed with /speckit.specify
  Type 'no' to exit (then run /speckit.quick instead)
  "

  Wait for user response.

  IF user responds "no", "quick", "switch", or "exit":
    Output: "✅ Exiting /speckit.specify. Please run `/speckit.quick` instead for fast implementation."
    → STOP workflow (user will run /speckit.quick)

  ELSE IF user responds "yes", "full", "continue", or "proceed":
    Output: "✅ Continuing with full workflow (comprehensive documentation)."
    → Proceed to Step 1 (Vagueness Detection)

ELSE (feature meets COMPLEX criteria):
  → Skip recommendation entirely
  → Proceed directly to Step 1 (full workflow required)
```

**Example Scenarios**:

**Scenario 1** (Recommend Quick):
- Description: "Create user profile edit form with name, email, bio fields"
- Analysis: Simple component, clear requirements, < 2 days, LOW risk
- Recommendation: `/speckit.quick` (saves 2-4 hours)

**Scenario 2** (No Recommendation - Complex):
- Description: "Add multi-tenant billing system with Stripe integration"
- Analysis: HIGH risk, external integration, complex business logic, 5+ days
- Action: Proceed directly with full workflow (no prompt shown)

**Scenario 3** (Borderline - User Choice):
- Description: "Fix date picker validation bug (allow only future dates)"
- Analysis: Meets SIMPLE criteria but touches multiple files
- Recommendation: Offer choice, recommend `/speckit.quick`

---

1. **Vagueness Detection & Pre-Spec Discovery** (Adaptive Dialogue):

   **Purpose**: If the feature description is too high-level or vague, engage in brief discovery dialogue BEFORE generating the spec to build better understanding and prevent assumption-heavy specifications.

   **Vagueness Detection Heuristics**:

   Evaluate the user's feature description against these criteria:

   - **Length**: < 30 words (insufficient detail)
   - **Uncertainty markers**: Contains "maybe", "thinking about", "not sure", "possibly", "could", "might"
   - **Generic terms without specifics**:
     - "dashboard" (without saying what data/metrics)
     - "authentication" (without specifying type: OAuth, JWT, sessions)
     - "API" (without endpoints or purpose)
     - "admin panel" (without specific admin functions)
   - **Lack of concrete verbs**: "add something", "make it better", "improve"
   - **Missing context**: Who/what/why/when are unclear

   **Decision Flow**:

   ```yaml
   IF description meets ANY vagueness criteria:

     Present discovery option to user:

     "🤔 Your description seems high-level. To create a better spec, I can:

     **A) Generate spec now** (fast, ~1 min, may include assumptions)
     **B) Ask 3-5 discovery questions first** (better spec, ~2-3 min)

     Type 'A' for fast generation, 'B' for discovery, or just 'proceed' to skip.

     Recommendation: Choose B for important features or when requirements are unclear."

     Wait for user response.

     IF user responds with "A", "fast", "skip", or "proceed":
       → Skip discovery, proceed to spec generation (note: will make informed assumptions)

     ELSE (user chooses "B", "discovery", or "questions"):
       → Run discovery dialogue (see below)

   ELSE (description is detailed and specific):
     → Skip detection entirely, proceed directly to Step 2 (short name generation)
   ```

   **Discovery Dialogue** (if user chooses discovery):

   Ask 3-5 targeted questions to build understanding. Present questions ONE at a time, wait for each answer before proceeding.

   **Question 1 (Outcome Focus)**:
   ```
   ❓ Outcome & Problem

   What user outcome or problem does this feature solve?

   Examples to guide your answer:
   - "Users can't currently access protected resources" → Authentication needed
   - "Users can't see real-time sales data" → Real-time dashboard needed
   - "Admins spend hours manually approving requests" → Automated workflow needed

   Your answer: [wait for response]
   ```

   **Question 2 (Scope Control)**:
   ```
   ❓ Scope & Priorities

   What are the must-haves vs nice-to-haves for this feature?

   Must-have: Core functionality without which the feature doesn't work
   Nice-to-have: Enhancements that improve UX but aren't critical

   Example:
   - Must: Email/password login, password reset
   - Nice: OAuth providers, 2FA, remember me

   Your answer: [wait for response]
   ```

   **Question 3 (Risk Awareness)**:
   ```
   ❓ Scale & Risk Considerations

   What happens if usage is 10x higher than expected?
   (Or: What are the key technical risks or constraints?)

   This helps identify:
   - Performance bottlenecks to consider
   - Scalability requirements
   - Infrastructure needs

   Example: "Need rate limiting, caching critical, async processing"

   Your answer: [wait for response]
   ```

   **Optional Question 4** (if scope still unclear):
   ```
   ❓ Out of Scope

   What's explicitly OUT of scope for this feature?

   Clarifying what NOT to build is often as important as what to build.

   Example: "No social login, no magic links, no biometric auth"

   Your answer: [wait for response]
   ```

   **Optional Question 5** (if constraints mentioned):
   ```
   ❓ Constraints

   What are the hard constraints (time, budget, technology, compliance)?

   Example: "Must launch in 2 weeks, must use existing auth system, must comply with GDPR"

   Your answer: [wait for response]
   ```

   **After Discovery Completion**:

   Synthesize insights and store them for use during spec generation:

   ```yaml
   Discovery Insights:
     user_outcome: [answer to Q1]
     scope_must_haves: [answer to Q2 - must]
     scope_nice_to_haves: [answer to Q2 - nice]
     risks_and_scale: [answer to Q3]
     out_of_scope: [answer to Q4 if asked]
     constraints: [answer to Q5 if asked]
   ```

   These insights will be used to:
   - Generate more accurate functional requirements
   - Create User Outcome section in spec
   - Define Scope Boundaries section
   - Populate Risks & Assumptions section
   - Reduce [NEEDS CLARIFICATION] markers

   Output to user:
   ```
   ✅ Discovery complete! Generating spec with enriched understanding...
   ```

2. **Generate a concise short name** (2-4 words) for the branch:
   - Analyze the feature description and extract the most meaningful keywords
   - Create a 2-4 word short name that captures the essence of the feature
   - Use action-noun format when possible (e.g., "add-user-auth", "fix-payment-bug")
   - Preserve technical terms and acronyms (OAuth2, API, JWT, etc.)
   - Keep it concise but descriptive enough to understand the feature at a glance
   - Examples:
     - "I want to add user authentication" → "user-auth"
     - "Implement OAuth2 integration for the API" → "oauth2-api-integration"
     - "Create a dashboard for analytics" → "analytics-dashboard"
     - "Fix payment processing timeout bug" → "fix-payment-timeout"

3. **Check for existing branches before creating new one**:
   
   a. First, fetch all remote branches to ensure we have the latest information:
      ```bash
      git fetch --all --prune
      ```
   
   b. Find the highest feature number across all sources for the short-name:
      - Remote branches: `git ls-remote --heads origin | grep -E 'refs/heads/feature/[0-9]{6}-<short-name>$'`
      - Local branches: `git branch | grep -E '^[* ]*feature/[0-9]{6}-<short-name>$'`
      - Specs directories: Check for directories matching `specs/[0-9]{6}-<short-name>`
   
   c. Determine the next available number:
      - Extract all numbers from all three sources
      - Find the highest number N
      - Use N+1 for the new branch number
   
   d. Run the script `{SCRIPT}` with the calculated number and short-name:
      - Pass `--number N+1` and `--short-name "your-short-name"` along with the feature description
      - Bash example: `{SCRIPT} --json --number 5 --short-name "user-auth" "Add user authentication"`
      - PowerShell example: `{SCRIPT} -Json -Number 5 -ShortName "user-auth" "Add user authentication"`
   
   **IMPORTANT**:
   - Check all three sources (remote branches, local branches, specs directories) to find the highest number
   - Only match branches/directories with the exact short-name pattern
   - If no existing branches/directories found with this short-name, start with number 1
   - You must only ever run this script once per feature
   - The JSON is provided in the terminal as output - always refer to it to get the actual content you're looking for
   - The JSON output will contain BRANCH_NAME and SPEC_FILE paths
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot")

4. Load `templates/spec-template.md` to understand required sections.

5. Follow this execution flow:

    1. Parse user description from Input
       If empty: ERROR "No feature description provided"
    2. Extract key concepts from description
       Identify: actors, actions, data, constraints
    3. For unclear aspects:
       - Make informed guesses based on context and industry standards
       - Only mark with [NEEDS CLARIFICATION: specific question] if:
         - The choice significantly impacts feature scope or user experience
         - Multiple reasonable interpretations exist with different implications
         - No reasonable default exists
       - **LIMIT: Maximum 3 [NEEDS CLARIFICATION] markers total**
       - Prioritize clarifications by impact: scope > security/privacy > user experience > technical details
    4. Fill User Scenarios & Testing section
       If no clear user flow: ERROR "Cannot determine user scenarios"
    5. Generate Functional Requirements
       **Use EARS Format** (Easy Approach to Requirements Syntax) for structured, testable requirements:

       **Four Core Patterns**:
       - **Event-Driven**: `WHEN [event] THEN [system] SHALL [response]`
         Example: `WHEN a user submits valid credentials THEN the Authentication Service SHALL grant access within 2 seconds`

       - **State-Driven**: `IF [condition] THEN [system] SHALL [response]`
         Example: `IF credentials are invalid THEN the Authentication Service SHALL reject access and log the attempt`

       - **Continuous**: `WHILE [duration] THE [system] SHALL [behavior]`
         Example: `WHILE a report is generating THE UI SHALL display a progress indicator`

       - **Contextual**: `WHERE [context] THE [system] SHALL [behavior]`
         Example: `WHERE multi-tenant isolation is required THE Data Layer SHALL enforce row-level security`

       **Requirement ID Format**: `REQ-{CATEGORY}-{NUMBER}.{SUB}`
       - Examples: REQ-AUTH-1.1, REQ-AUTH-1.2, REQ-DASH-2.1
       - Category: 2-4 letter acronym (AUTH, DASH, API, DATA, etc.)
       - Number: Sequential per category
       - Sub: Sequential within requirement

       Each requirement must be:
       - Testable (clear success criteria)
       - Measurable (specific metrics where applicable)
       - Unambiguous (structured EARS format)
       - Traceable (unique ID for spec → plan → tasks → tests)

       Use reasonable defaults for unspecified details (document assumptions in Assumptions section)

       See `.specify/templates/requirements-ears.md` for detailed guidance and examples.
    6. Define Success Criteria
       Create measurable, technology-agnostic outcomes
       Include both quantitative metrics (time, performance, volume) and qualitative measures (user satisfaction, task completion)
       Each criterion must be verifiable without implementation details
    7. Identify Key Entities (if data involved)
    8. Return: SUCCESS (spec ready for planning)

6. Write the specification to SPEC_FILE using the template structure, replacing placeholders with concrete details derived from the feature description (arguments) while preserving section order and headings.

7. **Specification Quality Validation**: After writing the initial spec, validate it against quality criteria:

   a. **Generate Quality Scores**: Assess the spec across four dimensions and fill in the "Specification Quality Assessment" section:

      **Clarity Score (0-10)**: Language precision and understandability
      - 8-10: Crystal clear, unambiguous, consistent terminology
      - 5-7: Mostly clear with some ambiguous sections
      - 2-4: Significant ambiguity requiring clarification
      - 0-1: Unclear and confusing throughout

      **Assessment criteria**:
      - Are requirements atomic (one testable behavior each)?
      - Is terminology consistent across the spec?
      - Are vague terms ("gracefully", "efficiently", "user-friendly") avoided?
      - Are dependencies and preconditions explicit?

      **Completeness Score (0-10)**: Coverage of scenarios, edge cases, and constraints
      - 8-10: All scenarios, edge cases, and constraints covered
      - 5-7: Core scenarios covered, missing some edge cases
      - 2-4: Significant gaps in scenario coverage
      - 0-1: Incomplete, major sections missing

      **Assessment criteria**:
      - Are all user stories with P1-P3 priorities present?
      - Are edge cases and error conditions identified?
      - Are non-functional requirements specified?
      - Are all affected stakeholders identified?

      **Testability Score (0-10)**: Measurability and validation capability
      - 8-10: Concrete acceptance criteria, specific examples, measurable outcomes
      - 5-7: Some acceptance criteria, lacking specificity
      - 2-4: Vague criteria, difficult to validate
      - 0-1: Untestable requirements

      **Assessment criteria**:
      - Do all FRs have measurable acceptance criteria?
      - Are Given/When/Then scenarios specific (not abstract)?
      - Can automated tests verify each requirement?
      - Are success criteria quantifiable?

      **Consistency Score (0-10)**: Internal coherence and contradiction detection
      - 8-10: Fully consistent, no contradictions
      - 5-7: Mostly consistent with minor conflicts
      - 2-4: Several contradictions requiring resolution
      - 0-1: Major contradictions throughout

      **Assessment criteria**:
      - Do requirements contradict each other?
      - Are similar behaviors handled consistently?
      - Is the scope clearly bounded without overlap?
      - Are assumptions aligned across sections?

      **Overall Quality**: Average of the four scores

      **Review Status**:
      - ✅ Ready for Clarify: Overall score ≥ 7, no critical issues
      - ⚠️ Needs Review: Overall score 4-6, some issues present
      - ❌ Major Issues: Overall score < 4, significant problems

      **Areas Needing Attention**: List top 3 issues found during assessment (with section references)

   b. **Generate UX Quality Scores** (for interface-heavy features): If the feature involves user interfaces, workflows, or interactions, assess UX quality and fill in the "UX Quality Assessment" section:

      **Usability Score (0-10)**: Ease of use and learnability
      - 8-10: Intuitive, self-evident, follows established UX patterns
      - 5-7: Mostly usable with some friction points
      - 2-4: Confusing, requires training or documentation
      - 0-1: Unusable, violates basic usability heuristics

      **Assessment criteria** (Nielsen's Heuristics):
      - Is system status always visible (loading states, progress)?
      - Does interface use user's language (not system jargon)?
      - Can users undo/exit actions easily?
      - Are interaction patterns consistent across flows?
      - Does design prevent errors before they occur?
      - Are options visible vs. requiring memory?

      **Efficiency Score (0-10)**: Task completion speed and input burden
      - 8-10: Optimized for frequent tasks, smart defaults, minimal steps (≤3 clicks)
      - 5-7: Reasonable efficiency with some unnecessary steps
      - 2-4: Inefficient workflows, excessive clicks/typing
      - 0-1: Painful to use, major inefficiencies

      **Assessment criteria**:
      - Can users complete primary tasks in ≤3 clicks/taps?
      - Are smart defaults provided for common values?
      - Is progressive disclosure used (complexity revealed only when needed)?
      - Is context switching minimized (no leaving app for info)?

      **Accessibility Score (0-10)**: Universal access and WCAG compliance
      - 8-10: WCAG 2.1 AA compliant, keyboard accessible, screen reader friendly
      - 5-7: Mostly accessible with minor gaps
      - 2-4: Significant accessibility barriers
      - 0-1: Inaccessible to users with disabilities

      **Assessment criteria** (WCAG 2.1 AA):
      - Can entire flow be completed without mouse (keyboard only)?
      - Are ARIA labels and screen reader support specified?
      - Is color contrast sufficient (4.5:1 for text, 3:1 for large)?
      - Is information conveyed beyond color alone?
      - Are focus indicators clearly visible?

      **Error Prevention Score (0-10)**: Ability to prevent and recover from mistakes
      - 8-10: Errors prevented by design, easy recovery, forgiving formats
      - 5-7: Basic error handling, some prevention
      - 2-4: Errors easily made, poor recovery
      - 0-1: Error-prone, no recovery mechanisms

      **Assessment criteria**:
      - Are destructive actions confirmed ("Are you sure?")?
      - Are invalid states prevented (disabled buttons, constraints)?
      - Are error messages specific, actionable, and blame-free?
      - Can users fix errors inline (not start over)?
      - Does system accept multiple input formats (dates, phone)?

      **Delight Score (0-10)**: User satisfaction and emotional response
      - 8-10: Delightful interactions, exceeds expectations
      - 5-7: Functional but not memorable
      - 2-4: Frustrating or tedious
      - 0-1: Actively unpleasant to use

      **Assessment criteria**:
      - Are success states celebrated (not just dismissed)?
      - Do transitions feel smooth (not jarring)?
      - Are loading states engaging (skeleton screens, progress)?
      - Is micro-feedback provided (button press animations)?

      **Overall UX Quality**: Average of the five scores

      **UX Status**:
      - ✅ UX Ready: Overall score ≥ 7, no accessibility violations
      - ⚠️ UX Needs Improvement: Overall score 4-6, some friction points
      - ❌ UX Issues: Overall score < 4 or WCAG violations present

      **UX Areas Needing Attention**: List top 3 UX issues found (with Nielsen's heuristic or WCAG guideline violated)

      **Note**: Skip UX scoring for backend-only, API-only, or infrastructure features. Focus on features with user-facing interfaces, workflows, or interactions.

   c. **Generate Risk Assessment** (for all features): Identify high-risk requirements requiring extra scrutiny during planning and implementation:

      **Risk Scoring Criteria** (0-10 scale per requirement):

      1. **Data Sensitivity Risk** (0-3 points):
         - 3: Personal data (PII), financial data, health information, passwords, API keys
         - 2: Business-sensitive data (revenue, contracts, proprietary algorithms)
         - 1: User-generated content, logs, analytics
         - 0: Public data, read-only reference data

      2. **Access Control Risk** (0-3 points):
         - 3: Authentication, authorization, role management, tenant isolation, admin functions
         - 2: User permissions, resource access, data filtering by user/tenant
         - 1: Public endpoints with rate limiting
         - 0: Anonymous read-only access

      3. **External Integration Risk** (0-2 points):
         - 2: Payment processors, financial APIs, identity providers (OAuth), webhooks receiving sensitive data
         - 1: 3rd party APIs, external services, file uploads, email sending
         - 0: No external integrations

      4. **Performance Impact Risk** (0-1 point):
         - 1: High-traffic endpoints (>100 req/s), complex queries, large data exports, search/reporting
         - 0: Low-traffic CRUD operations

      5. **Complexity Risk** (0-1 point):
         - 1: Multi-step workflows, state machines, distributed transactions, concurrency handling
         - 0: Simple CRUD operations, single-step actions

      6. **Business Impact Risk** (0-2 points):
         - 2: Revenue-affecting (billing, payments, subscriptions), compliance-affecting (audit trails, GDPR)
         - 1: User satisfaction-affecting (core features, data integrity)
         - 0: Nice-to-have features, cosmetic changes

      **Total Risk Score**: Sum of all criteria (0-12)

      **Risk Classifications**:
      - 🔴 **HIGH RISK** (8-12): Requires security review, comprehensive testing, staged rollout
      - 🟠 **MEDIUM RISK** (4-7): Requires extra scrutiny during implementation, integration testing
      - 🟢 **LOW RISK** (0-3): Standard development process

      **Risk Assessment Output Format**:

      ```markdown
      ## Risk Assessment

      **Overall Feature Risk**: [🔴 HIGH | 🟠 MEDIUM | 🟢 LOW]

      ### High-Risk Requirements (Score ≥ 8)

      - **FR-XXX**: [Requirement title]
        - **Risk Score**: [X]/12
        - **Risk Factors**: [Data Sensitivity: X, Access Control: X, External Integration: X, ...]
        - **Mitigation Required**:
          - Security review during planning (OWASP Top 10 checklist)
          - Threat modeling for attack vectors
          - Comprehensive edge case testing (boundary values, concurrency, failure scenarios)
          - Staged rollout with monitoring
          - Penetration testing before production

      ### Medium-Risk Requirements (Score 4-7)

      - **FR-XXX**: [Requirement title]
        - **Risk Score**: [X]/12
        - **Risk Factors**: [List]
        - **Mitigation Required**:
          - Extra code review scrutiny
          - Integration testing with external services
          - Performance testing under load
          - Error handling validation

      ### Low-Risk Requirements (Score 0-3)

      [List of requirement IDs - no detailed mitigation needed]

      **Testing Strategy Recommendations**:
      - **High-Risk**: Security testing, penetration testing, chaos engineering, staged rollout
      - **Medium-Risk**: Integration testing, performance testing, error handling validation
      - **Low-Risk**: Unit testing, basic integration testing

      **Security Review Triggers**:
      [IF any requirement has Data Sensitivity ≥ 2 OR Access Control ≥ 2 OR External Integration = 2]:
        → ⚠️ Security review REQUIRED during planning phase
        → Run OWASP Top 10 checklist
        → Conduct threat modeling

      **Performance Review Triggers**:
      [IF any requirement has Performance Impact = 1]:
        → ⚠️ Performance testing REQUIRED
        → Load testing, query optimization, pagination validation
      ```

      **Append Risk Assessment to spec.md** under "## Risk Assessment" section

7.5. **Large Spec Detection** (Proactive Supplement Recommendation):

   **Purpose**: Detect complex multi-domain specifications early and recommend hierarchical spec structure to improve maintainability, token efficiency, and team ownership.

   **Detection Logic**:

   After the spec file is written and validated, check if it exceeds recommended complexity thresholds:

   ```bash
   # Calculate spec size and requirement count
   SPEC_SIZE_KB=$(du -k "${FEATURE_SPEC}" | cut -f1)
   FR_COUNT=$(grep -c "^### FR-" "${FEATURE_SPEC}" || echo 0)
   NFR_COUNT=$(grep -c "^### NFR-" "${FEATURE_SPEC}" || echo 0)
   TOTAL_REQUIREMENTS=$((FR_COUNT + NFR_COUNT))

   # Check if spec exceeds recommended thresholds
   if [ "$SPEC_SIZE_KB" -gt 100 ] || [ "$TOTAL_REQUIREMENTS" -gt 60 ]; then
     echo ""
     echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
     echo "💡 RECOMMENDATION: Large Specification Detected"
     echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
     echo ""
     echo "Spec Size: ${SPEC_SIZE_KB}KB (threshold: 100KB)"
     echo "Requirements: ${TOTAL_REQUIREMENTS} (threshold: 60)"
     echo ""
     echo "Consider creating supplementary specs for domain-specific details:"
     echo ""
     echo "  /speckit.supplement ui-ux \"UI/UX implementation details\""
     echo "    → Creates UI-SPEC.md for interface-specific requirements"
     echo ""
     echo "  /speckit.supplement api-contracts \"REST API contracts\""
     echo "    → Creates API-SPEC.md for endpoint specifications"
     echo ""
     echo "  /speckit.supplement technical \"Architecture and infrastructure\""
     echo "    → Creates TECHNICAL-SPEC.md for technical decisions"
     echo ""
     echo "Benefits:"
     echo "  - ✅ Token efficiency (~47% reduction for agents reading specs)"
     echo "  - ✅ Clear ownership (UI team owns UI-SPEC.md, Backend team owns API-SPEC.md)"
     echo "  - ✅ Maintainability (update domains independently)"
     echo "  - ✅ Better organization (complex features stay manageable)"
     echo ""
     echo "You can:"
     echo "  A) Create supplementary specs now (recommended for multi-domain features)"
     echo "  B) Continue with single spec.md (acceptable for focused features)"
     echo ""
     echo "Note: You can always run /speckit.supplement later if needed."
     echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
     echo ""
   fi
   ```

   **Thresholds Explained**:

   - **Size Threshold: 100KB**
     - Rationale: Early warning (before hitting 150KB pain point)
     - Benefit: User can proactively split specs before pain becomes acute
     - Context: Allows for natural growth without immediate action

   - **Requirements Threshold: 60 FRs/NFRs**
     - Rationale: ~60 requirements often indicate multi-domain features
     - Benefit: Suggests domain separation even if file size is manageable
     - Context: Quality-based signal (not just file size)

   - **Triggering Logic**: `IF size > 100KB OR requirements > 60`
     - Detects both large files AND complex features
     - Either condition triggers recommendation
     - User always has final say

   **User Decision Flow**:

   ```yaml
   IF large spec detected:
     → Display recommendation with examples
     → User decides:
       A) Run /speckit.supplement → Create hierarchical specs now
       B) Continue → Keep single spec.md (no penalty)
       C) Defer → Run /speckit.supplement later when needed

     → Preserve user control (no automatic file creation)
     → No workflow blocking (recommendation only)
   ```

   **CRITICAL - Manual Control Preserved**:

   - ✅ Display recommendation with clear examples
   - ✅ Show benefits (token efficiency, ownership, maintainability)
   - ✅ Let user choose (A/B/C decision)
   - ❌ **DO NOT** create supplementary spec files automatically
   - ❌ **DO NOT** block workflow if user declines
   - ❌ **DO NOT** require action before proceeding

   This is a **recommendation only** - the user retains full control over whether to create supplementary specs.

   d. **Create Spec Quality Checklist**: Generate a checklist file at `FEATURE_DIR/checklists/requirements.md` using the checklist template structure with these validation items:

      ```markdown
      # Specification Quality Checklist: [FEATURE NAME]
      
      **Purpose**: Validate specification completeness and quality before proceeding to planning
      **Created**: [DATE]
      **Feature**: [Link to spec.md]
      
      ## Content Quality
      
      - [ ] No implementation details (languages, frameworks, APIs)
      - [ ] Focused on user value and business needs
      - [ ] Written for non-technical stakeholders
      - [ ] All mandatory sections completed
      
      ## Requirement Completeness
      
      - [ ] No [NEEDS CLARIFICATION] markers remain
      - [ ] Requirements are testable and unambiguous
      - [ ] Success criteria are measurable
      - [ ] Success criteria are technology-agnostic (no implementation details)
      - [ ] All acceptance scenarios are defined
      - [ ] Edge cases are identified
      - [ ] Scope is clearly bounded
      - [ ] Dependencies and assumptions identified
      
      ## Feature Readiness
      
      - [ ] All functional requirements have clear acceptance criteria
      - [ ] User scenarios cover primary flows
      - [ ] Feature meets measurable outcomes defined in Success Criteria
      - [ ] No implementation details leak into specification
      
      ## Notes
      
      - Items marked incomplete require spec updates before `/speckit.clarify` or `/speckit.plan`
      ```

   d. **Run Validation Check**: Review the spec against each checklist item:
      - For each item, determine if it passes or fails
      - Document specific issues found (quote relevant spec sections)

   e. **Handle Validation Results**:

      - **If all items pass**: Mark checklist complete and proceed to step 6

      - **If items fail (excluding [NEEDS CLARIFICATION])**:
        1. List the failing items and specific issues
        2. Update the spec to address each issue
        3. Re-run validation until all items pass (max 3 iterations)
        4. If still failing after 3 iterations, document remaining issues in checklist notes and warn user

      - **If [NEEDS CLARIFICATION] markers remain**:
        1. Extract all [NEEDS CLARIFICATION: ...] markers from the spec
        2. **LIMIT CHECK**: If more than 3 markers exist, keep only the 3 most critical (by scope/security/UX impact) and make informed guesses for the rest
        3. For each clarification needed (max 3), present options to user in this format:

           ```markdown
           ## Question [N]: [Topic]
           
           **Context**: [Quote relevant spec section]
           
           **What we need to know**: [Specific question from NEEDS CLARIFICATION marker]
           
           **Suggested Answers**:
           
           | Option | Answer | Implications |
           |--------|--------|--------------|
           | A      | [First suggested answer] | [What this means for the feature] |
           | B      | [Second suggested answer] | [What this means for the feature] |
           | C      | [Third suggested answer] | [What this means for the feature] |
           | Custom | Provide your own answer | [Explain how to provide custom input] |
           
           **Your choice**: _[Wait for user response]_
           ```

        4. **CRITICAL - Table Formatting**: Ensure markdown tables are properly formatted:
           - Use consistent spacing with pipes aligned
           - Each cell should have spaces around content: `| Content |` not `|Content|`
           - Header separator must have at least 3 dashes: `|--------|`
           - Test that the table renders correctly in markdown preview
        5. Number questions sequentially (Q1, Q2, Q3 - max 3 total)
        6. Present all questions together before waiting for responses
        7. Wait for user to respond with their choices for all questions (e.g., "Q1: A, Q2: Custom - [details], Q3: B")
        8. Update the spec by replacing each [NEEDS CLARIFICATION] marker with the user's selected or provided answer
        9. Re-run validation after all clarifications are resolved

   d. **Update Checklist**: After each validation iteration, update the checklist file with current pass/fail status

8. **Specification Review Gate** (Self-Check Pattern):

   **Purpose**: Validate specification completeness and present review summary before proceeding.

   **Self-Evaluation Checklist** (MUST answer honestly):

   ❓ **"Was large spec detection executed?"**
      ```yaml
      Action Required:
        - Check if spec size > 100KB or requirements > 60
        - Show ACTUAL spec size and requirement count
        - Report: Recommendation displayed (if applicable) or not needed

      Expected Evidence:
        Spec Analysis:
        ✓ Size: [N]KB (threshold: 100KB)
        ✓ Requirements: [M] total (threshold: 60)
        ✓ Recommendation: [DISPLAYED / NOT NEEDED]

        IF recommendation displayed:
          → User notified of hierarchical spec option
          → Examples provided (/speckit.supplement commands)
          → Benefits communicated (token efficiency, ownership)

        IF not needed (size ≤100KB AND requirements ≤60):
          → Spec size manageable for single file
          → No proactive recommendation required
      ```

   ❓ **"Are all requirements included?"**
      ```yaml
      Action Required:
        - Compare spec.md content vs user's feature description
        - Verify no key features omitted
        - Check all user stories covered

      Expected Check:
        ✅ All requirements from description covered
        ✅ User scenarios complete
        ✅ Success criteria defined

        OR identify:
        ❌ Missing: [list any requirements not covered]
      ```

   ❓ **"Are there ambiguous statements remaining?"**
      ```yaml
      Action Required:
        - Scan spec for vague adjectives ("fast", "intuitive", "robust") without metrics
        - Count [NEEDS CLARIFICATION] markers (should be ≤3)
        - Check success criteria are measurable

      Expected Check:
        ✅ No vague terms without quantification
        ✅ [NEEDS CLARIFICATION] markers: [N]/3 (within limit)
        ✅ All success criteria measurable

        OR identify:
        ⚠️ Ambiguous terms found: [list with line numbers]
        ⚠️ [NEEDS CLARIFICATION] count: [N] (over limit if >3)
      ```

   ❓ **"Are requirements testable?"**
      ```yaml
      Action Required:
        - Verify each functional requirement has acceptance criteria
        - Check no implementation details leaked (languages, frameworks, APIs)
        - Validate user stories have clear outcomes

      Expected Check:
        ✅ All requirements have acceptance criteria
        ✅ No tech stack mentioned in spec
        ✅ User stories testable

        OR identify:
        ❌ Requirements without acceptance criteria: [list]
        ❌ Implementation details found: [quote sections]
      ```

   ❓ **"Are user stories clear?"**
      ```yaml
      Action Required:
        - Validate user journey makes sense (no missing steps)
        - Check scenarios cover happy path and edge cases
        - Verify user roles/personas defined if multiple actors

      Expected Check:
        ✅ User journey complete (no gaps)
        ✅ Edge cases identified
        ✅ Multiple user types distinguished (if applicable)

        OR identify:
        ⚠️ Missing steps in journey: [describe gaps]
        ⚠️ Edge cases unclear: [list areas]
      ```

   **Evidence Collection**:
   ```yaml
   Sections Completed:
     - User Scenarios & Testing: ✅ / ⚠️ / ❌
     - Functional Requirements: ✅ / ⚠️ / ❌
     - Success Criteria: ✅ / ⚠️ / ❌
     - Key Entities: ✅ / ⚠️ / ❌ / N/A
     - Edge Cases: ✅ / ⚠️ / ❌
     - Assumptions: ✅ / ⚠️ / ❌

   Quality Metrics:
     - [NEEDS CLARIFICATION] markers: [N] (max 3)
     - Requirements count: [N]
     - Success criteria count: [N]
     - Quality checklist: [X]/[Y] items passed

   Issues Found:
     - [List any issues from self-check, or "None"]
   ```

   **Determine Status**:
   ```yaml
   Calculate overall readiness:

   ✅ READY:
     - All checklist items passed
     - ≤3 [NEEDS CLARIFICATION] markers
     - No critical issues

   ⚠️ NEEDS REVIEW:
     - Some checklist items failed but not critical
     - 1-3 [NEEDS CLARIFICATION] markers present
     - Minor ambiguities found

   ❌ NOT READY:
     - Multiple critical checklist failures
     - >3 [NEEDS CLARIFICATION] markers
     - Major sections missing
   ```

   **Output Format** (Present to User):
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   📋 Specification Review Complete
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

   Branch: [branch-name]
   Spec: [path/to/spec.md]
   Checklist: [path/to/checklists/requirements.md]

   ## Self-Check Results

   Large Spec Detection:
   ✓ Size: [N]KB (threshold: 100KB)
   ✓ Requirements: [M] total (threshold: 60)
   [IF size >100KB OR requirements >60:]
   💡 Recommendation displayed: Consider hierarchical specs (/speckit.supplement)
   [ELSE:]
   ✅ Spec size manageable for single file

   Requirements Coverage:
   ✅ All requirements from description included ([N] total)
   [OR: ❌ Missing requirements: [list]]

   Ambiguity Check:
   [IF ≤3 markers:]
   ⚠️ [N] clarifications needed (marked in spec)
   [ELSE IF >3 markers:]
   ❌ Too many clarifications needed ([N] found, reduced to 3 most critical)
   [ELSE:]
   ✅ No ambiguous statements

   Testability:
   ✅ All requirements testable ([N]/[N] with acceptance criteria)
   ✅ No implementation details leaked
   [OR: ❌ Issues found: [list]]

   User Stories:
   ✅ User journey complete
   ✅ Edge cases identified ([N] cases)
   [OR: ⚠️ Gaps found: [list]]

   ## Quality Checklist Status

   [Show checklist results from step 6]
   Quality Score: [X]/[Y] items passed ([%]%)

   ## Issues to Address

   [IF any issues found:]
   Priority Issues:
   - [Issue 1 with line number]
   - [Issue 2 with line number]

   Clarifications Needed:
   - [NEEDS CLARIFICATION]: [Topic] (line [N])
   - [NEEDS CLARIFICATION]: [Topic] (line [N])
   [up to 3]

   [IF no issues:]
   ✅ None - specification is complete and clear

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Next Steps
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [IF ✅ READY with no clarifications:]
   1. Review spec: [spec.md path]
   2. Proceed to planning: /speckit.plan

   [IF ⚠️ NEEDS REVIEW with clarifications:]
   1. Review spec: [spec.md path]
   2. Address clarifications: /speckit.clarify
   3. OR proceed if acceptable: /speckit.plan

   [IF ❌ NOT READY:]
   1. Review spec: [spec.md path]
   2. Address critical issues: [list specific actions]
   3. Re-run: /speckit.specify (to regenerate)

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Ready to proceed? (yes/review spec/address issues/regenerate)
   ```

   **User Interaction**:
   ```yaml
   IF user says "yes" or "proceed":
     → Suggest next command based on status:
        - If READY with no clarifications → /speckit.plan
        - If clarifications exist → /speckit.clarify
        - If NOT READY → List required fixes

   IF user says "review" or "review spec":
     → Provide spec file path
     → Wait for user feedback
     → Ask if they want to make changes

   IF user says "address issues":
     → List specific issues with line numbers
     → Offer to make corrections
     → Re-run review gate after fixes

   IF user says "regenerate":
     → Warn: "This will recreate the spec from scratch"
     → Ask for confirmation
     → If confirmed, start over from step 1

   IF user provides feedback or changes:
     → Apply requested changes to spec
     → Re-run validation (step 6)
     → Re-run review gate (step 7)
   ```

   **Benefits** (from PM Agent patterns):
   ```yaml
   Evidence-Based Validation:
     - ✅ No "looks good" without checking
     - ✅ Actual issue detection before proceeding
     - ✅ Transparent quality metrics

   Early Issue Detection:
     - ✅ Catches problems before planning phase
     - ✅ Prevents downstream rework (5K-20K token savings)
     - ✅ User confidence in spec quality

   Hallucination Prevention:
     - ✅ Forces self-check against actual requirements
     - ✅ No claiming "complete" without evidence
     - ✅ Explicit status (READY/NEEDS REVIEW/NOT READY)
   ```

9. Report completion with branch name, spec file path, checklist results, and readiness for the next phase (`/speckit.clarify` or `/speckit.plan`).

**NOTE:** The script creates and checks out the new branch and initializes the spec file before writing.

## General Guidelines

## Quick Guidelines

- Focus on **WHAT** users need and **WHY**.
- Avoid HOW to implement (no tech stack, APIs, code structure).
- Written for business stakeholders, not developers.
- DO NOT create any checklists that are embedded in the spec. That will be a separate command.

### Section Requirements

- **Mandatory sections**: Must be completed for every feature
- **Optional sections**: Include only when relevant to the feature
- When a section doesn't apply, remove it entirely (don't leave as "N/A")

### For AI Generation

When creating this spec from a user prompt:

1. **Make informed guesses**: Use context, industry standards, and common patterns to fill gaps
2. **Document assumptions**: Record reasonable defaults in the Assumptions section
3. **Limit clarifications**: Maximum 3 [NEEDS CLARIFICATION] markers - use only for critical decisions that:
   - Significantly impact feature scope or user experience
   - Have multiple reasonable interpretations with different implications
   - Lack any reasonable default
4. **Prioritize clarifications**: scope > security/privacy > user experience > technical details
5. **Think like a tester**: Every vague requirement should fail the "testable and unambiguous" checklist item
6. **Common areas needing clarification** (only if no reasonable default exists):
   - Feature scope and boundaries (include/exclude specific use cases)
   - User types and permissions (if multiple conflicting interpretations possible)
   - Security/compliance requirements (when legally/financially significant)

**Examples of reasonable defaults** (don't ask about these):

- Data retention: Industry-standard practices for the domain
- Performance targets: Standard web/mobile app expectations unless specified
- Error handling: User-friendly messages with appropriate fallbacks
- Authentication method: Standard session-based or OAuth2 for web apps
- Integration patterns: RESTful APIs unless specified otherwise

### Success Criteria Guidelines

Success criteria must be:

1. **Measurable**: Include specific metrics (time, percentage, count, rate)
2. **Technology-agnostic**: No mention of frameworks, languages, databases, or tools
3. **User-focused**: Describe outcomes from user/business perspective, not system internals
4. **Verifiable**: Can be tested/validated without knowing implementation details

**Good examples**:

- "Users can complete checkout in under 3 minutes"
- "System supports 10,000 concurrent users"
- "95% of searches return results in under 1 second"
- "Task completion rate improves by 40%"

**Bad examples** (implementation-focused):

- "API response time is under 200ms" (too technical, use "Users see results instantly")
- "Database can handle 1000 TPS" (implementation detail, use user-facing metric)
- "React components render efficiently" (framework-specific)
- "Redis cache hit rate above 80%" (technology-specific)

## State Management Integration (Phase 1.2)

After specification is written and validated:

1. **Initialize Feature State**: Create state tracking file at `.specify/memory/features/SPEC_DIR_NAME/state.json`

   ```json
   {
     "feature_num": "FEATURE_NUM",
     "feature_name": "SPEC_DIR_NAME",
     "branch_name": "BRANCH_NAME",
     "spec_dir": "SPEC_DIR_NAME",
     "phase": "SPECIFYING",
     "progress": 15,
     "created_at": "[ISO-8601-TIMESTAMP]",
     "updated_at": "[ISO-8601-TIMESTAMP]",
     "gates_passed": ["spec_created"],
     "gates_failed": [],
     "commands_executed": [
       {
         "command": "specify",
         "timestamp": "[ISO-8601-TIMESTAMP]",
         "success": true,
         "clarifications_pending": [COUNT of [NEEDS CLARIFICATION] markers]
       }
     ],
     "clarifications_count": 0,
     "review_scores": {
       "clarity": null,
       "completeness": null,
       "testability": null,
       "consistency": null
     },
     "blockers": [],
     "notes": "Initial spec created. Ready for clarification phase.",
     "reconciliation_cycles": 0,
     "gaps_identified": [],
     "sync_reports": [],
     "supplementary_specs": {
       "enabled": false,
       "specs": []
     }
   }
   ```

   **Note on Hierarchical Specs (v2.1)**: The `supplementary_specs` object tracks hierarchical specifications:
   - `enabled`: Set to `true` when `/speckit.supplement` creates UI-SPEC.md, API-SPEC.md, or TECHNICAL-SPEC.md
   - `specs`: Array of supplementary spec metadata (populated by supplement command)

   Example after `/speckit.supplement ui-ux "..."`:
   ```json
   "supplementary_specs": {
     "enabled": true,
     "specs": [
       {
         "filename": "UI-SPEC.md",
         "scope": "ui-ux-implementation",
         "created_at": "[ISO-8601-TIMESTAMP]",
         "target_agents": ["frontend-architect"],
         "size_kb": 182,
         "fr_coverage": "FR-001 through FR-080"
       }
     ]
   }
   ```

2. **Update PM Context**: Update `.specify/memory/pm_context.md` with new active feature

3. **Update Next Actions**: Write `.specify/memory/next_actions.md`:

   ```markdown
   # Next Actions

   **Updated**: [TIMESTAMP]
   **Feature**: SPEC_DIR_NAME

   ## Immediate (Next Command)

   - `/speckit.clarify` - Clarify [COUNT] underspecified areas in spec

   ## Prerequisites

   - [x] Specification created
   - [ ] Clarifications completed
   - [ ] Plan created

   ## Notes

   Spec has [COUNT] [NEEDS CLARIFICATION] markers. Run `/speckit.clarify` to resolve them before planning.
   ```

3. **Create Specification Metadata**: Create `spec-metadata.json` in the spec directory for phase tracking and approval gates:

   ```json
   {
     "$schema": "http://json-schema.org/draft-07/schema#",
     "version": "2.3.0",
     "feature_name": "{short-name}",
     "phase": "specification",
     "approvals": {
       "specification": {
         "generated": true,
         "approved": false,
         "timestamp": "{ISO 8601 timestamp}"
       },
       "gap_analysis": {
         "generated": false,
         "approved": false,
         "timestamp": null
       },
       "planning": {
         "generated": false,
         "approved": false,
         "timestamp": null
       },
       "tasks": {
         "generated": false,
         "approved": false,
         "timestamp": null
       },
       "implementation": {
         "generated": false,
         "approved": false,
         "timestamp": null
       },
       "reconciliation": {
         "generated": false,
         "approved": false,
         "timestamp": null
       }
     },
     "metadata": {
       "created_at": "{ISO 8601 timestamp}",
       "updated_at": "{ISO 8601 timestamp}",
       "created_by": "speckit",
       "spec_version": "1.0",
       "risk_level": "{HIGH/MEDIUM/LOW from Risk Assessment section}",
       "overall_quality": {quality_score_average}
     }
   }
   ```

   **Determine Next Phase Based on Risk**:
   - IF risk_level == "HIGH":
     → Set phase to "gap_analysis" (requires feasibility assessment)
     → Recommend: "⚠️ HIGH RISK feature - run `/speckit.validate-gap` before planning"
   - ELSE IF risk_level == "MEDIUM":
     → Set phase to "gap_analysis" (gap analysis optional but recommended)
     → Recommend: "📋 MEDIUM RISK - consider running `/speckit.validate-gap`"
   - ELSE (LOW risk):
     → Set phase to "specification" (gap analysis optional)
     → Recommend: "Run `/speckit.clarify` (optional) or proceed to `/speckit.plan`"

**Note**: If `.specify/memory/` directory doesn't exist, create it with necessary structure (memory/, memory/features/).
