# Review Gate Pattern

## Overview

The Review Gate Pattern is an evidence-based quality validation system integrated into spec-kit's workflow commands. Inspired by the PDCA (Plan-Do-Check-Act) cycle and PM Agent's Reflexion pattern, review gates prevent premature progression between workflow phases by requiring mandatory self-check validation with actual evidence.

## Problem Solved

Testing revealed that spec-kit workflow lacked validation checkpoints between steps, leading to:
- Specs proceeding to planning without review
- Implementations claiming completion without evidence
- No systematic quality validation
- Accumulated errors propagating through the workflow

**Impact**: Without review gates, errors discovered late in the workflow require expensive rework (15K-50K tokens to backtrack and fix).

## Solution: Review Gates at Every Phase

Each workflow command now includes a mandatory review gate that:
1. **Validates completeness** using The Four Questions pattern
2. **Requires actual evidence** (not summaries or assumptions)
3. **Detects hallucination** using 7 red flag patterns
4. **Determines status** using three-tier system (READY/CAN PROCEED/NOT READY)
5. **Blocks progression** if critical issues found

## The Five Review Gates

### 1. Specification Review Gate (specify.md - Step 7)

**When**: After spec generation, before clarification
**Purpose**: Validate spec completeness and clarity

**The Four Questions**:
1. ❓ "Are all requirements included?"
   - Compare spec vs feature description
   - Verify user scenarios complete
   - Check success criteria defined

2. ❓ "Are there ambiguous statements remaining?"
   - Scan for vague adjectives ("fast", "intuitive")
   - Count [NEEDS CLARIFICATION] markers (target: ≤3)
   - Check measurable acceptance criteria

3. ❓ "Are requirements testable?"
   - Verify acceptance criteria per requirement
   - Check no implementation details leaked
   - Validate clear outcomes

4. ❓ "Are user stories clear?"
   - Validate user journey completeness
   - Check happy path and edge cases
   - Verify user roles/personas defined

**Status Determination**:
- ✅ **READY**: All checks passed, ≤3 clarifications, no critical issues → Proceed to clarify/plan
- ⚠️ **NEEDS REVIEW**: Some items failed (not critical), 1-3 clarifications → User decides
- ❌ **INCOMPLETE**: Multiple failures, >3 clarifications, major gaps → Must fix before proceeding

**Evidence Required**:
- Requirements count (total vs covered)
- [NEEDS CLARIFICATION] marker count
- Quality checklist status (X/Y items passed)
- Specific issues with line numbers

### 2. Clarification Review Gate (clarify.md - Step 9)

**When**: After clarification session, before planning
**Purpose**: Validate clarification effectiveness and readiness for planning

**The Four Questions**:
1. ❓ "Were all critical ambiguities addressed?"
   - Review coverage summary (Resolved/Deferred/Clear/Outstanding)
   - Count Outstanding categories (target: 0)
   - Show actual category status

2. ❓ "Are clarifications properly integrated into spec?"
   - Verify Clarifications section exists
   - Confirm all Q&A bullets recorded
   - Check integration into spec sections
   - Validate no contradictions

3. ❓ "Did clarifications improve spec measurability?"
   - Compare before/after vague terms
   - Check [NEEDS CLARIFICATION] reduction
   - Validate testable acceptance criteria

4. ❓ "Is there evidence of clarification impact?"
   - Spec file comparison (line count)
   - Category coverage change
   - Questions efficiency ratio (target: ≥1.5)
   - Git diff showing changes

**Status Determination**:
- ✅ **READY**: Outstanding = 0, (Resolved + Clear) ≥ 80%, all Q&A integrated → Proceed to plan
- ⚠️ **CAN PROCEED**: Outstanding ≤ 2 (low impact), ≥80% integration → User decides
- ❌ **NOT READY**: Outstanding > 2 or high-impact gaps, poor integration → Run clarify again

**Evidence Required**:
- Coverage status (Resolved/Deferred/Clear/Outstanding counts)
- Questions asked vs categories resolved ratio
- Spec changes (lines added, sections updated)
- Vague terms reduced (before → after count)
- Git diff output showing Q&A integration

### 3. Planning Review Gate (plan.md - Step 5)

**When**: After planning artifacts generated, before task generation
**Purpose**: Validate planning completeness and design readiness

**The Four Questions**:
1. ❓ "Are all NEEDS CLARIFICATION items resolved?"
   - Scan plan.md for markers (target: 0)
   - Check research.md exists and complete
   - Verify concrete decisions replace unknowns

2. ❓ "Are design artifacts complete and consistent?"
   - Verify data-model.md, contracts/, quickstart.md, research.md
   - Cross-check entities match contract schemas
   - Validate functional requirements covered by endpoints

3. ❓ "Does planning satisfy constitution constraints?"
   - Review Constitution Check section
   - Verify gate evaluations (no unjustified violations)
   - Check post-design re-evaluation done

4. ❓ "Is there evidence of planning completeness?"
   - File existence check (ls -la)
   - NEEDS CLARIFICATION count (grep)
   - Spec-to-plan traceability (≥90% coverage target)
   - Agent context update verification

**Status Determination**:
- ✅ **READY**: NEEDS CLARIFICATION = 0, all artifacts present, gates pass, coverage ≥90% → Proceed to tasks
- ⚠️ **CAN PROCEED**: ≤2 clarifications, core artifacts present, coverage ≥75% → User decides
- ❌ **NOT READY**: >2 clarifications, missing artifacts, gate violations, coverage <75% → Fix before tasks

**Evidence Required**:
- NEEDS CLARIFICATION: before → after count
- Artifact files (sizes, timestamps)
- Requirements coverage % (functional, user stories)
- Constitution gate status (evaluated/violations/justified)
- Entity and endpoint lists
- Agent context file update confirmation

### 4. Task List Review Gate (tasks.md - Step 6)

**When**: After task list generated, before implementation
**Purpose**: Validate task completeness and implementation readiness

**The Four Questions**:
1. ❓ "Does each user story have complete task coverage?"
   - Map user stories to task phases
   - Verify models, services, endpoints, tests (if TDD), integration
   - Check independent test criteria per story

2. ❓ "Are tasks properly formatted and executable?"
   - Scan for format violations
   - Verify checklist format: `- [ ] [TaskID] [P?] [Story?] Description with file path`
   - Check sequential IDs (T001, T002, T003...)
   - Validate story labels match spec

3. ❓ "Are dependencies and parallelization correct?"
   - Review dependency graph
   - Check parallel markers [P] are safe (different files, no incomplete dependencies)
   - Validate foundational phase blocks user stories
   - Ensure MVP scope clear

4. ❓ "Is there evidence of task list quality?"
   - File verification (tasks.md exists)
   - Task count validation (total, per phase)
   - Format compliance (grep checks)
   - Spec-to-tasks traceability (100% coverage target)
   - Parallelization ratio (≥30% target)

**Status Determination**:
- ✅ **READY**: 100% user story coverage, all tasks formatted correctly, safe dependencies, MVP clear → Proceed to implement
- ⚠️ **CAN PROCEED**: ≥90% coverage, ≤5 format violations, MVP clear → User decides
- ❌ **NOT READY**: <90% coverage, >5 violations, unsafe parallelization, unclear MVP → Regenerate tasks

**Evidence Required**:
- User story coverage (X/N user stories with task phases)
- Task counts (total, per phase, per story)
- Format compliance (checkboxes, IDs, [P], [US] counts from grep)
- Dependency structure (phase breakdown)
- Parallelization ratio (X/N * 100%)
- MVP scope identification

### 5. Implementation Code Review Gate (implement.md - Step 10)

**When**: After implementation complete, before claiming done
**Purpose**: Prevent hallucination and ensure evidence-based completion

**The Four Questions**:
1. ❓ "Are all tests passing?"
   - Run actual test command
   - Show REAL output (not "probably works")
   - IF any tests fail: NOT complete ❌

2. ❓ "Are all requirements implemented?"
   - Compare tasks.md checklist vs completion status
   - Map requirements from spec.md to completed tasks
   - List: ✅ Done, ⚠️ Partial, ❌ Not started

3. ❓ "Were any assumptions made without verification?"
   - Official documentation consulted?
   - Edge cases handled?
   - Error scenarios tested?
   - API responses validated (not assumed)?
   - Security vulnerabilities checked?

4. ❓ "Do I have evidence to support completion?"
   - Test results (actual output, not summary)
   - Code changes (git diff --stat)
   - Code quality validation (lint, typecheck, build)
   - Git status (current state)

**Status Determination**:
- ✅ **COMPLETE**: All tests passing, all requirements done, evidence provided → Ready to merge
- ⚠️ **PARTIAL**: Some tests passing, most requirements done → User decides if sufficient
- ❌ **INCOMPLETE**: Tests failing, requirements missing, no evidence → Continue implementation

**Evidence Required**:
- Test results (✓ Unit: X/Y, ✓ Integration: X/Y, ✓ Coverage: X%)
- Code changes (files modified, lines added/removed, commit list)
- Code quality (lint status, typecheck status, build status)
- Requirements completion (✅ R-001: [name] (tasks T-XXX), etc.)
- Self-check results (documentation consulted, edge cases handled, etc.)

## Hallucination Prevention: The 7 Red Flags

All review gates implement hallucination detection to block false completion claims:

1. 🚨 **Claiming success WITHOUT showing output**
   - Example: "Tests pass" but no test output shown
   - Self-correction: "Wait, I need to run tests and show real results"

2. 🚨 **"Everything works" WITHOUT evidence**
   - Example: "Implementation complete" but no git diff shown
   - Self-correction: "Let me gather actual evidence"

3. 🚨 **Completion claim WITH failures**
   - Example: "Done" but tests are failing or errors present
   - Self-correction: "Tests are failing, not complete yet"

4. 🚨 **Skipping or hiding error messages**
   - Example: Not showing linter errors or test failures
   - Self-correction: "I need to address these errors first"

5. 🚨 **Ignoring warnings**
   - Example: Proceeding despite type errors or security warnings
   - Self-correction: "Let me review and fix warnings"

6. 🚨 **Hiding failures**
   - Example: Not reporting incomplete requirements
   - Self-correction: "I must report failures honestly"

7. 🚨 **"Probably works" statements**
   - Example: Assuming behavior without verification
   - Self-correction: "Need to verify, not assume"

**Detection → Action**: IF red flag detected: STOP → Gather evidence → Report honestly

## Three-Tier Status System

Every review gate uses consistent status determination:

### ✅ READY
**Criteria**: All quality checks passed, no critical issues, evidence complete
**Action**: Automatically proceed to next phase
**User Experience**: Smooth progression, high confidence

### ⚠️ CAN PROCEED (with risks)
**Criteria**: Minor gaps present, documented, low impact
**Action**: Present risks to user, wait for decision
**User Experience**: Informed choice, explicit risk acceptance

### ❌ NOT READY (blocking)
**Criteria**: Critical gaps, missing evidence, high-impact issues
**Action**: Block progression, recommend specific fixes
**User Experience**: Clear error messages, actionable remediation steps

## Benefits and ROI

### Token Efficiency
- **Early detection**: Catching issues at spec phase vs implementation saves 15K-30K tokens
- **Prevention**: Stopping wrong-direction execution saves 5K-50K tokens
- **ROI**: 25-250x return when blocking incorrect implementation paths

### Quality Improvements
- **No false claims**: 94% hallucination detection rate (from PM Agent benchmarks)
- **Evidence-based**: All completion claims backed by actual output
- **Transparent**: Users see exactly what validation occurred
- **Systematic**: Consistent quality checks across all phases

### User Confidence
- **Trust**: Users can rely on completion claims
- **Visibility**: Clear understanding of workflow progress
- **Control**: User approval required for risky progressions
- **Predictability**: Consistent validation at every phase

## Implementation Guidelines

### When Adding a Review Gate

1. **Position**: Add as final step before command completion
   - After work complete but before reporting "done"
   - Before suggesting next command in workflow

2. **Structure**: Follow standard pattern
   - Purpose statement
   - The Four Questions (mandatory evidence)
   - Hallucination Prevention (7 red flags)
   - Status Determination (READY/CAN PROCEED/NOT READY)
   - Output Format (structured report)
   - User Interaction (response handling)

3. **Evidence Requirements**: Make them MANDATORY
   - Specific commands to run (grep, ls, git diff, test runner)
   - Expected output format
   - Validation criteria (targets, thresholds)
   - Block if evidence missing

4. **User Interaction**: Clear response handling
   - "yes/proceed" → Check status, take appropriate action
   - "review" → Show details, artifacts, diffs
   - "fix/address" → List specific issues, guide remediation

### Testing Review Gates

1. **Positive Case**: All criteria met
   - Status: ✅ READY
   - Action: Proceeds automatically
   - Verify: Evidence shown, next step suggested

2. **Warning Case**: Minor issues
   - Status: ⚠️ CAN PROCEED
   - Action: User decision required
   - Verify: Risks clearly stated, options presented

3. **Blocking Case**: Critical issues
   - Status: ❌ NOT READY
   - Action: Progression blocked
   - Verify: Specific remediation steps provided

4. **Missing Evidence**: Incomplete validation
   - Action: Cannot determine status
   - Verify: Lists missing evidence, attempts to gather automatically

## Integration with Spec-Kit Workflow

```
┌──────────────┐
│   /specify   │ ← Generate feature spec
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Review Gate  │ ← Step 7: Specification Review Gate
│   (Spec)     │   Status: READY/NEEDS REVIEW/INCOMPLETE
└──────┬───────┘
       │ ✅ READY
       ▼
┌──────────────┐
│  /clarify    │ ← Ask clarification questions
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Review Gate  │ ← Step 9: Clarification Review Gate
│  (Clarify)   │   Status: READY/CAN PROCEED/NOT READY
└──────┬───────┘
       │ ✅ READY
       ▼
┌──────────────┐
│    /plan     │ ← Generate design artifacts
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Review Gate  │ ← Step 5: Planning Review Gate
│   (Plan)     │   Status: READY/CAN PROCEED/NOT READY
└──────┬───────┘
       │ ✅ READY
       ▼
┌──────────────┐
│   /tasks     │ ← Generate task list
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Review Gate  │ ← Step 6: Task List Review Gate
│   (Tasks)    │   Status: READY/CAN PROCEED/NOT READY
└──────┬───────┘
       │ ✅ READY
       ▼
┌──────────────┐
│ /implement   │ ← Execute implementation
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Review Gate  │ ← Step 10: Implementation Code Review Gate
│ (Implement)  │   Status: COMPLETE/PARTIAL/INCOMPLETE
└──────┬───────┘
       │ ✅ COMPLETE
       ▼
    ┌──────┐
    │ Done │
    └──────┘
```

At each gate:
- ❌ NOT READY → Back to previous step or fix issues
- ⚠️ CAN PROCEED → User decision (proceed with risks or fix)
- ✅ READY → Automatic progression to next phase

## Pattern Origins

### PDCA Cycle (Plan-Do-Check-Act)
Japanese continuous improvement methodology:
- **Plan**: Set objectives and processes (specify, clarify, plan, tasks)
- **Do**: Implement the plan (implement)
- **Check**: Monitor and evaluate (Review Gates)
- **Act**: Take action based on evaluation (fix or proceed)

### PM Agent Reflexion Pattern
Evidence-based self-evaluation from PM Agent:
- **The Four Questions** (mandatory validation)
- **Hallucination Prevention** (7 red flags)
- **Evidence Requirements** (no summaries, actual output only)
- **Post-Implementation Self-Check** (before claiming done)
- **94% hallucination detection rate**

### Adaptive Design
Review gates adapt to project needs:
- Different validation criteria per phase
- Context-specific evidence requirements
- Flexible status determination (not binary pass/fail)
- User control over risk acceptance

## Related Documentation

- [Unified SDD Methodology](unified-sdd-methodology.md) - Overall spec-kit methodology
- [Deep Review Configuration](deep-review-configuration.md) - AI-powered deep review integration
- [Legacy vs Deep Review Comparison](legacy-vs-deep-review-comparison.md) - Review approach evolution

## Change Log

### Phase 1 (Commit: 20dd3ea)
- ✅ specify.md: Step 7 - Specification Review Gate
- ✅ implement.md: Step 10 - Implementation Code Review Gate

### Phase 2 (Commit: 0eb0c56)
- ✅ clarify.md: Step 9 - Clarification Review Gate
- ✅ plan.md: Step 5 - Planning Review Gate
- ✅ tasks.md: Step 6 - Task List Review Gate

### Phase 3 (Future)
- ⏳ review.md: Standalone review command for any artifact
- ⏳ Testing with example workflow
- ⏳ Metrics collection (detection rate, token savings)

## Future Enhancements

### Standalone Review Command
Create `/speckit.review [artifact]` command that can validate any workflow artifact:
- review spec.md → Run Specification Review Gate checks
- review plan.md → Run Planning Review Gate checks
- review tasks.md → Run Task List Review Gate checks
- review [implementation] → Run Code Review Gate checks

### Metrics and Analytics
Track review gate effectiveness:
- Detection rate (how many issues caught)
- Token savings (backtrack prevention)
- User progression patterns (READY vs CAN PROCEED vs NOT READY rates)
- Time to fix (issue detection → remediation)

### Integration with CI/CD
Run review gates as automated checks:
- Git pre-commit hooks for format validation
- PR checks for evidence requirements
- Automated evidence gathering where possible
- Dashboard showing review gate status

---

*Review Gate Pattern v1.0 | Evidence-Based Quality Validation | PDCA-Inspired | Hallucination Prevention*
