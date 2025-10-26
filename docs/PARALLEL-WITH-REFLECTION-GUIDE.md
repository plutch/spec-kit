# Parallel-with-Reflection Pattern Guide

**Purpose**: Refactor spec-kit commands for 2-3x faster execution and better quality control.

**Pattern**: Wave → Checkpoint → Wave

**Source**: Adapted from [pm-agent.md](../../SuperClaude_Framework/superclaude/agents/pm-agent.md)

---

## Pattern Overview

Traditional sequential execution:

```yaml
OLD (Sequential):
  1. Read file A (wait 500ms)
  2. Read file B (wait 500ms)
  3. Read file C (wait 500ms)
  4. Process all
  5. Write file D (wait 300ms)
  Total: ~2000ms
```

Parallel-with-Reflection execution:

```yaml
NEW (Parallel-with-Reflection):
  Wave 1 - PARALLEL Read (concurrent):
    - Read file A │
    - Read file B ├─ Execute simultaneously
    - Read file C │
    Total: ~500ms (fastest completes)

  Checkpoint (200 tokens, ~100ms):
    ❓ All files loaded?
    ❓ Data consistent?
    ❓ Ready to proceed?

    IF confidence < 70%: STOP
    ELSE: Continue

  Wave 2 - Process + Write:
    - Process data
    - Write file D
    Total: ~500ms

  Total: ~1100ms (2x faster)
```

**Key Insight**: Most time is spent waiting for I/O. Parallelize I/O operations and add confidence checkpoints to prevent wrong-direction work.

---

## Core Components

### 1. Wave Pattern

**Definition**: Group of operations executed in parallel.

**Rules**:
- All operations in a wave must be independent (no dependencies)
- Tools support multiple calls in one message
- Operations can be: Read, Bash, Grep, Glob

**Example**:

```yaml
Wave 1 (Information Gathering):
  PARALLEL:
    - Read: constitution.md
    - Read: spec-template.md
    - Bash: git rev-parse --abbrev-ref HEAD
    - Bash: git status --porcelain
    - Glob: specs/**/spec.md
```

### 2. Checkpoint Pattern

**Definition**: Brief (200-token) self-assessment to validate loaded data.

**Structure**:

```yaml
Checkpoint Questions (3-5 questions):
  ❓ "Critical data loaded?"
     → Check: Required files exist and parsed
     → Evidence: File contents or error messages

  ❓ "Data consistent?"
     → Check: No contradictions
     → Evidence: Cross-reference key values

  ❓ "Ready to proceed?"
     → Calculate: Confidence 0-100%
     → Target: ≥70% to continue

Decision Logic:
  IF any_critical_issues OR confidence < 70%:
    → STOP immediately
    → Report specific issues
    → Request user clarification
    → DO NOT guess or proceed

  ELSE (confidence ≥ 70%):
    → High confidence
    → Proceed to next wave or processing
```

**Anti-Pattern** (don't do this):

```yaml
❌ BAD Checkpoint:
  "Files loaded. Processing..."  # No validation!
  "Looks good, continuing..."    # No confidence assessment!
  "Assuming X is Y..."            # Guessing instead of stopping!
```

**Good Checkpoint**:

```yaml
✅ GOOD Checkpoint:
  ❓ "constitution.md loaded?"
     → YES: 5 articles parsed
  ❓ "spec-template.md loaded?"
     → YES: 12 sections found
  ❓ "Current branch valid?"
     → YES: feature/000001-auth
  ❓ "Data conflicts?"
     → NO conflicts detected

  Confidence: 95% (all checks passed)
  → Proceeding to spec generation
```

### 3. Evidence Requirement

**Rule**: Every checkpoint answer needs evidence.

**Good Evidence**:
- File size: "4.2KB loaded"
- Line count: "145 lines parsed"
- Key data: "Branch: feature/000001-auth"
- Specific values: "5 articles found in constitution"

**Bad Evidence**:
- "File seems okay" (vague)
- "Probably loaded" (uncertain)
- "Should be fine" (assumption)

---

## Refactoring Cookbook

### Recipe 1: Basic Command Refactor

**Before**:

```markdown
1. Run script {SCRIPT}
2. Read spec-template.md
3. Read constitution.md
4. Generate spec content
5. Write spec.md
```

**After**:

```markdown
### Wave 1 - PARALLEL Context Load

Execute these operations in parallel:

1. Bash: {SCRIPT} --json "{ARGS}"
2. Read: .specify/templates/spec-template.md
3. Read: .specify/memory/constitution.md

### Checkpoint - Validation (200 tokens max)

❓ "Script executed successfully?"
   → Check: JSON output parsed
   → Evidence: BRANCH_NAME and SPEC_FILE present

❓ "Templates loaded?"
   → Check: spec-template has required sections
   → Evidence: Section count = 12

❓ "Constitution loaded?"
   → Check: Articles parsed
   → Evidence: Article count = 9

❓ "Ready to generate spec?"
   → Confidence: [Calculate 0-100%]

IF confidence < 70%:
  → STOP and report issues
ELSE:
  → Proceed to spec generation

### Wave 2 - Spec Generation

4. Generate spec content using loaded templates
5. Write: specs/[FEATURE]/spec.md
```

### Recipe 2: Multi-File Read Refactor

**Before** (Sequential):

```markdown
1. Read spec.md
2. Read plan.md
3. Read data-model.md
4. Read constitution.md
5. Process all
```

**After** (Parallel):

```markdown
### Wave 1 - PARALLEL File Load

PARALLEL Read:
  - specs/[FEATURE]/spec.md
  - specs/[FEATURE]/plan.md
  - specs/[FEATURE]/data-model.md
  - .specify/memory/constitution.md

### Checkpoint - Completeness (200 tokens)

❓ "All files loaded?"
   → spec.md: ✅ (145 lines)
   → plan.md: ✅ (89 lines)
   → data-model.md: ❌ (not found)
   → constitution.md: ✅ (234 lines)

❓ "Missing files critical?"
   → data-model.md: Optional for this phase
   → Continue: YES

❓ "File contents valid?"
   → spec.md has requirements: ✅
   → plan.md has implementation: ✅

Confidence: 85%
→ Proceed

### Processing

Process loaded files...
```

### Recipe 3: State Update Pattern

**Before**:

```markdown
7. Update state after command completes
   - Read state.json
   - Modify phase and progress
   - Write state.json back
```

**After**:

```markdown
### Wave 3 - State Update (Final)

PARALLEL Operations:
  1. Read: .specify/memory/features/[FEATURE]/state.json
  2. Prepare: Updated state data

### Checkpoint - State Consistency

❓ "Current state loaded?"
   → Evidence: phase = "PLANNING", progress = 45

❓ "Update valid?"
   → New phase: "TASKING"
   → New progress: 70
   → Gates passed: +1 (plan_complete)

Confidence: 100%
→ Write updated state

### Write Operation

Write: state.json with updated values
```

---

## Command-Specific Patterns

### /speckit.specify Refactor

```yaml
Wave 1 - Context Load:
  PARALLEL:
    - Bash: create-new-feature.sh (creates branch)
    - Read: spec-template.md
    - Read: constitution.md
    - Bash: git rev-parse --abbrev-ref HEAD

Checkpoint:
  ❓ Branch created?
  ❓ Templates loaded?
  ❓ Ready to generate?

Wave 2 - Spec Generation:
  - Generate spec content
  - Write: spec.md

Wave 3 - State Init:
  PARALLEL:
    - Write: state.json
    - Write: next_actions.md
```

### /speckit.clarify Refactor

```yaml
Wave 1 - Load Spec:
  PARALLEL:
    - Read: spec.md
    - Read: state.json
    - Grep: "\[NEEDS CLARIFICATION\]" in spec.md

Checkpoint:
  ❓ Spec loaded?
  ❓ State valid?
  ❓ Clarifications found? (count = X)

Interactive Clarification Loop:
  (Sequential, can't parallelize user interaction)

Wave 2 - Update:
  PARALLEL:
    - Write: spec.md (updated)
    - Write: state.json (updated phase)
    - Write: next_actions.md
```

### /speckit.plan Refactor

```yaml
Wave 1 - Load Context:
  PARALLEL:
    - Read: spec.md
    - Read: constitution.md
    - Read: state.json
    - Grep: "\[NEEDS CLARIFICATION\]" in spec.md

Checkpoint:
  ❓ All files loaded?
  ❓ Clarifications resolved? (count = 0)
  ❓ Constitution articles accessible?

Wave 2 - Plan Generation:
  - Generate plan.md
  - Check constitutional gates
  - Write: plan.md

Wave 3 - State Update:
  PARALLEL:
    - Write: state.json
    - Write: next_actions.md
```

### /speckit.tasks Refactor

```yaml
Wave 1 - Load Plan:
  PARALLEL:
    - Read: plan.md
    - Read: data-model.md (if exists)
    - Glob: contracts/*.json
    - Read: state.json

Checkpoint:
  ❓ Plan loaded and valid?
  ❓ Dependencies identified?
  ❓ Ready to generate tasks?

Wave 2 - Task Generation:
  - Parse plan sections
  - Generate task list
  - Write: tasks.md

Wave 3 - State Update:
  PARALLEL:
    - Write: state.json
    - Write: next_actions.md
```

---

## Implementation Checklist

Refactoring a command:

- [ ] **Identify I/O Operations**: List all Read, Write, Bash, Grep, Glob
- [ ] **Group Into Waves**: Cluster independent operations
- [ ] **Design Checkpoint**: Write 3-5 validation questions with evidence
- [ ] **Set Confidence Threshold**: Usually 70%, adjust based on criticality
- [ ] **Add Error Handling**: What if checkpoint fails?
- [ ] **Test Parallel Execution**: Verify operations are truly independent
- [ ] **Measure Performance**: Compare old vs new execution time
- [ ] **Document Evidence**: What specific checks validate success?

---

## Performance Gains

**Expected Improvements**:

| Command | Before | After | Speedup |
|---------|--------|-------|---------|
| specify | ~3000ms | ~1200ms | 2.5x |
| clarify | ~2500ms | ~1000ms | 2.5x |
| plan | ~4000ms | ~1500ms | 2.7x |
| tasks | ~2000ms | ~800ms | 2.5x |

**Why It Works**:
- I/O operations are slowest (500-1000ms each)
- Parallelizing I/O collapses wait time
- Checkpoints are fast (100-200ms)
- Prevents expensive wrong-direction work (5K-50K tokens wasted)

**ROI Calculation**:

```
Traditional:
  Wrong direction: 50% chance
  Wasted tokens: 10,000 tokens average
  Cost: High (retry entire flow)

Parallel-with-Reflection:
  Checkpoint cost: 200 tokens
  Early stop: Prevents 10,000 token waste
  ROI: 50x token savings when catching errors early
```

---

## Anti-Patterns to Avoid

### ❌ Parallelizing Dependent Operations

```yaml
BAD:
  PARALLEL:
    - Read: spec.md
    - Process: spec content (depends on read!)
    - Write: output.md (depends on process!)

GOOD:
  Wave 1 - PARALLEL Read:
    - Read: spec.md

  Checkpoint: Validate

  Wave 2 - Sequential Processing:
    - Process: spec content
    - Write: output.md
```

### ❌ Skipping Checkpoints

```yaml
BAD:
  Wave 1: Load files
  Wave 2: Process immediately (no validation!)

GOOD:
  Wave 1: Load files
  Checkpoint: Validate (200 tokens)
  Wave 2: Process with confidence
```

### ❌ Verbose Checkpoints

```yaml
BAD (1500 tokens):
  "I loaded the constitution which contains the project
   guidelines. It has 9 articles covering library-first principles,
   CLI interface mandates, test-first imperatives..."

GOOD (200 tokens):
  ❓ Constitution loaded?
     → YES: 9 articles, 234 lines
  ❓ Required articles present?
     → YES: Articles I, II, III, VII, VIII, IX
  Confidence: 95%
```

### ❌ Proceeding Without Confidence

```yaml
BAD:
  Checkpoint:
    ❓ File loaded? → Error: file not found
    Confidence: 30%
    → Continuing anyway... (WRONG!)

GOOD:
  Checkpoint:
    ❓ File loaded? → Error: file not found
    Confidence: 30%
    → STOPPING - reporting error to user
```

---

## Testing Refactored Commands

### Validation Tests

1. **Parallel Execution Test**:
   - Verify operations execute concurrently
   - Check total time < sequential time
   - Confirm results identical to sequential

2. **Checkpoint Accuracy Test**:
   - Inject errors (missing files, corrupted data)
   - Verify checkpoint catches them
   - Confirm execution stops at <70% confidence

3. **Performance Benchmark**:
   - Measure execution time (old vs new)
   - Target: 2-3x speedup
   - Profile I/O wait times

4. **Edge Case Handling**:
   - Missing files
   - Corrupted JSON
   - Git errors
   - Permission issues

---

## Rollout Strategy

### Phase 1: Pilot (Week 1)
- Refactor `/speckit.specify`
- Measure performance
- Gather feedback
- Refine checkpoint questions

### Phase 2: Core Commands (Week 2)
- Refactor `/speckit.plan`
- Refactor `/speckit.tasks`
- Compare metrics

### Phase 3: Remaining Commands (Week 3)
- Refactor `/speckit.clarify`
- Refactor `/speckit.implement`
- Final benchmarks

### Phase 4: Documentation (Week 4)
- Update command docs
- Add examples
- Create troubleshooting guide

---

## Success Metrics

- ✅ All commands use Parallel-with-Reflection
- ✅ 2-3x performance improvement measured
- ✅ <10% confidence failure rate (checkpoint stops execution)
- ✅ 100% accuracy (results identical to sequential)
- ✅ Token consumption reduced by checkpoints preventing wrong work

---

**Last Updated**: 2025-01-26
**Version**: 2.0.0
**Implementation Status**: Guide Complete - Ready for Command Refactoring
