---
description: Fast UX quality analysis with actionable recommendations - lightweight usability review without interactive questions
scripts:
  sh: .specify/scripts/bash/check-prerequisites.sh --json
  ps: .specify/scripts/powershell/check-prerequisites.ps1 -Json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

**Purpose**: Perform a fast, non-interactive UX quality analysis of the feature specification to identify top usability issues and provide actionable recommendations without the full review process.

**When to Use**:
- Quick UX check before planning
- After UI spec updates to validate improvements
- When you need UX scores but not full clarification
- Before committing to implementation
- As a pre-check before UX Lens Mode
- Validate accessibility compliance (WCAG 2.1 AA)

**Output**: UX quality scores + top 5 issues by priority + Quick Wins

---

## Execution Steps

1. **Load Feature Context** (v2.3 Enhanced):

   Run `{SCRIPT}` from repo root to get:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - Optional: `IMPL_PLAN`, `TASKS`

   **Load spec-metadata.json** (NEW v2.3):
   ```bash
   IF specs/[FEATURE]/spec-metadata.json exists:
     → Load metadata for context:
       - phase: Current workflow phase
       - metadata.risk_level: HIGH/MEDIUM/LOW (affects analysis depth)
       - metadata.overall_quality: Previous quality score (track improvement)
       - metadata.ux_quality: Previous UX score (track UX improvement)
       - approvals: Phase approval status
   ```

   **Context-Aware UX Analysis**:
   - IF risk_level == HIGH: Apply extra scrutiny to accessibility and error prevention
   - IF ux_quality from previous analysis: Compare to detect improvement/regression
   - IF UI-SPEC.md exists: Load for component system audit (Angular/Kendo integration)

2. **Read Specification**: Load the complete spec.md file

3. **Validate Feature Type**:
   - Check if feature is interface-heavy (user interfaces, workflows, interactions)
   - If backend-only, API-only, or infrastructure: Report "N/A - Feature has no user interface"
   - If interface-heavy: Proceed with UX analysis

4. **Calculate UX Quality Scores**: Assess across five dimensions (0-10 scale)

   **Usability Score**: Ease of use and learnability

   Evaluate using Nielsen's 10 Usability Heuristics:
   - #1 Visibility of system status: Are users always informed (loading states, progress)?
   - #2 Match between system and real world: Does it speak user's language (not jargon)?
   - #3 User control and freedom: Can users undo/exit actions easily?
   - #4 Consistency and standards: Are similar actions handled consistently?
   - #5 Error prevention: Does design prevent errors before they occur?
   - #6 Recognition over recall: Are options visible vs. requiring memory?
   - #7 Flexibility and efficiency: Are there shortcuts for expert users?
   - #8 Aesthetic and minimalist design: Is every element necessary?
   - #9 Error recovery: Are error messages actionable and blame-free?
   - #10 Help and documentation: Is context-sensitive help available?

   Scoring:
   - 8-10: Intuitive, self-evident, follows established UX patterns
   - 5-7: Mostly usable with some friction points
   - 2-4: Confusing, requires training or documentation
   - 0-1: Unusable, violates basic usability heuristics

   **Efficiency Score**: Task completion speed and input burden

   Evaluate:
   - Can users complete primary tasks in ≤ 3 clicks/taps?
   - Are smart defaults provided for common values?
   - Is progressive disclosure used (complexity revealed only when needed)?
   - Is context switching minimized (no leaving app for info)?
   - Are multi-step processes kept minimal?

   Scoring:
   - 8-10: Optimized for frequent tasks, smart defaults, minimal steps (≤3 clicks)
   - 5-7: Reasonable efficiency with some unnecessary steps
   - 2-4: Inefficient workflows, excessive clicks/typing
   - 0-1: Painful to use, major inefficiencies

   **Accessibility Score**: Universal access and WCAG 2.1 AA compliance

   Evaluate:
   - Can entire flow be completed without mouse (keyboard only)?
   - Are ARIA labels and screen reader support specified?
   - Is color contrast sufficient (4.5:1 for text, 3:1 for large)?
   - Is information conveyed beyond color alone?
   - Are focus indicators clearly visible?
   - Does layout work at 200% zoom?
   - Are images, icons, and charts described (alt text)?
   - Can animations be disabled (prefers-reduced-motion)?

   Scoring:
   - 8-10: WCAG 2.1 AA compliant, keyboard accessible, screen reader friendly
   - 5-7: Mostly accessible with minor gaps
   - 2-4: Significant accessibility barriers
   - 0-1: Inaccessible to users with disabilities

   **Error Prevention Score**: Ability to prevent and recover from mistakes

   Evaluate:
   - Are destructive actions confirmed ("Are you sure?")?
   - Are invalid states prevented (disabled buttons, constraints)?
   - Are error messages specific, actionable, and blame-free?
   - Can users fix errors inline (not start over)?
   - Does system accept multiple input formats (dates, phone)?
   - Is forgiving input parsing used?

   Scoring:
   - 8-10: Errors prevented by design, easy recovery, forgiving formats
   - 5-7: Basic error handling, some prevention
   - 2-4: Errors easily made, poor recovery
   - 0-1: Error-prone, no recovery mechanisms

   **Delight Score**: User satisfaction and emotional response

   Evaluate:
   - Are success states celebrated (not just dismissed)?
   - Do transitions feel smooth (not jarring)?
   - Are loading states engaging (skeleton screens, progress)?
   - Is micro-feedback provided (button press animations)?
   - Are empty states helpful (not just blank)?
   - Is the interface pleasant to use?

   Scoring:
   - 8-10: Delightful interactions, exceeds expectations
   - 5-7: Functional but not memorable
   - 2-4: Frustrating or tedious
   - 0-1: Actively unpleasant to use

   **Overall UX Quality**: Average of the five scores

5. **Detect UI Specifications** (Optional - when UI-SPEC.md present):

   Check if `${FEATURE_DIR}/UI-SPEC.md` exists:
   - IF exists → Enable UI Design Quality scoring + Component System Audit (Steps 6-7)
   - IF not exists → Skip to Step 8 (UX-only mode)

6. **Calculate UI Design Quality Scores** (0-10 scale, ONLY when UI-SPEC.md present):

   **Visual Clarity**: Simplicity, purpose-driven design (8-10: minimal/clear, 0-1: chaotic)
   **Component Consistency**: Design system adherence, reuse (8-10: excellent tokens, 0-1: no system)
   **Visual Hierarchy**: Emphasis, scannability (8-10: clear primary/secondary, 0-1: flat)
   **Responsive Design**: Mobile-first, touch targets ≥44px (8-10: mobile-first, 0-1: not responsive)
   **Design System Adherence**: Framework usage ≥80% (8-10: high framework usage, 0-1: none)

   **Overall Design Quality**: Average of UX (5) + UI (5) = 10 dimensions

7. **Component System Audit** (ONLY when UI-SPEC.md present):

   **Framework Detection**:
   Default: Angular + Kendo UI (project standard)

   Prompt: "Component library framework? [Angular/Kendo] (or type to change)"

   Available alternatives (for future framework changes):
   - React + Material-UI (MUI)
   - React + Chakra UI
   - Angular + Angular Material
   - Vue + Vuetify
   - Generic (custom framework)
   - Skip (no component audit)

   If user accepts default (Enter): Use Angular/Kendo template
   If user types alternative: Use specified framework template
   If user types "skip": Skip component audit

   **Audit Process**:
   a. Load template from `.specify/templates/component-audit-templates/[framework].md`
   b. Scan UI-SPEC.md for component usage, hardcoded values, custom components
   c. Categorize: ✅ Used Correctly | ❌ Custom Reimplementation | ⚠️ Inconsistent
   d. Check design tokens: Colors, Spacing, Typography (% adherence)
   e. Calculate Component Reuse Score: (Library / Total) × 100%
   f. Optional MCP: Query component docs if MCP available (e.g., kendo-angular-assistant)
   g. Generate recommendations with before/after examples

8. **Identify Top UX Issues**: Scan spec for common problems, prioritized by severity

   **Severity Classification**:
   - 🔴 CRITICAL: WCAG Level A/AA violations (legal risk), task-blocking issues affecting primary workflows, no mobile optimization for mobile-first app, data loss risks
   - 🟠 MAJOR: Significant friction in common tasks, poor error recovery, confusing navigation, inconsistent patterns, missing accessibility features (non-WCAG)
   - 🟡 MEDIUM: Efficiency opportunities, nice-to-have patterns, edge case improvements
   - 🟢 LOW: Delight enhancements, minor polish, cosmetic improvements
   - ℹ️ INFO: UX observations, pattern suggestions, best practices

   **UX Issue Categories to Check**:

   a. **Nielsen's Heuristics Violations**:
      - Invisible system status (no loading indicators, no progress feedback)
      - System jargon instead of user language
      - No undo/exit for critical actions
      - Inconsistent interaction patterns
      - No error prevention (validation only on submit)
      - Hidden options requiring recall (no autocomplete, no visible choices)

   b. **Accessibility Gaps** (WCAG 2.1 AA):
      - No keyboard navigation specified
      - Missing ARIA labels/roles
      - Color-only status indicators (red/green without text)
      - Low color contrast (< 4.5:1 for text)
      - Placeholder text used as labels
      - No focus indicators
      - Images without alt text
      - No text resize support

   c. **Mobile Optimization Issues**:
      - Touch targets < 44px (iOS) / 48px (Android)
      - Actions outside thumb zones
      - No responsive design specified
      - Desktop-first approach for mobile-first app
      - Multi-column layouts on mobile
      - No gesture support (swipe, pinch, long-press)

   d. **Task Efficiency Issues**:
      - Primary tasks require > 3 clicks/taps
      - No smart defaults for common values
      - Required fields users don't have at hand
      - Multi-step processes without save draft
      - Context switching (must leave app for info)
      - No progressive disclosure

   e. **Form Design Issues**:
      - Forms with > 7 fields without grouping
      - No inline validation
      - Error messages only on submit
      - No mobile-optimized keyboards (tel, email, number)
      - Labels not above fields (poor mobile scanning)
      - Dropdowns with > 10 options (no search/autocomplete)

   f. **Error Prevention/Recovery Issues**:
      - Destructive actions without confirmation
      - No undo for critical operations
      - Error messages blaming user
      - No inline error recovery
      - Starting over required after errors
      - No forgiving input formats

   **Limit**: Flag top 5 issues maximum (by severity, then impact)

   **For Each Issue Category**: Apply Socratic validation to verify significance:
   - "What user goal is blocked or delayed?" (specific task/workflow)
   - "How many users are affected?" (all, mobile, accessibility, cohort, % estimate)
   - "What is the cost?" (time per task, abandonment, support, legal risk)
   - "Why this severity?" (WCAG violation = legal, task-blocking = abandonment)

   **Socratic Filter**: If you cannot answer these 4 questions with specifics, the issue may be a false positive. Only include issues where all 4 questions have clear, evidence-based answers focusing on actual user impact.

6. **Identify Quick Wins**: High-impact, low-effort improvements

   Find 3-5 changes that:
   - Significantly improve UX with minimal development effort
   - Address common friction points
   - Have measurable impact (time saved, errors prevented)
   - Can be implemented quickly

   Examples:
   - Add autocomplete to search fields (saves 15 seconds per search)
   - Increase button size to 48px (prevents 30% of mobile tap errors)
   - Add "Save draft" button (prevents data loss on timeout)
   - Use forgiving date formats (accepts "01/15/25", "1-15-2025", "Jan 15")
   - Add keyboard shortcuts for power users

7. **Generate Structured UX Recommendations**: Convert UX scores into actionable, prioritized improvement recommendations

   **Purpose**: Transform UX scores into specific, actionable fixes following Nielsen's heuristic evaluation format.

   **Recommendation Structure**:

   For each UX dimension scoring <7/10, generate recommendations:

   ```markdown
   ### [Severity]: [Dimension] Issues (Score: [X]/10)

   **Heuristic Violated**: [Nielsen Heuristic or WCAG Guideline]

   **Problem**:
   - [Specific issue from spec.md with section reference]
   - [Specific issue from spec.md with section reference]
   - [Specific issue from spec.md with section reference]

   **Recommended Fix**:
   [Numbered list of specific, actionable steps]

   **Example** (from similar feature):
   [Concrete example or pattern reference]

   **Impact**: +[N]% [dimension] score ([current]/10 → [target]/10)
   ```

   **Severity Prioritization**:
   1. **🔴 CRITICAL**: Scores 0-4/10 (WCAG Level A/AA violations, task-blocking issues)
   2. **🟠 MAJOR**: Scores 4-6/10 (significant usability problems, efficiency gaps)
   3. **🟡 MEDIUM**: Scores 6-7/10 (minor friction, optimization opportunities)

   **Nielsen's Heuristic Categories**:
   1. Visibility of system status
   2. Match between system and real world
   3. User control and freedom
   4. Consistency and standards
   5. Error prevention
   6. Recognition rather than recall
   7. Flexibility and efficiency of use
   8. Aesthetic and minimalist design
   9. Help users recognize, diagnose, and recover from errors
   10. Help and documentation

   **WCAG 2.1 AA Guidelines** (for Accessibility dimension):
   - Guideline 1.1: Text Alternatives
   - Guideline 1.3: Adaptable
   - Guideline 2.1: Keyboard Accessible
   - Guideline 2.4: Navigable
   - Guideline 3.2: Predictable
   - Guideline 4.1: Compatible

   ---

   **Quick Wins Identification**:

   **Criteria for Quick Wins**:
   - **High Impact**: Improves score by ≥2 points
   - **Low Effort**: ≤4 hours development time
   - **Clear Implementation**: Well-documented pattern or example

   **Format**:
   ```markdown
   ## Quick Wins (High Impact, Low Effort)

   1. **[Fix Title]** ([N] hours dev, +[M] points [dimension])
      - Problem: [Brief description]
      - Fix: [One-sentence action]
      - Example: [Reference or pattern]

   2. **[Fix Title]** ([N] hours dev, +[M] points [dimension])
      [Same format]

   3-5. [Additional quick wins, max 5 total]
   ```

   ---

   **Summary**:

   **Total Improvements**: [N] critical, [M] major, [P] medium
   **Expected Score Gain**: [Current]/10 → [Target]/10 (+[X]%)
   **Development Effort**: ~[N] hours
   **User Time Saved**: ~[X]% per task (from efficiency improvements)

8. **Generate Analysis Report**: Create concise, actionable output

9. **UX Analysis Review Gate (Evidence-Based Self-Check)**

**Purpose**: Validate UX analysis completeness before presenting results.

### Evidence Collection (Mandatory)

❓ **"Were all 5 UX dimensions scored?"**
Action Required:
  - Verify all 5 UX scores calculated (Usability, Efficiency, Accessibility, Error Prevention, Delight)
  - Show ACTUAL scores with justifications
  - Report: Score for each dimension + overall

Expected Evidence:
  ✓ Usability score: [X]/10 - [Nielsen heuristic assessment]
  ✓ Efficiency score: [X]/10 - [Task completion assessment]
  ✓ Accessibility score: [X]/10 - [WCAG 2.1 AA compliance]
  ✓ Error Prevention score: [X]/10 - [Error handling assessment]
  ✓ Delight score: [X]/10 - [User satisfaction assessment]
  ✓ Overall UX quality: [X.X]/10 (average of 5 dimensions)

❓ **"Was WCAG 2.1 AA compliance checked?"**
Action Required:
  - Verify WCAG guidelines validated
  - Show ACTUAL guideline violations (if any)
  - Report: Compliance status + specific violations

Expected Evidence:
  ✓ WCAG compliance status: [🟢 Compliant | 🟡 Minor Gaps | 🔴 Violations]
  ✓ Specific violations: [List WCAG guideline numbers if violations found]
  ✓ Accessibility gaps: [List missing features: keyboard nav, ARIA, contrast, etc.]

❓ **"Were usability issues prioritized?"**
Action Required:
  - Count issues by severity (🔴/🟠/🟡/🟢/ℹ️)
  - Show ACTUAL severity breakdown
  - Report: Impact prioritization

Expected Evidence:
  ✓ 🔴 CRITICAL: [N] issues (WCAG violations, task-blocking)
  ✓ 🟠 MAJOR: [N] issues (significant friction)
  ✓ 🟡 MEDIUM: [N] issues (efficiency opportunities)
  ✓ 🟢 LOW: [N] issues (delight enhancements)
  ✓ ℹ️ INFO: [N] observations
  ✓ Total issues flagged: ≤5 (top issues only)

❓ **"Are recommendations specific and actionable?"**
Action Required:
  - Show top 3-5 recommendations with section references
  - Verify heuristics/guidelines cited (Nielsen or WCAG)
  - Check quick wins identified (high impact, low effort)

Expected Evidence:
  Recommendations Summary:
  ✓ Critical: [N] (WCAG violations, task blockers)
  ✓ Major: [M] (usability/efficiency issues)
  ✓ Medium: [P] (optimization opportunities)
  ✓ Quick Wins: [Q] identified (≤4 hours, ≥2 points impact)
  Total expected improvement: [X]% ([Current]/10 → [Target]/10)

IF any evidence is MISSING:
  ❌ CANNOT report completion
  → Gather missing evidence first
  → Re-run this step with complete evidence

### Hallucination Prevention (7 Red Flags for UX Analysis)

```yaml
Detect and BLOCK these patterns:

🚨 "UX score X/10" WITHOUT showing dimension breakdown
   → Self-correction: "Wait, I need to show all 5 UX dimension scores"

🚨 "Accessibility issues" WITHOUT citing WCAG guidelines
   → Self-correction: "Must reference specific WCAG 2.1 AA guidelines"

🚨 "Usability problems" WITHOUT specific examples from spec
   → Self-correction: "Need to quote actual spec sections with issues"

🚨 "Recommendations provided" WITHOUT impact estimates
   → Self-correction: "Must explain expected impact (time saved, errors prevented)"

🚨 Scoring >7/10 WITHOUT justification for high-quality features
   → Self-correction: "High scores need evidence of excellent UX patterns"

🚨 Generic Nielsen heuristics NOT tied to actual spec
   → Self-correction: "Must apply heuristics to specific spec content"

🚨 Missing quick wins (high-impact, low-effort improvements)
   → Self-correction: "I should identify at least 3-5 quick wins"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

✅ **UX READY**:
```yaml
Criteria (ALL must be met):
  - All 5 UX dimensions scored with justifications
  - WCAG 2.1 AA compliance validated (no Level A/AA violations)
  - No 🔴 CRITICAL UX issues found
  - Top 5 issues identified with severity and recommendations
  - Overall UX quality ≥ 7/10
  - Quick wins identified

IF ALL criteria met:
  → Proceed with recommendation: /speckit.clarify or /speckit.plan
```

⚠️ **NEEDS REVIEW** (can proceed with awareness):
```yaml
Criteria:
  - Overall UX quality 4-6/10
  - Minor WCAG gaps (not Level A/AA violations)
  - Minor UX issues (🟠 MAJOR or 🟡 MEDIUM only)
  - Recommendations actionable but non-blocking

IF criteria met:
  → Present issues to user
  → Recommend: /speckit.clarify --ux for comprehensive UX review
```

❌ **NOT READY** (more work needed):
```yaml
Criteria (ANY triggers NOT READY):
  - 🔴 CRITICAL UX issues (WCAG Level A/AA violations, task-blocking)
  - Overall UX quality < 4/10
  - Missing accessibility features (keyboard nav, screen reader support)
  - Mobile optimization missing for mobile-first app
  - Analysis incomplete (feature not interface-heavy, skip UX analysis)

IF NOT READY:
  → Present critical UX issues with evidence
  → Recommend: "Revise spec to address WCAG violations and task-blocking issues"
  → STOP workflow progression
```

### Output Format (Present to User - ONLY if evidence provided)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎨 UX Quality Analysis Review
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ UX READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**Spec Analyzed**: specs/[FEATURE]/spec.md

**UX Quality Scores**:
  - Usability: [X]/10 - [Nielsen heuristic assessment]
  - Efficiency: [X]/10 - [Task completion assessment]
  - Accessibility: [X]/10 - [WCAG 2.1 AA compliance]
  - Error Prevention: [X]/10 - [Error handling assessment]
  - Delight: [X]/10 - [User satisfaction assessment]

**Overall UX Quality**: [X.X]/10

**WCAG 2.1 AA Compliance**: [🟢 Compliant | 🟡 Minor Gaps | 🔴 Violations]
**Mobile Optimization**: [🟢 Optimized | 🟡 Partial | 🔴 Desktop-Only]

**Issues by Severity**:
  - 🔴 CRITICAL: [N] (WCAG violations, task-blocking)
  - 🟠 MAJOR: [N] (significant friction)
  - 🟡 MEDIUM: [N] (efficiency opportunities)
  - 🟢 LOW: [N] (delight enhancements)
  - ℹ️ INFO: [N]

**Recommendations Summary**:
  - 🔴 CRITICAL: [N] (WCAG violations, task blockers)
  - 🟠 MAJOR: [M] (usability/efficiency issues)
  - 🟡 MEDIUM: [P] (optimization opportunities)
  - Quick Wins: [Q] identified (≤4 hours, ≥2 points impact)
  - Expected improvement: [Current]/10 → [Target]/10 (+[X]%)

**Top Recommendations**:
  1. [Severity] [Issue] - [Fix] (+[N] points [dimension])
  2. [Severity] [Issue] - [Fix] (+[N] points [dimension])
  3. [Severity] [Issue] - [Fix] (+[N] points [dimension])

**Quick Wins** (High Impact, Low Effort):
  - [Quick win 1]: [X] hours dev, +[N] points [dimension]
  - [Quick win 2]: [X] hours dev, +[N] points [dimension]
  - [Quick win 3]: [X] hours dev, +[N] points [dimension]

Next Action: [Proceed to clarify/plan OR Fix WCAG violations]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Output Format

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎨 UX Quality Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Feature: [FEATURE_NAME]
Spec: [path/to/spec.md]
Analyzed: [DATE]

## UX Quality Dashboard

- **Usability**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Efficiency**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Accessibility**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Error Prevention**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Delight**: [X]/10 [⚠️ if <7] - [Brief assessment]

**Overall UX Quality**: [X.X]/10

**Status**: [🟢 UX Ready | 🟡 UX Needs Improvement | 🔴 UX Issues]
- 🟢 UX Ready: Overall score ≥ 7, no WCAG violations
- 🟡 UX Needs Improvement: Overall score 4-6, some friction points
- 🔴 UX Issues: Overall score < 4 or WCAG violations present

**WCAG 2.1 AA Compliance**: [🟢 Compliant | 🟡 Minor Gaps | 🔴 Violations Present]

**Mobile Optimization**: [🟢 Optimized | 🟡 Partial | 🔴 Desktop-Only]

---

## UI Design Quality (When UI-SPEC.md Present)

[Include this section ONLY if UI-SPEC.md was detected]

- **Visual Clarity**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Component Consistency**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Visual Hierarchy**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Responsive Design**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Design System Adherence**: [X]/10 [⚠️ if <7] - [Brief assessment]

**Overall UI Quality**: [X.X]/10
**Overall Design Quality**: [X.X]/10 (average of UX + UI)

---

## Component System Audit (When UI-SPEC.md Present)

[Include this section ONLY if UI-SPEC.md was detected AND user didn't skip]

**Framework Detected**: [React/Angular/Vue] + [Component Library]

**Component Consistency**:
- ✅ **Used Correctly**: [List library components with counts] = [N] total
  - Example: Button (12), TextField (8), Card (5) = 25 components
- ❌ **Custom Reimplementation**: [Components duplicating library functionality]
  - Example: "BlueButton" duplicates MUI Button, "CustomTable" duplicates MUI Table
- ⚠️ **Inconsistent Usage**: [Mix of library + custom for same purpose]
  - Example: Mix of MUI DatePicker (3) and native `<input type="date">` (2)

**Design Token Adherence**:
- ✅ **Colors**: [X]% using design tokens ([framework theme references])
- ❌ **Colors**: [Y]% hardcoded ([hex codes]) - should use theme
- ✅ **Spacing**: [X]% using spacing scale ([framework spacing system])
- ⚠️ **Typography**: [X]% using type variants ([Y]% custom font sizes)

**Recommendations**:
1. [Replace custom component X with library component Y]
   [If MCP available: Include official docs link and validated props]
2. [Standardize on library component Z]
3. [Replace hardcoded values with design tokens]
4. [Additional recommendations...]

**Component Reuse Score**: [X]% ([Excellent ≥80% | Good 60-79% | Needs Improvement <60%])

---

## UX Recommendations (Prioritized by Impact)

### 🔴 CRITICAL: [Dimension] Issues (Score: [X]/10)

**Heuristic Violated**: [Nielsen Heuristic or WCAG Guideline]

**Problem**:
- [Specific issue from spec.md with section reference]
- [Specific issue from spec.md with section reference]
- [Specific issue from spec.md with section reference]

**Socratic Validation**:
- "What user goal is blocked or delayed?" → [Specific task/workflow affected by these issues]
- "How many users are affected?" → [All users, mobile users, accessibility users, specific cohort, % estimate]
- "What is the cost of this issue?" → [Time waste per task, abandonment rate, support costs, legal risk (WCAG)]
- "Why CRITICAL severity?" → [WCAG Level A/AA violation = legal risk, OR task-blocking = workflow failure]

**Recommended Fix**:
1. [Specific, actionable step]
2. [Specific, actionable step]
3. [Specific, actionable step]

**Example** (from similar feature):
[Concrete example or pattern reference]

**Impact**: +[N]% [dimension] score ([current]/10 → [target]/10)
**Priority Rationale**: [Frequency × Impact × User Segment Size]
**Effort Estimate**: [High (>5 days) | Medium (2-5 days) | Low (<2 days)]

---

### 🟠 MAJOR: [Dimension] Issues (Score: [X]/10)

[Same format as CRITICAL]

---

### 🟡 MEDIUM: [Dimension] Issues (Score: [X]/10)

[Same format as CRITICAL]

---

## Quick Wins (High Impact, Low Effort)

1. **[Fix Title]** ([N] hours dev, +[M] points [dimension])
   - Problem: [Brief description]
   - Fix: [One-sentence action]
   - Example: [Reference or pattern]

2. **[Fix Title]** ([N] hours dev, +[M] points [dimension])
   - Problem: [Brief description]
   - Fix: [One-sentence action]
   - Example: [Reference or pattern]

3. **[Fix Title]** ([N] hours dev, +[M] points [dimension])
   - Problem: [Brief description]
   - Fix: [One-sentence action]
   - Example: [Reference or pattern]

---

## Summary

**Total Improvements**: [N] critical, [M] major, [P] medium
**Expected Score Gain**: [Current]/10 → [Target]/10 (+[X]%)
**Development Effort**: ~[N] hours
**User Time Saved**: ~[X]% per task (from efficiency improvements)

---

## Recommendations

**Immediate Actions** (Before Planning):
1. [Action 1 with screen/section reference]
2. [Action 2 with screen/section reference]
3. [Action 3 with screen/section reference]

**Suggested Next Step**:

[IF Overall UX Quality ≥ 7 AND no WCAG violations]:
  ✅ UX quality acceptable. Proceed to `/speckit.clarify` or `/speckit.plan`

[IF Overall UX Quality 4-6 OR minor WCAG gaps]:
  ⚠️ Run `/speckit.clarify --ux` for comprehensive UX review

[IF Overall UX Quality < 4 OR WCAG Level A/AA violations]:
  ❌ Revise spec to address critical UX issues before proceeding
  → WCAG violations create legal compliance risk (Section 508, ADA)
  → Task-blocking issues will cause user abandonment and support costs

**Optional Deep Dive**:
- Run `/speckit.clarify --ux` for multi-perspective UX expert review
- Conduct user testing on high-risk flows before implementation
- Review Nielsen's heuristics: https://www.nngroup.com/articles/ten-usability-heuristics/
- Review WCAG guidelines: https://www.w3.org/WAI/WCAG21/quickref/

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

9. **Update spec-metadata.json** (NEW v2.3)

   Save UX quality scores to `specs/[FEATURE]/spec-metadata.json`:

   ```json
   {
     "metadata": {
       "ux_quality": [calculated overall UX score 0-10],
       "ux_dimensions": {
         "usability": [score],
         "efficiency": [score],
         "accessibility": [score],
         "error_prevention": [score],
         "delight": [score]
       },
       "ui_design_quality": [calculated UI design score 0-10],
       "ui_design_dimensions": {
         "visual_clarity": [score],
         "component_consistency": [score],
         "visual_hierarchy": [score],
         "responsive_design": [score],
         "design_system_adherence": [score]
       },
       "component_audit": {
         "framework": "[detected framework]",
         "component_consistency_score": [score],
         "reuse_score": [score],
         "design_token_adherence": [score]
       },
       "critical_ux_issues": [count],
       "major_ux_issues": [count],
       "wcag_compliance": "[AA|A|None]",
       "last_ux_analysis_date": "[ISO 8601 timestamp]",
       "last_analysis_command": "analyze-ux"
     }
   }
   ```

   **UX Quality Trend Tracking**:
   - IF previous ux_quality exists:
     → Calculate delta: new_score - previous_score
     → Report: "UX quality improved by +X.X points" OR "UX quality regressed by -X.X points"
   - ELSE:
     → Report: "Baseline UX quality established: X.X/10"

---

## Notes

**Differences from Full UX Review**:
- ✅ **Fast**: No interactive questions, immediate results (30 seconds)
- ✅ **Lightweight**: Focuses on top 5 issues, not exhaustive
- ✅ **Non-blocking**: Provides guidance without requiring fixes
- ✅ **Quick Wins**: Highlights high-impact, low-effort improvements
- ❌ **Not comprehensive**: Use UX Lens Mode for thorough review
- ❌ **No clarification**: Identifies issues but doesn't resolve them

**When to Use Full UX Review**:
- Complex/high-stakes features → `/speckit.clarify --ux`
- Need multi-perspective analysis → Full UX expert review
- Pre-implementation quality gate → Comprehensive UX validation

**Analysis Frequency**:
- After major UI spec changes
- Before transitioning to planning
- As a quick health check during development
- When UX quality scores drop below threshold
- Before accessibility compliance audits

**WCAG Compliance Notes**:
- Level A: Minimum accessibility (critical)
- Level AA: Recommended standard (target for most projects)
- Level AAA: Enhanced accessibility (nice to have, not required)

**Mobile-First Considerations**:
- If 60%+ users are mobile, optimize for mobile FIRST
- Touch targets ≥ 44px (iOS) / 48px (Android)
- Primary actions in thumb zones (bottom third of screen)
- Use mobile-optimized keyboards (tel, email, number, url)

---

**Command Version**: 2.3.0
**Last Updated**: 2025-11-05
**Compatibility**: SpecKit v2.3+
**New in v2.3**: spec-metadata.json integration, context-aware UX analysis (risk level, previous UX quality), UX trend tracking, component audit metadata
