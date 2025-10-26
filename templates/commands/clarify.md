---
description: Identify underspecified areas in the current feature spec by asking up to 5 highly targeted clarification questions and encoding answers back into the spec.
scripts:
   sh: scripts/bash/check-prerequisites.sh --json --paths-only
   ps: scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

Goal: Detect and reduce ambiguity or missing decision points in the active feature specification and record the clarifications directly in the spec file.

Note: This clarification workflow is expected to run (and be completed) BEFORE invoking `/speckit.plan`. If the user explicitly states they are skipping clarification (e.g., exploratory spike), you may proceed, but must warn that downstream rework risk increases.

Execution steps:

1. Run `{SCRIPT}` from repo root **once** (combined `--json --paths-only` mode / `-Json -PathsOnly`). Parse minimal JSON payload fields:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - (Optionally capture `IMPL_PLAN`, `TASKS` for future chained flows.)
   - If JSON parsing fails, abort and instruct user to re-run `/speckit.specify` or verify feature branch environment.
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. Load the current spec file. Perform a structured ambiguity & coverage scan using this taxonomy. For each category, mark status: Clear / Partial / Missing. Produce an internal coverage map used for prioritization (do not output raw map unless no questions will be asked).

   Functional Scope & Behavior:
   - Core user goals & success criteria
   - Explicit out-of-scope declarations
   - User roles / personas differentiation

   Domain & Data Model:
   - Entities, attributes, relationships
   - Identity & uniqueness rules
   - Lifecycle/state transitions
   - Data volume / scale assumptions

   Interaction & UX Flow:
   - Critical user journeys / sequences
   - Error/empty/loading states
   - Accessibility or localization notes

   Non-Functional Quality Attributes:
   - Performance (latency, throughput targets)
   - Scalability (horizontal/vertical, limits)
   - Reliability & availability (uptime, recovery expectations)
   - Observability (logging, metrics, tracing signals)
   - Security & privacy (authN/Z, data protection, threat assumptions)
   - Compliance / regulatory constraints (if any)

   Integration & External Dependencies:
   - External services/APIs and failure modes
   - Data import/export formats
   - Protocol/versioning assumptions

   Edge Cases & Failure Handling:
   - Negative scenarios
   - Rate limiting / throttling
   - Conflict resolution (e.g., concurrent edits)

   Constraints & Tradeoffs:
   - Technical constraints (language, storage, hosting)
   - Explicit tradeoffs or rejected alternatives

   Terminology & Consistency:
   - Canonical glossary terms
   - Avoided synonyms / deprecated terms

   Completion Signals:
   - Acceptance criteria testability
   - Measurable Definition of Done style indicators

   Misc / Placeholders:
   - TODO markers / unresolved decisions
   - Ambiguous adjectives ("robust", "intuitive") lacking quantification

   For each category with Partial or Missing status, add a candidate question opportunity unless:
   - Clarification would not materially change implementation or validation strategy
   - Information is better deferred to planning phase (note internally)

3. Generate (internally) a prioritized queue of candidate clarification questions (maximum 5). Do NOT output them all at once. Apply these constraints:
    - Maximum of 10 total questions across the whole session.
    - Each question must be answerable with EITHER:
       - A short multiple‑choice selection (2–5 distinct, mutually exclusive options), OR
       - A one-word / short‑phrase answer (explicitly constrain: "Answer in <=5 words").
    - Only include questions whose answers materially impact architecture, data modeling, task decomposition, test design, UX behavior, operational readiness, or compliance validation.
    - Ensure category coverage balance: attempt to cover the highest impact unresolved categories first; avoid asking two low-impact questions when a single high-impact area (e.g., security posture) is unresolved.
    - Exclude questions already answered, trivial stylistic preferences, or plan-level execution details (unless blocking correctness).
    - Favor clarifications that reduce downstream rework risk or prevent misaligned acceptance tests.
    - If more than 5 categories remain unresolved, select the top 5 by (Impact * Uncertainty) heuristic.

4. Sequential questioning loop (interactive):
    - Present EXACTLY ONE question at a time.
    - For multiple‑choice questions:
       - **Analyze all options** and determine the **most suitable option** based on:
          - Best practices for the project type
          - Common patterns in similar implementations
          - Risk reduction (security, performance, maintainability)
          - Alignment with any explicit project goals or constraints visible in the spec
       - Present your **recommended option prominently** at the top with clear reasoning (1-2 sentences explaining why this is the best choice).
       - Format as: `**Recommended:** Option [X] - <reasoning>`
       - Then render all options as a Markdown table:

       | Option | Description |
       |--------|-------------|
       | A | <Option A description> |
       | B | <Option B description> |
       | C | <Option C description> (add D/E as needed up to 5) |
       | Short | Provide a different short answer (<=5 words) (Include only if free-form alternative is appropriate) |

       - After the table, add: `You can reply with the option letter (e.g., "A"), accept the recommendation by saying "yes" or "recommended", or provide your own short answer.`
    - For short‑answer style (no meaningful discrete options):
       - Provide your **suggested answer** based on best practices and context.
       - Format as: `**Suggested:** <your proposed answer> - <brief reasoning>`
       - Then output: `Format: Short answer (<=5 words). You can accept the suggestion by saying "yes" or "suggested", or provide your own answer.`
    - After the user answers:
       - If the user replies with "yes", "recommended", or "suggested", use your previously stated recommendation/suggestion as the answer.
       - Otherwise, validate the answer maps to one option or fits the <=5 word constraint.
       - If ambiguous, ask for a quick disambiguation (count still belongs to same question; do not advance).
       - Once satisfactory, record it in working memory (do not yet write to disk) and move to the next queued question.
    - Stop asking further questions when:
       - All critical ambiguities resolved early (remaining queued items become unnecessary), OR
       - User signals completion ("done", "good", "no more"), OR
       - You reach 5 asked questions.
    - Never reveal future queued questions in advance.
    - If no valid questions exist at start, immediately report no critical ambiguities.

5. Integration after EACH accepted answer (incremental update approach):
    - Maintain in-memory representation of the spec (loaded once at start) plus the raw file contents.
    - For the first integrated answer in this session:
       - Ensure a `## Clarifications` section exists (create it just after the highest-level contextual/overview section per the spec template if missing).
       - Under it, create (if not present) a `### Session YYYY-MM-DD` subheading for today.
    - Append a bullet line immediately after acceptance: `- Q: <question> → A: <final answer>`.
    - Then immediately apply the clarification to the most appropriate section(s):
       - Functional ambiguity → Update or add a bullet in Functional Requirements.
       - User interaction / actor distinction → Update User Stories or Actors subsection (if present) with clarified role, constraint, or scenario.
       - Data shape / entities → Update Data Model (add fields, types, relationships) preserving ordering; note added constraints succinctly.
       - Non-functional constraint → Add/modify measurable criteria in Non-Functional / Quality Attributes section (convert vague adjective to metric or explicit target).
       - Edge case / negative flow → Add a new bullet under Edge Cases / Error Handling (or create such subsection if template provides placeholder for it).
       - Terminology conflict → Normalize term across spec; retain original only if necessary by adding `(formerly referred to as "X")` once.
    - If the clarification invalidates an earlier ambiguous statement, replace that statement instead of duplicating; leave no obsolete contradictory text.
    - Save the spec file AFTER each integration to minimize risk of context loss (atomic overwrite).
    - Preserve formatting: do not reorder unrelated sections; keep heading hierarchy intact.
    - Keep each inserted clarification minimal and testable (avoid narrative drift).

6. Validation (performed after EACH write plus final pass):
   - Clarifications session contains exactly one bullet per accepted answer (no duplicates).
   - Total asked (accepted) questions ≤ 5.
   - Updated sections contain no lingering vague placeholders the new answer was meant to resolve.
   - No contradictory earlier statement remains (scan for now-invalid alternative choices removed).
   - Markdown structure valid; only allowed new headings: `## Clarifications`, `### Session YYYY-MM-DD`.
   - Terminology consistency: same canonical term used across all updated sections.

7. Write the updated spec back to `FEATURE_SPEC`.

8. Report completion (after questioning loop ends or early termination):
   - Number of questions asked & answered.
   - Path to updated spec.
   - Sections touched (list names).
   - Coverage summary table listing each taxonomy category with Status: Resolved (was Partial/Missing and addressed), Deferred (exceeds question quota or better suited for planning), Clear (already sufficient), Outstanding (still Partial/Missing but low impact).
   - If any Outstanding or Deferred remain, recommend whether to proceed to `/speckit.plan` or run `/speckit.clarify` again later post-plan.
   - Suggested next command.

9. **Clarification Review Gate** (Evidence-Based Self-Check):

   **Purpose**: Validate clarification effectiveness and assess readiness to proceed to planning phase.

   **MANDATORY: The Four Clarification Questions** (MUST answer with ACTUAL evidence):

   ❓ **"Were all critical ambiguities addressed?"**
      ```yaml
      Action Required:
        - Review coverage summary table from Step 8
        - Count: Resolved vs Outstanding vs Deferred categories
        - IF any category remains "Outstanding" with HIGH impact: NOT ready ❌
        - Show ACTUAL category status counts, not summaries

      Expected Evidence Format:
        Coverage Status:
        ✓ Resolved: [X] categories (was Partial/Missing, now Clear)
        ⚠️ Deferred: [Y] categories (better suited for planning phase)
        ✓ Clear: [Z] categories (already sufficient)
        ❌ Outstanding: [N] categories (still Partial/Missing)

        Outstanding Categories (if any):
        - [Category Name]: [Impact Level] - [Why still unresolved]

      Readiness Determination:
        IF Outstanding = 0 AND (Resolved + Clear) > 80% of taxonomy:
          → ✅ READY to proceed to planning
        IF Outstanding ≤ 2 AND all Outstanding are LOW impact:
          → ⚠️ CAN PROCEED (with noted risks)
        IF Outstanding > 2 OR any HIGH impact Outstanding:
          → ❌ NOT READY (recommend another clarify pass)
      ```

   ❓ **"Are clarifications properly integrated into spec?"**
      ```yaml
      Action Required:
        - Verify ## Clarifications section exists with Session YYYY-MM-DD
        - Confirm all [X] Q&A bullets recorded (count from Step 4)
        - Check each answer integrated into appropriate spec section
        - Validate no contradictory statements remain

      Expected Evidence Format:
        Integration Status:
        ✓ Clarifications section: [Present/Missing]
        ✓ Q&A bullets recorded: [X]/[X] (all questions answered)
        ✓ Sections updated: [list section names]
        ✓ Contradictions removed: [Yes/No - if No, list them]

      Validation Checklist:
        - [ ] Each accepted answer appears in Clarifications section
        - [ ] Each answer also integrated into relevant spec section
        - [ ] No obsolete/conflicting text remains
        - [ ] Terminology normalized across all updated sections
        - [ ] Markdown structure valid (no broken headings)

      IF any checklist item fails:
        → ❌ Integration incomplete
        → Fix integration issues before proceeding
      ```

   ❓ **"Did clarifications improve spec measurability?"**
      ```yaml
      Action Required:
        - Compare spec BEFORE vs AFTER clarification
        - Count vague adjectives remaining ("robust", "intuitive", "fast")
        - Check if [NEEDS CLARIFICATION] markers reduced
        - Validate acceptance criteria are more testable

      Expected Evidence Format:
        Measurability Improvements:
        Before: [N] vague adjectives, [M] [NEEDS CLARIFICATION] markers
        After: [N'] vague adjectives (-[N-N']), [M'] markers (-[M-M'])

        Specific Improvements:
        - Functional Requirements: [Example of vague → specific]
        - Non-Functional: [Example of added measurable target]
        - Data Model: [Example of clarified constraint/type]
        - Edge Cases: [Example of added failure scenario]

      Quality Check:
        IF (vague adjectives reduced by ≥50%) AND (markers reduced to ≤3):
          → ✅ Significant improvement
        IF some improvement but markers still >3:
          → ⚠️ Partial improvement (may need another pass)
        IF no measurable change:
          → ❌ Questions asked but spec not clarified
      ```

   ❓ **"Is there evidence of clarification impact?"**
      ```yaml
      Required Evidence (ALL must be provided):

      1. Spec File Comparison (MANDATORY):
         Before clarification line count: [N] lines
         After clarification line count: [M] lines
         Net change: [M-N] lines ([describe: additions in Clarifications + section updates])

      2. Category Coverage Change (MANDATORY):
         Run: Compare taxonomy status before/after
         Output:
           Categories changed from Partial→Clear: [list]
           Categories changed from Missing→Clear: [list]
           Categories still Partial/Missing: [list with reasons]

      3. Questions Efficiency (MANDATORY):
         Questions asked: [X]/5 quota used
         Categories resolved per question: [Y]/[X] = [Y/X] ratio
         High-impact categories resolved: [list]

         Efficiency Validation:
           IF ratio ≥ 1.5 (each question resolved 1-2 categories):
             → ✅ Efficient questioning
           IF ratio < 1.0:
             → ⚠️ Low impact questions (review question selection)

      4. Spec Diff Summary (MANDATORY):
         Show actual file changes:
         ```
         git diff HEAD -- [FEATURE_SPEC path]
         ```
         Output: [paste relevant diff sections showing Q&A + integration]

      IF any evidence is MISSING:
        ❌ CANNOT report completion
        → Gather missing evidence first
        → Re-run this step with complete evidence
      ```

   **Hallucination Prevention (7 Red Flags for Clarification):**
   ```yaml
   Detect and BLOCK these patterns:

   🚨 "All ambiguities resolved" WITHOUT showing coverage table
      → Self-correction: "Wait, I need to show actual coverage status"

   🚨 "Spec updated" WITHOUT showing what changed
      → Self-correction: "Let me show the actual diff and integration"

   🚨 "Questions answered" WITH Outstanding high-impact categories
      → Self-correction: "High-impact gaps remain, not ready yet"

   🚨 Claiming clarification WITHOUT reducing vague terms
      → Self-correction: "Spec measurability hasn't improved"

   🚨 Skipping integration validation
      → Self-correction: "I need to verify all answers are in spec"

   🚨 Hiding remaining ambiguities
      → Self-correction: "I must report Outstanding categories honestly"

   🚨 "Ready for planning" statements WITHOUT evidence
      → Self-correction: "Need to verify readiness criteria first"

   IF detected: STOP → Gather evidence → Report honestly
   ```

   **Determine Status**:

   ✅ **READY for Planning**:
   ```yaml
   Criteria (ALL must be met):
     - Outstanding categories = 0
     - (Resolved + Clear) ≥ 80% of taxonomy categories
     - All Q&A bullets recorded and integrated
     - Measurability improved (markers ≤3, vague terms reduced)
     - No contradictions in spec
     - Git diff shows actual content changes

   IF ALL criteria met:
     → Proceed to Step 10 (present completion report)
   ```

   ⚠️ **CAN PROCEED** (with risks noted):
   ```yaml
   Criteria:
     - Outstanding categories ≤ 2
     - All Outstanding are LOW impact
     - Most Q&A integrated (≥80%)
     - Some measurability improvement visible

   IF criteria met:
     → Present risks to user
     → Ask: "Low-impact gaps remain. Proceed to planning or run another clarify pass?"
     → Wait for user decision before Step 10
   ```

   ❌ **NOT READY** (more clarification needed):
   ```yaml
   Criteria (ANY triggers NOT READY):
     - Outstanding categories > 2
     - Any Outstanding category is HIGH impact
     - Q&A integration incomplete (<80%)
     - No measurability improvement
     - Contradictions remain in spec

   IF NOT READY:
     → Present issues to user with evidence
     → Recommend: "Run /speckit.clarify again to address [specific categories]"
     → STOP before Step 10 (do not proceed to planning)
   ```

   **Output Format** (Present to User - ONLY if evidence provided):
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   📋 Clarification Review Complete
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Status: [✅ READY | ⚠️ CAN PROCEED | ❌ NOT READY]

   Branch: [branch-name]
   Spec: [path/to/spec.md]

   ## Coverage Status

   ✓ Resolved: [X] categories (Partial/Missing → Clear)
   ✓ Clear: [Y] categories (already sufficient)
   ⚠️ Deferred: [Z] categories (planning phase)
   ❌ Outstanding: [N] categories

   [IF Outstanding > 0:]
   Outstanding Categories (need attention):
   - [Category]: [Impact] - [Why unresolved]

   ## Clarification Effectiveness

   Questions Asked: [X]/5 quota used
   Categories per Question: [Y]/[X] = [ratio] (target: ≥1.5)

   Spec Changes:
   - Lines added: +[N] (Clarifications + integrations)
   - Sections updated: [list names]
   - Vague terms reduced: [N] → [M] (-[N-M])
   - [NEEDS CLARIFICATION] markers: [X] → [Y] (-[X-Y])

   Integration Status:
   ✅ All Q&A recorded in Clarifications section
   ✅ All answers integrated into spec sections
   ✅ No contradictions remaining
   ✅ Terminology normalized

   ## Measurability Improvements

   [Show before/after examples:]
   - Functional: "[vague statement]" → "[specific statement]"
   - Non-Functional: "[missing target]" → "[measurable target: X ms]"
   - Data Model: "[unclear type]" → "[explicit type with constraints]"

   ## Readiness Assessment

   [IF ✅ READY:]
   ✅ All critical ambiguities resolved
   ✅ Spec measurability significantly improved
   ✅ No high-impact gaps remaining
   ✅ Ready to proceed to planning phase

   [IF ⚠️ CAN PROCEED:]
   ⚠️ Minor gaps remain (low impact):
      - [Gap 1]: [Why low impact]
      - [Gap 2]: [Why low impact]

   Risk: These gaps may require clarification during planning.

   [IF ❌ NOT READY:]
   ❌ Critical gaps remain:
      - [Gap 1]: [HIGH impact] - [Why blocks planning]
      - [Gap 2]: [HIGH impact] - [Why blocks planning]

   Recommendation: Run /speckit.clarify again focusing on:
      - [Specific category 1]
      - [Specific category 2]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Evidence: Spec Changes
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   ```diff
   [Paste relevant git diff output showing:]
   + ## Clarifications
   + ### Session YYYY-MM-DD
   + - Q: [question 1] → A: [answer 1]
   + ... [and integration into spec sections]
   ```

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Next Steps
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [IF ✅ READY:]
   1. Proceed to planning phase:
      /speckit.plan

   2. Or review spec changes:
      git diff HEAD -- [FEATURE_SPEC]

   [IF ⚠️ CAN PROCEED:]
   Choose one:
   1. Proceed with noted risks:
      /speckit.plan

   2. Address remaining gaps:
      /speckit.clarify [focus on specific categories]

   3. Review current spec state:
      git diff HEAD -- [FEATURE_SPEC]

   [IF ❌ NOT READY:]
   REQUIRED: Address critical gaps before planning:
   1. Run another clarify pass:
      /speckit.clarify [targeting high-impact categories]

   2. Review what needs clarification:
      [List specific sections/questions to focus on]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Ready to proceed? (yes/review spec/run clarify again/address gaps)
   ```

   **User Interaction:**
   ```yaml
   IF user says "yes" or "proceed":
     IF status = READY:
       → Suggest: /speckit.plan
     IF status = CAN PROCEED:
       → Warn about risks, then suggest: /speckit.plan
     IF status = NOT READY:
       → Block: "Cannot proceed - critical gaps remain"
       → Suggest: /speckit.clarify [with focus areas]

   IF user says "review" or "review spec":
     → Run: git diff HEAD -- [FEATURE_SPEC]
     → Show detailed spec changes

   IF user says "clarify again" or "run clarify":
     → Suggest: /speckit.clarify [focus on Outstanding categories]
     → List specific questions to target

   IF user says "address gaps":
     → List Outstanding categories with:
        - Why they're important (impact)
        - Suggested questions to ask
        - Which spec sections need updating
     → Ask: "Ready to run /speckit.clarify to address these?"

   IF EVIDENCE MISSING:
     ❌ "Cannot complete clarification review without evidence."
     → List missing evidence
     → Gather evidence automatically where possible
     → Re-run review gate
   ```

   **Benefits** (from PM Agent Reflexion pattern):
   - ✅ Prevents premature planning with unresolved ambiguities
   - ✅ Evidence-based readiness assessment
   - ✅ No false "ready" claims without coverage proof
   - ✅ Transparent validation of clarification effectiveness
   - ✅ User confidence in spec quality before planning

10. Report final completion summary (after review gate passes).

Behavior rules:

- If no meaningful ambiguities found (or all potential questions would be low-impact), respond: "No critical ambiguities detected worth formal clarification." and suggest proceeding.
- If spec file missing, instruct user to run `/speckit.specify` first (do not create a new spec here).
- Never exceed 5 total asked questions (clarification retries for a single question do not count as new questions).
- Avoid speculative tech stack questions unless the absence blocks functional clarity.
- Respect user early termination signals ("stop", "done", "proceed").
- If no questions asked due to full coverage, output a compact coverage summary (all categories Clear) then suggest advancing.
- If quota reached with unresolved high-impact categories remaining, explicitly flag them under Deferred with rationale.

Context for prioritization: {ARGS}
