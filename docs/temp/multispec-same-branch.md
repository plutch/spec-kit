# Multi-Spec Shared Branch Enhancement Plan for GitHub Spec Kit

**Document Version**: 1.0
**Created**: 2025-11-10
**Status**: PROPOSED
**Author**: Claude Code (Plan Mode)

---

## Executive Summary

**Objective**: Enable GitHub Spec Kit to handle multiple active specifications on a single branch while maintaining 100% backward compatibility with existing single-spec workflows.

**Strategy**: Incremental 4-phase rollout (Foundation → Commands → Advanced → Polish) with feature flagging and graceful degradation.

**Timeline**:
- 1 Developer (Sequential): 6-8 weeks
- 2 Developers (Parallel): 3-4 weeks
- **MVP Option**: 5-6 weeks (skip Phase 3 advanced features)

**Risk Level**: 🟢 **LOW** - Backward compatible design, incremental rollout with safety nets

---

## Current State Analysis

### Architecture Review

**Current Spec Kit Workflow** (v2.3):
- **Feature Detection**: Branch-based via `common.sh:get_current_branch()`
- **State Management**: Dual-state system
  - `.specify/memory/features/*/state.json` (v2.1 fallback)
  - `specs/*/spec-metadata.json` (v2.3 primary)
- **Commands**: 17 slash commands (`/speckit.*`)
- **Assumption**: Single active feature per branch

### Identified Gaps

1. **Multi-Spec Detection**: No logic to handle 2+ active specs on same branch
2. **Context Management**: No persistent "last used spec" tracking
3. **Batch Operations**: Cannot view status across multiple specs
4. **Dependency Tracking**: No cross-spec dependency awareness
5. **Conflict Detection**: No file overlap detection between specs

### Current Workaround

```bash
# Manual context switching (existing)
export SPECIFY_FEATURE=000004-company-switching
/speckit.next
```

**Problem**: Must set manually every command, no persistence across terminal sessions.

---

## PHASE 1: FOUNDATION (Week 1-2)

**Goal**: Enable multi-spec detection and basic context management with zero breaking changes.

### 1.1 Multi-Spec Detection & Mode Selection

#### Implementation

**File**: `.specify/scripts/bash/common.sh`

**New Function**: `detect_multi_spec_mode()`

```bash
detect_multi_spec_mode() {
    local repo_root="$1"
    local current_branch="$2"
    local specs_dir="$repo_root/specs"

    # Count active specs (those with state files OR metadata files)
    local active_specs=()

    for dir in "$specs_dir"/*; do
        if [[ -d "$dir" ]]; then
            local dirname=$(basename "$dir")
            local state_file="$repo_root/.specify/memory/features/$dirname/state.json"
            local metadata_file="$dir/spec-metadata.json"

            # Spec is "active" if has state OR metadata in planning+ phase
            if [[ -f "$state_file" ]] || [[ -f "$metadata_file" ]]; then
                local phase=$(get_spec_phase "$dir")
                # Filter out COMPLETED specs (don't count as "active")
                if [[ "$phase" != "COMPLETED" && "$phase" != "complete" ]]; then
                    active_specs+=("$dirname")
                fi
            fi
        fi
    done

    echo "${#active_specs[@]}"  # Return count

    # Store in env for commands to check
    export SPECKIT_MULTI_SPEC_COUNT="${#active_specs[@]}"
    export SPECKIT_ACTIVE_SPECS="${active_specs[*]}"
}

# NEW FUNCTION: Get phase from spec (unified v2.1/v2.3)
get_spec_phase() {
    local spec_dir="$1"
    local metadata_file="$spec_dir/spec-metadata.json"
    local dirname=$(basename "$spec_dir")
    local state_file="$repo_root/.specify/memory/features/$dirname/state.json"

    # Priority: spec-metadata.json (v2.3) > state.json (v2.1)
    if [[ -f "$metadata_file" ]]; then
        # Extract phase from metadata (lowercase)
        grep -oP '"phase":\s*"\K[^"]+' "$metadata_file" | head -1
    elif [[ -f "$state_file" ]]; then
        # Extract phase from state (uppercase)
        grep -oP '"phase":\s*"\K[^"]+' "$state_file" | head -1
    else
        echo "UNKNOWN"
    fi
}
```

#### Activation Logic

```yaml
IF SPECKIT_MULTI_SPEC_COUNT > 1:
  IF $SPECIFY_FEATURE set:
    → Use manual selection (existing workaround preserved)
  ELSE IF git branch matches single spec:
    → Use single-spec mode (backward compatibility)
  ELSE:
    → Activate multi-spec mode (interactive selection)
ELSE:
  → Single-spec mode (existing behavior - no changes)
```

#### Benefits

- ✅ Zero breaking changes (existing commands work unchanged)
- ✅ Env var workaround preserved (`$SPECIFY_FEATURE`)
- ✅ Auto-detection based on actual state, not assumptions
- ✅ Foundation for Phase 2 enhancements

---

### 1.2 Persistent Context Management

#### New File: `.specify/memory/multi-spec-context.json`

```json
{
  "version": "2.4.0",
  "current_spec": "000004-company-switching",
  "last_updated": "2025-11-10T12:34:56Z",
  "recent_specs": [
    "000004-company-switching",
    "000003-billing-console",
    "000002-devops-infrastructure"
  ],
  "session_context": {
    "terminal_id": "tty-pts-2",
    "git_branch": "004-company-switching",
    "active_since": "2025-11-10T10:00:00Z"
  }
}
```

#### Integration Points

**All Commands Load Context** (add to command initialization):

```bash
# At start of each command (after common.sh sourcing)
load_multi_spec_context() {
    local context_file="$REPO_ROOT/.specify/memory/multi-spec-context.json"

    if [[ -f "$context_file" ]] && [[ -z "$SPECIFY_FEATURE" ]]; then
        # Extract current_spec from JSON
        export SPECIFY_FEATURE=$(grep -oP '"current_spec":\s*"\K[^"]+' "$context_file")
    fi
}

# Call in every command
load_multi_spec_context
```

#### Graceful Degradation

- **Missing file**: Falls back to branch-based detection
- **Stale session**: Prompts for confirmation if > 24 hours old
- **Conflicting state**: User override prompt

---

### 1.3 Batch Status Command (First Multi-Spec Feature)

#### Enhancement: `/speckit.status --all`

**Modify**: `.claude/commands/speckit.status.md`

**New Behavior**:

```yaml
IF --all flag OR multi-spec mode detected:
  FOR EACH active spec:
    - Read state/metadata
    - Calculate progress
    - Show phase
    - Display risk level
    - Show quality score

  OUTPUT (Markdown Table):
    | Spec | Phase | Progress | Risk | Quality | Next Action |
    |------|-------|----------|------|---------|-------------|
    | 000004-company-switching | TASKS | 70% | 🔴 HIGH | 9.0/10 | /speckit.implement |
    | 000003-billing-console | RECONCILING | 95% | 🟠 MEDIUM | 8.2/10 | /speckit.reconcile |
    | 000002-devops-infrastructure | PLANNING | 40% | 🟢 LOW | 7.5/10 | /speckit.tasks |

    Current Context: 000004-company-switching

    Switch context: export SPECIFY_FEATURE=000003-billing-console
    Or use: /speckit.pm --select
ELSE:
  → Existing single-spec behavior (no changes)
```

**Example Output**:

```markdown
📊 Multi-Spec Status Report

┌─────────────────────────────────────────────────────────────────────────┐
│ Spec: 000004-company-switching                                          │
│ Phase: IMPLEMENTING (70%)                                               │
│ Risk: 🔴 HIGH | Quality: 9.0/10                                         │
│ Next: /speckit.implement                                                │
│ Updated: 2 hours ago                                                    │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│ Spec: 000003-billing-console                                            │
│ Phase: RECONCILING (95%)                                                │
│ Risk: 🟠 MEDIUM | Quality: 8.2/10                                       │
│ Next: /speckit.reconcile                                                │
│ Updated: 3 days ago                                                     │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│ Spec: 000002-devops-infrastructure                                      │
│ Phase: PLANNING (40%)                                                   │
│ Risk: 🟢 LOW | Quality: 7.5/10                                          │
│ Next: /speckit.tasks                                                    │
│ Updated: 1 week ago                                                     │
└─────────────────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 Current Context: 000004-company-switching
🎯 Aggregate: 3 specs | Avg Quality: 8.2/10 | Overall Progress: 68%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💡 Switch context: export SPECIFY_FEATURE=000003-billing-console
💡 Interactive selector: /speckit.pm --select
```

#### Benefits

- **Read-only** (minimal risk, no state changes)
- **Immediate value** (visibility across all active specs)
- **Validates detection logic** (proves Phase 1.1 works)
- **Introduces multi-spec UX patterns** (foundation for Phase 2)

---

### Phase 1 Deliverables

- [ ] Enhanced `common.sh` with multi-spec detection
- [ ] `multi-spec-context.json` schema and management
- [ ] `/speckit.status --all` batch command
- [ ] Integration tests (single-spec regression + multi-spec scenarios)
- [ ] Documentation update (`.specify/README.md`)

### Phase 1 Success Criteria

- ✅ Multi-spec detection works 100% of time (tested with 2, 5, 10 specs)
- ✅ Context persistence survives terminal restart
- ✅ `/speckit.status --all` shows accurate aggregate data
- ✅ Zero regressions in single-spec mode (all 17 commands tested)
- ✅ Performance < 100ms overhead per command

---

## PHASE 2: ENHANCED COMMANDS (Week 3-4)

**Goal**: Make all commands multi-spec aware with context management.

### 2.1 Context Selector for PM Agent

#### Enhancement: `/speckit.pm --select`

**Modify**: `.claude/commands/speckit.pm.md`

**Session Start Protocol (Multi-Spec Enhanced)**:

```yaml
Wave 1 - PARALLEL Context Load:
  1-7: Existing reads (pm_context, state files, git status)
  8: NEW: Read multi-spec-context.json
  9: NEW: Count active specs (detect_multi_spec_mode)

Checkpoint - Confidence Check (Enhanced):
  CHECK 5: "Multi-spec mode detected correctly?" (20%)
    PASS: Mode matches reality (1 spec = single, 2+ = multi)
    PASS: Context file valid (or gracefully degraded)
    FAIL: Mode detection broken → abort with diagnostics

IF multi-spec mode AND no valid context:
  → Launch interactive selector
```

**Interactive Selector UI**:

```markdown
🔍 Multiple active specs detected on branch '004-company-switching'

Select a spec to work on:

1. 🔴 000004-company-switching [HIGH RISK] [70%]
   Phase: IMPLEMENTING → Next: /speckit.implement
   Quality: 9.0/10 | Updated: 2 hours ago
   Dependencies: ✅ 000003-billing-console (satisfied)

2. 🟠 000003-billing-console [MEDIUM] [95%]
   Phase: RECONCILING → Next: /speckit.reconcile
   Quality: 8.2/10 | Updated: 3 days ago
   Dependencies: None

3. 🟢 000002-devops-infrastructure [LOW] [40%]
   Phase: PLANNING → Next: /speckit.tasks
   Quality: 7.5/10 | Updated: 1 week ago
   Dependencies: None

Enter selection (1-3), or type 'all' for batch status: █
```

**Selection Handling**:

```yaml
IF user selects number (1-3):
  → Update multi-spec-context.json with selection
  → Export SPECIFY_FEATURE=<selected-spec>
  → Continue PM session with selected spec
  → Show confirmation: "✅ Context set to 000004-company-switching"

IF user types 'all':
  → Run /speckit.status --all
  → Exit (don't start PM session)

IF user types 'quit' or Ctrl+C:
  → Exit gracefully
```

---

### 2.2 Spec-Scoped vs. Branch-Scoped Commands

#### Command Classification

| Command | Scope | Multi-Spec Behavior |
|---------|-------|---------------------|
| `/speckit.status` | **Branch** | Batch mode with `--all` (Phase 1.3 ✓) |
| `/speckit.pm` | **Branch** | Interactive selector (Phase 2.1) |
| `/speckit.next` | **Spec** | Uses context, operates on single spec |
| `/speckit.specify` | **Spec** | Context-aware, can create new spec |
| `/speckit.clarify` | **Spec** | Uses context, operates on single spec |
| `/speckit.analyze` | **Spec** | Uses context, operates on single spec |
| `/speckit.plan` | **Spec** | Uses context, operates on single spec |
| `/speckit.validate-gap` | **Spec** | Uses context, operates on single spec |
| `/speckit.tasks` | **Spec** | Uses context, operates on single spec |
| `/speckit.implement` | **Spec** | Uses context, operates on single spec |
| `/speckit.reconcile` | **Spec** | Uses context, operates on single spec |
| `/speckit.amend-technical` | **Spec** | Uses context, operates on single spec |
| `/speckit.constitution` | **Branch** | Global, not spec-specific |
| `/speckit.memory` | **Branch** | Global, not spec-specific |

#### Design Pattern: Spec-Scoped Commands

**Template for all spec-scoped commands**:

```bash
#!/bin/bash
set -euo pipefail

# Source common functions (includes multi-spec detection)
source "$(dirname "$0")/../../.specify/scripts/bash/common.sh"

# Load multi-spec context
load_multi_spec_context

# Detect multi-spec mode
REPO_ROOT=$(git rev-parse --show-toplevel)
CURRENT_BRANCH=$(git branch --show-current)
detect_multi_spec_mode "$REPO_ROOT" "$CURRENT_BRANCH"

# If multi-spec mode AND no context, prompt selector
if [[ "$SPECKIT_MULTI_SPEC_COUNT" -gt 1 ]] && [[ -z "$SPECIFY_FEATURE" ]]; then
    echo "🔍 Multiple specs detected. Please select:"
    run_spec_selector  # Interactive selector
    exit 0
fi

# Continue with command logic (operates on $SPECIFY_FEATURE)
# ... existing command implementation ...
```

#### Design Pattern: Branch-Scoped Commands

**Template for batch operations**:

```bash
#!/bin/bash
set -euo pipefail

source "$(dirname "$0")/../../.specify/scripts/bash/common.sh"

# Detect multi-spec mode
REPO_ROOT=$(git rev-parse --show-toplevel)
CURRENT_BRANCH=$(git branch --show-current)
detect_multi_spec_mode "$REPO_ROOT" "$CURRENT_BRANCH"

# If multi-spec mode OR --all flag, batch operation
if [[ "$SPECKIT_MULTI_SPEC_COUNT" -gt 1 ]] || [[ "$1" == "--all" ]]; then
    for spec in $SPECKIT_ACTIVE_SPECS; do
        echo "Processing spec: $spec"
        # ... operation for each spec ...
    done
else
    # Single-spec operation (backward compat)
    # ... existing command implementation ...
fi
```

---

### 2.3 Cross-Spec Dependency Tracking

#### New File: `.specify/memory/spec-dependencies.json`

```json
{
  "version": "1.0.0",
  "dependencies": [
    {
      "spec": "000004-company-switching",
      "depends_on": [
        {
          "spec": "000003-billing-console",
          "reason": "Requires multi-tenant auth infrastructure from billing console",
          "blocking": false,
          "status": "SATISFIED",
          "satisfied_at": "2025-11-01T14:30:00Z"
        }
      ]
    },
    {
      "spec": "000005-ui-spec",
      "depends_on": [
        {
          "spec": "000004-api-spec",
          "reason": "UI requires API endpoints to be implemented first",
          "blocking": true,
          "status": "PENDING",
          "required_phase": "IMPLEMENTING"
        }
      ]
    }
  ]
}
```

#### Dependency Schema

```typescript
interface SpecDependency {
  spec: string;                    // Dependent spec ID
  depends_on: Array<{
    spec: string;                  // Required spec ID
    reason: string;                // Human-readable explanation
    blocking: boolean;             // true = cannot proceed, false = warning only
    status: 'PENDING' | 'SATISFIED' | 'BROKEN';
    required_phase?: string;       // Min phase needed (e.g., "IMPLEMENTING")
    satisfied_at?: string;         // ISO timestamp when satisfied
  }>;
}
```

#### Integration: `/speckit.next` (Enhanced)

**Dependency Check Logic**:

```yaml
WHEN /speckit.next runs:
  1. Load current spec context
  2. Read spec-dependencies.json
  3. Find dependencies for current spec
  4. FOR EACH dependency:
     a. Check dependency spec phase
     b. IF blocking=true AND status=PENDING:
        → Show ERROR, block proceed
     c. IF blocking=false AND status=PENDING:
        → Show WARNING, allow proceed
     d. IF status=BROKEN (dependency regressed):
        → Show CRITICAL, suggest reconciliation
```

**Example Output** (with dependency):

```markdown
📍 Current Spec: 000004-company-switching
🔹 Phase: IMPLEMENTING (80%)

⚠️ Dependency Notice:

  Depends on: 000003-billing-console
  Reason: Requires multi-tenant auth infrastructure from billing console
  Status: ✅ SATISFIED (003 is in RECONCILING phase, ahead of 004)
  Satisfied: 2025-11-01 14:30:00

  💡 Tip: If 003 changes, you may need to run /speckit.reconcile on 004

🎯 Next Action: /speckit.implement (continue implementation)
```

**Example Output** (blocking dependency):

```markdown
📍 Current Spec: 000005-ui-spec
🔹 Phase: PLANNING (40%)

🛑 BLOCKING DEPENDENCY:

  Depends on: 000004-api-spec
  Reason: UI requires API endpoints to be implemented first
  Status: ❌ PENDING (004 is in PLANNING, needs to reach IMPLEMENTING)
  Required Phase: IMPLEMENTING

  Cannot proceed until 000004-api-spec reaches IMPLEMENTING phase.

  🎯 Recommendation:
    1. Switch context: export SPECIFY_FEATURE=000004-api-spec
    2. Complete API implementation first
    3. Then return to UI spec

Override: Use --force flag to proceed anyway (not recommended)
```

#### Manual Dependency Management

**New Command**: `/speckit.dependency add <spec> <depends-on> "<reason>" [--blocking]`

```bash
# Example: Add dependency
/speckit.dependency add 000005-ui-spec 000004-api-spec \
  "UI requires API endpoints" --blocking

# Example: Mark dependency satisfied
/speckit.dependency satisfy 000005-ui-spec 000004-api-spec

# Example: List all dependencies
/speckit.dependency list
```

---

### Phase 2 Deliverables

- [ ] Interactive spec selector in PM agent
- [ ] All 17 commands classified (spec-scoped vs. branch-scoped)
- [ ] Context loading integrated into all spec-scoped commands
- [ ] Batch operations for branch-scoped commands
- [ ] `spec-dependencies.json` schema and management
- [ ] Dependency checking in `/speckit.next`
- [ ] `/speckit.dependency` command suite
- [ ] Integration tests (context switching, dependency blocking)
- [ ] Documentation updates (command reference)

### Phase 2 Success Criteria

- ✅ PM agent selector < 500ms response time
- ✅ Context switches persist across commands
- ✅ Dependency tracking prevents spec conflicts
- ✅ All commands respect spec context (no ambiguity)
- ✅ Single-spec regression tests still pass (backward compat)

---

## PHASE 3: ADVANCED FEATURES (Week 5-6) - OPTIONAL

**Goal**: Advanced multi-spec capabilities for power users.

⚠️ **Note**: Phase 3 is **optional** - Assess necessity after Phase 2 user feedback. Can ship MVP without these features.

### 3.1 Parallel Spec Execution (Experimental)

#### Command: `/speckit.implement --parallel <spec1> <spec2>`

**Use Case**: Execute implementation of two non-conflicting specs simultaneously to save time.

#### Safety Constraints

```yaml
ALLOW parallel execution IF:
  ✅ Both specs in IMPLEMENTING phase
  ✅ Planning approved for both
  ✅ No file conflicts detected (parse tasks.md)
  ✅ Git working directory clean
  ✅ User explicitly opts in (--parallel flag)

BLOCK parallel execution IF:
  ❌ File conflicts detected
  ❌ Specs in different phases
  ❌ State file locks present
  ❌ Git working directory dirty
```

#### Pre-Flight Conflict Detection

```yaml
WHEN --parallel flag used:
  1. Parse tasks.md from both specs
  2. Extract "Files Modified" section from both plan.md
  3. Check for file path overlaps
  4. IF overlap found:
     → Show conflict report
     → Abort with error
     → Suggest resolution
  5. ELSE:
     → Proceed with parallel execution
```

**Conflict Detection Example**:

```markdown
🛑 Cannot execute in parallel - file conflicts detected:

Spec 1: 000004-company-switching
Spec 2: 000003-billing-console

Conflicting files (3):
  - workers/src/routes/auth.ts (both specs modify)
  - workers/src/lib/session.ts (both specs modify)
  - workers/src/middleware/auth.ts (both specs modify)

Resolution Options:
  1. Complete 000003 first (older spec, higher priority)
  2. Merge 000003, rebase 000004, retry parallel execution
  3. Run sequentially: /speckit.implement (one at a time)

Cannot proceed with --parallel flag.
```

#### Execution Strategy

```yaml
IF all safety checks pass:
  → Fork execution into background processes:

  Process 1 (Foreground):
    - Spec: <spec1>
    - Shell: Current terminal
    - Output: Real-time display

  Process 2 (Background):
    - Spec: <spec2>
    - Shell: Background process (nohup)
    - Output: Buffered to .specify/memory/parallel-execution-<spec2>.log

  → Monitor both executions:
    - Poll every 5 seconds
    - Check for state file updates
    - Aggregate quality gate results

  → Report combined status:
    - Show progress bars for both
    - Display any failures immediately
    - Wait for both to complete
    - Aggregate test results
```

**Example Output**:

```markdown
🚀 Parallel Execution Started

┌───────────────────────────────────────────────────────────────────┐
│ Spec 1: 000004-company-switching [FOREGROUND]                     │
│ Progress: ████████████░░░░░░░░ 60% (12/20 tasks)                 │
│ Status: Implementing task T-012 (Company context service)         │
│ Quality Gates: 3/3 passed                                         │
└───────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────┐
│ Spec 2: 000003-billing-console [BACKGROUND]                       │
│ Progress: ██████████████████░░ 80% (16/20 tasks)                 │
│ Status: Implementing task T-016 (Invoice generation)              │
│ Quality Gates: 3/3 passed                                         │
└───────────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Overall Progress: 70% | Estimated completion: 45 minutes
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Press 'Ctrl+C' to cancel both executions
Press 'l' to view spec 2 logs
```

#### Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| **Git conflicts** | Pre-flight check, abort if working dir dirty |
| **State file corruption** | Atomic writes with file locks |
| **Resource exhaustion** | Limit to 2 parallel specs max |
| **One fails, other succeeds** | Rollback strategy documented |
| **User confusion** | Clear UX indicators for both processes |

---

### 3.2 Visual Dashboard

#### Command: `/speckit.dashboard` (New)

**Purpose**: Real-time multi-spec progress visualization in terminal.

**ASCII Dashboard Layout**:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Spec Kit Dashboard - Branch: 004-company-switching
 Last Updated: 2025-11-10 14:30:15 | Auto-refresh: 5s | Press 'q' to quit
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────────────────────────────────────────────────────────────────────┐
│ 🔴 000004-company-switching [HIGH RISK]                             │
│ Phase: IMPLEMENTING (70%) ██████████████░░░░░░░                    │
│ Tasks: 14/20 complete | Quality: 9.0/10 | Critical Issues: 0       │
│ Next: /speckit.implement                                            │
│ Updated: 2 hours ago                                                │
│ Dependencies: ✅ 000003 (satisfied)                                 │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ 🟠 000003-billing-console [MEDIUM]                                  │
│ Phase: RECONCILING (95%) ███████████████████░                      │
│ Tasks: 19/20 complete | Quality: 8.2/10 | Critical Issues: 0       │
│ Next: /speckit.reconcile                                            │
│ Updated: 3 days ago                                                 │
│ Dependencies: None                                                   │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ 🟢 000002-devops-infrastructure [LOW]                               │
│ Phase: PLANNING (40%) ████████░░░░░░░░░░                           │
│ Tasks: Not generated yet | Quality: 7.5/10 | Critical Issues: 0    │
│ Next: /speckit.tasks                                                │
│ Updated: 1 week ago                                                 │
│ Dependencies: None                                                   │
└─────────────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 📊 Aggregate Metrics
 Active Specs: 3 | Avg Quality: 8.2/10 | Overall Progress: 68%
 Critical Issues: 0 | Blockers: 0 | Last Activity: 2 hours ago
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Commands:
 [1-3] Switch to spec | [s] Status all | [d] Dependencies | [q] Quit
 [r] Refresh now | [+/-] Adjust refresh rate

Press a key: █
```

**Features**:

- **Real-time updates**: Auto-refresh every 5 seconds (configurable)
- **Interactive**: Press number to switch context
- **Color-coded**: Risk levels (🔴 HIGH, 🟠 MEDIUM, 🟢 LOW)
- **Progress bars**: Visual task completion
- **Quick actions**: Single-key commands

**Implementation**:

```bash
#!/bin/bash
# .claude/commands/speckit.dashboard.md

# Main loop
while true; do
    clear
    render_dashboard

    # Wait for user input (5 second timeout for auto-refresh)
    read -t 5 -n 1 key

    case "$key" in
        1|2|3) switch_to_spec "$key" ;;
        s) run_status_all ;;
        d) show_dependencies ;;
        r) continue ;;  # Refresh now
        q) exit 0 ;;
        +) increase_refresh_rate ;;
        -) decrease_refresh_rate ;;
    esac
done
```

---

### 3.3 Conflict Detection & Resolution

#### Goal: Detect when multiple specs modify the same files

**Trigger Points**:

1. **During Planning** (`/speckit.plan`): Check "Files Modified" section
2. **Before Implementation** (`/speckit.implement`): Re-check task file paths
3. **On Demand** (`/speckit.conflicts`): Manual conflict check

#### Conflict Detection Algorithm

```yaml
WHEN /speckit.plan completes:
  1. Extract "Files Modified" section from plan.md
  2. Parse file paths (remove markdown formatting)
  3. Query all other active specs:
     - Read their plan.md files
     - Extract "Files Modified" sections
  4. Find intersections (file path overlaps)
  5. IF overlap detected:
     → Generate conflict report
     → Store in .specify/memory/conflicts.json
     → Block proceed (or warn with --force)
```

**Conflict Report Schema**:

```json
{
  "version": "1.0.0",
  "conflicts": [
    {
      "detected_at": "2025-11-10T14:30:00Z",
      "specs": ["000004-company-switching", "000003-billing-console"],
      "conflicting_files": [
        "workers/src/routes/auth.ts",
        "workers/src/lib/session.ts",
        "workers/src/middleware/auth.ts"
      ],
      "severity": "HIGH",
      "status": "PENDING",
      "resolution_strategy": "COMPLETE_OLDER_FIRST"
    }
  ]
}
```

**Example Conflict Report** (shown to user):

```markdown
⚠️ FILE CONFLICT DETECTED

Multiple specs will modify the same files:

┌─────────────────────────────────────────────────────────────────────┐
│ Conflicting Specs                                                   │
├─────────────────────────────────────────────────────────────────────┤
│ • 000004-company-switching (IMPLEMENTING - 70%)                     │
│ • 000003-billing-console (RECONCILING - 95%)                        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ Conflicting Files (3)                                               │
├─────────────────────────────────────────────────────────────────────┤
│ 1. workers/src/routes/auth.ts                                       │
│    - 000004: Add company switching routes                           │
│    - 000003: Add billing portal auth                                │
│                                                                      │
│ 2. workers/src/lib/session.ts                                       │
│    - 000004: Add company context to session                         │
│    - 000003: Add subscription data to session                       │
│                                                                      │
│ 3. workers/src/middleware/auth.ts                                   │
│    - 000004: Add multi-company auth check                           │
│    - 000003: Add subscription status check                          │
└─────────────────────────────────────────────────────────────────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RECOMMENDED RESOLUTION STRATEGY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Option 1: Sequential Completion (RECOMMENDED)
  1. Complete 000003-billing-console first (95% done, almost finished)
  2. Merge 000003 to main branch
  3. Rebase 000004-company-switching on updated main
  4. Re-run /speckit.plan on 000004 (may need reconciliation)
  5. Continue 000004 implementation

Option 2: Separate Branches (SAFEST)
  1. Move 000004 to new branch: feature/000004-company-switching
  2. Complete 000003 on current branch
  3. Merge 000003 to main
  4. Merge main into 000004 branch (resolve conflicts manually)
  5. Continue 000004 implementation

Option 3: Manual Merge (ADVANCED)
  1. Continue both specs on same branch
  2. Manually resolve git conflicts during implementation
  3. Requires careful coordination and git expertise
  4. Not recommended for complex conflicts

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🛑 Implementation blocked to prevent conflicts.

Override: Use --force flag to proceed anyway (manual merge required)
```

#### Command: `/speckit.conflicts` (New)

**Purpose**: Check for conflicts between active specs on demand.

```bash
# Check all specs
/speckit.conflicts

# Check specific spec against others
/speckit.conflicts --spec=000004-company-switching

# Show resolved conflicts
/speckit.conflicts --resolved

# Mark conflict as resolved (manual)
/speckit.conflicts --resolve <conflict-id>
```

---

### Phase 3 Deliverables

- [ ] Parallel execution safety checks
- [ ] `/speckit.implement --parallel` command
- [ ] Background process monitoring
- [ ] `/speckit.dashboard` visual interface
- [ ] Real-time auto-refresh dashboard
- [ ] Interactive keyboard controls
- [ ] Conflict detection algorithm
- [ ] `/speckit.conflicts` command suite
- [ ] Conflict resolution strategies
- [ ] Integration tests (parallel execution, conflict scenarios)
- [ ] Documentation (advanced features guide)

### Phase 3 Success Criteria

- ✅ Parallel execution 2x faster than sequential (for non-conflicting specs)
- ✅ Conflict detection catches 100% of file overlaps
- ✅ Dashboard renders < 100ms, auto-refresh smooth
- ✅ Zero data corruption from parallel execution
- ✅ User confidence in conflict resolution strategies

---

## PHASE 4: POLISH & TESTING (Week 7-8)

**Goal**: Documentation, testing, and production readiness.

### 4.1 Documentation

#### New Documentation Files

**1. User Guide: `docs/MULTI-SPEC-WORKFLOWS.md` (15-20 pages)**

**Contents**:
- Introduction to multi-spec mode
- When to use (vs. separate branches)
- Setting up multi-spec context
- Context switching strategies
- Best practices
- Troubleshooting common issues
- Example workflows (3-5 scenarios)
- Command reference (multi-spec flags)

**2. Technical Design: `docs/MULTI-SPEC-ARCHITECTURE.md` (10-15 pages)**

**Contents**:
- System architecture overview
- State management design
- Context persistence mechanism
- Detection algorithm details
- Command classification rationale
- Dependency tracking implementation
- Conflict detection algorithm
- Testing strategy
- Performance considerations
- Future enhancements

**3. Best Practices: `.specify/memory/multi-spec-patterns.md` (5-7 pages)**

**Contents**:
- Pattern: Related features on same branch
- Pattern: Independent features on separate branches
- Pattern: Feature with multiple supplementary specs
- Anti-pattern: Too many specs on one branch (> 5)
- Anti-pattern: Conflicting file modifications
- Case study: Company switching + billing console
- Case study: API + UI specs coordination

#### Documentation Updates

**Update**: `.specify/README.md`

**New Section**: "Multi-Spec Branch Support (v2.4+)"

```markdown
## Multi-Spec Branch Support (v2.4+)

Starting in v2.4, Spec Kit supports multiple active specifications on a single branch.

### Quick Start

#### Check if multi-spec mode is active:
```bash
/speckit.status --all
```

#### Switch between specs:
```bash
export SPECIFY_FEATURE=000004-company-switching
/speckit.next
```

#### Interactive selector:
```bash
/speckit.pm --select
```

### When to Use Multi-Spec

✅ **Use multi-spec when**:
- Features are tightly coupled (A depends on B)
- Features share infrastructure (same API layer)
- Deploying together (atomic release)

❌ **Avoid multi-spec when**:
- Features are independent (can deploy separately)
- File conflicts likely (both modify same files)
- > 5 specs on same branch (complexity too high)

### See Also

- [Multi-Spec Workflows Guide](../docs/MULTI-SPEC-WORKFLOWS.md)
- [Multi-Spec Architecture](../docs/MULTI-SPEC-ARCHITECTURE.md)
- [Best Practices](memory/multi-spec-patterns.md)
```

**Update**: All 17 command files in `.claude/commands/speckit.*.md`

**Add to each command**:

```markdown
## Multi-Spec Support

**Mode**: [Spec-Scoped | Branch-Scoped]

**Multi-Spec Behavior**:
[Describe how command behaves when multiple specs detected]

**Context Management**:
- Auto-loads context from `.specify/memory/multi-spec-context.json`
- Falls back to `$SPECIFY_FEATURE` environment variable
- [Additional context notes if applicable]

**Examples**:

```bash
# Single-spec mode (existing behavior)
/speckit.[command]

# Multi-spec mode (explicit selection)
export SPECIFY_FEATURE=000004-company-switching
/speckit.[command]

# Multi-spec mode (interactive)
/speckit.pm --select
/speckit.[command]
```
```

---

### 4.2 Migration Guide

#### Document: `docs/MULTI-SPEC-MIGRATION.md` (3-5 pages)

**Contents**:

```markdown
# Upgrading to Multi-Spec Support (Spec Kit v2.4)

## Overview

Spec Kit v2.4 introduces multi-spec branch support with **100% backward compatibility**.

## Automatic Migration

**No action required!**

Existing workflows continue unchanged. Multi-spec features activate only when:
1. You have 2+ active specs on the same branch
2. You use `--all` flags (e.g., `/speckit.status --all`)
3. You manually set multi-spec context

## What's New

### Phase 1 (Foundation)
- Multi-spec detection (automatic)
- Persistent context tracking
- Batch status command (`/speckit.status --all`)

### Phase 2 (Enhanced Commands)
- Interactive spec selector (`/speckit.pm --select`)
- Context-aware commands (all 17 commands)
- Cross-spec dependency tracking
- `/speckit.dependency` command suite

### Phase 3 (Advanced - Optional)
- Parallel execution (`/speckit.implement --parallel`)
- Visual dashboard (`/speckit.dashboard`)
- Conflict detection (`/speckit.conflicts`)

## Opt-In Configuration

### Enable Multi-Spec Context Tracking

Create `.specify/config.yml`:

```yaml
multi_spec:
  enabled: true               # Enable multi-spec features
  persistent_context: true    # Track last-used spec
  auto_detect: true           # Auto-activate when 2+ specs
  max_specs_per_branch: 5     # Safety limit
```

### Try New Features

```bash
# Batch status (all specs)
/speckit.status --all

# Interactive selector
/speckit.pm --select

# Add dependency
/speckit.dependency add 000005 000004 "UI depends on API" --blocking

# Visual dashboard (Phase 3)
/speckit.dashboard

# Check conflicts (Phase 3)
/speckit.conflicts
```

## Breaking Changes

**None** - All existing commands work unchanged.

## Rollback

If issues arise, disable multi-spec features:

`.specify/config.yml`:
```yaml
multi_spec:
  enabled: false
```

Or set environment variable:
```bash
export SPECKIT_DISABLE_MULTI_SPEC=1
```

## Troubleshooting

### Issue: Context not persisting

**Solution**: Ensure `.specify/memory/multi-spec-context.json` has write permissions

### Issue: Wrong spec selected automatically

**Solution**: Explicitly set context:
```bash
export SPECIFY_FEATURE=000004-company-switching
```

### Issue: Commands showing selector every time

**Solution**: Set persistent context via `/speckit.pm --select` once

## Support

For issues, see:
- [Multi-Spec Workflows Guide](MULTI-SPEC-WORKFLOWS.md)
- [GitHub Issues](https://github.com/your-org/spec-kit/issues)
```

---

### 4.3 Testing Strategy

#### Test Matrix

**Dimensions**:
- **Spec Count**: 1 (single), 2 (dual), 5 (multi)
- **Commands**: 17 spec kit commands
- **Modes**: Single-spec, Multi-spec, Context switching

**Total Test Cases**: 3 × 17 = **51 scenarios**

#### Priority 1: Regression Tests (Backward Compatibility)

**Objective**: Ensure single-spec mode unchanged.

| Command | Single-Spec Test | Expected Behavior |
|---------|------------------|-------------------|
| `/speckit.status` | Run on branch with 1 spec | Shows current spec status (existing) |
| `/speckit.pm` | Run with 1 spec | PM session starts (existing) |
| `/speckit.next` | Run with 1 spec | Shows next action (existing) |
| ... | (all 17 commands) | Existing behavior preserved |

**Pass Criteria**: 100% backward compatibility (0 regressions)

#### Priority 2: Multi-Spec Mode Tests

**Objective**: Validate multi-spec features.

| Scenario | Test | Expected Behavior |
|----------|------|-------------------|
| **Detection** | 2 specs on branch | `SPECKIT_MULTI_SPEC_COUNT=2` |
| **Context Load** | Read context file | `SPECIFY_FEATURE` set correctly |
| **Batch Status** | `/speckit.status --all` | Shows 2 specs in table |
| **Selector** | `/speckit.pm --select` | Interactive UI appears |
| **Context Switch** | Select spec 2 | Context updated, spec 2 active |
| **Dependency Check** | Spec A depends on Spec B | Warning shown if B not ready |
| **Conflict Detect** | 2 specs modify same file | Conflict report generated |

**Pass Criteria**: All multi-spec features work as designed

#### Priority 3: Edge Cases

**Objective**: Handle unusual scenarios gracefully.

| Edge Case | Test | Expected Behavior |
|-----------|------|-------------------|
| **Context file missing** | Delete context file | Falls back to branch detection |
| **Context file stale** | 24h old context | Prompts re-selection |
| **Context file corrupt** | Invalid JSON | Graceful error, fallback |
| **10+ specs on branch** | Create 10 specs | Still works (performance test) |
| **Circular dependency** | A → B → A | Detects cycle, warns user |
| **Parallel execution** | 2 specs, no conflicts | Both complete successfully |
| **Parallel conflict** | 2 specs, same files | Pre-flight check blocks |

**Pass Criteria**: No crashes, clear error messages

#### Integration Tests

**Test Workflow 1**: Create → Plan → Implement (Multi-Spec)

```yaml
1. Create 2 specs on same branch
2. Run /speckit.specify on spec 1
3. Run /speckit.specify on spec 2
4. Run /speckit.status --all (should show both)
5. Switch context to spec 1
6. Run /speckit.plan on spec 1
7. Switch context to spec 2
8. Run /speckit.plan on spec 2
9. Run /speckit.implement on spec 1
10. Verify context persistence

Expected: All commands respect context, no state corruption
```

**Test Workflow 2**: Dependency Blocking

```yaml
1. Create 2 specs: A (API), B (UI)
2. Add dependency: B depends on A (blocking)
3. Try to implement B (should block)
4. Implement A to completion
5. Try to implement B (should succeed)

Expected: Dependency logic prevents premature B implementation
```

**Test Workflow 3**: Conflict Detection

```yaml
1. Create 2 specs on same branch
2. Plan spec 1 (modifies auth.ts)
3. Plan spec 2 (also modifies auth.ts)
4. Run /speckit.conflicts
5. Verify conflict detected
6. Try /speckit.implement on spec 2 (should block)

Expected: Conflict prevents implementation
```

#### Performance Tests

**Benchmark**: Multi-spec overhead

| Operation | Single-Spec | Multi-Spec (2) | Multi-Spec (5) | Max Overhead |
|-----------|-------------|----------------|----------------|--------------|
| `/speckit.status` | 50ms | 80ms | 150ms | +200% |
| `/speckit.pm` | 100ms | 150ms | 250ms | +150% |
| `/speckit.next` | 30ms | 50ms | 80ms | +167% |
| Context load | 0ms | 10ms | 20ms | +20ms |

**Pass Criteria**: Overhead < 300% for 5 specs, < 50ms absolute increase

#### Test Automation

**Framework**: Bash + GitHub Actions

```bash
# .github/workflows/test-multi-spec.yml
name: Multi-Spec Tests

on: [push, pull_request]

jobs:
  regression:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run single-spec regression tests
        run: ./tests/regression-single-spec.sh

  multi-spec:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run multi-spec feature tests
        run: ./tests/feature-multi-spec.sh

  edge-cases:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run edge case tests
        run: ./tests/edge-cases.sh

  integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run integration workflows
        run: ./tests/integration-workflows.sh
```

---

### Phase 4 Deliverables

- [ ] User guide (MULTI-SPEC-WORKFLOWS.md)
- [ ] Technical design doc (MULTI-SPEC-ARCHITECTURE.md)
- [ ] Best practices guide (multi-spec-patterns.md)
- [ ] Migration guide (MULTI-SPEC-MIGRATION.md)
- [ ] Updated README with multi-spec section
- [ ] Updated all 17 command docs with multi-spec notes
- [ ] Regression test suite (51 test cases)
- [ ] Integration test workflows (3 scenarios)
- [ ] Edge case test suite (8 scenarios)
- [ ] Performance benchmarks
- [ ] CI/CD integration (GitHub Actions)

### Phase 4 Success Criteria

- ✅ Documentation covers 100% of multi-spec features
- ✅ Migration guide tested on 3+ projects (real-world validation)
- ✅ Regression test suite passes (51/51 tests green)
- ✅ Integration workflows complete end-to-end
- ✅ Performance benchmarks meet targets (< 300% overhead for 5 specs)
- ✅ CI/CD pipeline green on all commits

---

## RISK ASSESSMENT

### Risk Matrix

| Risk | Severity | Likelihood | Phase | Mitigation |
|------|----------|------------|-------|------------|
| **Breaking existing workflows** | 🔴 HIGH | 🟢 LOW | Phase 1-4 | 100% backward compat design, feature flagging, regression tests |
| **State file corruption** | 🔴 HIGH | 🟡 MEDIUM | Phase 1-2 | Atomic writes, backup on change, validation, rollback |
| **Context confusion (wrong spec)** | 🟠 MEDIUM | 🟡 MEDIUM | Phase 2 | Clear UX indicators, always show current spec, confirmation prompts |
| **Performance degradation** | 🟡 LOW | 🟢 LOW | Phase 1-4 | Batch ops optimized, caching, lazy loading, benchmarks |
| **Parallel execution file conflicts** | 🟠 MEDIUM | 🟠 MEDIUM | Phase 3 | Pre-flight conflict detection, user confirmation, abort on conflict |
| **Parallel execution git conflicts** | 🟠 MEDIUM | 🟡 MEDIUM | Phase 3 | Require clean working dir, file-level conflict check, sequential fallback |
| **Documentation gaps** | 🟡 LOW | 🟡 MEDIUM | Phase 4 | Dedicated docs phase, user testing, iterative updates |
| **User adoption resistance** | 🟡 LOW | 🟡 MEDIUM | Phase 2-4 | Opt-in features, clear migration guide, backward compatibility |
| **Dependency cycle deadlock** | 🟡 LOW | 🟢 LOW | Phase 2 | Cycle detection algorithm, clear error messages, manual override |
| **Complex multi-spec debugging** | 🟠 MEDIUM | 🟡 MEDIUM | Phase 2-3 | Enhanced logging, dashboard visibility, conflict reports |

**Overall Risk Level**: 🟢 **LOW**

**Rationale**:
- Backward compatibility eliminates breaking change risk
- Incremental rollout allows early detection of issues
- Feature flags enable rollback at any point
- Extensive testing catches edge cases
- Optional Phase 3 reduces scope if needed

---

## TIMELINE ESTIMATES

### 1 Developer (Sequential)

**Total: 6-8 weeks**

| Phase | Duration | Tasks |
|-------|----------|-------|
| **Phase 1: Foundation** | 2 weeks | Detection (3d), Context (3d), Batch Status (3d), Testing (2d) |
| **Phase 2: Commands** | 2 weeks | PM Selector (4d), Scope Classification (3d), Dependency (3d), Testing (2d) |
| **Phase 3: Advanced** | 2 weeks | Parallel Exec (5d), Dashboard (4d), Conflicts (3d), Testing (2d) |
| **Phase 4: Polish** | 1-2 weeks | Docs (5d), Migration (2d), Testing (3d) |

**Critical Path**: Phase 1 → Phase 2 → Phase 4 (MVP: 5-6 weeks if Phase 3 skipped)

---

### 2 Developers (Parallel Tracks)

**Total: 3-4 weeks**

| Week | Developer A | Developer B |
|------|-------------|-------------|
| **Week 1** | Phase 1: Foundation (detection, context) | Phase 2: Commands (PM selector, scope classification) |
| **Week 2** | Phase 1: Batch Status, Testing | Phase 2: Dependency tracking, Testing |
| **Week 3** | Phase 3: Parallel exec, Dashboard | Phase 4: Documentation (user guide, technical) |
| **Week 4** | Phase 3: Conflicts, Testing | Phase 4: Migration guide, CI/CD, Final testing |

**Integration Points**:
- End of Week 1: Merge Phase 1+2
- End of Week 2: Integration testing
- End of Week 3: Merge Phase 3+4
- End of Week 4: Final QA, release

**Critical Path**: Phase 1 (Week 1) → Integration (Week 2) → Testing (Week 4)

---

### 3 Developers (Aggressive Parallel)

**Total: 3-4 weeks**

| Week | Dev A | Dev B | Dev C |
|------|-------|-------|-------|
| **Week 1** | Phase 1: Foundation | Phase 2: Commands (PM selector) | Phase 4: Docs (start user guide) |
| **Week 2** | Phase 1: Testing | Phase 2: Dependency, Testing | Phase 3: Parallel exec |
| **Week 3** | Integration & Bugfixes | Phase 3: Dashboard, Conflicts | Phase 4: Migration, Technical docs |
| **Week 4** | Final Integration Testing | Phase 4: CI/CD setup | Release Prep |

**Risks with 3 Developers**:
- Coordination overhead (daily standups required)
- Merge conflicts (frequent syncs needed)
- Context switching (harder to maintain consistency)

**Recommendation**: 2 developers optimal (best cost/speed trade-off)

---

## SUCCESS CRITERIA SUMMARY

### Phase 1 Success Criteria

- ✅ Multi-spec detection works 100% of time (tested with 1, 2, 5, 10 specs)
- ✅ Context persistence survives terminal restart
- ✅ `/speckit.status --all` shows accurate aggregate data
- ✅ Zero regressions in single-spec mode (all 17 commands)
- ✅ Performance overhead < 100ms per command

### Phase 2 Success Criteria

- ✅ PM agent selector < 500ms response time
- ✅ Context switches persist across commands
- ✅ Dependency tracking prevents spec conflicts (100% accuracy)
- ✅ All commands respect spec context (no ambiguity)
- ✅ Single-spec regression tests still pass (backward compat)

### Phase 3 Success Criteria (Optional)

- ✅ Parallel execution 2x faster than sequential (for non-conflicting specs)
- ✅ Conflict detection catches 100% of file overlaps
- ✅ Dashboard renders < 100ms, auto-refresh smooth
- ✅ Zero data corruption from parallel execution
- ✅ User confidence in conflict resolution strategies

### Phase 4 Success Criteria

- ✅ Documentation covers 100% of multi-spec features
- ✅ Migration guide tested on 3+ projects (real-world validation)
- ✅ Regression test suite passes (51/51 tests green)
- ✅ Integration workflows complete end-to-end
- ✅ Performance benchmarks meet targets (< 300% overhead for 5 specs)
- ✅ CI/CD pipeline green on all commits

---

## RECOMMENDATION

### MVP Strategy: Phase 1 + 2 + 4 (Skip Phase 3)

**Rationale**:
- Phase 3 (Advanced Features) adds complexity without essential value
- 80% of benefit comes from Phases 1-2 (detection, context, commands)
- Phase 4 (Polish) critical for adoption (docs, testing, migration)

**MVP Deliverables**:
1. Multi-spec detection and context management ✓
2. Batch status and interactive selector ✓
3. Context-aware commands (all 17) ✓
4. Dependency tracking ✓
5. Comprehensive documentation ✓
6. Complete testing suite ✓

**What's Deferred** (Phase 3):
- Parallel execution (nice-to-have, not essential)
- Visual dashboard (can use `--all` flag instead)
- Conflict detection (users can manage manually initially)

**Timeline**: 5-6 weeks (1 dev) | 3 weeks (2 devs)

**Value Proposition**: Proven multi-spec support with 60% less implementation risk.

---

### Post-MVP: Phase 3 as Iterative Enhancements

**After MVP Launch**:
1. Gather user feedback on Phases 1-2 (2-4 weeks)
2. Assess demand for Phase 3 features:
   - If users request parallel execution → prioritize
   - If users struggle with conflicts → add detection
   - If dashboard requested → implement
3. Incremental rollout of Phase 3 features (1-2 at a time)

**Benefits**:
- Reduced upfront risk
- User-driven prioritization
- Easier to validate each feature independently
- Allows course correction based on real usage

---

## ARCHITECTURAL DECISIONS

### File Structure Changes

```
.specify/
  memory/
    multi-spec-context.json          # NEW: Persistent context tracking
    spec-dependencies.json           # NEW: Cross-spec dependencies (Phase 2)
    conflicts.json                   # NEW: Conflict detection (Phase 3)
    features/
      000003-billing-console/
        state.json                   # EXISTING: v2.1 state
      000004-company-switching/
        state.json                   # EXISTING: v2.1 state
  scripts/
    bash/
      common.sh                      # MODIFIED: Add multi-spec detection
      multi-spec-selector.sh         # NEW: Interactive spec selector (Phase 2)
      conflict-detector.sh           # NEW: File conflict detection (Phase 3)
  config.yml                         # NEW: Multi-spec configuration

specs/
  000003-billing-console/
    spec-metadata.json               # EXISTING: v2.3 metadata
  000004-company-switching/
    spec-metadata.json               # EXISTING: v2.3 metadata
```

### Command API Changes (Non-Breaking)

**Global Flags** (all commands):
```bash
--spec=000004              # Explicit spec selection (overrides context)
--force                    # Override warnings/blocks
```

**Batch Commands** (Phase 1):
```bash
/speckit.status --all      # Show all specs
/speckit.status --spec=000003  # Show specific spec
```

**Interactive Commands** (Phase 2):
```bash
/speckit.pm --select       # Force spec selector
/speckit.pm --all          # Show all PM contexts
```

**Dependency Commands** (Phase 2):
```bash
/speckit.dependency add <spec> <depends-on> "<reason>" [--blocking]
/speckit.dependency list
/speckit.dependency satisfy <spec> <depends-on>
```

**Advanced Commands** (Phase 3 - Optional):
```bash
/speckit.implement --parallel <spec1> <spec2>
/speckit.dashboard
/speckit.conflicts
/speckit.conflicts --spec=000004
/speckit.conflicts --resolved
```

### State Management

**Dual-State Consistency** (v2.3 + v2.1 Unified):

```yaml
State Priority:
  1. spec-metadata.json (v2.3 primary)
  2. state.json (v2.1 fallback for backward compat)
  3. multi-spec-context.json (session-level context)

Update Protocol:
  WHEN command changes spec state:
    1. Update spec-metadata.json (primary)
    2. IF state.json exists: sync changes (v2.1 compat)
    3. Update multi-spec-context.json (last_updated timestamp)
```

**Atomic Writes** (Prevent Corruption):

```bash
# Pattern for all state file writes
write_state_file() {
    local file="$1"
    local content="$2"
    local temp_file="${file}.tmp"
    local backup_file="${file}.bak"

    # Backup existing
    if [[ -f "$file" ]]; then
        cp "$file" "$backup_file"
    fi

    # Write to temp
    echo "$content" > "$temp_file"

    # Validate JSON
    if jq empty "$temp_file" 2>/dev/null; then
        # Atomic rename
        mv "$temp_file" "$file"
        echo "State updated successfully"
    else
        # Restore backup on failure
        if [[ -f "$backup_file" ]]; then
            mv "$backup_file" "$file"
        fi
        echo "ERROR: State update failed, restored backup"
        return 1
    fi
}
```

### Migration Path

**Version Compatibility**:

| Spec Kit Version | Multi-Spec Support | Backward Compatible |
|------------------|-------------------|---------------------|
| v2.3 (current) | ❌ Single-spec only | - |
| v2.4 (MVP) | ✅ Phase 1+2+4 | ✅ 100% compatible with v2.3 |
| v2.5 (future) | ✅ Phase 1+2+3+4 | ✅ 100% compatible with v2.4 |

**Upgrade Path**:

```yaml
From v2.3 to v2.4 (MVP):
  1. Update command files (`.claude/commands/*.md`)
  2. Update common.sh (add detection logic)
  3. Add multi-spec-context.json (new file)
  4. Add spec-dependencies.json (new file)
  5. Add documentation (new files)
  6. NO state file schema changes (backward compatible)
  7. NO user action required (automatic)

From v2.4 to v2.5 (Full):
  1. Add conflicts.json (new file)
  2. Add parallel execution logic
  3. Add dashboard command
  4. Update documentation
  5. NO breaking changes (optional features)
```

### Rollback Strategy

**Feature Flag** (`.specify/config.yml`):

```yaml
multi_spec:
  enabled: false              # Disable multi-spec features
  persistent_context: false   # Disable context tracking
  auto_detect: false          # Disable auto-activation
```

**Environment Variable Override**:

```bash
# Disable multi-spec for single session
export SPECKIT_DISABLE_MULTI_SPEC=1

# Disable multi-spec globally
echo 'export SPECKIT_DISABLE_MULTI_SPEC=1' >> ~/.bashrc
```

**Graceful Degradation**:

```yaml
IF multi-spec disabled OR error detected:
  → Fall back to v2.3 single-spec behavior
  → Log warning (not error)
  → Continue operation
```

---

## NEXT STEPS

### Immediate Actions

1. **Review Plan** (1-2 days)
   - Share with Spec Kit team
   - Gather feedback on phases
   - Decide: MVP (Phase 1+2+4) vs. Full (1+2+3+4)

2. **Prioritize Phases** (1 day)
   - Confirm MVP scope
   - Set target launch date
   - Assign developers

3. **Kickoff Phase 1** (Day 1 of Week 1)
   - Set up branch: `feature/multi-spec-support`
   - Create initial files (`multi-spec-context.json` schema)
   - Begin implementation (detection logic)

### Decision Points

**After Phase 1** (End of Week 2):
- ✅ Validate detection logic works
- ✅ Review user feedback on batch status
- 🎯 **Decide**: Proceed to Phase 2 or adjust Phase 1

**After Phase 2** (End of Week 4):
- ✅ Validate context management works
- ✅ Review user feedback on commands
- 🎯 **Decide**: Skip Phase 3 (MVP) or proceed to Phase 3 (Full)

**After MVP Launch** (Week 6-8):
- ✅ Gather real-world usage data
- ✅ Identify pain points
- 🎯 **Decide**: Prioritize Phase 3 features or other enhancements

---

## CONCLUSION

**Multi-Spec Shared Branch enhancement is feasible, low-risk, and valuable.**

**Key Strengths**:
- 100% backward compatible (zero breaking changes)
- Incremental rollout (validate each phase before proceeding)
- Practical value (solves real pain point for related specs)
- Optional advanced features (can ship MVP without Phase 3)

**Recommended Path**:
1. **Immediate**: Review plan with team, get buy-in
2. **Week 1-2**: Implement Phase 1 (Foundation)
3. **Week 3-4**: Implement Phase 2 (Commands)
4. **Week 5-6**: Implement Phase 4 (Polish) - **MVP LAUNCH**
5. **Post-MVP**: Assess demand for Phase 3 (Advanced)

**Timeline**: 5-6 weeks to MVP (1 dev) | 3 weeks to MVP (2 devs)

**Risk**: 🟢 LOW (backward compatible, feature-flagged, well-tested)

---

*End of Plan - Ready for Team Review*
