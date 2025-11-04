---
description: Expert review system for workflow orchestration, persona coordination, and multi-phase execution quality.
---

## Overview

`/speckit.workflow-reviewer` provides specialized expert analysis of workflow orchestration quality, focusing on persona coordination, decision gate effectiveness, parallel execution optimization, and end-to-end workflow health.

**Purpose**: While `/speckit.review` focuses on specification and plan quality, this command reviews the *orchestration itself* - how well the workflow coordinates multiple personas, manages state, and executes the Spec-Driven Development process.

## Use Cases

1. **Post-Workflow Analysis**: Review completed workflows to identify bottlenecks and inefficiencies
2. **Mid-Workflow Checkpoint**: Validate workflow execution during long-running processes
3. **Workflow Template Optimization**: Improve workflow patterns and strategies
4. **Persona Coordination Review**: Assess how well personas collaborate and handoff work
5. **Parallel Execution Analysis**: Identify missed parallelization opportunities

## Expert Panel

### Workflow-Specific Experts

| Expert | Specialty | Focus Areas |
|--------|-----------|-------------|
| **Gene Kim** | DevOps & Flow | Lead time, deployment frequency, workflow bottlenecks, value stream mapping |
| **Nicole Forsgren** | Metrics & Performance | Workflow metrics, throughput, quality indicators, data-driven optimization |
| **David Anderson** | Kanban & Flow | Work-in-progress limits, flow efficiency, cycle time, blockers |
| **Dominica DeGrandis** | Time Management | Time theft, dependencies, context switching, workflow delays |
| **Matthew Skelton** | Team Topologies | Team interaction modes, cognitive load, communication patterns |
| **Elisabeth Hendrickson** | Agile Testing | Test strategy integration, quality gates, feedback loops |
| **Allen Holub** | Agile Workflows | Value delivery, iterative refinement, adaptive planning |

## Usage

```bash
# Review completed workflow
/speckit.workflow-reviewer feature/000001-auth

# Review workflow strategy effectiveness
/speckit.workflow-reviewer feature/000001-auth --focus strategy

# Analyze persona coordination
/speckit.workflow-reviewer feature/000001-auth --focus personas

# Identify parallelization opportunities
/speckit.workflow-reviewer feature/000001-auth --focus parallelization

# Quick health check
/speckit.workflow-reviewer feature/000001-auth --quick
```

## Review Dimensions

### 1. Workflow Efficiency

**Metrics**:
- **Lead Time**: Time from workflow start to completion
- **Cycle Time**: Active work time (excluding wait states)
- **Flow Efficiency**: Cycle time / Lead time ratio
- **Throughput**: Features completed per time period

**Expert**: Gene Kim

**Analysis**:
```yaml
Lead Time Analysis:
  Total Duration: 4 hours 30 minutes
  Active Work: 2 hours 45 minutes
  Wait Time: 1 hour 45 minutes

  Flow Efficiency: 61% (2:45 / 4:30)

  Bottlenecks Identified:
    - Phase 2 (Clarification): 45 min wait for user response
    - Phase 5 (Implementation): 30 min sequential tasks that could be parallel
    - Phase 6 (Validation): 30 min test execution delay

  Recommendations:
    1. Reduce clarification rounds (strategy adjustment)
    2. Parallelize frontend/backend implementation (Phase 5)
    3. Run tests concurrently with code review (Phase 6)

  Potential Improvement: Lead time could reduce to 3 hours (33% faster)
```

### 2. Persona Coordination

**Metrics**:
- **Handoff Efficiency**: Time between persona transitions
- **Collaboration Quality**: Number of coordination points vs conflicts
- **Cognitive Load**: Number of active personas per phase
- **Context Switching**: Frequency of persona changes

**Expert**: Matthew Skelton (Team Topologies)

**Analysis**:
```yaml
Persona Coordination Map:

Phase 1 (Specification):
  Lead: Analyzer
  Support: None
  Duration: 30 minutes
  Handoff Quality: ✅ Clean (no rework)

Phase 2 (Clarification):
  Lead: Analyzer
  Support: Security (late involvement)
  Duration: 45 minutes
  Issues:
    - Security persona brought in at end, causing rework
    - Should have been parallel from start

Phase 3 (Planning):
  Lead: Architect
  Support: Backend, Frontend, DevOps
  Duration: 60 minutes
  Collaboration Quality: ✅ Excellent
  Context: All personas engaged early

Phase 5 (Implementation):
  Rotating: Frontend → Backend → DevOps
  Duration: 90 minutes
  Issues:
    - Sequential execution (Frontend → Backend)
    - Could be parallel (separate components)

Recommendations:
  1. Involve Security persona in Phase 1 for auth features
  2. Parallelize Frontend/Backend in Phase 5
  3. Reduce context switching by batching persona work
```

### 3. Decision Gate Quality

**Metrics**:
- **Gate Pass Rate**: Percentage of gates passed on first attempt
- **Gate Rework**: Time spent addressing gate failures
- **Constitutional Alignment**: Adherence to constitution principles

**Expert**: Elisabeth Hendrickson (Quality Gates)

**Analysis**:
```yaml
Constitutional Gate Results (Phase 3):

Article I (Library-First): ✅ PASS
  - Used bcrypt for password hashing
  - Used Passport.js for OAuth
  - No custom crypto implementations

Article II (CLI Interface): ⚠️ WARNING
  - Plan includes web UI focus
  - CLI interface mentioned but not prioritized
  - Recommendation: Ensure CLI parity

Article III (Test-First): ❌ FAIL
  - Tests mentioned in plan but not contracts
  - No clear test-before-implementation strategy
  - Required Rework: Add contract test approach

Article VII (Simplicity): ✅ PASS
  - Simple JWT approach (no complex session management)
  - Clear authentication flow

Article VIII (Anti-Abstraction): ✅ PASS
  - Concrete implementations, no premature abstraction
  - Direct database access (no ORM initially)

Gate Summary:
  Pass Rate: 80% (4/5)
  Failures: 1 (Test-First)
  Warnings: 1 (CLI Interface)
  Rework Time: 15 minutes (adding contract tests)

Recommendation:
  - Improve test-first discipline in planning phase
  - Add "CLI consideration" checkpoint for web-focused features
```

### 4. Parallelization Opportunities

**Metrics**:
- **Parallel Execution Ratio**: Parallel work / Total work
- **Dependency Graph Depth**: Longest dependency chain
- **Idle Resource Time**: Personas waiting for work

**Expert**: David Anderson (Flow)

**Analysis**:
```yaml
Task Dependency Analysis:

Current Execution:
  Wave 1: Task 1 (Frontend UI) - 30 min
  Wave 2: Task 2 (Backend API) - 40 min  [Blocked by Task 1]
  Wave 3: Task 3 (DevOps setup) - 20 min [Blocked by Task 2]
  Wave 4: Task 4 (Integration tests) - 30 min [Blocked by Task 3]

  Total Time: 120 minutes
  Parallel Ratio: 0% (all sequential)

Optimized Execution:
  Wave 1 (Parallel):
    - Task 1 (Frontend UI) - 30 min
    - Task 2 (Backend API) - 40 min
    - Task 3 (DevOps setup) - 20 min

  Wave 2 (Synchronization):
    - Task 4 (Integration tests) - 30 min

  Total Time: 70 minutes (42% reduction!)
  Parallel Ratio: 57% (40 min parallel / 70 min total)

Dependency Analysis:
  - Tasks 1, 2, 3 have NO hard dependencies
  - Only Task 4 requires completion of all three
  - Current workflow incorrectly marked dependencies

Recommendations:
  1. Update task.md to mark Tasks 1-3 as [P] (parallel)
  2. Coordinate personas to start simultaneously
  3. Use synchronization point before Task 4
  4. Potential time savings: 50 minutes (42%)
```

### 5. Strategy Effectiveness

**Metrics**:
- **Strategy Fit**: Appropriateness of chosen strategy for feature complexity
- **Throughput vs Quality**: Balance between speed and thoroughness
- **Scope Creep**: Unplanned work discovered during execution

**Expert**: Nicole Forsgren (Metrics)

**Analysis**:
```yaml
Strategy: Systematic
Feature Complexity: Medium (OAuth integration)
Expected Duration: 3-4 hours
Actual Duration: 4 hours 30 minutes

Strategy Assessment:

  Appropriateness: ✅ GOOD
    - OAuth requires security review (systematic strategy includes this)
    - Multiple integration points (Google, GitHub, Microsoft)
    - Production-critical feature (deserves thorough approach)

  Over-Engineering Indicators: ⚠️ MINOR
    - 5 clarification rounds (4 would have sufficed)
    - All constitutional gates applied (could skip Article VIII for this feature)
    - Deep architecture discussion (good but lengthy)

  Under-Engineering Indicators: ✅ NONE
    - Security properly considered
    - All edge cases addressed
    - Comprehensive test coverage

  Scope Creep: 15% (acceptable)
    - Added: Token refresh mechanism (should have been in spec)
    - Added: Rate limiting (good defensive addition)

Recommendation:
  - Strategy choice was appropriate
  - Consider "Fast" strategy for simpler OAuth provider additions
  - Clarification rounds could be reduced to 3-4 for similar features

Comparison with Alternative Strategies:
  - Fast Strategy: 2 hours estimated, but likely missing security considerations
  - Deep Strategy: 6+ hours, excessive for this feature type
  - Systematic: ✅ Optimal balance
```

### 6. State Management Quality

**Metrics**:
- **State Consistency**: Accuracy of state.json throughout workflow
- **Resume Capability**: Ability to resume from any phase
- **Audit Trail**: Completeness of workflow history

**Expert**: Dominica DeGrandis (Dependencies)

**Analysis**:
```yaml
State File Analysis: .specify/memory/workflows/000001-auth.json

State Updates:
  Phase 1: ✅ Updated correctly
  Phase 2: ✅ Updated correctly
  Phase 3: ⚠️ Delayed update (15 min lag)
  Phase 4: ✅ Updated correctly
  Phase 5: ❌ Missing intermediate updates (90 min gap)
  Phase 6: ✅ Updated correctly

Resume Capability Test:
  From Phase 1: ✅ Possible
  From Phase 2: ✅ Possible
  From Phase 3: ⚠️ Possible but state stale
  From Phase 4: ✅ Possible
  From Phase 5: ❌ Not possible (no checkpoints during implementation)
  From Phase 6: ✅ Possible

Issues:
  1. Phase 5 has 90-minute gap without state updates
     - If workflow crashes during implementation, lose progress
     - Cannot resume mid-implementation

  2. Phase 3 state update delayed by 15 minutes
     - State shows "in_progress" when actually completed

Recommendations:
  1. Add checkpoint after each task completion in Phase 5
  2. Update state immediately upon phase completion
  3. Add "sub-phase" tracking for long phases (>30 min)
  4. Include persona handoff timestamps in state
```

## Workflow Health Score

Composite score based on all dimensions:

```yaml
Overall Workflow Health: 7.2/10 (Good)

Dimension Scores:
  - Efficiency: 6.5/10 (Fair - bottlenecks identified)
  - Persona Coordination: 7.0/10 (Good - minor issues)
  - Decision Gates: 8.0/10 (Good - 80% pass rate)
  - Parallelization: 5.0/10 (Fair - missed opportunities)
  - Strategy Fit: 9.0/10 (Excellent)
  - State Management: 6.5/10 (Fair - gaps in tracking)

Interpretation:
  - 9-10: Excellent - Workflow optimized
  - 7-8: Good - Minor improvements available
  - 5-6: Fair - Significant optimization potential
  - 3-4: Poor - Major workflow issues
  - 0-2: Critical - Workflow breakdown

Primary Improvement Area: Parallelization (5.0/10)
  - Potential time savings: 50 minutes (42% reduction)
  - Low effort, high impact
```

## Review Modes

### 1. Post-Mortem Mode (Default)

**Purpose**: Comprehensive analysis after workflow completion

**Process**:
1. Load workflow state and history
2. Calculate all metrics
3. Identify bottlenecks and inefficiencies
4. Generate improvement recommendations
5. Compare against similar workflows

**Output**: Full report with all dimensions

### 2. Live Monitoring Mode

**Purpose**: Real-time workflow health during execution

**Process**:
1. Monitor workflow state changes
2. Alert on anomalies (long wait times, gate failures)
3. Suggest interventions
4. Track progress against expected timeline

**Output**: Real-time dashboard with alerts

```bash
/speckit.workflow-reviewer feature/000001-auth --live

# Output:
📊 Live Workflow Monitor: 000001-auth-system

Current Phase: IMPLEMENTATION (Phase 5/6)
Progress: 65% complete
Elapsed: 3 hours 15 minutes
Expected Remaining: 1 hour 5 minutes

⚠️ Alert: Sequential execution detected
   - Frontend task complete (30 min)
   - Backend task running (20 min elapsed)
   - DevOps task waiting
   - Recommendation: Start DevOps task in parallel

🟢 Health: 7.0/10 (On track)
```

### 3. Comparative Mode

**Purpose**: Compare workflow execution against historical data

**Process**:
1. Load current workflow metrics
2. Query historical workflows of similar type
3. Compare efficiency, duration, quality
4. Identify regression or improvement trends

**Output**: Comparative analysis with trends

```yaml
Comparative Analysis: OAuth Implementation Workflows

Current Workflow: 000001-auth-system
Duration: 4 hours 30 minutes
Health Score: 7.2/10

Historical Average (last 5 OAuth workflows):
Duration: 3 hours 45 minutes
Health Score: 7.8/10

Comparison:
  Duration: 🔴 20% slower than average
  Health: 🟡 8% below average

  Better Than Average:
    - Strategy Fit (9.0 vs 8.2)
    - Decision Gates (8.0 vs 7.5)

  Worse Than Average:
    - Parallelization (5.0 vs 7.5) ⚠️
    - Efficiency (6.5 vs 8.0)
    - State Management (6.5 vs 7.8)

Root Cause:
  - Missed parallelization opportunities in Phase 5
  - Inefficient task sequencing

Learning:
  - Review tasks.md from workflow 000042-github-oauth (similar feature)
  - Their parallel execution pattern is superior
```

### 4. Strategy Recommendation Mode

**Purpose**: Suggest optimal strategy for upcoming workflow

**Process**:
1. Analyze feature description or PRD
2. Assess complexity indicators
3. Review historical data for similar features
4. Recommend strategy and customizations

**Output**: Strategy recommendation with rationale

```yaml
Feature: "Add SAML SSO authentication"
Complexity Analysis:
  - Technology: SAML (complex protocol)
  - Security: High (enterprise authentication)
  - Integration: 3 external systems
  - Risk: High (production authentication)

Recommended Strategy: DEEP

Rationale:
  1. SAML complexity requires extended discovery
  2. Security-sensitive (needs threat modeling)
  3. Enterprise compliance requirements
  4. High risk of rework if underspecified

Strategy Customizations:
  - Involve Security persona from Phase 1
  - Add compliance review gate (GDPR, SOC2)
  - Extend clarification to 6-8 rounds
  - Include security architect in planning

Alternative: Systematic strategy with security focus
  - If team has SAML experience
  - If this is similar to previous SAML implementations
  - Savings: 1-2 hours, but higher rework risk

Estimated Duration:
  - Deep Strategy: 6-8 hours
  - Systematic + Security: 4-6 hours (riskier)
```

## Command Options

```bash
# Review mode
--mode post-mortem    # Default: full analysis after completion
--mode live           # Real-time monitoring during execution
--mode comparative    # Compare against historical workflows
--mode strategy       # Recommend strategy for new workflow

# Focus areas
--focus efficiency    # Lead time, cycle time, bottlenecks
--focus personas      # Persona coordination and handoffs
--focus gates         # Decision gate quality and pass rate
--focus parallelization  # Parallel execution opportunities
--focus strategy      # Strategy effectiveness assessment
--focus state         # State management quality

# Output control
--quick               # Top 3 issues only
--detailed            # Full analysis with examples
--metrics-only        # Just the numbers, no recommendations

# Comparison
--compare [workflow_id]  # Compare against specific workflow
--compare average     # Compare against historical average
```

## Integration with Workflow

```yaml
Recommended Usage:

Before Starting Workflow:
  → /speckit.workflow-reviewer --mode strategy [feature description]
  → Choose optimal strategy based on recommendation

During Workflow (Optional):
  → /speckit.workflow-reviewer --mode live feature/000001-auth
  → Monitor for issues and optimize in real-time

After Workflow Completion:
  → /speckit.workflow-reviewer feature/000001-auth --mode post-mortem
  → Learn from execution, improve future workflows

Periodic Review:
  → /speckit.workflow-reviewer --mode comparative --focus efficiency
  → Track workflow improvement trends over time
```

## Metrics Storage

Workflow metrics stored in `.specify/memory/analytics/workflows.json`:

```json
{
  "workflows": [
    {
      "id": "wf-001-000001-auth",
      "feature": "000001-auth-system",
      "strategy": "systematic",
      "started_at": "2025-01-26T10:00:00Z",
      "completed_at": "2025-01-26T14:30:00Z",
      "lead_time_minutes": 270,
      "cycle_time_minutes": 165,
      "flow_efficiency": 0.61,
      "health_score": 7.2,
      "dimension_scores": {
        "efficiency": 6.5,
        "persona_coordination": 7.0,
        "decision_gates": 8.0,
        "parallelization": 5.0,
        "strategy_fit": 9.0,
        "state_management": 6.5
      },
      "bottlenecks": [
        {
          "phase": "clarification",
          "delay_minutes": 45,
          "reason": "User response wait time"
        },
        {
          "phase": "implementation",
          "delay_minutes": 30,
          "reason": "Sequential tasks (should be parallel)"
        }
      ],
      "personas_used": ["analyzer", "architect", "security", "frontend", "backend", "devops"],
      "gate_results": {
        "total_gates": 5,
        "passed": 4,
        "failed": 1,
        "rework_minutes": 15
      }
    }
  ],
  "aggregates": {
    "total_workflows": 47,
    "average_lead_time": 225,
    "average_health_score": 7.8,
    "most_common_bottleneck": "clarification_wait"
  }
}
```

## Success Criteria

- ✅ Accurate workflow metrics calculation
- ✅ Expert frameworks applied to orchestration quality
- ✅ Bottleneck identification with time impact
- ✅ Parallelization opportunity detection
- ✅ Strategy effectiveness assessment
- ✅ Historical comparison and trend analysis
- ✅ Actionable recommendations for optimization
- ✅ Integration with workflow state management

## Example Output

```bash
/speckit.workflow-reviewer feature/000001-auth --detailed

# Output:
═══════════════════════════════════════════════════════════
🔍 WORKFLOW REVIEW: 000001-auth-system
═══════════════════════════════════════════════════════════

Strategy: Systematic
Duration: 4 hours 30 minutes
Status: Completed
Health Score: 7.2/10 (Good)

───────────────────────────────────────────────────────────
📊 DIMENSION SCORES
───────────────────────────────────────────────────────────

Efficiency:           6.5/10 (Fair)
Persona Coordination: 7.0/10 (Good)
Decision Gates:       8.0/10 (Good)
Parallelization:      5.0/10 (Fair) ⚠️
Strategy Fit:         9.0/10 (Excellent)
State Management:     6.5/10 (Fair)

───────────────────────────────────────────────────────────
🚨 TOP ISSUES
───────────────────────────────────────────────────────────

1. 🔴 HIGH IMPACT: Missed Parallelization (Phase 5)

   Current: Sequential execution of Frontend → Backend → DevOps
   Time: 90 minutes

   Optimal: Parallel execution of all three
   Time: 40 minutes

   Savings: 50 minutes (56% reduction in Phase 5)

   Recommendation: Mark tasks 1-3 as [P] in tasks.md

2. 🟡 MEDIUM IMPACT: Clarification Overhead (Phase 2)

   Rounds: 5 (could be 3-4)
   Time: 45 minutes (15 min excess)

   Recommendation: Pre-populate common OAuth questions in templates

3. 🟢 LOW IMPACT: State Update Gaps (Phase 5)

   Gap: 90 minutes without checkpoint
   Risk: Cannot resume mid-implementation if crash occurs

   Recommendation: Add checkpoints after each task completion

───────────────────────────────────────────────────────────
💡 EXPERT INSIGHTS
───────────────────────────────────────────────────────────

Gene Kim (DevOps & Flow):
"Your flow efficiency is 61%, which is decent but can improve.
The 1 hour 45 minutes of wait time is mostly in clarification
(45 min) and unnecessary sequential execution (30 min). Focus
on reducing these delays for future workflows."

Matthew Skelton (Team Topologies):
"Persona coordination is good overall. One improvement: bring
Security persona into Phase 1 for auth features, not Phase 2.
This eliminates the rework we saw when Security raised concerns
during clarification that should have been in the spec."

David Anderson (Kanban & Flow):
"You have clear parallelization opportunities that weren't
utilized. Tasks 1, 2, and 3 in Phase 5 have no dependencies
on each other. Running them in parallel would save 50 minutes."

───────────────────────────────────────────────────────────
📈 COMPARATIVE ANALYSIS
───────────────────────────────────────────────────────────

vs. Historical Average (OAuth workflows):
  Duration: 4:30 vs 3:45 (20% slower) 🔴
  Health: 7.2 vs 7.8 (8% lower) 🟡

vs. Best Workflow (000042-github-oauth):
  Duration: 4:30 vs 2:50 (59% slower) 🔴
  Health: 7.2 vs 9.1 (21% lower) 🔴

  Learn from 000042:
    - Excellent parallel execution (saved 60 min)
    - Streamlined clarification (only 2 rounds)
    - Pre-planned security review (no rework)

───────────────────────────────────────────────────────────
✅ WHAT WENT WELL
───────────────────────────────────────────────────────────

1. Excellent strategy choice (9.0/10)
   - Systematic was appropriate for OAuth complexity
   - Security properly considered
   - Quality gates prevented issues

2. Strong decision gate performance (8.0/10)
   - 80% pass rate (4/5 gates)
   - Only minor rework needed
   - Constitutional alignment maintained

3. Good persona collaboration (7.0/10)
   - Clean handoffs between phases
   - Clear role definitions
   - Minimal conflicts

───────────────────────────────────────────────────────────
🎯 IMPROVEMENT ACTIONS
───────────────────────────────────────────────────────────

Priority 1 (High Impact, Low Effort):
  → Update task breakdown process to identify parallel work
  → Could save 50 minutes on similar workflows

Priority 2 (Medium Impact, Low Effort):
  → Create OAuth clarification template (common questions)
  → Could save 15 minutes on OAuth workflows

Priority 3 (Low Impact, Medium Effort):
  → Add state checkpoints every 30 minutes in Phase 5
  → Improves resume capability and progress visibility

───────────────────────────────────────────────────────────
📊 METRICS SUMMARY
───────────────────────────────────────────────────────────

Lead Time: 4 hours 30 minutes (270 min)
Cycle Time: 2 hours 45 minutes (165 min)
Flow Efficiency: 61% (cycle/lead)

Phase Breakdown:
  Phase 1 (Specification):   30 min (11%)
  Phase 2 (Clarification):   45 min (17%)
  Phase 3 (Planning):        60 min (22%)
  Phase 4 (Tasks):           15 min (6%)
  Phase 5 (Implementation):  90 min (33%)
  Phase 6 (Validation):      30 min (11%)

Potential Optimized Time: 3 hours 5 minutes
Savings: 1 hour 25 minutes (31% reduction)

═══════════════════════════════════════════════════════════
```

---

**Implementation**: Phase 3.8
**Dependencies**:
  - Phase 1.2 (State Management)
  - Phase 3.7 (Workflow Orchestration)
  - Phase 3.5 (Review System)
**Pattern**: Meta-analysis of workflow execution with optimization recommendations
**Complexity**: High (multi-dimensional analysis, historical comparison)
**Token Budget**: 2000-5000 tokens depending on mode
