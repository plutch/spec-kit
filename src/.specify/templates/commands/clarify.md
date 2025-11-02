---
description: Identify underspecified areas in the current feature spec by asking up to 5 highly targeted clarification questions and encoding answers back into the spec.
scripts:
   sh: .specify/scripts/bash/check-prerequisites.sh --json --paths-only
   ps: .specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

Goal: Detect and reduce ambiguity or missing decision points in the active feature specification and record the clarifications directly in the spec file.

Note: This clarification workflow is expected to run (and be completed) BEFORE invoking `/speckit.plan`. If the user explicitly states they are skipping clarification (e.g., exploratory spike), you may proceed, but must warn that downstream rework risk increases.

## Mode Selection

**Available Modes**:
1. **Normal Mode** (Default): Gap-filling clarification
2. **Challenge Mode**: Adversarial stress-testing
3. **Expert Lens Mode**: Multi-perspective requirements review
4. **UX Lens Mode**: Multi-perspective UX/usability review
5. **Edge Case Detection Mode**: Systematic boundary and error analysis

**Mode Detection**:

```yaml
IF $ARGUMENTS contains "--challenge":
  → Activate Challenge Mode (adversarial questioning)
  → Follow Challenge Mode execution steps

ELSE IF $ARGUMENTS contains "--expert" OR "--review":
  → Activate Expert Lens Mode (multi-expert review)
  → Follow Expert Lens execution steps

ELSE IF $ARGUMENTS contains "--ux":
  → Activate UX Lens Mode (UX/usability review)
  → Follow UX Lens execution steps

ELSE IF $ARGUMENTS contains "--edge-cases" OR "--edge":
  → Activate Edge Case Detection Mode (boundary/error analysis)
  → Follow Edge Case Detection execution steps

ELSE:
  → Normal Mode (gap-filling clarification)
  → Follow standard execution steps
```

---

### Challenge Mode Execution (Adversarial Stress-Testing)

**Purpose**: Stress-test spec assumptions through adversarial questioning. Instead of filling gaps, CHALLENGE the spec's direction, assumptions, and approach to surface hidden risks and alternatives BEFORE committing to planning.

**When to Use Challenge Mode**:
- High-stakes features (security, payments, compliance, core infrastructure)
- Features with many assumptions or unclear requirements
- Features with significant resource commitments
- When spec seems "too clean" (may be hiding complexity)
- Before major architecture decisions

**Challenge Mode Steps**:

1. Run `{SCRIPT}` to load FEATURE_SPEC (same as normal mode)

2. **Load and Analyze Spec**: Read spec.md completely and identify:
   - Stated assumptions (explicit in spec)
   - Implied assumptions (read between the lines)
   - Approach/solution chosen (technical decisions)
   - Scope boundaries (what's in/out)
   - Risk statements (what's mentioned/missing)

3. **Generate Challenge Questions** (3-5 adversarial questions):

   **Question Categories**:

   a. **Assumption Challenge**:
      ```
      ❓ Assumption Validation (Challenge)

      The spec assumes [stated assumption]. But have we validated this?

      Challenge: [Why this assumption might be wrong]

      Alternative: [What if the opposite is true]

      Question: How confident are we in this assumption? What's the evidence?

      Options:
      A) High confidence - validated with users/data
      B) Medium confidence - based on best practices
      C) Low confidence - educated guess
      D) Should validate before proceeding
      ```

   b. **Alternative Exploration**:
      ```
      ❓ Approach Alternatives (Challenge)

      The spec proposes [stated approach]. But what alternatives exist?

      Current: [Summarize chosen approach]

      Alternatives:
      - [Alternative 1]: [Pros/cons]
      - [Alternative 2]: [Pros/cons]
      - [Simplest option]: [Pros/cons]

      Question: What's the rationale for the chosen approach vs simpler alternatives?

      Options:
      A) Chosen approach is optimal - [reason]
      B) Alternative [X] is better - [reason]
      C) Need to compare alternatives first
      D) Simplest approach sufficient
      ```

   c. **Simplification Pressure**:
      ```
      ❓ Scope Reduction (Challenge)

      If we had 50% of the time/budget, what would we cut?

      Current scope: [List key features from spec]

      Question: What's the MINIMAL viable version?

      Rate each feature:
      - Must-have (breaks feature without it)
      - Should-have (significantly degrades UX)
      - Nice-to-have (polish, not critical)
      - Gold-plating (over-engineered)

      Options:
      A) All current features are must-haves
      B) [List features] could be deferred
      C) Minimal version is just [core features]
      D) Need to re-scope significantly
      ```

   d. **Risk Exposure**:
      ```
      ❓ Downside Scenarios (Challenge)

      What's the worst case if our assumptions are wrong?

      Scenarios to consider:
      - User adoption 10x LOWER than expected
      - User adoption 10x HIGHER than expected
      - Security requirements more stringent
      - Integration dependencies fail/change
      - Timeline compressed by 50%

      Question: What are the hidden risks not addressed in the spec?

      Options:
      A) Risks are well-covered in spec
      B) [Specific risk] not addressed
      C) Need contingency plans for [scenarios]
      D) Spec is too optimistic
      ```

   e. **Competitive/Market Challenge**:
      ```
      ❓ Market Positioning (Challenge)

      Why would users choose this over alternatives?

      Current alternatives users have:
      - [Existing solution 1]
      - [Existing solution 2]
      - [Do nothing / workaround]

      Question: What's our unique value proposition?

      Options:
      A) Clear differentiation - [specific advantage]
      B) Better UX than alternatives
      C) Cheaper/faster than alternatives
      D) Not clearly differentiated yet
      ```

4. **Present Challenge Questions** (one at a time, same interaction pattern as normal mode):
   - Ask question
   - Wait for user response
   - Record answer
   - Move to next question or user signals "done"

5. **Generate Challenge Report**:

   After questions complete, synthesize findings:

   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   🔍 Spec Challenge Report
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Feature: [Name]
   Spec: [Path]
   Challenge Mode: [N] adversarial questions asked

   ## Assumptions Challenged

   Assumption 1: [Stated assumption]
   - Confidence: [High/Medium/Low]
   - Evidence: [User's response]
   - Risk if wrong: [Impact]
   - Action: [Validate / Accept / Revise]

   [Repeat for each assumption]

   ## Alternatives Explored

   Current Approach: [Summary]
   - Rationale: [Why chosen]

   Alternative 1: [Name]
   - Pros: [...]
   - Cons: [...]
   - Decision: [Chosen / Rejected / Needs evaluation]

   [Repeat for each alternative]

   ## Scope Recommendations

   Minimal Viable Version:
   - Must-have: [List core features]
   - Should-have: [List important features]
   - Nice-to-have: [List polish features]
   - Deferred: [List features to cut if needed]

   ## Risks Identified

   1. [Risk category]: [Description]
      - Mitigation: [Strategy]
      - Contingency: [Plan]

   [Repeat for each risk]

   ## Recommendations

   Based on challenge analysis:

   ✅ Proceed as-is: [If spec validated]
     - Rationale: [Why assumptions hold]

   ⚠️ Revise spec: [If issues found]
     - Changes needed: [List specific revisions]
     - Focus areas: [What to rethink]

   ❌ Re-scope: [If fundamental issues]
     - Core problems: [What's wrong]
     - Suggested direction: [Alternative approach]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Next Steps
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [IF ✅ Proceed:]
   1. Continue to planning: /speckit.plan
   2. Update spec with challenge insights (optional)

   [IF ⚠️ Revise:]
   1. Update spec.md with recommended changes
   2. Run /speckit.clarify again (normal mode)
   3. Then proceed to /speckit.plan

   [IF ❌ Re-scope:]
   1. Major spec revision needed
   2. Consider /speckit.specify [revised description]
   3. Or manually update spec.md with new direction

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Ready to proceed? (yes/revise/re-scope)
   ```

6. **Update Spec (Optional)**: Based on challenge findings, optionally update spec.md with:
   - Challenge Insights section
   - Validated Assumptions section
   - Alternatives Considered section
   - Risk Mitigation section

7. **User Decision**: Wait for user to decide:
   - Proceed to planning (spec validated)
   - Revise spec (issues found)
   - Re-scope (fundamental problems)

**Benefits of Challenge Mode**:
- Surfaces hidden assumptions early
- Explores alternatives explicitly
- Identifies simplification opportunities
- Exposes hidden risks
- Documents decision rationale
- Prevents expensive late-stage pivots

---

### Expert Lens Mode Execution (Multi-Perspective Review)

**Purpose**: Apply systematic multi-expert review methodology to identify gaps across requirements quality, testability, production resilience, and design clarity dimensions.

**When to Use Expert Lens Mode**:
- Complex features requiring high quality standards
- Features with regulatory, security, or compliance requirements
- After significant spec revisions
- Before committing to implementation planning
- When comprehensive quality assessment is needed

**Expert Lens Steps**:

1. Run `{SCRIPT}` to load FEATURE_SPEC (same as normal mode)

2. **Execute Multi-Expert Review** across four lenses:

   **Lens 1: Requirements Quality (Karl Wiegers)**
   Focus: SMART criteria, measurability, stakeholder validation

   Review questions:
   - "How would you validate compliance in production?"
   - "Are acceptance criteria specific and measurable?"
   - "Can automated tests verify the requirement?"
   - "What stakeholder needs are unaddressed?"
   - "Can this be interpreted multiple ways?"

   Flag when:
   - Requirements use vague terms ("gracefully", "efficiently")
   - No quantifiable acceptance criteria
   - Ambiguous stakeholder or scope
   - Missing validation criteria

   **Lens 2: Testability (Gojko Adzic + Lisa Crispin)**
   Focus: Concrete examples, Given/When/Then scenarios, edge case coverage

   Review questions:
   - "Can you provide concrete examples demonstrating this?"
   - "How would the testing team validate this requirement?"
   - "Are all execution paths covered by scenarios?"
   - "Are boundary conditions tested (empty, zero, max, null)?"
   - "Can this be tested independently?"

   Flag when:
   - Generic scenarios without specific values
   - Missing error handling scenarios
   - No boundary condition testing
   - Untestable requirements (no observable behavior)

   **Lens 3: Production Resilience (Michael Nygard)**
   Focus: Failure modes, operational concerns, resilience patterns

   Review questions:
   - "What happens when this component fails?"
   - "Where are timeout specifications?"
   - "How does this behave under load?"
   - "What are the retry and fallback mechanisms?"
   - "How will this be monitored in production?"

   Flag when:
   - No error handling for external dependencies
   - Missing timeout specifications
   - No recovery or fallback mechanisms
   - Missing operational monitoring requirements
   - No consideration of distributed system challenges

   **Lens 4: Design Clarity (Martin Fowler + Alistair Cockburn)**
   Focus: Goal alignment, design coherence, simplification opportunities

   Review questions:
   - "Who is the primary stakeholder and what goal are they achieving?"
   - "Does this requirement directly support a business goal?"
   - "What breaks if we remove this requirement?"
   - "Does this violate separation of concerns?"
   - "Can we achieve the same outcome more simply?"

   Flag when:
   - Requirements lack clear business justification
   - Overcomplicated solutions for simple problems
   - Contradictions with established patterns
   - Violations of design principles
   - Inconsistent handling of similar scenarios

3. **Synthesize Findings**: For each lens, identify top 2-3 issues (max 10 total across all lenses)

4. **Prioritize by Severity**:
   - 🔴 CRITICAL: Security, data loss, compliance, payment integrity
   - 🟠 MAJOR: Missing error handling, UX issues, untestable requirements
   - 🟡 MEDIUM: Edge cases, tech debt, documentation gaps
   - 🟢 LOW: Optimizations, simplification suggestions
   - ℹ️ INFO: Observations, pattern recommendations

5. **Generate Expert Review Questions** (3-5 questions, prioritized by severity):

   ```markdown
   ## Expert Review Question [N]: [Title]

   **Expert Lens**: [Wiegers/Adzic/Nygard/Fowler] - [Focus Area]
   **Severity**: [🔴 CRITICAL | 🟠 MAJOR | 🟡 MEDIUM | 🟢 LOW | ℹ️ INFO]
   **Location**: [Section reference from spec]

   **Issue**:
   > [Quote relevant spec section]

   [Specific description of the gap or problem]

   **Why This Matters**:
   - **User Impact**: [How does this affect end users?]
   - **Technical Risk**: [What could break or go wrong?]
   - **Business Impact**: [Cost, compliance, reputation implications]

   **Question**: [Specific clarification needed]

   **Suggested Answers**:

   | Option | Answer | Implications |
   |--------|--------|--------------|
   | A      | [First option] | [Impact and tradeoffs] |
   | B      | [Second option] | [Impact and tradeoffs] |
   | C      | [Third option] | [Impact and tradeoffs] |
   | Custom | Provide your own answer | [How to provide custom input] |

   **Your choice**: _[Wait for user response]_
   ```

6. **Wait for User Responses**: Collect answers for all questions

7. **Update Spec**: Apply answers and improvements to spec.md, documenting expert review findings

8. **Output Expert Review Summary**:
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   🔍 Expert Review Complete
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Expert Lenses Applied:
   ✅ Requirements Quality (Wiegers)
   ✅ Testability (Adzic/Crispin)
   ✅ Production Resilience (Nygard)
   ✅ Design Clarity (Fowler/Cockburn)

   Findings: [N] issues ([X] critical, [Y] major, [Z] minor)
   Questions Asked: [N]
   Spec Updates: [N] sections improved

   Quality Impact:
   - Clarity: +[X]% improvement
   - Completeness: +[Y]% improvement
   - Testability: +[Z]% improvement
   - Consistency: +[W]% improvement

   Status: ✅ Ready for Planning
   ```

**Benefits of Expert Lens Mode**:
- Systematic quality assessment across multiple dimensions
- Evidence-based findings with clear impact
- Prioritized issues by severity
- Multi-expert validation prevents blind spots
- Actionable recommendations with examples
- Comprehensive coverage of quality aspects

---

### UX Lens Mode Execution (Multi-Perspective UX Review)

**Purpose**: Apply world-class UX, usability, and interaction design methodologies to validate interface-heavy specifications. Identify usability issues, accessibility gaps, and friction points BEFORE planning to prevent UX debt.

**When to Use**:
- Features with user interfaces, workflows, or interactions
- Complex multi-step processes requiring user input
- Mobile or responsive interfaces
- Forms, data entry, or navigation-heavy features
- Any feature where UX quality impacts user success

**Prerequisites**:
1. Run `{SCRIPT}` from repo root once to get FEATURE_DIR and FEATURE_SPEC
2. Read FEATURE_SPEC completely
3. Identify if feature is interface-heavy (skip if backend/API/infrastructure only)

**Execute Four Systematic UX Review Passes**:

#### Pass 1: Usability & Cognitive Load (Nielsen + Norman Lens)

**Focus**: Heuristic evaluation, mental models, cognitive friction

For each interface and interaction specified:

1. **Apply Nielsen's 10 Usability Heuristics**:
   - **#1 Visibility of system status**: Are users always informed (loading states, progress)?
   - **#2 Match between system and real world**: Does it speak user's language (not jargon)?
   - **#3 User control and freedom**: Can users undo/exit actions easily?
   - **#4 Consistency and standards**: Are similar actions handled consistently?
   - **#5 Error prevention**: Does the design prevent errors before they occur?
   - **#6 Recognition over recall**: Are options visible vs. requiring memory?
   - **#7 Flexibility and efficiency**: Are there shortcuts for expert users?
   - **#8 Aesthetic and minimalist design**: Is every element necessary?
   - **#9 Error recovery**: Are error messages actionable and blame-free?
   - **#10 Help and documentation**: Is context-sensitive help available?

2. **Analyze Cognitive Load**:
   - Working memory demands (can users complete without memorizing state?)
   - Decision complexity (are users paralyzed by too many options?)
   - Visual complexity (can users scan and find key information quickly?)
   - Mental model clarity (does interface match users' mental models?)

3. **Evaluate Norman's Design Principles**:
   - **Affordances**: Do interactive elements look clickable/tappable?
   - **Signifiers**: Are clickable areas clearly indicated?
   - **Constraints**: Does design prevent invalid actions?
   - **Mapping**: Are controls spatially related to their effects?
   - **Feedback**: Do actions produce immediate, clear responses?

**Flag when**:
- Critical actions lack confirmation dialogs (destructive operations)
- System status is invisible (loading states, background processes)
- Terminology doesn't match user domain language (tech jargon in business context)
- Users must remember information across screens
- No visual hierarchy (everything looks equally important)
- Inconsistent interaction patterns (button placement varies)

---

#### Pass 2: Task Efficiency & User Goals (Spool + Adzic Lens)

**Focus**: User goals, task flows, mental models, path length

For each user journey and workflow specified:

1. **Goal-Task Alignment**:
   - What job is the user trying to accomplish?
   - Are there unnecessary steps blocking the primary goal?
   - Does the flow match the user's mental model of the task?
   - Are there hidden assumptions about user knowledge?

2. **Task Efficiency Analysis**:
   - **Path length**: Can users complete primary tasks in ≤ 3 clicks/taps?
   - **Input burden**: Are smart defaults provided? Is typing minimized?
   - **Context switching**: Must users leave the app to gather information?
   - **Progressive disclosure**: Is complexity revealed only when needed?

3. **Mental Model Validation**:
   - Does interface structure match how users conceptualize the domain?
   - Are labels and categories aligned with user vocabulary (not system internals)?
   - Do navigation structures reflect user tasks (not database tables)?

**Flag when**:
- Workflows optimized for edge cases instead of common tasks (80/20 rule violated)
- Required fields that users typically don't have at hand
- Task sequences that don't match real-world workflows
- No defaults for fields users typically set to the same value
- Multi-step processes without ability to save partial progress
- Navigation organized by system architecture (not user tasks)

---

#### Pass 3: Interaction Patterns & Mobile Design (Cooper + Wroblewski Lens)

**Focus**: Form design, responsive behavior, micro-interactions

For each interactive component and pattern specified:

1. **Form Design Excellence (Wroblewski)**:
   - **Input types**: Are mobile-optimized keyboards triggered (tel, email, number)?
   - **Label placement**: Are labels above fields (better for mobile scanning)?
   - **Inline validation**: Is feedback immediate (not just on submit)?
   - **Smart defaults**: Are common choices pre-filled?
   - **Optional vs. required**: Is the distinction clear (not just asterisks)?
   - **Error recovery**: Can users fix errors without losing data?

2. **Interaction Patterns (Cooper)**:
   - **Mode awareness**: Are users always aware of current mode (edit vs. view)?
   - **Direct manipulation**: Can users interact with objects directly vs. through dialogs?
   - **Undo/redo**: Can all actions be reversed?

3. **Responsive & Mobile Design**:
   - **Touch targets**: Are buttons ≥ 44px (iOS) / 48px (Android) on mobile?
   - **Thumb zones**: Are primary actions in easy-to-reach areas?
   - **Orientation**: Does layout adapt to portrait/landscape?
   - **Offline behavior**: Are offline states designed (not just error messages)?

4. **Micro-interactions & Feedback**:
   - **Loading states**: Are skeleton screens or progress indicators shown?
   - **Transitions**: Do state changes use smooth animations (not jarring)?
   - **Success states**: Are completions celebrated (not just dismissed)?

**Flag when**:
- Forms with > 7 fields without grouping or steps
- Tiny touch targets on mobile (< 44px)
- No loading states for async operations
- Destructive actions without undo
- Dropdowns with > 10 options (use search/autocomplete instead)
- No visual feedback for clicks/taps
- Multi-column layouts on mobile

---

#### Pass 4: Accessibility & Error Prevention (WCAG + Krug Lens)

**Focus**: WCAG 2.1 AA compliance, simplicity, error prevention

For each user-facing decision point and content:

1. **Accessibility (WCAG 2.1 AA)**:
   - **Keyboard navigation**: Can entire flow be completed without mouse?
   - **Screen reader support**: Are ARIA labels, roles, and landmarks defined?
   - **Color contrast**: Is text readable (4.5:1 for body, 3:1 for large)?
   - **Color independence**: Is information conveyed beyond color alone?
   - **Focus indicators**: Are focused elements clearly visible?
   - **Text resize**: Does layout work at 200% zoom?
   - **Alternative text**: Are images, icons, and charts described?

2. **Simplicity (Krug - "Don't Make Me Think")**:
   - **Obviousness**: Can users understand the page in < 5 seconds?
   - **Clarity**: Are labels and instructions crystal clear (no jargon)?
   - **Scannability**: Can users find what they need by scanning?
   - **Navigation clarity**: Is "You are here" always obvious?

3. **Error Prevention & Recovery**:
   - **Confirmation**: Are destructive actions confirmed?
   - **Constraints**: Are invalid states impossible to enter?
   - **Forgiving formats**: Does system accept multiple input formats?
   - **Error messages**: Are they specific, actionable, polite, and blame-free?
   - **Inline recovery**: Can users fix errors inline (not start over)?

**Flag when**:
- Color is the only indicator of status (red/green without text)
- No keyboard shortcuts or tab order defined
- Placeholder text used as labels (disappears on focus)
- Error messages blame the user ("You entered an invalid value")
- Critical actions have same visual weight as secondary actions
- No "Are you sure?" for delete/archive/permanent operations
- Instructions require users to think or interpret
- No way to recover from errors except starting over

---

**UX Quality Scoring**:

Assess the specification across five UX dimensions (0-10 scale):

- **Usability Score** (0-10): Ease of use, learnability, heuristic compliance
- **Efficiency Score** (0-10): Task completion speed, input burden, minimal steps
- **Accessibility Score** (0-10): WCAG 2.1 AA compliance, keyboard access, screen reader
- **Error Prevention Score** (0-10): Ability to prevent/recover from mistakes
- **Delight Score** (0-10): User satisfaction, emotional response

**Overall UX Quality**: Average of five scores

---

**Output Format for UX Lens Mode**:

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎨 UX Quality Assessment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Feature: [FEATURE_NAME]
Spec: [path/to/spec.md]
Reviewed: [DATE]

## UX Scores

- **Usability**: [X]/10 - [Brief assessment]
- **Efficiency**: [X]/10 - [Brief assessment]
- **Accessibility**: [X]/10 - [Brief assessment]
- **Error Prevention**: [X]/10 - [Brief assessment]
- **Delight**: [X]/10 - [Brief assessment]

**Overall UX Quality**: [X.X]/10

**Status**: [✅ UX Ready | ⚠️ UX Needs Improvement | ❌ UX Issues]

---

## UX Findings

### ❌ Critical Issues (WCAG violations, task-blocking)

---
**Heuristic Violated**: [Nielsen's heuristic #X or WCAG guideline]
**Location**: [Screen/flow - e.g., "Invoice Creation > Line Item Entry"]
**Title**: [User-focused title]

**User Scenario**:
```
User: [Persona - e.g., "Rosa, accountant at small retail shop"]
Goal: [What user is trying to accomplish]
Friction Point: [Specific UX issue]
Consequence: [Impact on user]
```

**Issue**: [Description of the problem]

**Impact**:
- **User Impact**: [How affects task completion, time, errors]
- **Affected Users**: [Who is impacted - all users, mobile users, accessibility users]
- **Business Impact**: [Abandonment, support costs, compliance risk]

**Recommendation**: [Specific, actionable fix with UX pattern reference]

**Example**:
```
BEFORE: [Description of problematic design]
AFTER: [Description of improved design]
Pattern: [e.g., "Inline editing (Cooper)", "Forgiving formats (Wroblewski)"]
```

**Measurable Impact**:
- Reduces task time by ~X% (or Y seconds)
- Prevents ~X% of errors
- Improves accessibility for [user group]

---

[Repeat for each critical finding]

### ⚠️ Major Issues (Significant friction in common tasks)

[Same format as critical issues]

### ⚠️ Minor Issues (Efficiency opportunities, edge cases)

[Same format as critical issues]

---

## Quick Wins (High Impact, Low Effort)

1. **[Quick win title]** - [What to change] → [Expected impact]
2. **[Quick win title]** - [What to change] → [Expected impact]
3. **[Quick win title]** - [What to change] → [Expected impact]

---

## Recommendations

**Immediate Actions** (Before Planning):
1. [Action 1 with screen/section reference]
2. [Action 2 with screen/section reference]
3. [Action 3 with screen/section reference]

**Suggested Next Step**:

[IF Overall UX Quality ≥ 7 AND no WCAG violations]:
  ✅ UX quality acceptable. Proceed to `/speckit.plan`

[IF Overall UX Quality 4-6 OR minor WCAG gaps]:
  ⚠️ Address major UX issues before planning or proceed with noted risks

[IF Overall UX Quality < 4 OR WCAG Level A/AA violations]:
  ❌ Critical UX issues present. Revise spec to address before planning
  → WCAG violations create legal compliance risk
  → Task-blocking issues will cause user abandonment

**Optional Enhancements**:
- Run `/speckit.analyze-ux` for faster UX validation during iterations
- Review Nielsen's heuristics documentation for pattern examples
- Conduct user testing on high-risk flows before implementation

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Append UX findings to spec.md**:
- Add "UX Review Findings" section
- Include top 5 critical/major issues
- Update UX Quality Assessment scores
- Add identified UX Friction Points

**Benefits of UX Lens Mode**:
- Early UX validation prevents costly redesigns post-implementation
- WCAG 2.1 AA compliance validation reduces legal risk
- Nielsen's heuristics ensure industry-standard usability
- Mobile-first optimization for 60%+ mobile traffic
- Evidence-based findings with user scenarios
- Prioritized quick wins for immediate improvements
- Accessibility compliance built into workflow

---

### Edge Case Detection Mode Execution (Systematic Boundary & Error Analysis)

**Purpose**: Systematically identify missing edge cases, boundary conditions, error scenarios, and state transitions to ensure production resilience and comprehensive testing coverage.

**Activation**: `/speckit.clarify --edge-cases` or `/speckit.clarify --edge`

**Methodology**: Apply Quality Engineering best practices for boundary value testing, error condition discovery, state transition validation, and race condition analysis.

**Execution Steps**:

1. **Load Feature Context**: Run `{SCRIPT}` to get `FEATURE_DIR` and `FEATURE_SPEC`

2. **Read Complete Specification**: Load spec.md including all User Stories, FRs, BRs, and acceptance criteria

3. **Systematic Edge Case Analysis**: Apply 6 detection categories to each requirement

   **Category 1: Boundary Conditions**
   - **Empty/Zero/Null**: What happens with empty strings, zero quantities, null values?
   - **Maximum Values**: What are upper limits? (string length, array size, numeric max, file size)
   - **Minimum Values**: What are lower limits? (negative numbers, below threshold)
   - **Edge of Range**: Values at boundaries (exactly 0, exactly max, one above/below limit)
   - **Large Scale**: Performance at 10x, 100x, 1000x expected volume

   **Flag when**:
   - No specification of maximum string lengths, array sizes, or numeric limits
   - No handling of empty/null inputs specified
   - No validation of minimum values or negative numbers
   - No performance characteristics at scale defined

   **Category 2: Error Conditions**
   - **External Failures**: API timeout, network error, service unavailable, rate limit exceeded
   - **Data Validation Failures**: Invalid format, type mismatch, constraint violation
   - **Authorization Failures**: Insufficient permissions, expired token, missing access
   - **Resource Exhaustion**: Out of memory, connection pool exhausted, disk full
   - **Concurrent Conflicts**: Optimistic lock failure, version mismatch, race condition

   **Flag when**:
   - No error handling specified for external dependencies
   - No timeout values defined
   - No retry/backoff strategy documented
   - No fallback or degraded mode specified
   - No concurrent access handling defined

   **Category 3: State Transitions**
   - **Invalid Transitions**: Can state go from A → C without B? What prevents it?
   - **Terminal States**: Can the system recover from error/cancelled/failed states?
   - **State Persistence**: What happens if app crashes mid-workflow?
   - **Idempotency**: Can the same operation be safely retried?
   - **Concurrent State Changes**: Two users editing same record simultaneously

   **Flag when**:
   - No state machine or valid transitions documented
   - No handling of invalid state transitions
   - No crash recovery or resume-from-checkpoint logic
   - No idempotency guarantees for critical operations
   - No optimistic locking or version control specified

   **Category 4: Data Quality Issues**
   - **Duplicate Data**: Same entity created twice, duplicate emails/usernames
   - **Orphaned Data**: Parent deleted but children remain (or blocked by FK)
   - **Stale Data**: Cached data outdated, UI showing old state
   - **Inconsistent Data**: Related fields out of sync (total != sum of items)
   - **Corrupted Data**: Malformed JSON, invalid date formats, encoding issues

   **Flag when**:
   - No uniqueness constraints defined
   - No cascade delete or orphan handling specified
   - No cache invalidation strategy documented
   - No data integrity checks or validation
   - No handling of malformed input data

   **Category 5: Race Conditions & Concurrency**
   - **Read-Modify-Write Races**: Two users updating same field simultaneously
   - **TOCTOU (Time-of-Check-Time-of-Use)**: State changes between check and action
   - **Double Processing**: Webhook delivered twice, job executed concurrently
   - **Distributed Locks**: Multiple workers claiming same task
   - **Eventual Consistency**: Reads seeing stale data after writes

   **Flag when**:
   - No optimistic/pessimistic locking strategy
   - No idempotency keys for critical operations
   - No duplicate detection for webhooks/messages
   - No distributed coordination mechanism (if needed)
   - No acknowledgment of eventual consistency windows

   **Category 6: Multi-Tenant Isolation**
   - **Cross-Tenant Data Leakage**: User A sees user B's data
   - **Quota/Rate Limit Bypass**: Tenant exceeds allocated resources
   - **Tenant Context Loss**: Request processed without tenant filter
   - **Shared Resource Contention**: One tenant starves others
   - **Migration/Deletion Issues**: Orphaned data after tenant deletion

   **Flag when** (if multi-tenant system):
   - No tenant_id filtering on all queries
   - No row-level security or data isolation
   - No per-tenant quotas or rate limits
   - No tenant context validation
   - No cleanup procedures for tenant deletion

4. **Prioritize Edge Cases by Risk**:
   - 🔴 CRITICAL: Data corruption, security breach, financial loss, compliance violation
   - 🟠 MAJOR: User-blocking errors, data loss, incorrect calculations, poor UX
   - 🟡 MEDIUM: Performance degradation, confusing errors, minor data issues
   - 🟢 LOW: Edge cases in rare scenarios, cosmetic issues
   - ℹ️ INFO: Potential future improvements, defensive checks

5. **Generate Edge Case Questions** (max 5 questions, prioritized by risk):

   For each critical edge case gap, create a targeted question:

   ```markdown
   ## Edge Case Question [N]: [Category] - [Brief Title]

   **Category**: [Boundary Conditions | Error Conditions | State Transitions | Data Quality | Race Conditions | Multi-Tenant Isolation]
   **Severity**: [🔴 CRITICAL | 🟠 MAJOR | 🟡 MEDIUM | 🟢 LOW | ℹ️ INFO]
   **Affected Requirement**: [FR-XXX or User Story X]

   **Missing Edge Case**:
   [Describe the unhandled scenario with concrete example]

   **Example Failure Scenario**:
   ```
   Given: [Specific setup - e.g., "User has 999 items in cart (max is 1000)"]
   When: [Action - e.g., "User adds one more item"]
   Then: [What happens? What SHOULD happen?]
   Actual Behavior: [Not specified OR spec says nothing]
   Expected Behavior: [What needs to be defined]
   ```

   **Why This Matters**:
   - **User Impact**: [Specific consequence - e.g., "Order total miscalculated by $X"]
   - **Production Risk**: [What breaks in prod - e.g., "500 error, user can't checkout"]
   - **Testing Gap**: [Why this won't be caught - e.g., "No boundary tests for cart limits"]

   **Suggested Options**:

   | Option | Behavior | Tradeoffs |
   |--------|----------|-----------|
   | A) [Option 1] | [What happens] | [Pros/Cons - implementation cost, UX impact] |
   | B) [Option 2] | [What happens] | [Pros/Cons] |
   | C) [Option 3] | [What happens] | [Pros/Cons] |

   **Industry Best Practice**: [Pattern from production systems - e.g., "Stripe: idempotency key required", "GitHub: optimistic locking with retry"]

   **Recommended Approach**: [Which option + why]

   **Your Decision**: _[Wait for user response]_
   ```

6. **Present ALL Questions at Once**: Show all 5 questions together before waiting for responses

7. **Wait for User Input**: User responds with choices for all questions (e.g., "Q1: A, Q2: Custom - [details], Q3: B")

8. **Update Specification**: For each answered question:
   - Add new Given/When/Then scenarios for edge cases
   - Update FRs with boundary conditions and error handling
   - Add new BRs if constraints discovered
   - Update acceptance criteria with specific test cases
   - Add Risk Flags if high-severity edge cases identified

9. **Generate Edge Case Summary**: Create structured output

**Output Format**:

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 Edge Case Detection Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Feature: [FEATURE_NAME]
Spec: [path/to/spec.md]
Analyzed: [DATE]

## Edge Cases Identified

**Total Edge Cases Found**: X
- 🔴 CRITICAL: X (data/security/financial risks)
- 🟠 MAJOR: X (user-blocking errors, data loss)
- 🟡 MEDIUM: X (degraded performance, confusing UX)
- 🟢 LOW: X (rare scenarios)
- ℹ️ INFO: X (defensive improvements)

## Coverage by Category

| Category | Critical | Major | Medium | Low | Total |
|----------|----------|-------|--------|-----|-------|
| Boundary Conditions | X | X | X | X | X |
| Error Conditions | X | X | X | X | X |
| State Transitions | X | X | X | X | X |
| Data Quality | X | X | X | X | X |
| Race Conditions | X | X | X | X | X |
| Multi-Tenant Isolation | X | X | X | X | X |

## Questions Asked (Max 5)

[Edge Case Questions in format above]

---

## Specification Updates Applied

**User Stories Updated**: [List with edge case scenarios added]
**Functional Requirements Updated**: [FRs with boundary/error handling]
**Business Rules Added**: [New constraints discovered]
**Risk Flags Added**: [High-severity edge cases flagged]

**Edge Case Test Scenarios Added**: X new Given/When/Then scenarios

---

## Production Readiness Assessment

**Boundary Handling**: [🟢 Comprehensive | 🟡 Partial | 🔴 Gaps Remain]
**Error Recovery**: [🟢 Comprehensive | 🟡 Partial | 🔴 Gaps Remain]
**State Management**: [🟢 Comprehensive | 🟡 Partial | 🔴 Gaps Remain]
**Data Integrity**: [🟢 Comprehensive | 🟡 Partial | 🔴 Gaps Remain]
**Concurrency Handling**: [🟢 Comprehensive | 🟡 Partial | 🔴 Gaps Remain]
**Tenant Isolation**: [🟢 Comprehensive | 🟡 Partial | 🔴 Gaps Remain | N/A]

**Overall Edge Case Coverage**: [🟢 Production-Ready | 🟡 Needs Attention | 🔴 Significant Gaps]

---

## Remaining Gaps (Deferred to Future)

[Edge cases identified but not addressed in this session]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**When to Use Edge Case Detection Mode**:
- Before planning high-risk features (payment, security, financial)
- After initial specification to validate production resilience
- When feature involves state machines, workflows, or complex transitions
- For multi-tenant features requiring strict data isolation
- When integrating with external APIs or event-driven systems
- Before implementation of features with scale/performance requirements

**Benefits of Edge Case Detection Mode**:
- Prevents production incidents from unhandled edge cases
- Identifies race conditions and concurrency issues early
- Ensures multi-tenant data isolation (if applicable)
- Validates error recovery and resilience patterns
- Creates comprehensive test scenarios for QA
- Reduces post-launch bugs and support tickets
- Builds production-ready specifications from the start

---

### Standard Execution Steps (Normal Mode)

If NOT in Challenge Mode, follow these steps:

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

   **ENHANCEMENT: Context-Aware Question Selection** (Smart Questioning):

   For EACH ambiguity identified, generate question VARIANTS from different perspectives, then select the BEST one based on context:

   **Question Perspectives**:

   - **Technical** (How/What): Specific implementation details, numbers, formats
     - Example: "What's the target API response time? (<100ms / <500ms / <1s / <5s)"
     - Best when: Spec mentions technical requirements but lacks specifics

   - **User** (Who/Why): User priorities, workflows, experience impacts
     - Example: "Which user workflows need the fastest response time? (search/filter / data export / all operations)"
     - Best when: Multiple use cases exist with potentially different needs

   - **Business** (Value/Tradeoff): Cost-benefit analysis, priority decisions, ROI
     - Example: "What's the user impact of 500ms vs 2s response time? (critical / moderate / minimal)"
     - Best when: Trade-offs exist between complexity and value

   - **Risk** (What-if): Scale considerations, failure modes, edge cases
     - Example: "What happens if API response time degrades under 10x load? (acceptable / degraded UX / critical failure)"
     - Best when: Scalability or reliability concerns exist

   **Context-Based Selection Algorithm**:

   For each ambiguity, analyze the spec context to choose the best perspective:

   ```yaml
   IF spec mentions "real-time", "critical path", or "performance-sensitive":
     → Choose Technical perspective (need specific numbers)
     → Rationale: Precision required for technical implementation

   ELSE IF spec has vague priorities ("fast", "good UX", "responsive"):
     → Choose User perspective (understand what matters most)
     → Rationale: Priorities unclear, need user-centric clarification

   ELSE IF spec has competing concerns (speed vs cost, features vs time):
     → Choose Business perspective (understand trade-offs)
     → Rationale: Decision requires cost-benefit analysis

   ELSE IF spec lacks error handling, scale considerations, or edge cases:
     → Choose Risk perspective (surface hidden issues)
     → Rationale: Resilience and scale not addressed

   ELSE (default):
     → Choose Technical perspective (most direct clarification)
   ```

   **Example: Performance Ambiguity**

   Ambiguity detected: "The API should respond quickly"

   Generated variants:
   - Technical: "What's the target response time? (<100ms / <500ms / <1s / <5s)"
   - User: "Which operations need fastest response? (search / export / all)"
   - Business: "What's the user impact of slower response? (critical / moderate / minimal)"
   - Risk: "What happens if response degrades under load?"

   Context analysis:
   - Spec mentions "search functionality" and "user experience"
   - No "real-time" or "critical" keywords
   - Multiple user workflows described

   Selected: **User perspective**
   Rationale: Multiple workflows exist, need to understand which ones prioritize speed

   Final question:
   ```
   ❓ Performance Priorities (User Perspective)

   Which user workflows require the fastest API response times?

   This helps prioritize optimization efforts and set appropriate targets for different operations.

   **Recommended:** Search/filter operations (typically interactive, users expect <500ms)

   | Option | Description |
   |--------|-------------|
   | A | Search and filter operations (interactive queries) |
   | B | Data export and report generation (background operations) |
   | C | All operations equally (uniform performance target) |
   | D | Real-time dashboard updates (continuous data flow) |
   | Short | Specify other workflows (<=5 words) |

   You can reply with the option letter, accept the recommendation ("yes"), or specify your own answer.
   ```

   **Question Annotation Format**:

   When presenting questions, include perspective label to help user understand the question's purpose:

   - `❓ [Topic] (Technical)` - For implementation specifics
   - `❓ [Topic] (User Perspective)` - For user priorities and workflows
   - `❓ [Topic] (Business Perspective)` - For value and trade-off decisions
   - `❓ [Topic] (Risk Perspective)` - For scale and failure considerations

   **Benefits of Context-Aware Selection**:
   - Same 5-question limit (no workflow change)
   - Questions reveal deeper insights (not just technical gaps)
   - Smart selection based on context (not random perspective choice)
   - Annotations help user understand WHY each question matters
   - Reduces follow-up clarification rounds (get it right first time)

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
