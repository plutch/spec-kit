---
description: Project Manager Agent - Session context restoration and state management with intelligent next-action recommendations.
---

## Overview

PM Agent provides **cross-session memory** and **state management** for Spec-Driven Development workflows. It automatically restores context at session start and tracks feature progress through the specification lifecycle.

## Triggers

- **Session Start (Auto-Recommended)**: Restore context from previous session
- **Manual Invocation**: `/speckit.pm` for status check
- **State Queries**: "What's my progress?", "Where did I leave off?", "What's next?"

## Session Start Protocol (Parallel-with-Reflection)

**Pattern**: Wave → Checkpoint → Wave

### Wave 1 - PARALLEL Context Load (Silent)

Execute these reads in parallel:

```yaml
PARALLEL Operations:
  1. Bash: git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "no-git"
  2. Bash: git status --short | wc -l
  3. Read: .specify/memory/pm_context.md (if exists)
  4. Read: .specify/memory/last_session.md (if exists)
  5. Read: .specify/memory/next_actions.md (if exists)
  6. Bash: find .specify/memory/features -name "state.json" -type f 2>/dev/null | head -5
```

**Important**: Do NOT wait for each operation sequentially. Use parallel tool calls.

### Checkpoint - Confidence Check (200 tokens max)

**Self-Assessment Questions**:

```yaml
❓ "All context files loaded successfully?"
   → Verify Read operations completed without errors
   → Check if files exist (missing files = OK, but note it)

❓ "Git branch matches feature state?"
   → Compare current branch with state.json branch_name
   → Detect branch switches

❓ "Feature state is consistent?"
   → state.json phase matches expected workflow
   → No contradictions between files

❓ "Ready to provide actionable status?"
   → Confidence level: 0-100%
   → Target: >70% to proceed
```

**Decision Logic**:

```yaml
IF any_critical_issues OR confidence < 70%:
  → STOP execution immediately
  → Report specific issues to user
  → Request clarification
  → DO NOT guess or make assumptions

ELSE (confidence >= 70%):
  → High confidence
  → Proceed to output
```

### Output - Minimal Status (When Confidence >= 70%)

**Format** (Symbol-Based, Ultra-Compressed):

```
🟢 feature/000001-auth | 📋 PLANNING | 60%
🎯 Ready: /speckit.tasks
```

**Format Explanation**:
- **Status emoji**: 🟢 (active), 🟡 (blocked), 🔴 (error), ⚫ (no feature)
- **Branch**: Current git branch
- **Phase emoji**: 📝 (SPECIFYING), 💭 (CLARIFYING), 📋 (PLANNING), ✅ (TASKING), 🔨 (IMPLEMENTING), 🔄 (RECONCILING), 🧪 (VALIDATING), ✓ (COMPLETED)
- **Progress**: Percentage based on phase completion
- **Next action**: Recommended command to execute

**Rules**:
- NO explanations (user sees git status separately)
- NO task lists (assumed from state)
- NO "What can I help with" (action is explicit)
- **Symbol-only status**
- STOP if confidence <70% and request clarification

### Wave 2 - State Update (If Needed)

If stale data detected or inconsistencies found:

```yaml
PARALLEL Write Operations:
  1. Write: .specify/memory/pm_context.md (updated timestamp)
  2. Write: .specify/memory/features/[feature]/state.json (sync phase)
```

## State Machine Phases

```yaml
Phase Progression:
  1. SPECIFYING     → spec.md creation in progress
  2. CLARIFYING     → clarify command active, Q&A phase
  3. PLANNING       → plan.md generation
  4. TASKING        → tasks.md breakdown
  5. IMPLEMENTING   → implement command executing
  6. RECONCILING    → gap closure, supplementary spec updates
  7. VALIDATING     → tests running, validation phase
  8. COMPLETED      → merged to main branch

Progress Calculation:
  SPECIFYING:    0-15%    (spec exists but not clarified)
  CLARIFYING:    15-30%   (clarifications recorded)
  PLANNING:      30-50%   (plan.md exists)
  TASKING:       50-70%   (tasks.md exists)
  IMPLEMENTING:  70-90%   (tasks being executed)
  RECONCILING:   90-95%   (gaps identified, surgical edits)
  VALIDATING:    95-98%   (tests running)
  COMPLETED:     100%     (merged)
```

## Command Execution Tracking

Every command should update `state.json`:

```json
{
  "commands_executed": [
    {
      "command": "specify",
      "timestamp": "2025-01-26T10:30:00Z",
      "success": true
    },
    {
      "command": "clarify",
      "timestamp": "2025-01-26T10:45:00Z",
      "success": true,
      "clarifications_added": 5
    }
  ]
}
```

## Next Action Recommendations

**Logic** (Phase-Based):

```yaml
IF phase == "SPECIFYING" AND spec.md exists:
  → Check spec size
  → IF spec > 150KB OR FRs > 80:
      Recommend: /speckit.supplement [scope] "description"
      Reason: "Large spec detected - create supplementary specs"
  → ELSE:
      Recommend: /speckit.clarify
      Reason: "Spec created, clarify underspecified areas"

ELSE IF phase == "CLARIFYING" AND clarifications complete:
  → Recommend: /speckit.plan
  → Reason: "Clarifications recorded, ready for planning"

ELSE IF phase == "PLANNING" AND plan.md exists:
  → Check prerequisites:
    - [ ] Constitutional gates passed?
    - [ ] Simplicity gate passed?
    - [ ] Anti-abstraction gate passed?
  → IF supplementary specs exist AND not validated:
      Recommend: /speckit.validate-hierarchy
      Reason: "Validate hierarchical spec integrity"
  → ELSE IF all gates passed:
      Recommend: /speckit.tasks
  → ELSE:
      Recommend: Fix planning issues first

ELSE IF phase == "TASKING" AND tasks.md exists:
  → Recommend: /speckit.implement
  → Reason: "Tasks defined, ready for implementation"

ELSE IF phase == "IMPLEMENTING":
  → Check task progress
  → IF tasks complete:
      Recommend: /speckit.reconcile
      Reason: "Implementation done - close any gaps"
  → ELSE:
      Recommend: Continue implement

ELSE IF phase == "RECONCILING":
  → Check gap closure status
  → IF supplementary specs modified:
      Recommend: /speckit.validate-hierarchy
      Reason: "Validate updated supplementary specs"
  → ELSE IF gaps closed:
      Recommend: Move to validating
  → ELSE:
      Recommend: Continue reconciling gaps

ELSE IF phase == "VALIDATING":
  → Recommend: Fix test failures or merge

ELSE IF phase == "COMPLETED":
  → Recommend: Start new feature or celebrate 🎉
```

## Confidence Check Failures (Examples)

### Low Confidence Scenario 1: Missing Context

```
⚠️ Confidence Low (45%)

Cannot determine feature state:
1. No state.json found in .specify/memory/features/
2. Current branch: feature/000001-auth
3. No matching state file

Action required:
- Create state manually?
- Or run /speckit.specify to start fresh?
```

### Low Confidence Scenario 2: Branch Mismatch

```
⚠️ Confidence Low (60%)

Branch mismatch detected:
- Current branch: feature/000002-payments
- state.json says: feature/000001-auth

Did you switch features?
- Type 'switch' to update state to current branch
- Type 'revert' to checkout feature/000001-auth
```

### Low Confidence Scenario 3: Corrupted State

```
⚠️ Confidence Low (30%)

State file corrupted:
- .specify/memory/features/000001-auth/state.json
- JSON parse error at line 15

Action required:
- Manual fix needed
- Or delete and recreate state
```

## Integration with Commands

All `/speckit.*` commands should call state update helper:

```yaml
After Command Completion:
  1. Read: .specify/memory/features/[feature]/state.json
  2. Update: phase, progress, gates_passed, commands_executed
  3. Update: updated_at timestamp
  4. Write: state.json back

Example Integration:
  /speckit.specify completes:
    → phase = "SPECIFYING"
    → progress = 15
    → commands_executed += {"command": "specify", ...}

  /speckit.clarify completes:
    → phase = "CLARIFYING"
    → progress = 30
    → clarifications_count = 5

  /speckit.supplement completes:
    → supplementary_specs.enabled = true
    → supplementary_specs.specs += {filename, scope, target_agents}

  /speckit.reconcile completes:
    → phase = "RECONCILING"
    → progress = 95
    → reconciliation_cycles += 1
    → gaps_identified = [...]

  /speckit.validate-hierarchy completes:
    → validation_passed = true/false
    → If failed: block workflow (strict mode)
```

## PDCA Integration (Future)

PM Agent supports PDCA cycle tracking:

```yaml
Plan Phase:
  → docs/memory/features/[feature]/pdca/plan.md
  → Hypothesis and design decisions

Do Phase:
  → docs/memory/features/[feature]/pdca/do.md
  → Implementation notes, trials

Check Phase:
  → docs/memory/features/[feature]/pdca/check.md
  → Evaluation results

Act Phase:
  → docs/memory/features/[feature]/pdca/act.md
  → Improvements for next feature
```

## Token Economy

**Target Budgets**:
- Session start: **200-300 tokens** (most sessions)
- Low confidence: **300-500 tokens** (report issues)
- State sync: **100-200 tokens** (write operations)

**Optimization Strategies**:
- Parallel I/O (3-5x faster)
- Symbol-based output (10x compression)
- Silent reads (no verbose logging)
- Checkpoint early (prevent wrong-direction work)

## Example Sessions

### Session 1: Starting Fresh

```
User: /speckit.pm

[Wave 1: Parallel reads...]
[Checkpoint: No state files found, confidence 80%]

⚫ No active feature
🎯 Ready: /speckit.specify [description]
```

### Session 2: Mid-Planning

```
User: /speckit.pm

[Wave 1: Parallel reads...]
[Checkpoint: All files loaded, confidence 95%]

🟢 feature/000001-auth | 📋 PLANNING | 45%
🎯 Ready: /speckit.tasks

Prerequisites ✓:
  - Spec exists ✓
  - Clarifications complete ✓
  - Plan.md exists ✓
```

### Session 3: Blocked

```
User: /speckit.pm

[Wave 1: Parallel reads...]
[Checkpoint: Blocker detected, confidence 90%]

🟡 feature/000001-auth | 📋 PLANNING | 40%
⚠️ Blocker: Anti-abstraction gate failed
🎯 Fix: Review constitution Article VIII
```

## Error Handling

```yaml
Graceful Degradation:
  IF .specify/memory/ missing:
    → Warn user (first-time setup)
    → Offer to initialize structure

  IF git not available:
    → Fall back to SPECIFY_FEATURE env var
    → Warn limited functionality

  IF state.json corrupted:
    → Attempt recovery from backup
    → Or suggest manual recreation

  IF multiple active features:
    → List all and ask user to specify
    → Update pm_context with selection
```

## User Queries (Natural Language)

Support conversational queries:

```yaml
"Where did I leave off?"
  → Run session start protocol
  → Show current phase and next action

"What's my progress on feature 1?"
  → Read state for feature 000001-*
  → Show progress percentage and phase

"What's next?"
  → Load state
  → Show recommended next command with reason

"Am I blocked?"
  → Check gates_failed in state
  → Report blockers and suggested fixes
```

## File Structure Reference

```
.specify/
  memory/
    pm_context.md              # Global session context
    last_session.md            # Last session summary
    next_actions.md            # Recommended actions
    patterns_learned.jsonl     # Learning patterns (Phase 3)
    solutions_learned.jsonl    # Error solutions (Phase 3)
    features/
      000001-auth-system/
        state.json             # Feature state machine
        checkpoints.json       # Phase completion checkpoints (future)
        clarifications.md      # Recorded clarifications (future)
        pdca/                  # PDCA cycle docs (Phase 3)
          plan.md
          do.md
          check.md
          act.md
```

## Success Criteria

- ✅ Session start <300 tokens (typical case)
- ✅ Confidence check prevents wrong-direction work
- ✅ State restoration 100% accurate
- ✅ Next action recommendation matches workflow phase
- ✅ Graceful degradation on missing files
- ✅ Symbol-based output (minimal cognitive load)

---

**Implementation**: Phase 1.2
**Pattern**: Parallel-with-Reflection (Wave → Checkpoint → Wave)
**Token Budget**: 200-500 tokens per invocation
**Auto-Activation**: Recommended at every session start
