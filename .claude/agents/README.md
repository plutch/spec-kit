# Spec-Kit Agents

This directory contains specialized review agents for the Spec-Kit framework. These agents are designed to analyze and improve the framework itself, not individual project specifications.

## Available Agents

### workflow-reviewer.md

**Purpose**: Comprehensive workflow coherence and process quality analysis for the Spec-Kit framework

**Persona**: The Process Architect (Cockburn + Gothelf + Kim + Fowler + Nygard)

**When to Use**:
- Reviewing changes to Spec-Kit commands before release
- Analyzing cross-command integration after adding new features
- Validating quality framework enhancements
- Identifying simplification opportunities
- Assessing framework maturity for production readiness

**What It Analyzes**:
1. **Workflow Coherence** - State machine integrity, review gate consistency, command chaining
2. **Developer Experience** - Token economy, session continuity, error handling, cognitive load
3. **Quality Framework** - Risk → Test → Security flow, coverage completeness, gate enforcement
4. **Process Clarity** - Documentation alignment, mode selection, distribution model, simplification
5. **Resilience** - Error recovery, graceful degradation, edge cases, operational robustness

**Output Format**:
- **Framework Coherence Assessment**: 5 dimensional scores (0-10) + overall score
- **Executive Summary**: Findings count, themes, strengths, critical path, quick wins
- **Detailed Findings**: Critical/Major/Medium/Low/Info issues with evidence and recommendations
- **Simplification Opportunities**: Top 5 with complexity reduction percentages
- **Cross-Command Integration Analysis**: Quality flow, state machine, review gate consistency tables
- **Expert Consensus**: Priority recommendations, positive observations, maturity assessment

## How to Use These Agents

### Method 1: Manual Review (Recommended for Deep Analysis)

1. **Read the agent file** to understand the methodology
2. **Follow the multi-pass review process** systematically
3. **Document findings** using the agent's output format
4. **Generate actionable recommendations** with evidence

Example:
```
# In Claude Code
1. Open /home/localdev/work/spec-kit/src/.claude/agents/workflow-reviewer.md
2. Read the 5-pass review process
3. Apply each pass to the framework:
   - Pass 1: Workflow Coherence (Cockburn + Kim)
   - Pass 2: Developer Experience (Gothelf + Kim)
   - Pass 3: Quality Framework (Fowler + Nygard)
   - Pass 4: Process Clarity (Fowler + Cockburn)
   - Pass 5: Resilience (Nygard)
4. Generate findings report
5. Save output to WORKFLOW-COHERENCE-REVIEW-{DATE}.md
```

### Method 2: Automated Review (Using Task Agent)

Use a Claude Code agent to apply the workflow-reviewer methodology:

```
# In Claude Code conversation
"Please perform a workflow coherence review of the spec-kit framework
using the methodology in src/.claude/agents/workflow-reviewer.md.
Focus on [specific aspect: workflow/DX/quality/clarity/resilience]."
```

### Method 3: Focused Review (Specific Command or Integration)

Apply the agent to a subset of the framework:

```
# Review a single command
"Using workflow-reviewer.md methodology, analyze the /speckit.clarify
command for developer experience issues, especially the 5-mode selection."

# Review an integration point
"Using workflow-reviewer.md methodology, validate that the Risk Assessment
in /speckit.specify correctly feeds into Test Strategy in /speckit.plan."
```

## Understanding Agent Output

### Coherence Scores

| Score | Rating | Meaning |
|-------|--------|---------|
| 9-10 | Excellent | Production-ready, minimal improvements needed |
| 8-9 | Advanced | Strong quality, polish and optimization opportunities |
| 7-8 | Good | Solid foundation, some coherence gaps to address |
| 6-7 | Adequate | Functional but needs significant improvements |
| 5-6 | Needs Work | Multiple coherence issues, DX friction |
| 0-5 | Poor | Major overhaul required |

### Issue Severity

- **🔴 CRITICAL**: Blocks framework adoption, causes data loss, state corruption, or workflow deadlock
- **🟠 MAJOR**: Significant DX friction, missing quality dimensions, documentation mismatch
- **🟡 MEDIUM**: Optimization opportunities, simplification potential, minor coherence gaps
- **🟢 LOW**: Polish, minor clarity improvements, documentation enhancements
- **ℹ️ INFO**: Future enhancements, alternative approaches, v2.2+ considerations

### Simplification Metrics

**Complexity Reduction Percentage**:
- Measures how much cognitive load or token usage can be reduced
- Calculated as: `(Current_Complexity - Proposed_Complexity) / Current_Complexity × 100`
- Example: "5 modes → 2 modes" = 60% complexity reduction

**Value Retained Percentage**:
- Measures how much functionality is preserved after simplification
- 80%+ value retention is generally acceptable
- Example: Consolidating clarify modes while keeping core functionality

## Example: Meta-Review Results

The spec-kit framework was reviewed on 2025-11-02 using workflow-reviewer.md:

**Overall Score**: 7.8/10 🟡 (Advanced, Needs Polish)

**Key Findings**:
- 3 Critical issues (review gate inconsistency, PM Agent ambiguity, reconcile validation)
- 8 Major issues (token economy, mode proliferation, quality integration gaps)
- 12 Medium issues (constitution integration, command redundancy, documentation gaps)

**Top Quick Win**:
- **Consolidate Clarify Modes**: 5 modes → 2 modes = 40% complexity reduction, 3 hours effort

**Full Report**: See `WORKFLOW-COHERENCE-META-REVIEW.md` in project root

## Agent Development Guidelines

When creating new agents for Spec-Kit:

### 1. Define Clear Persona
- Use expert composite (2-5 recognized methodologists)
- Examples: Wiegers + Adzic (requirements), Cockburn + Kim (workflow)
- Avoid generic "expert" personas

### 2. Multi-Pass Review Structure
- Minimum 3 passes with distinct perspectives
- Each pass should have specific questions and flag criteria
- Cross-validate findings between passes

### 3. Evidence-Based Analysis
- Require direct quotes from command text
- Cite specific file paths and line numbers
- Show before/after examples for recommendations
- Quantify impact (token reduction, complexity %, time savings)

### 4. Actionable Output
- Every finding must have a specific recommendation
- Include effort estimates (High/Medium/Low)
- Provide implementation examples
- Explain "why this matters" for each issue

### 5. Socratic Validation
- Apply Socratic questioning to validate findings
- Challenge assumptions ("Is this complexity necessary?")
- Justify severity levels with impact analysis

### 6. Cross-Expert Consensus
- Validate critical findings with multiple expert perspectives
- Show disagreements when experts have different views
- Provide synthesis and priority recommendations

## When NOT to Use These Agents

These agents are for **framework-level review**, not for:

❌ Reviewing individual project specifications (use spec-reviewer.md from other projects)
❌ Analyzing user code or implementations
❌ Validating business requirements or functional specs
❌ Testing individual features within a project using Spec-Kit

## Contributing New Agents

To add a new agent to Spec-Kit:

1. **Identify a gap**: What aspect of framework quality isn't covered?
2. **Define persona**: Which experts would analyze this aspect?
3. **Design multi-pass process**: 3-5 passes with distinct perspectives
4. **Create output format**: Scores, findings, recommendations, examples
5. **Test on spec-kit**: Run meta-review to validate agent effectiveness
6. **Document in this README**: Add agent entry with purpose, persona, usage

Example agents to consider:
- **accessibility-reviewer.md**: Validates WCAG 2.1 AA compliance in UX framework
- **security-reviewer.md**: Deep-dive on OWASP Top 10 coverage and threat modeling
- **documentation-reviewer.md**: Validates docs accuracy, completeness, clarity

## Meta-Review Schedule

Recommended frequency for framework reviews:

- **Pre-Release Review**: Before every major/minor version release
- **Post-Feature Review**: After adding new commands or significant enhancements
- **Quarterly Health Check**: Every 3 months for continuous improvement
- **Pre-Production Review**: Before promoting framework to production-ready status

## Questions or Issues?

If you have questions about using these agents or suggestions for improvements:

1. **Framework Issues**: Report at spec-kit repository issues
2. **Agent Methodology**: Reference the expert source material (Cockburn, Fowler, etc.)
3. **Custom Agents**: Follow the contribution guidelines above

---

**Spec-Kit Agents Version**: 1.0.0
**Last Updated**: 2025-11-02
**Maintained By**: Spec-Kit Framework Team
