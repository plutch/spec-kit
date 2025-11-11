# Spec-Kit vs CCPM: Comprehensive Comparison

**Date**: 2025-11-11
**Spec-Kit Version**: v2.6.0
**CCPM Version**: Latest (5.4k ⭐, 570 forks)

---

## Executive Summary

**Spec-Kit** and **CCPM** are both spec-driven development frameworks for Claude Code, but serve **different use cases**:

| Dimension | Spec-Kit | CCPM |
|-----------|----------|------|
| **Primary Use Case** | Single-feature development with constitutional enforcement | Multi-feature project management with parallel execution |
| **Team Size** | 1-3 developers (individual/small teams) | 3-20 developers (distributed teams) |
| **Complexity** | Simple (copy/paste install) | Medium (Git worktrees, GitHub integration) |
| **Governance** | Constitutional principles enforcement | GitHub Issues as collaboration protocol |
| **Execution Model** | Sequential (one feature at a time) | Parallel (5-12 agents simultaneously) |
| **Installation** | Zero dependencies | Requires Git worktrees + `gh` CLI |

**Key Insight**: Spec-Kit focuses on **quality and constitutional compliance** for individual features. CCPM focuses on **velocity and team coordination** for multi-feature projects.

---

## Philosophy & Design Goals

### Spec-Kit Philosophy

**Core Principle**: "Constitutional enforcement from specification through implementation"

**Design Goals**:
1. **Zero constitutional violations** reach production
2. **Evidence-based validation** prevents hallucinations
3. **Single implementation cycle** (50-70% token savings)
4. **Copy/paste installation** (no dependencies)
5. **Quality gates** at every phase

**Metaphor**: A **constitutional court** that validates every decision against project principles.

---

### CCPM Philosophy

**Core Principle**: "No Vibe Coding" - every line traces to specification

**Design Goals**:
1. **Parallel execution** (5-12 agents simultaneously)
2. **GitHub Issues** as collaboration database
3. **Context firewall** (agents compress output 80-90%)
4. **Team visibility** (human-AI handoffs)
5. **Distributed development** without separate PM tools

**Metaphor**: A **project management office** that coordinates parallel work streams.

---

## Detailed Feature Comparison

### 1. Workflow Model

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Phases** | 6 sequential (specify → gap → plan → tasks → implement → reconcile) | 5 phases (PRD → epic → decompose → sync → execute) | Tie (different models) |
| **Parallelism** | ❌ Sequential only | ✅ 5-12 parallel agents via Git worktrees | **CCPM** |
| **Phase Tracking** | ✅ spec-metadata.json (v2.3) | ✅ Local markdown + GitHub Issues | Tie |
| **Approval Gates** | ✅ Automated (blocks if previous phase unapproved) | ⚠️ Manual (user-driven) | **Spec-Kit** |
| **State Management** | ✅ Unified (spec-metadata.json) | ✅ Distributed (local + GitHub) | Tie |

**Key Difference**: Spec-Kit enforces **linear progression** with automated gates. CCPM enables **concurrent work** with manual coordination.

---

### 2. Context Management

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Memory System** | ✅ 4 memory files (constitution, API standards, testing, deployment) | ✅ `.claude/context/` files | Tie |
| **Context Firewall** | ❌ Analysis runs in main thread (15K-30K tokens) | ✅ Agents compress output 80-90% (2K-3K tokens) | **CCPM** 🔴 |
| **Session Persistence** | ✅ Memory files survive resets | ✅ Local context files | Tie |
| **Token Efficiency** | ⚠️ High for analysis commands | ✅ Excellent (firewall pattern) | **CCPM** |
| **Context Pollution** | ⚠️ Verbose analysis in main thread | ✅ Isolated in agents | **CCPM** |

**Key Difference**: CCPM's **context firewall** is a game-changer for token efficiency. Spec-Kit runs analysis inline, consuming 10x more tokens.

**Impact**: CCPM's approach scales better for large codebases (100K+ lines).

---

### 3. Constitutional Enforcement

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Constitution Support** | ✅ Core feature (v2.4 enforcement bridge) | ⚠️ Basic (`.claude/CLAUDE.md` as always-on instructions) | **Spec-Kit** 🏆 |
| **Automated Validation** | ✅ 3 enforcement points (pre-flight, quality gate, reconciliation) | ❌ Manual adherence | **Spec-Kit** |
| **Prohibited Patterns** | ✅ Machine-readable + automated scanning | ❌ Not available | **Spec-Kit** |
| **TDD Enforcement** | ✅ Git history analysis (tests before implementation) | ❌ Not enforced | **Spec-Kit** |
| **Simplicity Validation** | ✅ Framework count limits, complexity tracking | ❌ Not available | **Spec-Kit** |
| **Integration-First** | ✅ Contract test validation (v2.5/v2.6) | ❌ Not enforced | **Spec-Kit** |

**Key Difference**: Spec-Kit's v2.4-v2.6 constitutional enforcement is **unmatched**. CCPM relies on manual discipline.

**Impact**: Spec-Kit prevents architectural drift. CCPM trusts developers to follow principles.

---

### 4. Quality Framework

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Risk Assessment** | ✅ 6-criteria scoring (0-12 scale) | ❌ Not available | **Spec-Kit** |
| **Quality Scoring** | ✅ 4 dimensions (Clarity, Completeness, Testability, Consistency) | ❌ Not available | **Spec-Kit** |
| **Socratic Validation** | ✅ 4-question filter (40-60% false positive reduction) | ❌ Not available | **Spec-Kit** |
| **Evidence-Based Review** | ✅ 94% hallucination detection rate | ✅ Pre/post verification (context accuracy) | Tie |
| **UX Quality Framework** | ✅ 5-dimensional UX scoring + component audit | ❌ Not available | **Spec-Kit** |
| **OWASP Top 10** | ✅ Built-in security review (Phase 3) | ❌ Not available | **Spec-Kit** |

**Key Difference**: Spec-Kit has a **comprehensive quality framework**. CCPM focuses on context accuracy only.

**Impact**: Spec-Kit catches quality issues early. CCPM prevents hallucinated documentation.

---

### 5. Testing & Validation

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Test Strategy Planning** | ✅ Phase 2.5 (unit, integration, E2E, performance) | ⚠️ Basic (mentioned in PRDs) | **Spec-Kit** |
| **Test Isolation** | ❌ Tests run inline in /speckit.implement | ✅ `/testing:run` spawns agent (context firewall) | **CCPM** |
| **Coverage Thresholds** | ✅ 90% critical, 80% business logic, 75% overall | ❌ Not enforced | **Spec-Kit** |
| **Edge Case Detection** | ✅ `/speckit.clarify --edge-cases` (6 categories) | ❌ Not available | **Spec-Kit** |
| **Mock Strategy Validation** | ✅ Article V compliance (v2.5/v2.6) | ❌ Not available | **Spec-Kit** |
| **Test Execution** | ⚠️ Inline (verbose output) | ✅ Agent-based (compressed results) | **CCPM** |

**Key Difference**: Spec-Kit plans tests comprehensively but executes inline. CCPM isolates test execution in agents.

**Impact**: CCPM's `/testing:run` is cleaner. Spec-Kit's test planning is deeper.

---

### 6. Requirements Format

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Structured Requirements** | ✅ EARS format (4 patterns: WHEN/IF/WHILE/WHERE) | ⚠️ Natural language (guided brainstorming) | **Spec-Kit** |
| **Requirement IDs** | ✅ REQ-XX-Y.Z format | ❌ Not standardized | **Spec-Kit** |
| **Traceability** | ✅ Full (spec → plan → tasks → tests) | ⚠️ Partial (PRD → epic → task → code) | Tie |
| **Testability** | ✅ Every requirement has clear success criteria | ⚠️ Varies by PRD quality | **Spec-Kit** |
| **Ambiguity Prevention** | ✅ Structured patterns eliminate vague language | ⚠️ Relies on thorough brainstorming | **Spec-Kit** |

**Key Difference**: EARS format is **unambiguous and testable**. CCPM's natural language is flexible but less precise.

**Impact**: Spec-Kit requirements map directly to tests. CCPM requirements need interpretation.

---

### 7. GitHub Integration

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **GitHub Issues** | ❌ Local-only (no Issue sync) | ✅ Core feature (Issues as database) | **CCPM** 🏆 |
| **Team Visibility** | ❌ Requires repo access to see specs | ✅ Real-time progress via Issue comments | **CCPM** |
| **Human-AI Handoffs** | ⚠️ Manual (read specs, continue work) | ✅ Seamless (Issues show AI progress) | **CCPM** |
| **PR Creation** | ⚠️ Manual (git commands) | ✅ Automated (`/pm:issue-sync`) | **CCPM** |
| **Distributed Teams** | ⚠️ Manual coordination | ✅ Native support (multiple Claude instances) | **CCPM** |

**Key Difference**: CCPM's **GitHub-native approach** enables true team collaboration. Spec-Kit is single-developer focused.

**Impact**: For teams >3 developers, CCPM's visibility is essential.

---

### 8. Parallel Execution

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Git Worktrees** | ❌ Not supported | ✅ Core feature (one per agent) | **CCPM** 🏆 |
| **Concurrent Agents** | ❌ Sequential (1 task at a time) | ✅ 5-12 agents simultaneously | **CCPM** |
| **Merge Conflict Handling** | N/A | ✅ Isolated worktrees prevent collisions | **CCPM** |
| **Speed Improvement** | Baseline | ✅ 3x faster (5-8 parallel tasks vs 1) | **CCPM** |
| **Independence Detection** | ❌ Not available | ✅ Epic decomposition identifies parallel-safe tasks | **CCPM** |

**Key Difference**: CCPM's parallel model is **revolutionary** for multi-module features. Spec-Kit is sequential.

**Impact**: Large features (10+ tasks) complete 3x faster with CCPM.

**Trade-off**: Parallel execution adds complexity (learning curve, edge cases).

---

### 9. Installation & Dependencies

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Installation Method** | Copy/paste (2 commands) | `curl` script or manual copy | Tie |
| **Dependencies** | ✅ Zero (pure markdown) | ⚠️ Git worktrees + `gh` CLI (for GitHub sync) | **Spec-Kit** |
| **Compatibility** | ✅ Linux, macOS, Windows | ✅ Linux, macOS (Windows with WSL) | Tie |
| **Setup Time** | 2 minutes | 5-10 minutes (gh auth, worktree setup) | **Spec-Kit** |
| **Maintenance** | ✅ Zero (static templates) | ⚠️ GitHub API changes, gh CLI updates | **Spec-Kit** |

**Key Difference**: Spec-Kit's **zero-dependency philosophy** is simpler. CCPM requires tooling for full features.

**Impact**: Spec-Kit works out-of-the-box. CCPM needs environment setup.

---

### 10. Command Design

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Command Count** | 17 commands | ~20 commands (PRD, epic, issue, workflow, sync, utility) | Tie |
| **Self-Contained** | ✅ No @include references | ⚠️ Some commands reference `.claude/context/` | **Spec-Kit** |
| **Tool Declarations** | ❌ Implicit tool usage | ✅ Frontmatter tool declarations | **CCPM** |
| **Error Handling** | ✅ Graceful degradation (v2.4 backward compat) | ✅ Fail-fast with clear errors | Tie |
| **Agent Spawning** | ⚠️ Rare (most work inline) | ✅ Common (context firewall pattern) | **CCPM** |

**Key Difference**: CCPM's **tool declarations** improve clarity. Spec-Kit's **self-contained commands** are simpler.

**Impact**: CCPM commands are more maintainable. Spec-Kit commands are easier to distribute.

---

### 11. Gap Closure & Reconciliation

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Post-Implementation Gaps** | ✅ `/speckit.reconcile` (surgical edits, append-only) | ⚠️ Manual (update specs, create new tasks) | **Spec-Kit** |
| **Gap Detection** | ✅ Git diff execution (live evidence) | ⚠️ User-reported gaps | **Spec-Kit** |
| **Silent Gap Detection** | ✅ Identifies gaps not in user's report | ❌ Not available | **Spec-Kit** |
| **Specification Integrity** | ✅ Append-only edits preserve history | ⚠️ Overwrite/replace common | **Spec-Kit** |
| **Reconciliation Workflow** | ✅ Structured (≤5 questions, sync impact report) | ⚠️ Ad-hoc (manual spec updates) | **Spec-Kit** |

**Key Difference**: Spec-Kit's **reconciliation workflow** is systematic. CCPM relies on manual gap closure.

**Impact**: Spec-Kit maintains spec accuracy post-implementation. CCPM specs may drift.

---

### 12. Documentation & Learning Curve

| Feature | Spec-Kit | CCPM | Winner |
|---------|----------|------|--------|
| **Documentation Quality** | ✅ Comprehensive (README, INSTALL, guides) | ✅ Comprehensive (README, COMMANDS, AGENTS, CONTEXT) | Tie |
| **Learning Curve** | ⚠️ Medium (17 commands, workflow phases) | ⚠️ Medium-High (parallel model, worktrees, GitHub) | **Spec-Kit** |
| **Onboarding Time** | 30 minutes (read README + try /speckit.specify) | 1-2 hours (understand PRD → epic → worktrees) | **Spec-Kit** |
| **Examples** | ✅ Inline in commands (EARS format, templates) | ✅ Inline + external tutorials | Tie |
| **Community Support** | ⚠️ Smaller (newer framework) | ✅ Large (5.4k stars, active Hacker News) | **CCPM** |

**Key Difference**: Spec-Kit is **easier to learn** but less mature. CCPM has proven **community traction**.

**Impact**: Spec-Kit: 30-minute ramp-up. CCPM: 1-2 hour investment with higher ceiling.

---

## Use Case Recommendations

### When to Use Spec-Kit

✅ **Constitutional Enforcement is Critical**
- Projects with strict architectural principles (TDD, simplicity, integration-first)
- Teams requiring automated governance (no manual discipline)
- Projects where architectural drift is a known risk

✅ **Single-Developer or Small Teams (1-3 people)**
- Solo developers building features sequentially
- Startups with lean teams
- Consulting projects with one primary developer

✅ **Quality-Driven Development**
- Risk-based prioritization (HIGH/MEDIUM/LOW)
- OWASP Top 10 validation required
- UX quality scoring needed (interface-heavy features)
- EARS requirements for testability

✅ **Zero-Dependency Environments**
- Air-gapped systems
- Corporate environments with restricted tooling
- Quick proof-of-concepts (copy/paste install)

**Example Projects**:
- SaaS product with 1-2 developers
- Enterprise feature development with strict governance
- High-security projects (fintech, healthcare) requiring OWASP validation

---

### When to Use CCPM

✅ **Parallel Execution is Essential**
- Large features with 10+ independent tasks
- Multi-module projects (API + UI + DB + docs)
- Teams needing 3x delivery speed

✅ **Distributed Teams (3-20 developers)**
- Multiple developers working on same project
- Human-AI handoffs required
- Team visibility needed (GitHub Issues)

✅ **Context Efficiency is Priority**
- Large codebases (100K+ lines)
- Complex analysis operations (10+ file reads)
- Token budget constraints (Claude Code API)

✅ **GitHub-Native Workflows**
- Teams already using GitHub Issues for PM
- Projects requiring stakeholder visibility
- Distributed collaboration (remote teams)

**Example Projects**:
- Open-source projects with contributors
- Agency teams managing multiple client features
- Startup product teams (5-10 developers)

---

### Hybrid Approach (Use Both)

**Scenario**: Medium-sized team (3-5 developers) with constitutional governance needs.

**Strategy**:
1. **Use Spec-Kit** for specification and planning:
   - `/speckit.specify` → EARS requirements
   - `/speckit.validate-gap` → Feasibility analysis
   - `/speckit.plan` → Constitutional validation (Phase -1)
   - Export plan to CCPM-compatible format

2. **Use CCPM** for execution:
   - `/pm:epic-decompose` → Break into parallel tasks
   - `/pm:epic-sync` → Push to GitHub Issues
   - `/pm:issue-start` (5-8 parallel agents) → Execute
   - `/pm:issue-sync` → Track progress

3. **Use Spec-Kit** for reconciliation:
   - `/speckit.reconcile` → Close gaps with append-only edits
   - `/speckit.validate-hierarchy` → Validate spec integrity

**Benefit**: Get **constitutional enforcement** from Spec-Kit + **parallel execution** from CCPM.

**Trade-off**: Requires manual coordination between frameworks (no native integration).

---

## Head-to-Head: Feature Delivery Example

### Scenario: Build a Billing Dashboard

**Requirements**:
- Multi-tenant SaaS billing dashboard
- Stripe integration (payments, subscriptions)
- Angular 20 + Kendo UI
- REST API + PostgreSQL
- 12 independent tasks (API routes, UI components, DB migrations, tests)

---

### Spec-Kit Workflow (Sequential)

**Timeline**: ~6 hours (12 tasks × 30 min each)

```bash
# Setup (one-time)
/speckit.memory  # 5 min

# Specification (30 min)
/speckit.specify "Build billing dashboard..."
# Output: spec.md with EARS requirements (REQ-BILLING-1.1, REQ-BILLING-1.2, etc.)

# Gap Analysis (20 min)
/speckit.validate-gap
# Output: gap-analysis.md (identifies reusable components, complexity: MEDIUM)

# Planning (45 min)
/speckit.plan
# Output: plan-1.md with:
# - Phase -1: Constitutional Validation ✅ PASS
# - Phase 2.5: Test Strategy (integration tests, no mocked DB)
# - Phase 2.6: Test Strategy Validation ✅ PASS
# - Phase 3: Security Review (OWASP Top 10)

# Tasks (15 min)
/speckit.tasks
# Output: tasks-1.md with 12 tasks (REQ references)

# Implementation (4 hours - SEQUENTIAL)
/speckit.implement
# → Task 1: API route (30 min)
# → Task 2: DB migration (20 min)
# → Task 3: Service layer (40 min)
# ... (9 more tasks sequentially)
# → Step 10.4: Constitutional Reviewer ✅ READY
#   - TDD compliance: Tests before implementation ✅
#   - Simplicity: No new frameworks ✅
#   - Prohibited patterns: None detected ✅

# Reconciliation (15 min)
/speckit.reconcile
# Output: Surgical edits to spec.md (menu entry gap closed)
```

**Total Time**: ~6 hours
**Token Usage**: ~50K tokens
**Deliverables**:
- ✅ Spec with EARS requirements (traceable)
- ✅ Constitutional compliance validated (3 checkpoints)
- ✅ OWASP Top 10 security review
- ✅ Test strategy validated (no mocked DB)
- ✅ Gap closure (append-only edits)

---

### CCPM Workflow (Parallel)

**Timeline**: ~2.5 hours (12 tasks × 12.5 min parallel avg)

```bash
# PRD Creation (20 min)
/pm:prd-new billing-dashboard
# Guided brainstorming → billing-dashboard.md

# Epic Planning (30 min)
/pm:prd-parse billing-dashboard
# Output: epic-billing-dashboard.md (technical architecture)

# Task Decomposition (25 min)
/pm:epic-decompose billing-dashboard
# Output: 12 task files in .claude/epics/billing-dashboard/tasks/
# - task-001-api-routes.md (parallel: true)
# - task-002-db-migration.md (parallel: true)
# ... (10 more tasks)

# GitHub Sync (5 min)
/pm:epic-sync billing-dashboard
# Creates GitHub Issue #123 (epic) + 12 task issues (#124-#135)

# Parallel Execution (1.5 hours - 8 AGENTS)
/pm:issue-start 124  # Agent 1: API routes → Worktree 1
/pm:issue-start 125  # Agent 2: DB migration → Worktree 2
/pm:issue-start 126  # Agent 3: Service layer → Worktree 3
... (5 more agents)

# → All 8 agents work simultaneously
# → Context firewall: Each agent returns 2K summary (not 15K full output)
# → Worktrees prevent merge conflicts

# Progress Tracking (ongoing)
/pm:standup
# Output: 8 tasks in-progress, 3 completed, 1 blocked

# Merge & Close (15 min)
/pm:issue-close 124  # Merge worktree 1
/pm:issue-close 125  # Merge worktree 2
... (10 more)
```

**Total Time**: ~2.5 hours (60% faster than Spec-Kit)
**Token Usage**: ~30K tokens (40% less - context firewall)
**Deliverables**:
- ✅ PRD with business requirements
- ✅ Epic with technical architecture
- ✅ GitHub Issues for team visibility
- ✅ Parallel execution (8 tasks simultaneously)
- ⚠️ No constitutional enforcement
- ⚠️ No OWASP Top 10 validation
- ⚠️ Test strategy not validated (may use mocked DB)

---

### Comparison Summary

| Metric | Spec-Kit | CCPM | Winner |
|--------|----------|------|--------|
| **Time to Complete** | 6 hours | 2.5 hours | **CCPM** (60% faster) |
| **Token Usage** | 50K tokens | 30K tokens | **CCPM** (40% less) |
| **Constitutional Compliance** | ✅ Validated (3 checkpoints) | ❌ Not enforced | **Spec-Kit** |
| **Security Review** | ✅ OWASP Top 10 | ❌ Manual | **Spec-Kit** |
| **Test Strategy Validation** | ✅ Article V compliance | ❌ Not checked | **Spec-Kit** |
| **Team Visibility** | ⚠️ Repo access required | ✅ GitHub Issues (real-time) | **CCPM** |
| **Parallel Execution** | ❌ Sequential | ✅ 8 agents | **CCPM** |
| **Spec Integrity** | ✅ Append-only edits | ⚠️ Manual updates | **Spec-Kit** |

**Verdict**:
- **For Speed + Team Coordination**: CCPM wins (2.5x faster, team visibility)
- **For Quality + Governance**: Spec-Kit wins (constitutional enforcement, security validation)
- **Ideal**: Use Spec-Kit for planning → CCPM for execution → Spec-Kit for reconciliation

---

## Token Economy Comparison

### Spec-Kit Token Profile

**Specification Phase** (10K-15K tokens):
- `/speckit.specify`: 5K-8K (EARS requirements generation)
- `/speckit.validate-gap`: 3K-5K (codebase analysis)
- `/speckit.clarify`: 2K-3K (targeted questions)

**Planning Phase** (10K-15K tokens):
- `/speckit.plan`: 8K-12K (including Phase -1 constitutional validation, Phase 2.5 test strategy, Phase 2.6 validation)
- `/speckit.tasks`: 2K-3K (task breakdown with REQ references)

**Implementation Phase** (20K-30K tokens):
- `/speckit.implement`: 18K-28K (includes Step 10.4 constitutional reviewer)
- **Issue**: Analysis runs inline (verbose output in main thread)

**Reconciliation Phase** (5K-8K tokens):
- `/speckit.reconcile`: 5K-8K (git diff, surgical edits)

**Total**: ~50K tokens per feature

---

### CCPM Token Profile

**PRD Phase** (8K-12K tokens):
- `/pm:prd-new`: 8K-12K (guided brainstorming)

**Epic Phase** (8K-12K tokens):
- `/pm:prd-parse`: 6K-9K (technical architecture)
- `/pm:epic-decompose`: 2K-3K (task breakdown)

**Execution Phase** (12K-18K tokens):
- `/pm:issue-start` × 8: 1.5K-2K per agent (context firewall compression)
- **Key**: Agents return summaries only (80-90% compression)

**Sync Phase** (2K-3K tokens):
- `/pm:issue-sync`: 2K-3K (GitHub API calls, PR creation)

**Total**: ~30K tokens per feature (40% less than Spec-Kit)

---

### Token Efficiency Analysis

**Where CCPM Wins**:
- **Context Firewall**: Agents compress 15K output → 2K summary (87% reduction)
- **Parallel Execution**: No token duplication (independent worktrees)
- **Focused Analysis**: Only essential info returned to main thread

**Where Spec-Kit Wins**:
- **Single Implementation Cycle**: 50-70% savings vs. ad-hoc coding (no rework loops)
- **Gap Analysis**: Prevents late-stage surprises (avoids 20K-50K rework)
- **Constitutional Enforcement**: Catching 1 violation saves 10K-20K tokens

**Net Result**:
- **CCPM**: Lower per-feature cost (30K vs 50K)
- **Spec-Kit**: Lower total project cost (fewer rework cycles)

**Recommendation**: For features <10 tasks → Spec-Kit. For features >10 tasks → CCPM.

---

## Adoption Strategy Matrix

### Maturity vs. Team Size

```
Team Size │
    20+   │                           ✅ CCPM
          │                      (Parallel + GitHub)
    10    │
          │              ⚠️ Hybrid Approach
     5    │         (Spec-Kit planning +
          │          CCPM execution)
     3    │
          │  ✅ Spec-Kit
     1    │  (Constitutional
          │   enforcement)
          └────────────────────────────────────
            Low         Medium         High
                Project Complexity
```

**Quadrant Recommendations**:

1. **Low Complexity + Small Team (1-3 devs)**:
   - ✅ **Spec-Kit** (simple features, constitutional enforcement)

2. **High Complexity + Small Team (1-3 devs)**:
   - ⚠️ **Hybrid** (Spec-Kit for governance, consider parallelism for large modules)

3. **Low Complexity + Large Team (5+ devs)**:
   - ✅ **CCPM** (team visibility, parallel execution)

4. **High Complexity + Large Team (5+ devs)**:
   - ✅ **CCPM** (parallel execution essential, GitHub Issues for coordination)

---

## Migration Path: Spec-Kit → CCPM

If your project **outgrows Spec-Kit** (team expands, need parallelism):

### Step 1: Export Spec-Kit Artifacts

```bash
# Convert spec.md → PRD format
cat specs/000042-billing/spec.md | \
  sed 's/## Functional Requirements/## Requirements/' | \
  sed 's/REQ-\([A-Z]*\)-\([0-9]*\)\.\([0-9]*\)/\1-\2-\3/' \
  > .claude/prds/billing-dashboard.md

# Convert plan-1.md → Epic format
cat specs/000042-billing/plan-1.md | \
  sed 's/## Phase/## Technical Approach/' \
  > .claude/epics/billing-dashboard/epic.md

# Convert tasks-1.md → CCPM task files
# (Manual: split tasks-1.md into individual .md files)
```

### Step 2: Initialize CCPM

```bash
# Sync to GitHub
/pm:epic-sync billing-dashboard
# Creates GitHub Issues from exported artifacts

# Start parallel execution
/pm:issue-start 124
/pm:issue-start 125
... (spawn 5-8 agents)
```

### Step 3: Maintain Constitutional Governance

**Challenge**: CCPM doesn't enforce constitution.

**Solution**: Manual validation workflow:
1. Before `/pm:issue-start`: Review epic against constitution.md
2. During execution: Periodic code reviews (check TDD compliance, simplicity)
3. After `/pm:issue-close`: Run Spec-Kit's `/speckit.reconcile` for gap closure

**Trade-off**: Lose automated enforcement, gain parallel execution.

---

## Migration Path: CCPM → Spec-Kit

If your project **needs governance** (architectural drift, quality issues):

### Step 1: Import CCPM Artifacts

```bash
# Convert PRD → spec.md format
cat .claude/prds/billing-dashboard.md | \
  sed 's/## Requirements/## Functional Requirements/' | \
  sed 's/\([A-Z]*\)-\([0-9]*\)-\([0-9]*\)/REQ-\1-\2.\3/' \
  > specs/000042-billing/spec.md

# Convert Epic → plan.md format
cat .claude/epics/billing-dashboard/epic.md | \
  sed 's/## Technical Approach/## Phase 1: Core Implementation/' \
  > specs/000042-billing/plan-1.md
```

### Step 2: Add Constitutional Infrastructure

```bash
# Create constitution
/speckit.constitution
# Follow prompts to define principles (TDD, simplicity, integration-first)

# Add Phase -1 to plan.md (manual edit)
# Add Phase 2.6 test strategy validation (manual edit)
```

### Step 3: Enable Enforcement

```bash
# Run retrospective validation
/speckit.implement --validate-only
# → Step 10.4 Constitutional Reviewer analyzes existing code
# → Reports violations (mocked DB, missing tests, etc.)

# Close gaps
/speckit.reconcile
# Surgical edits to bring code into constitutional compliance
```

**Trade-off**: Lose parallel execution, gain automated governance.

---

## Strengths & Weaknesses

### Spec-Kit Strengths

✅ **Constitutional Enforcement** (v2.4-v2.6) - Unmatched governance
✅ **Quality Framework** - Risk assessment, OWASP Top 10, UX scoring
✅ **EARS Requirements** - Testable, unambiguous, traceable
✅ **Gap Closure** - Systematic reconciliation with append-only edits
✅ **Zero Dependencies** - Copy/paste install, works everywhere
✅ **Evidence-Based Validation** - 94% hallucination detection
✅ **Single Implementation Cycle** - 50-70% token savings vs. ad-hoc

---

### Spec-Kit Weaknesses

❌ **No Parallel Execution** - Sequential workflow (bottleneck for large features)
❌ **Context Pollution** - Analysis runs inline (15K-30K tokens per command)
❌ **No GitHub Integration** - Local-only (no team visibility)
❌ **No Test Isolation** - Tests run inline (verbose output)
⚠️ **Learning Curve** - 17 commands, complex workflow phases
⚠️ **Smaller Community** - Newer framework (less mature)

---

### CCPM Strengths

✅ **Parallel Execution** (5-12 agents) - 3x faster for multi-module features
✅ **Context Firewall** - 80-90% token compression (agent summaries)
✅ **GitHub Integration** - Issues as database (team visibility)
✅ **Test Isolation** - `/testing:run` spawns agents (clean output)
✅ **Tool Declarations** - Clear frontmatter (maintainability)
✅ **Large Community** - 5.4k stars, proven at scale

---

### CCPM Weaknesses

❌ **No Constitutional Enforcement** - Manual adherence (architectural drift risk)
❌ **No Quality Framework** - No risk scoring, OWASP, UX analysis
❌ **No EARS Requirements** - Natural language (ambiguity risk)
❌ **No Gap Closure Workflow** - Ad-hoc spec updates (integrity risk)
⚠️ **Dependencies** - Requires Git worktrees + `gh` CLI
⚠️ **Complexity** - Parallel model, worktree management (learning curve)

---

## Final Verdict

### Choose Spec-Kit If...

✅ You need **constitutional governance** (TDD, simplicity, integration-first)
✅ You're a **solo developer** or **small team** (1-3 people)
✅ **Quality > speed** (OWASP validation, UX scoring, risk assessment)
✅ You want **zero dependencies** (copy/paste install)
✅ You value **testable requirements** (EARS format)
✅ **Sequential workflow** is acceptable (no parallelism needed)

**Ideal for**: Enterprise features, high-security projects, startups with lean teams

---

### Choose CCPM If...

✅ You need **parallel execution** (5-12 agents, 3x speed)
✅ You have a **distributed team** (3-20 developers)
✅ **Speed > governance** (fast delivery, manual quality control)
✅ You use **GitHub Issues** for project management
✅ You have **large codebases** (100K+ lines, token efficiency critical)
✅ You're comfortable with **Git worktrees** and **gh CLI**

**Ideal for**: Open-source projects, agency teams, startup product teams (5-10 devs)

---

### Hybrid Approach If...

✅ You need **both governance and speed**
✅ You have a **medium-sized team** (3-5 developers)
✅ You can **manually bridge** Spec-Kit planning → CCPM execution
✅ You're willing to **export/import artifacts** between frameworks

**Ideal for**: Growing teams transitioning from solo → collaborative development

---

## Conclusion

**Spec-Kit** and **CCPM** are **complementary, not competitive**:

- **Spec-Kit** = **Constitutional enforcement** + **quality framework** (governance-first)
- **CCPM** = **Parallel execution** + **team coordination** (velocity-first)

**The Future**: A framework that combines **Spec-Kit's governance** with **CCPM's parallelism** would be ideal.

**Recommendation for Spec-Kit v2.7.0+**:
1. Adopt CCPM's **context firewall pattern** (Phase 1 priority)
2. Add CCPM's **test isolation** (`/speckit.test` command)
3. Consider **optional parallel execution** (`/speckit.parallel` - experimental)
4. Keep **constitutional enforcement** (competitive advantage)

**Result**: Best of both worlds - **governance + velocity**.

---

**Report Version**: 1.0
**Date**: 2025-11-11
**Author**: Claude Code Agent
**Source**: CCPM review + spec-kit documentation analysis
