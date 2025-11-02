# Spec-Kit Framework Improvements Implementation Plan

**Based on**: Workflow Coherence Meta-Review (2025-11-02)
**Priority**: Immediate Actions (Top 5 Quick Wins)
**Total Effort**: 11 hours (1-2 days)

---

## Implementation Summary

### Completed ✅
1. **Review Gate Template Created** - `/src/.specify/templates/review-gate-template.md`
2. **Workflow Reviewer Agent Created** - `/src/.claude/agents/workflow-reviewer.md`
3. **Meta-Review Completed** - `WORKFLOW-COHERENCE-META-REVIEW.md`

### In Progress 🔄
Implementing the Top 5 Priority Recommendations:

1. **CRITICAL-001: Standardize Review Gates** (4 hours)
2. **CRITICAL-003: Add Reconcile Review Gate** (1 hour)
3. **MAJOR-002: Consolidate Clarify Modes** (3 hours)
4. **MAJOR-001: Optimize PM Agent** (1 hour)
5. **CRITICAL-002: Fix PM Agent Confidence Check** (2 hours)

---

## Detailed Implementation Plan

### 1. FIX PM AGENT CONFIDENCE CHECK (CRITICAL-002)
**File**: `src/.claude/commands/speckit.pm.md`
**Effort**: 2 hours
**Priority**: 🔴 CRITICAL

#### Current Issue:
```yaml
# Lines 40-55 have ambiguous confidence calculation:
❓ "All context files loaded successfully?"
❓ "Git branch matches feature state?"
❓ "Feature state is consistent?"
❓ "Ready to provide actionable status?"
   → Confidence level: 0-100% (NO FORMULA)
   → Examples show 45%, 60%, 30% (inconsistent)
```

#### Changes Required:
1. **Define explicit confidence formula** (lines 40-69):
   ```yaml
   Confidence Calculation:
   - 4 checks total (each worth 25%)
   - Check 1 (Context files): 25%
   - Check 2 (Git branch match): 25%
   - Check 3 (State consistency): 25%
   - Check 4 (Actionable status): 25%

   Confidence = (Passed Checks / 4) * 100

   Examples:
   - All 4 pass: 100% → PROCEED
   - 3 pass: 75% → PROCEED
   - 2 pass: 50% → BLOCK (< 70% threshold)
   - 1 pass: 25% → BLOCK
   ```

2. **Remove fuzzy percentages** from examples (lines 211-253)

3. **Add binary pass/fail** per check with clear criteria

#### Expected Impact:
- **User**: Clear, predictable confidence scores
- **Technical**: Deterministic behavior
- **Business**: Reliable session restoration

---

### 2. ADD RECONCILE REVIEW GATE (CRITICAL-003)
**File**: `src/.claude/commands/speckit.reconcile.md`
**Effort**: 1 hour
**Priority**: 🔴 CRITICAL

#### Current Issue:
```markdown
# reconcile.md has 4 steps but NO review gate:
1. Parse Gap Report
2. Ask Clarifying Questions (≤5)
3. Surgical Edits
4. Generate Sync Impact Report ← NO VALIDATION HERE
```

#### Changes Required:
1. **Add Step 5: Reconciliation Review Gate** after Sync Impact Report

2. **Gate Content** (based on review-gate-template.md):
```markdown
## Step 5: Reconciliation Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate surgical edits and gap closure before marking reconciliation complete.

### MANDATORY: The Four Reconciliation Questions

#### ❓ Question 1: Were all surgical edits append-only?
**Action Required**:
- Review git diff for modified files
- Verify no deletions occurred (strikethrough is acceptable for versioning)
- Check [NEEDS CLARIFICATION] markers added, not removed

**Expected Evidence Format**:
```bash
Run: git diff --staged specs/[FEATURE_ID]/
Output: Show additions (+) and modifications
Verification: Count deletions (-) = 0 OR only strikethrough formatting
```

**Readiness Determination**:
```yaml
IF deletions found (not strikethrough):
  → ❌ NOT APPEND-ONLY
IF only additions/strikethrough:
  → ✅ APPEND-ONLY VERIFIED
```

#### ❓ Question 2: Are [NEEDS CLARIFICATION] markers ≤ 3?
**Action Required**:
- Count [NEEDS CLARIFICATION] markers in modified specs
- Verify constraint met (≤3 total per reconciliation cycle)

**Expected Evidence Format**:
```bash
Run: grep -c "\[NEEDS CLARIFICATION\]" specs/[FEATURE_ID]/*.md
Output: Count per file
Total: [X] markers (target: ≤3)
```

**Readiness Determination**:
```yaml
IF total_markers > 3:
  → ❌ TOO MANY CLARIFICATION MARKERS
IF total_markers ≤ 3:
  → ✅ MARKER CONSTRAINT MET
```

#### ❓ Question 3: Were integration tests added for critical gaps?
**Action Required**:
- Review Sync Impact Report for 🔴 CRITICAL gaps
- Verify integration tests added to tasks.md for wiring gaps
- Check test coverage for high-risk gap closures

**Expected Evidence Format**:
```yaml
Critical Gaps from Sync Impact Report:
- Gap 1 (🔴 CRITICAL): [Description]
  Integration Test: [Test ID added to tasks.md] ✅
- Gap 2 (🔴 CRITICAL): [Description]
  Integration Test: [Test ID added to tasks.md] ✅
- Gap 3 (🟠 MAJOR): [Description]
  Integration Test: [Optional] ⚠️
```

**Readiness Determination**:
```yaml
IF critical_gaps_without_tests > 0:
  → ❌ MISSING INTEGRATION TESTS
IF all_critical_gaps_have_tests:
  → ✅ TEST COVERAGE COMPLETE
```

#### ❓ Question 4: Are modified files syntactically valid?
**Required Evidence (ALL must be provided)**:
1. **File Existence Check (MANDATORY)**:
   ```bash
   Run: ls -la specs/[FEATURE_ID]/
   Output: Show modified files with timestamps
   ```

2. **Content Validation (MANDATORY)**:
   ```bash
   Run: grep -c "^---$" specs/[FEATURE_ID]/spec.md
   Output: 2 (YAML frontmatter delimiters)

   Run: grep -c "^# " specs/[FEATURE_ID]/spec.md
   Output: >0 (markdown headers present)
   ```

3. **Traceability (MANDATORY)**:
   ```yaml
   Modified Files:
   - spec.md: [X] surgical edits
   - UI-SPEC.md: [Y] surgical edits (if applicable)
   - plan.md: [Z] requirement additions
   - tasks.md: [W] new tasks appended

   Coverage: 100% of gaps from Gap Report addressed
   ```

**IF any evidence is MISSING**:
```
❌ CANNOT report reconciliation complete
→ Gather missing evidence first
→ Re-run this step with complete evidence
```

### Hallucination Prevention (7 Red Flags for Reconciliation)

```yaml
Detect and BLOCK these patterns:

🚨 "Reconciliation complete" WITHOUT git diff evidence
   → Self-correction: "Wait, I need to show surgical edits made"

🚨 "All gaps closed" WITHOUT showing which specs were modified
   → Self-correction: "Let me verify files actually changed"

🚨 "Append-only edits" WITH deletions in git diff
   → Self-correction: "Found deletions - not append-only, need to fix"

🚨 Claiming "gaps addressed" WITHOUT mapping to Gap Report
   → Self-correction: "Need to map each gap to specific edit"

🚨 Skipping [NEEDS CLARIFICATION] count
   → Self-correction: "I need to verify marker constraint (≤3)"

🚨 Hiding integration test gaps
   → Self-correction: "Must report missing tests for critical gaps"

🚨 "Ready for COMPLETED" statements WITHOUT validation
   → Self-correction: "Need to verify surgical edit integrity first"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

#### ✅ **READY for VALIDATING or COMPLETED**

**Criteria (ALL must be met)**:
- [ ] All surgical edits are append-only (strikethrough OK for versioning)
- [ ] [NEEDS CLARIFICATION] markers ≤ 3
- [ ] Integration tests added for all 🔴 CRITICAL gaps
- [ ] Modified files are syntactically valid (markdown, YAML frontmatter)

**IF ALL criteria met**:
→ Proceed to next step (present reconciliation completion report)

#### ⚠️ **CAN PROCEED** (with risks noted)

**Criteria**:
- Some 🟠 MAJOR gaps lack integration tests (acceptable if documented)
- [NEEDS CLARIFICATION] count = 3 (at limit, but acceptable)
- Minor syntax warnings (not blocking)

**IF criteria met**:
→ Present risks to user
→ Ask: "Minor gaps remain. Mark reconciliation complete or address gaps?"
→ Wait for user decision before proceeding

#### ❌ **NOT READY** (more reconciliation work needed)

**Criteria (ANY triggers NOT READY)**:
- Surgical edits contain deletions (not append-only)
- [NEEDS CLARIFICATION] markers > 3 (constraint violated)
- 🔴 CRITICAL gaps lack integration tests (blocking)
- Modified files have syntax errors (invalid YAML, broken markdown)

**IF NOT READY**:
→ Present issues to user with evidence
→ Recommend specific actions:
  - Fix non-append-only edits
  - Reduce [NEEDS CLARIFICATION] markers
  - Add missing integration tests
  - Fix syntax errors
→ STOP (do not mark reconciliation complete)
```

#### Expected Impact:
- **User**: Confidence in gap closure quality
- **Technical**: Prevents corrupted specs
- **Business**: Reconciliation integrity validated

---

### 3. CONSOLIDATE CLARIFY MODES (MAJOR-002)
**File**: `src/.claude/commands/speckit.clarify.md`
**Effort**: 3 hours
**Priority**: 🟠 MAJOR

#### Current Issue:
```yaml
# 5 modes create decision paralysis:
1. Standard Mode (default): Iterative Q&A
2. Deep Review Mode (--deep): Expert lens
3. Technical Constraints (--technical): Architecture deep dive
4. Edge Case Detection (--edge-cases): Boundary/error/state
5. UX Quality Review (--ux): 4-pass UX lens
```

#### Consolidation Strategy:
```yaml
# REDUCE TO 2 MODES:

1. **Standard Mode** (default):
   - Iterative Q&A (current Standard Mode)
   - + Edge Case Detection (integrated, no longer optional)
   - Max 5 questions per iteration

2. **Expert Lens Mode** (--deep):
   - Multi-perspective review (Fowler, Cockburn, Architecture, UX)
   - Combines: Deep Review + Technical + UX modes
   - 4-pass expert analysis
```

#### Changes Required:

1. **Remove mode branches** (lines ~50-300):
   - Delete: `--technical` mode logic
   - Delete: `--edge-cases` mode logic (integrate into Standard)
   - Delete: `--ux` mode logic (merge into --deep)

2. **Update Standard Mode** (integrate edge cases):
   ```markdown
   ### Standard Mode (Default): Iterative Clarification with Edge Case Detection

   This mode combines iterative Q&A with automatic edge case detection.

   #### Step 1: Initial Scan
   - Parse spec.md for ambiguities
   - Scan for boundary conditions, error handling, state transitions
   - Identify missing edge case scenarios

   #### Step 2: Question Generation (max 5)
   - Prioritize critical ambiguities
   - Include edge case questions:
     * Boundary conditions (empty, null, max values)
     * Error conditions (network failures, timeouts)
     * State transitions (invalid state changes)
     * Data quality (malformed inputs)
     * Race conditions (concurrent operations)
     * Multi-tenant isolation (if applicable)

   #### Step 3: User Responses
   [Existing Standard Mode logic]
   ```

3. **Update Expert Lens Mode** (--deep):
   ```markdown
   ### Expert Lens Mode (--deep): Multi-Perspective Expert Review

   This mode applies world-class methodologies from multiple experts.

   #### Pass 1: Requirements Quality (Karl Wiegers Lens)
   [Existing Deep Review Mode content]

   #### Pass 2: Architecture & Design (Martin Fowler + Alistair Cockburn Lens)
   [Merge Technical Constraints Mode content here]
   - SOLID principles
   - Design patterns
   - Overengineering detection

   #### Pass 3: UX & Usability (Nielsen + Norman + Cooper Lens)
   [Merge UX Quality Review Mode content here]
   - Usability heuristics
   - Task efficiency
   - Accessibility (WCAG 2.1 AA)

   #### Pass 4: Production Resilience (Michael Nygard Lens)
   [Existing Deep Review Mode content]
   ```

4. **Update command description**:
   ```yaml
   description: "Clarify ambiguities and edge cases through Q&A (default) or multi-perspective expert review (--deep)"
   ```

5. **Update usage examples**:
   ```markdown
   ## Usage

   ### Standard Clarification (with edge cases)
   /speckit.clarify

   ### Expert Multi-Perspective Review
   /speckit.clarify --deep
   ```

#### Expected Impact:
- **Complexity Reduction**: 40% (5 modes → 2 modes, -200 lines)
- **User**: Clear decision: "Need clarification? → `/speckit.clarify`" / "Need expert review? → `/speckit.clarify --deep`"
- **Technical**: Simpler command logic, easier to maintain

---

### 4. OPTIMIZE PM AGENT TOKEN ECONOMY (MAJOR-001)
**File**: `src/.claude/commands/speckit.pm.md`
**Effort**: 1 hour
**Priority**: 🟠 MAJOR

#### Current Issue:
```yaml
# File is 300 lines (~2400 tokens):
- Verbose output format (lines 71-91): Explanatory text
- Low-confidence examples (lines 211-253): 42 lines rarely used
- PDCA section (lines 292-300): Not implemented
```

#### Changes Required:

1. **Trim file to 150 lines** (50% reduction):

   **REMOVE**:
   - Lines 80-91: Legend explanations (assume symbol knowledge)
   - Lines 211-253: Low-confidence examples (move to docs/)
   - Lines 292-300: PDCA section (not implemented)

2. **Simplify output format**:
   ```yaml
   # Current (verbose):
   ### Output - Minimal Status (When Confidence >= 70%)
   Format (Symbol-Based, Ultra-Compressed):
   🟢 feature/000001-auth | 📋 PLANNING | 60%
   🎯 Ready: /speckit.tasks

   # Legend:
   - Status emoji: 🟢 (active), 🟡 (blocked), 🔴 (error)
   ...8 lines of legend

   # Proposed (pure symbols):
   🟢 feature/000001-auth | 📋 60% | /speckit.tasks
   ```

3. **Create companion documentation**: `src/.specify/docs/PM-AGENT-GUIDE.md`
   - Move verbose legend explanations here
   - Move low-confidence examples here
   - Add troubleshooting guide

#### Expected Impact:
- **Complexity Reduction**: 50% (file size), 40% (token usage)
- **User**: Faster session restoration
- **Technical**: Reduced context consumption

---

### 5. STANDARDIZE REVIEW GATES (CRITICAL-001)
**Files**: Multiple command files
**Effort**: 4 hours (after other changes complete)
**Priority**: 🔴 CRITICAL

#### Changes Required:

1. **Add Specification Review Gate** to `speckit.specify.md`:
   - Insert after spec.md generation (Step 8)
   - Based on review-gate-template.md
   - Validates: Requirements coverage, ambiguity check, testability, user stories

2. **Refactor existing gates** in:
   - `speckit.plan.md` (Planning Review Gate) - already good, minor formatting
   - `speckit.tasks.md` (Task List Review Gate) - already good, minor formatting
   - `speckit.implement.md` (Implementation Code Review Gate) - move from Step 13 to Step 12

3. **Ensure consistency**:
   - 4-question format
   - Evidence requirements
   - 7 Red Flags hallucination prevention
   - 3-tier status (READY / CAN PROCEED / NOT READY)

#### Expected Impact:
- **Complexity Reduction**: 15% (consolidate 3 variations into 1 template)
- **User**: Consistent validation experience
- **Technical**: Reliable workflow progression

---

## Implementation Order

### Phase 1: Foundation (Hours 1-2)
1. ✅ Create review gate template
2. 🔄 Fix PM Agent confidence check (CRITICAL-002)
3. 🔄 Add Reconcile review gate (CRITICAL-003)

### Phase 2: Simplification (Hours 3-7)
4. 🔄 Consolidate Clarify modes (MAJOR-002)
5. 🔄 Optimize PM Agent token economy (MAJOR-001)
6. 🔄 Create PM-AGENT-GUIDE.md documentation

### Phase 3: Standardization (Hours 8-11)
7. 🔄 Add Specification Review Gate to specify command
8. 🔄 Standardize existing review gates (plan, tasks, implement)
9. 🔄 Update README.md and CLAUDE.md with improvements
10. 🔄 Run meta-review again to validate improvements

---

## Success Criteria

### Target Framework Score: 8.5/10 (from current 7.8/10)

**Expected Improvements**:
- **Workflow Coherence**: 6/10 → 9/10 (review gate consistency)
- **Developer Experience**: 7/10 → 8.5/10 (clarify simplification, PM Agent optimization)
- **Review Gate Consistency**: 6.2/10 → 9/10 (unified template, all commands covered)
- **Process Clarity**: 8/10 → 9/10 (mode consolidation, clearer workflows)

### Validation:
- Run workflow-reviewer agent again after changes
- Compare scores before/after
- Verify all CRITICAL issues resolved
- Test commands in target project

---

## Next Steps

Would you like me to proceed with implementing these changes in the order outlined above?

**Estimated Timeline**:
- Phase 1 (Foundation): ~2 hours
- Phase 2 (Simplification): ~5 hours
- Phase 3 (Standardization): ~4 hours

**Total**: 11 hours over 1-2 days

I can execute these changes systematically, starting with the highest-priority fixes (PM Agent confidence, Reconcile gate) and then moving to the larger refactors (Clarify consolidation, gate standardization).
