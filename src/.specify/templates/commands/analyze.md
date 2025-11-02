---
description: Fast quality analysis with actionable recommendations - lightweight spec review without interactive questions
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

**Purpose**: Perform a fast, non-interactive quality analysis of the feature specification to identify top issues and provide actionable recommendations without the full review process.

**When to Use**:
- Quick quality check before planning
- After spec updates to validate improvements
- When you need scores but not full clarification
- Before committing to implementation
- As a pre-check before Expert Lens Mode

**Output**: Quality scores + top 5 issues by priority

---

## Execution Steps

1. **Load Feature Context**:

   Run `{SCRIPT}` from repo root to get:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - Optional: `IMPL_PLAN`, `TASKS`

2. **Read Specification**: Load the complete spec.md file

3. **Calculate Quality Scores**: Assess across four dimensions (0-10 scale)

   **Clarity Score**: Language precision and understandability

   Evaluate:
   - Are requirements atomic (one testable behavior each)?
   - Is terminology consistent across the spec?
   - Are vague terms ("gracefully", "efficiently", "user-friendly") avoided?
   - Are dependencies and preconditions explicit?
   - Can requirements be interpreted multiple ways?

   Scoring:
   - 8-10: Crystal clear, unambiguous, consistent terminology
   - 5-7: Mostly clear with some ambiguous sections
   - 2-4: Significant ambiguity requiring clarification
   - 0-1: Unclear and confusing throughout

   **Completeness Score**: Coverage of scenarios, edge cases, and constraints

   Evaluate:
   - Are all user stories with P1-P3 priorities present?
   - Are edge cases and error conditions identified?
   - Are non-functional requirements specified?
   - Are all affected stakeholders identified?
   - Are Given/When/Then scenarios comprehensive?

   Scoring:
   - 8-10: All scenarios, edge cases, and constraints covered
   - 5-7: Core scenarios covered, missing some edge cases
   - 2-4: Significant gaps in scenario coverage
   - 0-1: Incomplete, major sections missing

   **Testability Score**: Measurability and validation capability

   Evaluate:
   - Do all FRs have measurable acceptance criteria?
   - Are Given/When/Then scenarios specific (not abstract)?
   - Can automated tests verify each requirement?
   - Are success criteria quantifiable?
   - Are scenarios independently testable?

   Scoring:
   - 8-10: Concrete acceptance criteria, specific examples, measurable outcomes
   - 5-7: Some acceptance criteria, lacking specificity
   - 2-4: Vague criteria, difficult to validate
   - 0-1: Untestable requirements

   **Consistency Score**: Internal coherence and contradiction detection

   Evaluate:
   - Do requirements contradict each other?
   - Are similar behaviors handled consistently?
   - Is the scope clearly bounded without overlap?
   - Are assumptions aligned across sections?
   - Are user stories priorities logical?

   Scoring:
   - 8-10: Fully consistent, no contradictions
   - 5-7: Mostly consistent with minor conflicts
   - 2-4: Several contradictions requiring resolution
   - 0-1: Major contradictions throughout

   **Overall Quality**: Average of the four scores

4. **Identify Top Issues**: Scan spec for common problems, prioritized by severity

   **Severity Classification**:
   - 🔴 CRITICAL: Security vulnerabilities, data loss risks, compliance gaps, payment integrity
   - 🟠 MAJOR: Missing error handling, significant UX issues, untestable requirements, state inconsistencies
   - 🟡 MEDIUM: Edge cases in non-critical flows, tech debt, documentation gaps
   - 🟢 LOW: Style improvements, optimizations, minor simplifications
   - ℹ️ INFO: Observations, suggestions, recommendations

   **Issue Categories to Check**:

   a. **Missing Specifications**:
      - No error handling for critical flows
      - Missing timeout values for external calls
      - Undefined behavior for edge cases (null, empty, max values)
      - No recovery mechanisms specified

   b. **Ambiguity**:
      - Vague terms without quantification
      - Multiple interpretations possible
      - Unclear stakeholder/scope
      - Conflicting requirements

   c. **Testability**:
      - Generic scenarios without specific values
      - Acceptance criteria not measurable
      - Requirements cannot be validated
      - Missing Given/When/Then structure

   d. **Inconsistency**:
      - Contradicting requirements
      - Inconsistent terminology
      - Overlapping scope boundaries
      - Misaligned assumptions

   e. **Production Readiness**:
      - No monitoring/observability requirements
      - Missing performance characteristics
      - Undefined failure modes
      - No operational procedures

   **Limit**: Flag top 5 issues maximum (by severity, then impact)

5. **Generate Analysis Report**: Create concise, actionable output

---

## Output Format

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Specification Quality Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Feature: [FEATURE_NAME]
Spec: [path/to/spec.md]
Analyzed: [DATE]

## Quality Dashboard

- **Clarity**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Completeness**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Testability**: [X]/10 [⚠️ if <7] - [Brief assessment]
- **Consistency**: [X]/10 [⚠️ if <7] - [Brief assessment]

**Overall Quality**: [X.X]/10

**Status**: [🟢 Ready for Clarify | 🟡 Needs Review | 🔴 Major Issues]
- 🟢 Ready: Overall score ≥ 7, no critical issues
- 🟡 Needs Review: Overall score 4-6, some issues
- 🔴 Major Issues: Overall score < 4, significant problems

---

## Top Issues (By Impact)

### 1. [🔴 CRITICAL | 🟠 MAJOR | 🟡 MEDIUM | 🟢 LOW | ℹ️ INFO] [Issue Title]

**Location**: [Section reference - e.g., FR-005, User Story 2]

**Issue**:
> [Quote relevant spec text if applicable]

[Description of the problem]

**Impact**:
- **User**: [How this affects end users]
- **Technical**: [What could break]
- **Business**: [Cost/compliance/reputation risk]

**Recommendation**: [Specific, actionable fix]

---

### 2. [Severity] [Issue Title]

[Same format as #1]

---

[... Repeat for top 5 issues ...]

---

## Recommendations

**Immediate Actions** (Before Planning):
1. [Action 1 with section reference]
2. [Action 2 with section reference]
3. [Action 3 with section reference]

**Suggested Next Step**:
[IF Overall Quality ≥ 7]:
  ✅ Proceed to `/speckit.clarify` or `/speckit.plan`

[IF Overall Quality 4-6]:
  ⚠️ Run `/speckit.clarify --expert` for comprehensive review

[IF Overall Quality < 4]:
  ❌ Revise spec to address critical issues before proceeding

**Optional Deep Dive**:
- Run `/speckit.clarify --expert` for multi-perspective expert review
- Run `/speckit.clarify --challenge` to stress-test assumptions

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Notes

**Differences from Full Review**:
- ✅ **Fast**: No interactive questions, immediate results
- ✅ **Lightweight**: Focuses on top 5 issues, not exhaustive
- ✅ **Non-blocking**: Provides guidance without requiring fixes
- ❌ **Not comprehensive**: Use Expert Lens Mode for thorough review
- ❌ **No clarification**: Identifies issues but doesn't resolve them

**When to Use Full Review**:
- Complex/high-stakes features → `/speckit.clarify --expert`
- Need adversarial testing → `/speckit.clarify --challenge`
- Pre-implementation quality gate → Full expert review

**Analysis Frequency**:
- After major spec changes
- Before transitioning to planning
- As a quick health check during development
- When quality scores drop below threshold

---

**Command Version**: 2.1.0
**Last Updated**: 2025-01-15
**Compatibility**: SpecKit v2.1+
