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

1. **Load Feature Context**:

   Run `{SCRIPT}` from repo root to get:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - Optional: `IMPL_PLAN`, `TASKS`

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

5. **Identify Top UX Issues**: Scan spec for common problems, prioritized by severity

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

7. **Generate Analysis Report**: Create concise, actionable output

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

## Top UX Issues (By Impact)

### 1. [🔴 CRITICAL | 🟠 MAJOR | 🟡 MEDIUM | 🟢 LOW | ℹ️ INFO] [Issue Title]

**Heuristic Violated**: [Nielsen's heuristic #X or WCAG guideline Y.Z.Z]

**Location**: [Screen/flow reference - e.g., "Invoice Creation > Line Item Entry"]

**Issue**:
> [Quote relevant spec text if applicable]

[Description of the UX problem]

**User Scenario**:
```
User: [Persona - e.g., "Rosa, accountant at small retail shop"]
Goal: [What user is trying to accomplish]
Friction Point: [Specific UX issue]
Consequence: [Impact - e.g., "Abandons task, uses paper invoice"]
```

**Impact**:
- **User**: [How this affects end users - time, errors, frustration]
- **Technical**: [What could break or fail]
- **Business**: [Abandonment risk, support costs, compliance risk]

**Recommendation**: [Specific, actionable fix with UX pattern reference]

**Pattern**: [e.g., "Inline editing (Cooper)", "Forgiving formats (Wroblewski)", "Progressive disclosure (Tidwell)"]

---

### 2. [Severity] [Issue Title]

[Same format as #1]

---

[... Repeat for top 5 issues ...]

---

## Quick Wins (High Impact, Low Effort)

1. **[Quick win title]** - [What to change] → [Expected impact]
   - Effort: [Low/Very Low]
   - Impact: [Specific measurable improvement]

2. **[Quick win title]** - [What to change] → [Expected impact]
   - Effort: [Low/Very Low]
   - Impact: [Specific measurable improvement]

3. **[Quick win title]** - [What to change] → [Expected impact]
   - Effort: [Low/Very Low]
   - Impact: [Specific measurable improvement]

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

**Command Version**: 2.1.0
**Last Updated**: 2025-01-15
**Compatibility**: SpecKit v2.1+
