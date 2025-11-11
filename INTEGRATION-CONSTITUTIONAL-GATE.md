# Integration Point 3: Constitutional Approval Gate
# File: src/.claude/commands/speckit.plan.md
# Location: INSERT AFTER Phase -1 validation, BEFORE final plan write

---

## Phase -1 Approval Gate 🆕 (Mandatory Constitutional Approval)

**Purpose:** User reviews constitutional validation results and explicitly approves plan before implementation begins. This ensures plan adheres to project principles and prevents constitutional drift.

**Context:** This gate runs AFTER Phase -1 (Constitutional Validation) completes but BEFORE the plan is finalized. It reads the Phase -1 validation result and requires user confirmation.

---

### Step -1.1: Present Constitutional Validation Results

After Phase -1 validation completes, present results to user:

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
## Phase -1: Constitutional Validation Result
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Overall Status:** [Read from plan.md Phase -1 section: ✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL]

### Article-by-Article Results

[Read each article validation result from plan.md Phase -1 section:]

**Article I: Simplicity & Framework Minimalism**
- **Status:** [✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL]
- **New Frameworks Added:** [N]
- **Justification:** [Extract from plan.md Complexity Tracking if frameworks added]
- **Details:** [Extract specific findings from Phase -1 Article I validation]

**Article V: Integration-First Testing**
- **Status:** [✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL]
- **Test Strategy:** [Extract from Phase 2.5 Mock & Stub Strategy]
- **Integration Tests:** [Real dependencies | Mocked dependencies]
- **Article V Compliance:** [Extract Phase 2.6 validation result if exists]
- **Details:** [Specific findings about test strategy compliance]

**Article IX: Prohibited Patterns**
- **Status:** [✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL]
- **Patterns Detected:** [N prohibited patterns found]
[IF patterns found > 0:]
  **Violations:**
  [List each violation from Phase -1:]
  - Pattern: [name] (e.g., "Synchronous File Operations")
  - Location: [where it would be used, from plan phases]
  - Reason Prohibited: [why it violates constitution]
  - Remediation: [suggested fix]
[ELSE:]
  - No prohibited patterns detected in planned implementation
[END IF]

[... Continue for all other validated articles ...]

---

### Validation Summary

**Article Breakdown:**
- ✅ **PASS**: [COUNT] articles passed without issues
- ⚠️ **CONDITIONAL**: [COUNT] articles passed with conditions
- ❌ **FAIL**: [COUNT] articles failed validation

[IF CONDITIONAL count > 0:]

**Conditional Approvals Required:**

[For each CONDITIONAL article, extract from plan.md:]

1. **Article [Roman Numeral]: [Article Name]**
   - **Condition:** [What requires justification]
   - **Justification Provided:** [Extract from Complexity Tracking or Phase -1]
   - **Impact:** [How this affects implementation]
   - **Acceptable?** This requires your review and approval.

   [Example:]
   **Article I: Simplicity**
   - **Condition:** Plan adds Redis as new framework (violates minimalism)
   - **Justification:** "Reduces API latency by 80% (2s → 400ms), critical for user experience"
   - **Impact:** Adds Redis dependency, increases infrastructure complexity
   - **Acceptable?** Justification shows clear ROI, but increases operational burden

[END IF - conditional section]

[IF FAIL count > 0:]

**⛔ Violations Requiring Fix:**

[For each FAILED article, extract from plan.md:]

1. **Article [Roman Numeral]: [Article Name]**
   - **Violation:** [What violates the article]
   - **Location:** [Where in plan this occurs]
   - **Why This Matters:** [Impact of violation]
   - **Required Fix:** [What needs to change]

   [Example:]
   **Article V: Integration-First Testing**
   - **Violation:** Phase 2.5 Mock & Stub Strategy uses `jest.mock('auth0')` in integration tests
   - **Location:** Phase 2.5 → Mock & Stub Strategy
   - **Why This Matters:** Violates Integration-First principle - auth must use real Auth0 test tenant
   - **Required Fix:** Change test strategy to use real Auth0 test tenant with test credentials

[END IF - fail section]

---

### Security & Compliance Triggers

[IF any Phase 3 security issues found:]

**Security Review Required:**
[Extract from Phase 3 validation:]
- OWASP Top 10 items flagged: [COUNT]
- Data sensitivity level: [HIGH/MEDIUM/LOW]
- Recommended actions: [List from Phase 3]

[END IF]

---

### Post-Implementation Checkpoint

**Automated Enforcement Points:**

This plan, if approved, will be validated again during implementation:

**Step 2.5 (Pre-Flight Check):**
- Verify this planning approval before execution begins
- Block implementation if approval missing

**Step 10.4 (Constitutional Reviewer):**
- Re-validate constitutional compliance during Quality Gate
- Evidence-based checks:
  - 10.4.1: TDD Compliance (git history analysis)
  - 10.4.2: Simplicity (framework count validation)
  - 10.4.3: Anti-Abstraction (pattern detection)
  - 10.4.4: Integration-First (contract test presence)
  - 10.4.5: Prohibited Patterns (automated scanning)
  - 10.4.6: Complexity Drift (TECHNICAL.md validation)

These automated checks ensure compliance throughout implementation.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏸️ WAITING_FOR_CONSTITUTIONAL_APPROVAL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[IF status == ❌ FAIL:]

⛔ **CANNOT PROCEED** - Plan violates constitutional principles.

This plan has [COUNT] constitutional violations that must be fixed.

**You must either:**
1. **Fix violations** and re-run `/speckit.plan`
   - Address each violation listed above
   - Revise planning phases to comply with constitution
2. **Update constitution.md** if principles need revision
   - Only do this if the constitution is outdated
   - Document why principle change is necessary

**I cannot proceed to tasks/implementation until violations are resolved.**

Type "fix [violation]" to revise the plan, or "show constitution" to view constitution.md.

[ELSE IF status == ⚠️ CONDITIONAL:]

⚠️ **CONDITIONAL APPROVAL NEEDED**

This plan has [COUNT] conditional items requiring your review.

**Conditional items indicate:**
- Added complexity justified by clear benefits
- Trade-offs that need explicit approval
- Deviations from ideal that are acceptable

**Review the conditional items above and decide:**

**Options:**
1. **"approved"** - Accept conditional items (acknowledge trade-offs)
2. **"fix [item]"** - Revise plan to remove conditional complexity
3. **"explain [article]"** - Show detailed constitutional reasoning
4. **"show constitution"** - View full constitution.md

**I will not proceed until you explicitly approve or request fixes.**

[ELSE IF status == ✅ PASS:]

✅ **READY TO PROCEED** - Plan complies with all constitutional principles.

No constitutional violations detected. Plan adheres to project governance.

**Options:**
1. **"approved"** - Proceed to implementation
2. **"explain [article]"** - Show why specific article passed
3. **"show constitution"** - View full constitution.md
4. **"review plan"** - View complete plan.md before approving

**Type "approved" to proceed to tasks/implementation.**

[END IF - status-specific messaging]

Your response:
```

---

### Step -1.2: Wait for User Response and Process

Enter approval loop:

```yaml
WHILE constitutional approval not received:

  user_response = [wait for user input]

  # Calculate current status from Phase -1 validation
  status = [✅ PASS | ⚠️ CONDITIONAL | ❌ FAIL from plan.md]

  # Option 1: APPROVED
  IF user_response matches ["approved", "approve", "yes", "proceed", "ok"]:

    # Check if approval is allowed
    IF status == ❌ FAIL:
      → ⛔ ERROR: "Cannot approve a FAILED constitutional validation."
      → "The plan has [COUNT] constitutional violations that must be fixed first."
      → "Type 'fix [violation]' to revise the plan, or update constitution.md if principles changed."
      → Continue approval loop (don't approve)

    ELSE IF status == ⚠️ CONDITIONAL OR status == ✅ PASS:
      → Update spec-metadata.json:
        {
          "phase": "planning",
          "approvals": {
            "planning": {
              "generated": true,
              "approved": true,
              "timestamp": "[ISO 8601 current timestamp]",
              "approach": "[EXTEND | CREATE | HYBRID from gap analysis]",
              "constitutional_status": "[PASS | CONDITIONAL]",
              "conditional_items": [
                [IF CONDITIONAL, list items from Phase -1:]
                "Article I: Redis added (justified by 80% latency reduction)",
                ...
              ],
              "violations_fixed": [
                [Track if user fixed violations during this session:]
              ]
            }
          }
        }

      → Show confirmation:
        "✅ Planning Approved!

        Constitutional Status: [PASS | CONDITIONAL]
        Approval recorded at [timestamp]

        [IF CONDITIONAL:]
        Conditional Items Accepted:
        - [List conditional items]

        These will be re-validated during implementation (Step 10.4).
        [END IF]

        Next steps:
        - Optional: Run `/speckit.tasks` to break down into specific tasks
        - Required: Run `/speckit.implement` to begin implementation

        Implementation will include automated constitutional validation at:
        - Step 2.5: Pre-Flight Check (verify planning approval)
        - Step 10.4: Constitutional Reviewer (evidence-based validation)
        "

      → BREAK loop (exit approval gate, proceed to finalize plan)

  # Option 2: FIX violation
  ELSE IF user_response starts with "fix":

    → Extract what to fix:
      fix_target = [text after "fix " in user response]

    → Identify which phase/section needs revision:

      [IF fix_target mentions "test" OR "mock" OR "article v":]
        → Revision target: Phase 2.5 Mock & Stub Strategy
        → Guidance: "To fix Article V compliance, the test strategy must use real dependencies in integration tests."
        → Ask: "Which dependencies should use real implementations in integration tests?
                (e.g., 'Auth0, Database, Tenant Context')"

      [IF fix_target mentions "framework" OR "redis" OR "simplicity" OR "article i":]
        → Revision target: Phase where new framework introduced
        → Guidance: "To fix Article I compliance, either:
                     A) Remove the new framework (simpler solution)
                     B) Add stronger justification in Complexity Tracking"
        → Ask: "Which approach? (A - Remove framework, B - Justify complexity)"

      [IF fix_target mentions "pattern" OR "sync" OR "article ix":]
        → Revision target: Implementation phase using prohibited pattern
        → Guidance: "To fix Article IX, replace prohibited pattern with approved alternative."
        → Ask: "How should [prohibited pattern] be replaced?
                Constitution suggests: [extract remediation from constitution.md]"

      [GENERIC:]
        → Ask: "What specifically should be changed in the plan to fix: [fix_target]?"

    → Wait for fix clarification

    → Apply fix to plan.md:
      [Regenerate affected phases/sections based on fix guidance]
      [Mark as fixed in tracking]

    → Re-run Phase -1 validation:
      [Run constitutional validation again on updated plan]
      [Generate new validation results]

    → Track fixed violations:
      violations_fixed.append({
        "timestamp": "[ISO 8601]",
        "article": "[Article that was violated]",
        "fix_applied": "[Description of fix]"
      })

    → Return to Step -1.1:
      "✅ Fix applied: [summary]

      Re-running constitutional validation on updated plan...

      [Show updated Phase -1 validation results]"

      [If still ❌ FAIL, user needs to fix more]
      [If now ⚠️ CONDITIONAL or ✅ PASS, can approve]

  # Option 3: EXPLAIN article
  ELSE IF user_response starts with "explain":

    → Extract article reference:
      article = [text after "explain ", e.g., "article v", "article i", "simplicity"]

    → Map to article:
      [IF article mentions "v" OR "integration" OR "test":]
        article_section = "Article V: Integration-First Testing"

      [IF article mentions "i" OR "simplicity" OR "framework":]
        article_section = "Article I: Simplicity & Framework Minimalism"

      [IF article mentions "ix" OR "pattern" OR "prohibited":]
        article_section = "Article IX: Prohibited Patterns"

      [... map other articles ...]

    → Read constitution.md:
      [Locate article_section in constitution.md]
      [Extract full article text]

    → Read plan.md Phase -1 validation for this article:
      [Extract how plan was validated against this article]

    → Show explanation:
      "📖 Constitutional Article Explanation

      **[Article Section from constitution.md]**

      [Display full article text from constitution.md]

      ---

      **How This Plan Relates to [Article]:**

      [Extract from plan.md Phase -1 validation:]
      - Status: [PASS/CONDITIONAL/FAIL]
      - Validation Details: [What was checked]
      - Finding: [Why it passed/failed/conditional]

      [IF FAIL:]
      **Why This Failed:**
      [Explain violation]

      **How to Fix:**
      [Suggest remediation]

      [IF CONDITIONAL:]
      **Why This Requires Approval:**
      [Explain trade-off]

      **Justification Provided:**
      [Show justification from plan]

      [IF PASS:]
      **Why This Passed:**
      [Explain compliance]

      ---

      Does this clarify the constitutional requirement?

      You can:
      - Ask about another article ('explain [article]')
      - Request a fix ('fix [violation]')
      - Approve the plan ('approved')"

    → Continue approval loop

  # Option 4: SHOW CONSTITUTION
  ELSE IF user_response matches ["show constitution", "view constitution", "read constitution"]:

    → Check if constitution.md exists:

      IF constitution.md NOT found:
        → "ℹ️ No constitution.md found

        This project doesn't have a constitution file. Constitutional validation is running in PARTIAL mode:
        - TDD compliance checking
        - Simplicity validation (framework count)
        - Integration-first testing (basic checks)
        - NO prohibited pattern detection (requires constitution)

        To enable full constitutional validation:
        1. Create constitution.md via `/speckit.constitution`
        2. Define prohibited patterns
        3. Document project principles

        For now, you can proceed with PARTIAL validation.

        Type 'approved' to proceed with current validation level."

      ELSE:
        → Read and display constitution.md:
          "📜 Project Constitution (constitution.md)

          [Display full contents of constitution.md]

          ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

          End of constitution.

          This constitution defines the project's governance principles.
          The current plan was validated against these principles.

          Type 'approved' to approve the plan, or 'fix [violation]' to revise."

    → Continue approval loop

  # Option 5: REVIEW PLAN
  ELSE IF user_response matches ["review plan", "show plan", "view plan", "read plan"]:

    → Read and display plan.md:
      "📋 Implementation Plan (plan.md)

      [Display full contents of plan.md, or at least:
       - All implementation phases
       - Phase 2.5 (Test Strategy)
       - Phase 3 (Security Review)
       - Phase -1 (Constitutional Validation)
       - Complexity Tracking section]

      ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

      End of plan.

      Review the plan details and constitutional validation results above.

      Type 'approved' to approve, or 'fix [issue]' to revise."

    → Continue approval loop

  # Option 6: Unclear response
  ELSE:
    → "❓ I didn't understand that response.

    Current constitutional validation status: [PASS/CONDITIONAL/FAIL]

    Please respond with one of:
    - **'approved'** - Accept the plan [IF PASS or CONDITIONAL only]
    - **'fix [violation]'** - Revise plan to address issues
    - **'explain [article]'** - Show constitutional article details
    - **'show constitution'** - View full constitution.md
    - **'review plan'** - View complete plan.md

    [IF status == ❌ FAIL:]
    ⚠️ Note: Plan has constitutional violations - must fix before approving.
    [END IF]

    Your response:"

    → Continue approval loop

END WHILE (approval loop)
```

---

### Step -1.3: Configuration Override Handling

Before entering approval loop, check configuration:

```yaml
# Read .specify/config.yml (if exists)
IF config.yml exists AND has discovery_gates.constitutional_approval_gate section:

  IF discovery_gates.constitutional_approval_gate.enabled == false:
    → Skip approval gate entirely
    → Log: "Constitutional approval gate disabled via config"
    → Set spec-metadata.json:
      {
        "approvals": {
          "planning": {
            "approved": true,
            "approval_skipped": true,
            "skip_reason": "config disabled"
          }
        }
      }
    → Exit gate (proceed to finalize plan)

  ELSE IF discovery_gates.constitutional_approval_gate.auto_approve == true:
    → Auto-approve with warning:
      "⚠️ Auto-approval enabled via config (non-interactive mode)
       Constitutional validation status: [status]
       [IF FAIL:] ⛔ WARNING: Auto-approving FAILED validation - NOT RECOMMENDED"

    → Set spec-metadata.json with auto-approval flag
    → Exit gate

  ELSE IF discovery_gates.constitutional_approval_gate.block_on_fail == false:
    → IF status == ❌ FAIL:
      → Show warning instead of blocking:
        "⚠️ Constitutional validation FAILED but blocking is disabled.
         Violations: [list]

         You can proceed anyway, but this is NOT RECOMMENDED.
         Implementation will re-validate at Step 10.4.

         Proceed with violations? (yes/no)"

      → IF user confirms "yes":
        → Allow approval despite FAIL status
        → Set flag: approval_with_violations: true

ELSE (no config or default behavior):
  → Normal approval loop with blocking on FAIL

# Environment variable override
IF SPECKIT_AUTO_APPROVE == "true":
  → Same as auto_approve config above

# Command-line flag handling
IF command invoked with --auto-approve:
  → Auto-approve regardless of config

IF command invoked with --skip-gates:
  → Skip approval gate entirely
```

---

### Step -1.4: Graceful Degradation (No Constitution)

Handle case where constitution.md doesn't exist:

```yaml
IF constitution.md NOT found:

  → Show info message in Step -1.1:
    "ℹ️ **Constitutional Validation: PARTIAL MODE**

    No constitution.md found - running limited validation:
    ✅ TDD compliance checking (git history analysis during implementation)
    ✅ Simplicity validation (framework count)
    ✅ Integration-first testing (real vs mocked dependencies)
    ❌ Prohibited pattern detection (SKIPPED - requires constitution)

    **Recommendation:** Create constitution via `/speckit.constitution` for full validation.

    **Current Validation Result:** [Show what WAS validated]
    - [Results from checks that don't require constitution]

    You can proceed with PARTIAL validation, but full constitutional governance is disabled.

    Type 'approved' to proceed, or 'create constitution' to set up full validation."

  → IF user types "create constitution":
    → "To create a constitution, run: `/speckit.constitution`
       This will establish project principles and enable full validation.

       For now, type 'approved' to proceed with PARTIAL validation."

  → Allow approval with PARTIAL status:
    spec-metadata.json:
      {
        "approvals": {
          "planning": {
            "constitutional_status": "PARTIAL",
            "constitution_file_missing": true
          }
        }
      }
```

---

### Step -1.5: Integration with Implementation Pre-Flight

After approval recorded, implementation will validate:

**File: src/.claude/commands/speckit.implement.md**
**Location: Step 2.5 (Pre-Flight Check)**

Add to existing pre-flight check:

```markdown
## Step 2.5: Constitutional Pre-Flight Check (v2.7 Enhancement)

**Purpose:** Verify planning phase constitutional approval BEFORE code execution begins.

1. **Read spec-metadata.json**

   ```bash
   metadata_file="specs/[FEATURE]/spec-metadata.json"

   IF metadata file not found:
     → ⚠️ WARNING: "No metadata file - cannot verify planning approval"
     → Ask: "Planning approval unknown. Proceed anyway? (yes/no)"
     → IF no: EXIT

   planning_approved=$(jq -r '.approvals.planning.approved' "$metadata_file")
   constitutional_status=$(jq -r '.approvals.planning.constitutional_status' "$metadata_file")
   ```

2. **Validate Planning Approval**

   ```yaml
   IF planning_approved == false OR planning_approved == null:
     → ⛔ ERROR: "Planning not approved. Run /speckit.plan and approve before implementing."
     → Show: "Last planning phase: [extract from metadata]"
     → EXIT (cannot proceed)

   ELSE IF constitutional_status == "FAIL":
     → ⛔ ERROR: "Planning failed constitutional validation. Cannot implement until violations fixed."
     → Show violations from metadata
     → EXIT

   ELSE IF constitutional_status == "CONDITIONAL":
     → ⚠️ INFO: "Planning approved with conditional items:"
     → Show conditional items from metadata
     → Continue (conditional approval is valid)

   ELSE IF constitutional_status == "PASS":
     → ✅ INFO: "Planning approved - constitutional validation PASSED"
     → Continue

   ELSE IF constitutional_status == "PARTIAL":
     → ℹ️ INFO: "Planning approved - constitutional validation PARTIAL (no constitution.md)"
     → Continue

   ELSE:
     → ⚠️ WARNING: "Constitutional status unknown"
     → Continue with warning
   ```

3. **Show Pre-Flight Summary**

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ✅ Pre-Flight Check: PASSED
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Specification: ✅ Approved ([timestamp])
   Planning: ✅ Approved ([timestamp])
   Constitutional Status: [PASS/CONDITIONAL/PARTIAL]

   [IF CONDITIONAL:]
   Conditional Items Acknowledged:
   - [List from metadata]
   [END IF]

   Implementation can proceed.
   Constitutional re-validation will occur at Step 10.4 (Quality Gate).

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ```

This creates a complete gate system: Planning approval required → Implementation validates approval → Quality gate re-checks compliance
```

---

**END OF CONSTITUTIONAL APPROVAL GATE**

This is inserted into src/.claude/commands/speckit.plan.md after Phase -1 validation completes.

**Token Cost Analysis:**
- Validation results presentation: 100-150 tokens
- Approval loop (no fixes): 50-100 tokens
- Approval loop (1 fix cycle): +200-300 tokens
- Total: 100-200 tokens (mandatory, every feature)

**Integration Points:**
1. INSERT this gate after Phase -1 in speckit.plan.md
2. UPDATE speckit.implement.md Step 2.5 to validate planning approval
3. UPDATE spec-metadata.json schema to include constitutional_status fields
