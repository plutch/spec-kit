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

## Context Loading (v2.7 - Context Optimization)

**Purpose**: Load relevant memory content for quality analysis, optimized for token efficiency.

**Current Phase**: `quality_analysis`

**Context Loading Strategy**:

a. **Read Configuration** (if exists):
   - Check for `.specify/config.yml` or `.specify/config.example.yml`
   - Extract `context_budget` for quality analysis (default: 20KB)
   - Extract `memory.strict_phase_loading` (default: true)
   - Extract `budget_enforcement.mode` (default: strict)

b. **Load Memory Files** (phase-aware):

   For each memory file in `.specify/memory/`:

   i. **Read YAML Frontmatter**:
      - Extract `inclusion_mode`, `context_level`, `load_phases`, `exclude_phases`
      - If metadata missing: Default to `inclusion_mode: always, context_level: strategic`

   ii. **Determine if file should be loaded**:
      ```yaml
      IF inclusion_mode = "always":
        → Load strategic sections only (quality standards, severity guidelines)

      ELSE IF inclusion_mode = "conditional":
        IF "quality_analysis" IN load_phases OR "specification" IN load_phases:
          → Load strategic sections (quality criteria, best practices)
        ELSE:
          → SKIP

      ELSE IF inclusion_mode = "manual":
        → SKIP
      ```

   iii. **Filter Sections**:
      - Load strategic sections (quality standards, severity definitions)
      - Skip tactical sections (code examples, procedures)

   iv. **Track Budget**:
      - If exceeds 20KB budget:
        - **strict mode**: Warn user, prioritize quality standards
        - **warn mode**: Continue, warn about budget
        - **adaptive mode**: Filter to fit budget

c. **Context Loading Report** (verbose mode):

   IF `verbose_context_loading: true`:
   ```markdown
   📊 Context Loading Report

   Phase: quality_analysis
   Budget: 20KB

   Loaded:
   - quality-standards.md (strategic: criteria, severity levels): 4.8KB
   - Skipped: implementation-patterns.md (tactical: not needed for analysis)

   Total: 4.8KB / 20KB (24% used)
   Status: ✅ Within budget
   ```

d. **Backward Compatibility**:
   - If `.specify/memory/` doesn't exist → Skip (no error)
   - If files lack metadata → Load all content (v2.6 behavior)
   - Never fail due to missing memory files

**Expected Token Savings**: ~40% (15-20KB → 9-12KB for quality analysis)

**Key Focus**: Quality analysis loads STRATEGIC content (quality standards, severity guidelines, best practices) to evaluate specifications, not tactical implementation details.

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
       - metadata.risk_level: HIGH/MEDIUM/LOW (affects analysis focus)
       - metadata.overall_quality: Previous quality score (track improvement)
       - approvals: Phase approval status
   ```

   **Context-Aware Analysis**:
   - IF risk_level == HIGH: Apply extra scrutiny to security and edge cases
   - IF overall_quality from previous analysis: Compare to detect improvement/regression
   - IF phase == "clarifying": Expect [NEEDS CLARIFICATION] markers to exist

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

   **For Each Issue**: Apply Socratic validation to verify significance:
   - "What user goal is blocked or delayed?" (specific workflow/task)
   - "How many users are affected?" (all, cohort, percentage estimate)
   - "What is the cost?" (time, revenue, support, compliance, data loss)
   - "Why this severity?" (justify by frequency × impact × criticality)

   **Socratic Filter**: If you cannot answer these 4 questions with specifics, the issue may be a false positive. Only include issues where all 4 questions have clear, evidence-based answers.

5. **Identify Quick Wins**: After flagging issues, identify 3-5 high-impact, low-effort improvements

   **Quick Win Criteria**:
   - **High Impact**: Measurably improves quality (reduces errors, prevents data loss, improves clarity)
   - **Low Effort**: Can be fixed in 1-2 days (simple additions, clarifications, or refinements)
   - **Actionable**: Specific change with clear location in spec
   - **Independent**: Can be implemented without blocking on other changes

   **Examples of Quick Wins**:
   - Adding missing error handling for common failure modes (FR-XXX)
   - Clarifying vague acceptance criteria with specific values
   - Adding Given/When/Then structure to abstract scenarios
   - Defining timeout values for external API calls
   - Adding edge case specifications (empty, null, max values)

   **Quick Wins ≠ Low Severity Issues**: A quick win can address a CRITICAL issue if the fix is simple. Prioritize by impact/effort ratio, not just severity.

6. **Generate Analysis Report**: Create concise, actionable output

7. **Analysis Review Gate (Evidence-Based Self-Check)**

**Purpose**: Validate analysis completeness before presenting results.

### Evidence Collection (Mandatory)

❓ **"Was spec.md fully analyzed?"**
Action Required:
  - Verify spec.md was read completely
  - Show ACTUAL file size, line count, sections analyzed
  - Report: File path, size (KB), line count, sections

Expected Evidence:
  ✓ File path: specs/[FEATURE]/spec.md
  ✓ File size: [SIZE]KB
  ✓ Line count: [N] lines
  ✓ Sections analyzed: [List main sections]

❓ **"Were quality scores calculated?"**
Action Required:
  - Verify all 4 dimension scores calculated (Clarity, Completeness, Testability, Consistency)
  - Show ACTUAL scores with justifications
  - Report: Score for each dimension + overall

Expected Evidence:
  ✓ Clarity score: [X]/10 - [Justification]
  ✓ Completeness score: [X]/10 - [Justification]
  ✓ Testability score: [X]/10 - [Justification]
  ✓ Consistency score: [X]/10 - [Justification]
  ✓ Overall quality: [X.X]/10 (average of 4 dimensions)

❓ **"Were issues categorized by severity?"**
Action Required:
  - Count issues by severity (🔴/🟠/🟡/🟢/ℹ️)
  - Show ACTUAL count for each severity
  - Report: Severity breakdown

Expected Evidence:
  ✓ 🔴 CRITICAL: [N] issues
  ✓ 🟠 MAJOR: [N] issues
  ✓ 🟡 MEDIUM: [N] issues
  ✓ 🟢 LOW: [N] issues
  ✓ ℹ️ INFO: [N] issues
  ✓ Total issues flagged: ≤5 (top issues only)

❓ **"Were recommendations actionable?"**
Action Required:
  - Verify top 3-5 recommendations have section references
  - Show ACTUAL recommendations with locations
  - Report: Recommendations with section/FR references

Expected Evidence:
  ✓ Recommendation 1: [Action] → [Section reference: FR-XXX or Section Y]
  ✓ Recommendation 2: [Action] → [Section reference]
  ✓ Recommendation 3: [Action] → [Section reference]
  ✓ All recommendations are specific and actionable

IF any evidence is MISSING:
  ❌ CANNOT report completion
  → Gather missing evidence first
  → Re-run this step with complete evidence

### Hallucination Prevention (7 Red Flags for Analysis)

```yaml
Detect and BLOCK these patterns:

🚨 "Analysis complete" WITHOUT showing what was analyzed
   → Self-correction: "Wait, I need to show file size and sections analyzed"

🚨 "Quality score X/10" WITHOUT showing calculation basis
   → Self-correction: "I need to justify each dimension score"

🚨 "Found N issues" WITHOUT showing severity breakdown
   → Self-correction: "Must categorize issues by severity (🔴/🟠/🟡/🟢/ℹ️)"

🚨 "Recommendations provided" WITHOUT specific section references
   → Self-correction: "Need to link recommendations to actual spec sections"

🚨 Claiming "no issues" for specs >100 lines
   → Self-correction: "Large specs always have improvement opportunities"

🚨 Generic feedback NOT tied to actual spec content
   → Self-correction: "Must quote actual spec text in issue descriptions"

🚨 "Ready for planning" WITHOUT critical issue resolution
   → Self-correction: "CRITICAL issues block workflow, must report them"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

✅ **READY for Planning**:
```yaml
Criteria (ALL must be met):
  - All sections of spec.md analyzed
  - All 4 quality scores calculated with justifications
  - No 🔴 CRITICAL issues found
  - Top 5 issues identified with severity and recommendations
  - Overall quality score ≥ 7/10

IF ALL criteria met:
  → Proceed with recommendation: /speckit.clarify or /speckit.plan
```

⚠️ **NEEDS REVIEW** (can proceed with awareness):
```yaml
Criteria:
  - Overall quality score 4-6/10
  - Minor issues present (🟠 MAJOR or 🟡 MEDIUM only)
  - Recommendations actionable but non-blocking

IF criteria met:
  → Present issues to user
  → Recommend: /speckit.clarify --expert for comprehensive review
```

❌ **NOT READY** (more work needed):
```yaml
Criteria (ANY triggers NOT READY):
  - 🔴 CRITICAL issues found (security, data loss, compliance gaps)
  - Overall quality score < 4/10
  - Missing required sections in spec (Functional Requirements, User Stories)
  - Analysis incomplete (spec not fully read)

IF NOT READY:
  → Present critical issues with evidence
  → Recommend: "Revise spec to address critical issues before proceeding"
  → STOP workflow progression
```

### Output Format (Present to User - ONLY if evidence provided)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Specification Quality Analysis Review
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**Spec Analyzed**: specs/[FEATURE]/spec.md ([SIZE]KB, [N] lines)

**Quality Scores**:
  - Clarity: [X]/10 - [Brief assessment]
  - Completeness: [X]/10 - [Brief assessment]
  - Testability: [X]/10 - [Brief assessment]
  - Consistency: [X]/10 - [Brief assessment]

**Overall Quality**: [X.X]/10

**Issues by Severity**:
  - 🔴 CRITICAL: [N]
  - 🟠 MAJOR: [N]
  - 🟡 MEDIUM: [N]
  - 🟢 LOW: [N]
  - ℹ️ INFO: [N]

**Top Recommendations**:
  1. [Action 1] → [Section: FR-XXX or Section Y]
  2. [Action 2] → [Section: FR-XXX or Section Y]
  3. [Action 3] → [Section: FR-XXX or Section Y]

Next Action: [Proceed to clarify/plan OR Fix critical issues]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

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

**Socratic Validation**:
- "What user goal is blocked or delayed?" → [Specific goal/workflow affected]
- "How many users are affected?" → [All users, power users, specific cohort, % estimate]
- "What is the cost of this issue?" → [Time waste, revenue loss, support cost, compliance risk, data loss]
- "Why this severity level?" → [Justification: frequency × impact, criticality to business]

**Impact**:
- **User**: [How this affects end users]
- **Technical**: [What could break]
- **Business**: [Cost/compliance/reputation risk]

**Recommendation**: [Specific, actionable fix]

**Priority Rationale**: [Frequency × Impact - why this ranks in top 5]
**Effort Estimate**: [High (>5 days) | Medium (2-5 days) | Low (<2 days)]

---

### 2. [Severity] [Issue Title]

[Same format as #1]

---

[... Repeat for top 5 issues ...]

---

## Quick Wins (High Impact, Low Effort)

**Purpose**: Identify 3-5 improvements that provide significant value with minimal implementation effort. These are psychological momentum-builders and practical starting points.

### 1. **[Quick Win Title]**
- **Current State**: [What's wrong/missing now?]
- **Simple Fix**: [What's the specific change?]
- **Impact**: [Quantified benefit - reduces X by Y%, prevents Z, improves W]
- **Effort**: [1-2 days] [Why it's quick]
- **Location**: [Section: FR-XXX or specific requirement]

### 2. **[Quick Win Title]**
- **Current State**: [...]
- **Simple Fix**: [...]
- **Impact**: [...]
- **Effort**: [1-2 days] [Why it's quick]
- **Location**: [...]

### 3. **[Quick Win Title]**
- **Current State**: [...]
- **Simple Fix**: [...]
- **Impact**: [...]
- **Effort**: [1-2 days] [Why it's quick]
- **Location**: [...]

[Include 4-5 quick wins total if identified]

**Quick Wins Rationale**: These improvements are prioritized by impact/effort ratio. Completing quick wins builds team momentum and demonstrates immediate value while working on larger issues.

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

7. **Update spec-metadata.json** (NEW v2.3)

   Save quality scores to `specs/[FEATURE]/spec-metadata.json`:

   ```json
   {
     "metadata": {
       "overall_quality": [calculated score 0-10],
       "quality_dimensions": {
         "clarity": [score],
         "completeness": [score],
         "testability": [score],
         "consistency": [score]
       },
       "critical_issues": [count],
       "major_issues": [count],
       "last_analysis_date": "[ISO 8601 timestamp]",
       "last_analysis_command": "analyze"
     }
   }
   ```

   **Quality Trend Tracking**:
   - IF previous overall_quality exists:
     → Calculate delta: new_score - previous_score
     → Report: "Quality improved by +X.X points" OR "Quality regressed by -X.X points"
   - ELSE:
     → Report: "Baseline quality established: X.X/10"

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

**Command Version**: 2.3.0
**Last Updated**: 2025-11-05
**Compatibility**: SpecKit v2.3+
**New in v2.3**: spec-metadata.json integration, context-aware analysis (risk level, previous quality), quality trend tracking
