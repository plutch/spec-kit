---
description: Create or update the feature specification from a natural language feature description.
scripts:
  sh: scripts/bash/create-new-feature.sh --json "{ARGS}"
  ps: scripts/powershell/create-new-feature.ps1 -Json "{ARGS}"
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

The text the user typed after `/speckit.specify` in the triggering message **is** the feature description. Assume you always have it available in this conversation even if `{ARGS}` appears literally below. Do not ask the user to repeat it unless they provided an empty command.

Given that feature description, do this:

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
      - Remote branches: `git ls-remote --heads origin | grep -E 'refs/heads/[0-9]+-<short-name>$'`
      - Local branches: `git branch | grep -E '^[* ]*[0-9]+-<short-name>$'`
      - Specs directories: Check for directories matching `specs/[0-9]+-<short-name>`
   
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
       Each requirement must be testable
       Use reasonable defaults for unspecified details (document assumptions in Assumptions section)
    6. Define Success Criteria
       Create measurable, technology-agnostic outcomes
       Include both quantitative metrics (time, performance, volume) and qualitative measures (user satisfaction, task completion)
       Each criterion must be verifiable without implementation details
    7. Identify Key Entities (if data involved)
    8. Return: SUCCESS (spec ready for planning)

6. Write the specification to SPEC_FILE using the template structure, replacing placeholders with concrete details derived from the feature description (arguments) while preserving section order and headings.

7. **Specification Quality Validation**: After writing the initial spec, validate it against quality criteria:

   a. **Create Spec Quality Checklist**: Generate a checklist file at `FEATURE_DIR/checklists/requirements.md` using the checklist template structure with these validation items:

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

   b. **Run Validation Check**: Review the spec against each checklist item:
      - For each item, determine if it passes or fails
      - Document specific issues found (quote relevant spec sections)

   c. **Handle Validation Results**:

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

   ❌ INCOMPLETE:
     - Multiple critical checklist failures
     - >3 [NEEDS CLARIFICATION] markers
     - Major sections missing
   ```

   **Output Format** (Present to User):
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   📋 Specification Review Complete
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ INCOMPLETE]

   Branch: [branch-name]
   Spec: [path/to/spec.md]
   Checklist: [path/to/checklists/requirements.md]

   ## Self-Check Results

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

   [IF ❌ INCOMPLETE:]
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
        - If INCOMPLETE → List required fixes

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
     - ✅ Explicit status (READY/NEEDS REVIEW/INCOMPLETE)
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

**Note**: If `.specify/memory/` directory doesn't exist, create it with necessary structure (memory/, memory/features/).
