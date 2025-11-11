# Integration Point 2: Specification Review Gate
# File: src/.claude/commands/speckit.specify.md
# Location: INSERT BEFORE current Step 8 (line ~1076), becomes new Step 8
# Current Step 8 becomes Step 9

---

8. **Specification Review Gate** 🆕 (Mandatory Approval):

   **Purpose:** User reviews generated specification summary and explicitly approves before proceeding. This ensures AI-generated spec matches user intent and prevents costly downstream misunderstandings.

   **Step 8.1: Generate Specification Summary**

   Read the spec file that was just written and create a concise summary:

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   📋 Specification Review
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   I've generated a specification for **[FEATURE NAME from spec.md]**.

   Here's a summary for your review:

   ## Requirements Summary

   **Total Requirements:**
   - Functional: [COUNT of REQ-* requirements]
   - Non-Functional: [COUNT of NFR-* requirements]

   **Top Requirements (EARS Format):**

   [Extract first 5-7 EARS requirements from spec.md, showing full text]

   1. **[REQ-ID]**: [Full EARS requirement text]
      - Pattern: [WHEN/IF/WHILE/WHERE]
      - Verifiable: [How to test this]

   2. **[REQ-ID]**: [Full EARS requirement text]
      - Pattern: [WHEN/IF/WHILE/WHERE]
      - Verifiable: [How to test this]

   [... continue for top 5-7 requirements ...]

   [IF total requirements > 7:]
   _(Plus [N] additional requirements - see spec.md for full list)_

   ---

   ## Risk Assessment

   **Overall Feature Risk:** [Extract from spec.md Risk Assessment section]
   - Risk Level: [🔴 HIGH | 🟠 MEDIUM | 🟢 LOW]
   - Risk Score: [X]/12

   [IF risk level is MEDIUM or HIGH:]
   **High/Medium-Risk Areas:**
   [Extract top 3 risk factors from Risk Assessment section]
   1. [Risk area 1]: [Brief description + mitigation]
   2. [Risk area 2]: [Brief description + mitigation]
   3. [Risk area 3]: [Brief description + mitigation]

   **Testing Strategy Implications:**
   [Extract from Risk Assessment → Testing Strategy Recommendations]
   - [e.g., "Security testing required"]
   - [e.g., "Performance testing under load"]

   ---

   ## Architecture Impact

   **What This Feature Affects:**

   [Infer from functional requirements and spec sections:]

   **Modify Existing:**
   [If spec mentions extending existing code, list inferred files/components]
   - [e.g., "src/auth/auth.service.ts - Add OAuth2Provider"]
   - [e.g., "Database schema - Add oauth_tokens table"]

   **Create New:**
   [If spec describes new components, list inferred new files]
   - [e.g., "src/auth/providers/oauth2.provider.ts"]
   - [e.g., "tests/integration/auth/oauth2.integration.spec.ts"]

   **External Integrations:**
   [If spec mentions external services, list them]
   - [e.g., "Auth0 (OAuth provider)"]
   - [e.g., "Redis (token caching)"]

   [IF discovery_insights exist from Step 1:]
   **Based on Your Discovery Answers:**
   - [e.g., "OAuth 2.0 with Authorization Code flow"]
   - [e.g., "Tokens stored in httpOnly cookies"]
   - [e.g., "Multi-tenant support with row-level security"]

   ---

   ## Estimated Complexity

   [Calculate based on requirements count, risk score, and integrations:]

   **Size Estimate:** [🔴 LARGE | 🟠 MEDIUM | 🟢 SMALL]

   **Effort Estimate:**
   [Use this logic:]
   - SMALL: 1-3 days (0-3 requirements, LOW risk, no external integrations)
   - MEDIUM: 4-7 days (4-10 requirements, MEDIUM risk, 1-2 integrations)
   - LARGE: 8-15 days (10+ requirements, HIGH risk, 3+ integrations)

   Estimated: **[X-Y days]**

   **Complexity Factors:**
   - Requirements: [N] functional + [M] non-functional
   - Risk Level: [HIGH/MEDIUM/LOW] (security/performance considerations)
   - External Dependencies: [N] external services
   [IF multi-tenant from discovery:]
   - Multi-tenancy: Adds complexity to data isolation

   ---

   ## Quality Scores

   [Extract from spec.md Specification Quality Assessment section:]

   - **Clarity:** [X]/10 (language precision, unambiguous terms)
   - **Completeness:** [Y]/10 (scenario coverage, edge cases)
   - **Testability:** [Z]/10 (measurable acceptance criteria)
   - **Consistency:** [W]/10 (no contradictions)

   **Overall Quality:** [AVERAGE]/10

   **Quality Status:** [Extract from spec.md:]
   - [✅ Ready for Clarify | ⚠️ Needs Review | ❌ Major Issues]

   [IF quality score < 7 OR status is "Needs Review" or "Major Issues":]

   **⚠️ Areas Needing Attention:**

   [Extract from "Areas Needing Attention" in spec.md Quality Assessment:]
   1. [Issue 1 - Section reference]
   2. [Issue 2 - Section reference]
   3. [Issue 3 - Section reference]

   **Recommendation:**
   These quality issues should be addressed. You can:
   - Approve now and run `/speckit.clarify` to improve quality
   - Request revisions now (type "revise [aspect]")

   [END IF]

   [IF spec has UX Quality Assessment section:]

   ## UX Quality Scores

   [Extract from UX Quality Assessment:]

   - **Usability:** [X]/10 (ease of use, learnability)
   - **Efficiency:** [Y]/10 (task completion speed)
   - **Accessibility:** [Z]/10 (WCAG compliance)
   - **Error Prevention:** [W]/10 (mistake recovery)
   - **Delight:** [V]/10 (user satisfaction)

   **Overall UX Quality:** [AVERAGE]/10

   **UX Status:** [✅ UX Ready | ⚠️ UX Needs Improvement | ❌ UX Issues]

   [IF UX score < 7:]
   **UX Areas Needing Attention:**
   [Extract top 3 UX issues from spec.md]

   [END IF - UX section]

   ---

   ## Clarification Markers

   [Count [NEEDS CLARIFICATION] markers in spec.md:]

   **Remaining Ambiguities:** [COUNT] markers

   [IF count > 0:]
   The spec has [COUNT] areas marked [NEEDS CLARIFICATION]:
   [Extract and list each marker with context]
   1. [Section] - [Marker text]
   2. [Section] - [Marker text]
   [...]

   **Recommendation:** Run `/speckit.clarify` after approval to resolve these.

   [ELSE:]
   ✅ No ambiguities marked - spec is well-defined based on current understanding.

   [END IF]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ⏸️ WAITING_FOR_SPECIFICATION_APPROVAL
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Please review this summary and choose one option:

   **Options:**

   1. **"approved"** or **"yes"** or **"looks good"**
      → Proceed to next workflow phase
      → Updates spec-metadata.json with approval timestamp

   2. **"revise [aspect]"**
      → Modify specification (examples below)
      → Regenerate affected sections
      → Return to this review gate with updated spec

      **Revision examples:**
      - "revise token storage to use Redis instead of cookies"
      - "revise risk level - this is actually LOW risk"
      - "revise session duration to 24 hours instead of 15 minutes"

   3. **"clarify [question]"**
      → Ask follow-up questions about the spec
      → I'll answer based on spec.md content
      → Return to this review gate

      **Clarification examples:**
      - "clarify what 'multi-tenant support' means technically"
      - "clarify how error handling works"
      - "clarify the testing strategy for HIGH risk items"

   4. **"show full spec"** or **"show spec"**
      → Display complete spec.md file
      → Return to this review gate for approval

   5. **"skip"** or **"bypass"** (NOT RECOMMENDED)
      → Skip approval process
      → Warning will be shown
      → Requires confirmation

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   **I will not proceed until you explicitly approve or request revisions.**

   Your response:
   ```

   **Step 8.2: Wait for User Response and Process**

   Enter approval loop - block until valid response received:

   ```yaml
   # Initialize revision counter
   revision_count = 0
   revision_history = []

   WHILE approval not received:

     user_response = [wait for user input]

     # Option 1: APPROVED
     IF user_response matches ["approved", "approve", "yes", "looks good", "lgtm", "ok", "proceed"]:

       → Update spec-metadata.json:
         {
           "phase": "specification",
           "approvals": {
             "specification": {
               "generated": true,
               "approved": true,
               "timestamp": "[ISO 8601 current timestamp]",
               "discovery_mode": "[domain-specific | generic | skipped]",
               "discovery_questions_answered": [N from Step 1],
               "revisions_requested": [revision_count],
               "revision_history": [revision_history array],
               "approval_skipped": false
             }
           },
           "metadata": {
             "risk_level": "[HIGH/MEDIUM/LOW from spec.md]",
             "overall_quality": [quality score from spec.md],
             "markers_count": [count of [NEEDS CLARIFICATION]],
             "last_updated": "[ISO 8601 timestamp]"
           }
         }

       → Show confirmation:
         "✅ Specification approved!

         Approval recorded at [timestamp]
         Revisions during review: [revision_count]

         Next steps:
         [IF markers_count > 0:]
         - Recommended: Run `/speckit.clarify` to resolve [N] ambiguities
         [END IF]
         [IF risk_level == "HIGH":]
         - Required: Run `/speckit.validate-gap` (HIGH-risk feature)
         [ELSE IF risk_level == "MEDIUM":]
         - Recommended: Run `/speckit.validate-gap` or proceed to `/speckit.plan`
         [ELSE:]
         - Proceed to `/speckit.plan` when ready
         [END IF]"

       → BREAK loop (proceed to Step 9)

     # Option 2: REVISE
     ELSE IF user_response starts with "revise":

       → Extract aspect to revise:
         aspect = [text after "revise " in user response]
         # Example: "revise token storage to use Redis" → aspect = "token storage to use Redis"

       → Ask clarifying question about revision:
         "🔄 Revision Request: [aspect]

         To make this change accurately, I need to clarify:

         [Generate 1-2 clarifying questions based on aspect]

         [IF aspect mentions "token storage" or similar auth concept:]
         - Which storage mechanism? (Redis, database, in-memory, etc.)
         - What TTL/expiration should tokens have?

         [IF aspect mentions "risk level":]
         - Why should risk be [new level]?
         - Which risk factors changed? (data sensitivity, access control, etc.)

         [IF aspect mentions performance/timing:]
         - What's the new target metric?
         - Any infrastructure changes needed?

         [GENERIC fallback:]
         - What specifically should change in the spec?
         - How does this affect requirements/architecture?

         Your clarification:"

       → Wait for clarification response

       → Apply revision to spec.md:
         [Read spec.md, identify relevant sections based on aspect, regenerate those sections]

         [IF aspect is "token storage":]
           → Update auth requirements in Functional Requirements
           → Update Data Model if storage location changes
           → Update Risk Assessment if security implications change
           → Update Non-Functional Requirements for performance impacts

         [IF aspect is "risk level":]
           → Recalculate Risk Assessment section
           → Update Testing Strategy Recommendations
           → Adjust Estimated Complexity

         [IF aspect is general requirement change:]
           → Update specific requirement(s) in Functional Requirements
           → Check for cascading changes in related requirements
           → Update Success Criteria if affected

       → Write updated spec.md

       → Increment revision counter:
         revision_count += 1

       → Record in revision history:
         revision_history.append({
           "timestamp": "[ISO 8601 timestamp]",
           "aspect": "[aspect text]",
           "change": "[brief description of what was changed]"
         })

       → Return to Step 8.1:
         "✅ Revision applied: [brief summary of change]

         Regenerating specification summary with updates..."

         [Generate new summary with updated content, return to review gate]

     # Option 3: CLARIFY
     ELSE IF user_response starts with "clarify":

       → Extract question:
         question = [text after "clarify " in user response]

       → Answer question by reading relevant spec.md sections:

         [IF question mentions "risk" or "testing":]
           → Read Risk Assessment section
           → Explain risk factors and mitigation strategies
           → Quote specific requirements that drive risk score

         [IF question mentions "requirements" or specific REQ-ID:]
           → Read Functional Requirements section
           → Explain the requirement in detail
           → Show how it maps to user scenarios

         [IF question mentions "architecture" or "technical":]
           → Read Functional Requirements + Non-Functional Requirements
           → Infer architectural implications
           → Explain integration points

         [IF question mentions "testing" or "quality":]
           → Read Testing Strategy from Risk Assessment
           → Read Quality Assessment scores
           → Explain testing approach and quality measures

         [GENERIC:]
           → Search spec.md for keywords from question
           → Summarize relevant sections
           → Provide context

       → Show answer:
         "💡 Clarification: [question]

         [Answer based on spec.md content, 2-4 paragraphs]

         [Quote relevant sections from spec.md]

         Does this answer your question?

         You can:
         - Ask another clarification question
         - Request a revision if something should change
         - Approve the spec if satisfied"

       → Return to approval loop (don't change state, just re-ask for approval)

     # Option 4: SHOW FULL SPEC
     ELSE IF user_response matches ["show full spec", "show spec", "show complete spec", "view spec"]:

       → Read and display complete spec.md file:
         "📄 Complete Specification (spec.md)

         [Display full contents of spec.md with line numbers]

         ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

         End of specification.

         Please review and choose:
         - Type 'approved' to approve
         - Type 'revise [aspect]' to make changes
         - Type 'clarify [question]' for more details"

       → Return to approval loop

     # Option 5: SKIP (with warning and confirmation)
     ELSE IF user_response matches ["skip", "bypass", "skip approval", "no approval"]:

       → Show warning:
         "⚠️ WARNING: Skipping Approval Gate

         Skipping specification approval is NOT RECOMMENDED because:
         - Spec may contain assumptions that don't match your intent
         - Misunderstandings discovered late cost 10K-20K tokens in rework
         - Implementation may proceed in wrong direction

         This approval gate exists to catch these issues early.

         Are you sure you want to skip approval? (yes/no)

         [If you're testing or in CI/CD, consider using --auto-approve flag instead]"

       → Wait for confirmation

       IF user confirms with "yes":
         → Update spec-metadata.json with skipped flag:
           {
             "approvals": {
               "specification": {
                 "generated": true,
                 "approved": true,
                 "timestamp": "[ISO 8601 timestamp]",
                 "approval_skipped": true,
                 "skip_reason": "user requested",
                 "revisions_requested": 0
               }
             }
           }

         → Log warning to console/metadata:
           "⚠️ Specification approval skipped by user request"

         → BREAK loop (proceed to Step 9 with warning)

       ELSE:
         → "Approval skip cancelled. Returning to review gate."
         → Return to approval loop

     # Option 6: Unclear response
     ELSE:
       → "❓ I didn't understand that response.

       Please respond with one of:
       - **'approved'** - Accept the specification as-is
       - **'revise [aspect]'** - Make changes (e.g., 'revise token storage')
       - **'clarify [question]'** - Ask for more details
       - **'show full spec'** - View complete specification
       - **'skip'** - Skip approval (not recommended)

       Your response:"

       → Continue approval loop (ask again)

   END WHILE (approval loop)
   ```

   **Step 8.3: Record Approval Metadata**

   After breaking approval loop (either approved or skipped), ensure spec-metadata.json is written with complete approval context.

   ```json
   {
     "$schema": "http://json-schema.org/draft-07/schema#",
     "version": "2.7.0",
     "feature_name": "{short-name}",
     "phase": "specification",
     "approvals": {
       "specification": {
         "generated": true,
         "approved": true,
         "timestamp": "2025-11-11T14:30:00Z",
         "discovery_mode": "domain-specific",
         "discovery_domain": "authentication",
         "discovery_questions_answered": 4,
         "revisions_requested": 1,
         "revision_history": [
           {
             "timestamp": "2025-11-11T14:25:00Z",
             "aspect": "token storage",
             "change": "Changed from localStorage to httpOnly cookies for XSS protection"
           }
         ],
         "approval_skipped": false
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
       }
     },
     "metadata": {
       "created_at": "2025-11-11T14:00:00Z",
       "updated_at": "2025-11-11T14:30:00Z",
       "created_by": "speckit-v2.7",
       "spec_version": "1.0",
       "risk_level": "MEDIUM",
       "overall_quality": 82,
       "markers_count": 2,
       "ux_quality": 78
     }
   }
   ```

   **Step 8.4: Configuration Override Handling**

   Before entering approval loop in Step 8.2, check configuration:

   ```yaml
   # Read .specify/config.yml (if exists)
   IF config.yml exists AND has discovery_gates section:

     IF discovery_gates.specification_review_gate.enabled == false:
       → Skip entire Step 8 (go directly to Step 9)
       → Log: "Specification review gate disabled via config"
       → Set approval_skipped: true with skip_reason: "config disabled"

     ELSE IF discovery_gates.specification_review_gate.auto_approve == true:
       → Skip approval loop
       → Automatically approve with warning:
         "⚠️ Auto-approval enabled via config (non-interactive mode)"
       → Set approval_skipped: false but add auto_approved: true flag
       → Proceed to Step 9

     ELSE IF environment variable SPECKIT_AUTO_APPROVE == "true":
       → Same as auto_approve config above
       → Log: "Auto-approval enabled via SPECKIT_AUTO_APPROVE env var"

     ELSE:
       → Normal approval loop (Step 8.2)

   ELSE (no config or no discovery_gates section):
     → Normal approval loop (Step 8.2) with default behavior

   # Command-line flag handling
   IF command was invoked with --auto-approve flag:
     → Override config, auto-approve with warning
     → Set auto_approved: true

   IF command was invoked with --skip-gates flag:
     → Skip entire Step 8, set approval_skipped: true
   ```

   **Step 8.5: Next Steps Recommendation**

   After approval recorded, show context-aware next steps:

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ✅ Specification Approved
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Feature: [FEATURE NAME]
   Branch: [BRANCH NAME]
   Spec: [PATH to spec.md]

   **Approval Summary:**
   - Approved at: [timestamp]
   - Discovery mode: [domain-specific | generic | skipped]
   - Revisions made: [N]
   - Quality score: [X]/10
   - Risk level: [HIGH/MEDIUM/LOW]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Recommended Next Steps
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [IF risk_level == "HIGH":]
   **1. Gap Analysis (REQUIRED for HIGH-risk)**
      `/speckit.validate-gap`
      → Assess feasibility, identify reusable components
      → Required before planning for HIGH-risk features

   [IF markers_count > 0:]
   **2. Clarify Ambiguities (RECOMMENDED - [N] markers remain)**
      `/speckit.clarify`
      → Resolve [N] [NEEDS CLARIFICATION] markers
      → Improves spec quality before planning

   [IF risk_level == "MEDIUM":]
   **3. Gap Analysis (RECOMMENDED for MEDIUM-risk)**
      `/speckit.validate-gap`
      → Optional but recommended for feasibility check

   **4. Proceed to Planning**
      `/speckit.plan`
      → Generate implementation plan with constitutional validation

   [IF quality_score < 7:]
   **⚠️ Quality Note:**
   Overall quality score is [X]/10. Consider running `/speckit.clarify` or `/speckit.clarify --expert` to improve quality before planning.

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## View Specification
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   cat [PATH to spec.md]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ```

---

**END OF SPECIFICATION REVIEW GATE (STEP 8)**

This becomes the new Step 8, inserted before current Step 8 (presentation message).
Current Step 8 becomes Step 9 and is modified to reference the approval context.

**Modified Step 9 (previously Step 8):**

9. Present completion message with next steps (EXISTING, now references approval):

   [Simplified version since detailed next steps shown in Step 8.5:]

   ```markdown
   Feature specification complete!

   Specification file: [SPEC_FILE path]
   Metadata file: spec-metadata.json (tracks approvals and phase progression)

   [Show command to view spec]
   [Show command to proceed based on risk level - same as Step 8.5 recommendations]
   ```

---

**Token Cost Analysis for Step 8:**
- Summary generation: 200-300 tokens
- Approval loop (no revisions): 50-100 tokens
- Approval loop (1 revision): +150-200 tokens
- Total: 300-500 tokens (mandatory, every feature)
