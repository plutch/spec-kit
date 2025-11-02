---
description: Expert panel review system for specifications, plans, and architecture with multiple methodologies and quality scoring.
---

## Overview

`/speckit.review` provides multi-expert analysis of specifications using proven software engineering methodologies. It simulates a panel of domain experts reviewing your work with distinct perspectives and frameworks.

**Adapted from**: [spec-panel.md](../../SuperClaude_Framework/superclaude/commands/spec-panel.md)

## Expert Panel

### Core Experts

| Expert | Specialty | Focus Areas |
|--------|-----------|-------------|
| **Karl Wiegers** | Requirements Engineering | SMART criteria, testability, completeness, traceability |
| **Gojko Adzic** | Specification by Example | Given/When/Then scenarios, concrete examples, acceptance criteria |
| **Martin Fowler** | Architecture & Design | Patterns, refactoring, technical debt, modularity |
| **Michael Nygard** | Production Reliability | Failure modes, resilience patterns, operational concerns |
| **Lisa Crispin** | Quality & Testing | Test strategy, quality metrics, risk-based testing |
| **Alistair Cockburn** | Agile Methodologies | User stories, collaboration patterns, goal-driven design |
| **Sam Newman** | Microservices | Service boundaries, distributed systems, API design |
| **Gregor Hohpe** | Enterprise Integration | Integration patterns, message flows, decoupling strategies |

## Usage

```bash
# Review specification
/speckit.review --mode critique --focus requirements

# Architecture review
/speckit.review --mode discussion --focus architecture

# Interactive learning
/speckit.review --mode socratic

# Quick quality check
/speckit.review --quick
```

## Modes

### 1. Critique Mode (Default)

**Purpose**: Systematic review with improvement recommendations

**Process**:
1. Load document (spec.md or plan.md)
2. Apply expert frameworks
3. Identify issues and anti-patterns
4. Provide specific improvement recommendations
5. Generate quality scores

**Output**:
```markdown
## Expert Review: [FEATURE_NAME]

### Quality Scores
- Clarity: 7/10
- Completeness: 8/10
- Testability: 6/10
- Consistency: 9/10

**Overall**: 7.5/10 (Good, minor improvements needed)

### Findings by Expert

#### Karl Wiegers (Requirements)
🔍 **Findings**:
1. ⚠️ Requirement FR-3 not testable (vague "user-friendly")
2. ✅ Success criteria are SMART and measurable
3. ⚠️ Missing negative/error scenarios

💡 **Recommendations**:
- FR-3: Replace "user-friendly" with "task completion in <2 clicks"
- Add section: "Error Handling Scenarios"
- Define acceptance criteria for edge cases

#### Gojko Adzic (Specification by Example)
🔍 **Findings**:
1. ✅ Good Given/When/Then structure in user scenarios
2. ⚠️ Missing concrete examples for data formats
3. ⚠️ Acceptance criteria lack specific test values

💡 **Recommendations**:
- Add example: "Given user has 50 items, When searching..."
- Specify data ranges: "Price between $0.01 and $999,999.99"
- Include boundary conditions in scenarios

[... other experts ...]

### Priority Actions
1. 🔴 **High**: Fix untestable requirements (FR-3, FR-7)
2. 🟡 **Medium**: Add error handling scenarios
3. 🟢 **Low**: Enhance examples with concrete data
```

### 2. Discussion Mode

**Purpose**: Expert dialogue exploring design tradeoffs

**Process**:
1. Load document
2. Identify key decision points
3. Simulate expert discussion
4. Present alternative perspectives
5. User can ask follow-up questions

**Output**:
```markdown
## Expert Discussion: Authentication Architecture

**Martin Fowler**: "The spec proposes JWT tokens. Let's discuss tradeoffs."

**Michael Nygard**: "JWTs are stateless, which is good for scalability. But
what about token revocation? If a user's permissions change, how do we
invalidate existing tokens?"

**Fowler**: "Good point. The spec doesn't address this. We could:
1. Use short-lived tokens (5-15 min) + refresh tokens
2. Implement a token blacklist (adds state)
3. Use server-side sessions instead"

**Gojko Adzic**: "From a specification perspective, we need concrete scenarios:
- Given a user's role changes, When they access protected resource, Then...?
- Given a user is terminated, When their token is still valid, Then...?"

**Nygard**: "Right. And what's the incident response? If we detect compromise,
can we revoke ALL tokens for a user? The spec should address this."

**Sam Newman**: "Also consider the distributed system angle. If this is
microservices, each service needs to validate tokens. Do we centralize
this? Use API gateway? The architecture decision impacts the spec."

### Emerging Questions for Clarification:
1. How quickly must permission changes take effect?
2. What's the security posture on token revocation?
3. Is this a monolith or microservices architecture?

### Recommended Spec Updates:
- Add security section: Token lifecycle and revocation
- Define RTO (Recovery Time Objective) for security incidents
- Clarify system architecture (impacts auth strategy)
```

### 3. Socratic Mode

**Purpose**: Learning-focused questioning to deepen understanding

**Process**:
1. Expert asks probing questions
2. User answers (simulated or real)
3. Expert guides toward insights
4. Iterative refinement

**Output**:
```markdown
## Socratic Review with Karl Wiegers

**Wiegers**: "Let's examine requirement FR-5: 'System must be fast.'
What does 'fast' mean to your users?"

[User can type response, or continue for simulated learning]

**Wiegers**: "I see. Let me ask: If a user waits 2 seconds vs 200ms,
how does that impact their workflow?"

**Wiegers**: "Exactly. So 'fast' is contextual. For a search feature,
users expect instant results (<500ms). For a report generation, they
might accept 5-10 seconds. Let's make FR-5 specific:

Before: 'System must be fast'
After: 'Search results appear within 500ms for 95th percentile queries'

This is testable, measurable, and tied to user impact. Try this approach
for your other non-functional requirements."
```

## Focus Areas

### Requirements Focus

**Experts**: Wiegers, Adzic, Cockburn

**Criteria**:
- SMART requirements (Specific, Measurable, Achievable, Relevant, Time-bound)
- Testability and verification
- Completeness and coverage
- Traceability to business goals
- Acceptance criteria clarity

### Architecture Focus

**Experts**: Fowler, Newman, Hohpe, Nygard

**Criteria**:
- Design patterns appropriateness
- Modularity and coupling
- Scalability and resilience
- Technical debt awareness
- Operational considerations

### Testing Focus

**Experts**: Crispin, Adzic, Gregory

**Criteria**:
- Test strategy completeness
- Risk-based test prioritization
- Automated vs manual balance
- Test data requirements
- Quality metrics

### Compliance Focus

**Experts**: Wiegers, Nygard

**Criteria**:
- Security requirements
- Regulatory compliance (GDPR, HIPAA, etc.)
- Audit trail requirements
- Data privacy
- Access control

## Quality Scoring

### Scoring Rubric

**Clarity (0-10)**:
- 9-10: Crystal clear, no ambiguity
- 7-8: Mostly clear, minor clarifications needed
- 5-6: Some vagueness, needs refinement
- 3-4: Significant ambiguity
- 0-2: Incomprehensible or contradictory

**Completeness (0-10)**:
- 9-10: All essential elements covered
- 7-8: Core elements present, minor gaps
- 5-6: Significant gaps in coverage
- 3-4: Major sections missing
- 0-2: Skeletal or placeholder content

**Testability (0-10)**:
- 9-10: Every requirement verifiable with clear criteria
- 7-8: Most requirements testable
- 5-6: Some requirements vague
- 3-4: Many requirements untestable
- 0-2: No testable criteria

**Consistency (0-10)**:
- 9-10: No contradictions, aligned throughout
- 7-8: Mostly consistent, minor conflicts
- 5-6: Some contradictions or misalignments
- 3-4: Significant inconsistencies
- 0-2: Contradictory or incoherent

### Overall Score Interpretation

- **9-10**: Excellent - Ready for implementation
- **7-8**: Good - Minor improvements recommended
- **5-6**: Fair - Significant refinement needed
- **3-4**: Poor - Major rework required
- **0-2**: Insufficient - Start over

## Execution Flow

### Wave 1 - Context Load (Parallel)

```yaml
PARALLEL Operations:
  1. Read: specs/[FEATURE]/spec.md
  2. Read: specs/[FEATURE]/plan.md (if exists)
  3. Read: .specify/memory/constitution.md
  4. Read: .specify/memory/features/[FEATURE]/state.json
  5. Grep: "\[NEEDS CLARIFICATION\]" in spec.md
```

### Checkpoint - Validate Context

```yaml
❓ "Documents loaded?"
   → spec.md: ✅/❌
   → plan.md: ✅/❌/Optional
   → constitution: ✅/❌

❓ "Which document to review?"
   → Auto-detect based on args
   → Default: spec.md if plan missing

❓ "Focus area specified?"
   → Parse --focus flag
   → Default: requirements (for spec.md) or architecture (for plan.md)

Confidence: [Calculate]
→ If >= 70%: Proceed
```

### Wave 2 - Expert Analysis

Based on mode and focus, apply expert frameworks:

```yaml
IF mode == "critique":
  → Apply all relevant expert checklists
  → Generate scores
  → Identify issues
  → Recommend fixes

IF mode == "discussion":
  → Identify decision points
  → Simulate expert dialogue
  → Surface alternatives
  → Ask clarifying questions

IF mode == "socratic":
  → Select key weaknesses
  → Generate probing questions
  → Guide toward insights
```

### Wave 3 - State Update

```yaml
PARALLEL:
  - Write: review scores to state.json
  - Write: review notes to specs/[FEATURE]/reviews/[DATE]-review.md
  - Update: next_actions.md with recommended improvements
```

## Command-Line Options

```bash
# Mode selection
--mode critique      # Default: systematic review
--mode discussion    # Expert dialogue
--mode socratic      # Learning questions

# Focus areas
--focus requirements  # Default for spec.md
--focus architecture  # Default for plan.md
--focus testing
--focus compliance

# Quick mode
--quick              # Fast review, top 3 issues only

# Expert selection
--expert wiegers     # Single expert review
--expert fowler
--expert all         # Full panel (default)

# Output control
--verbose            # Detailed explanations
--brief              # Issues only, no examples
```

## Examples

### Example 1: Spec Review

```bash
/speckit.review --mode critique --focus requirements

# Output:
## Quality Scores
- Clarity: 8/10
- Completeness: 7/10
- Testability: 6/10
- Consistency: 9/10

**Overall**: 7.5/10 (Good)

### Top Issues:
1. 🔴 3 requirements not testable
2. 🟡 Missing error handling scenarios
3. 🟢 Add concrete examples to acceptance criteria

[Detailed findings by expert...]
```

### Example 2: Architecture Discussion

```bash
/speckit.review --mode discussion --focus architecture

# Output:
## Expert Discussion: Data Storage Architecture

**Martin Fowler**: The plan proposes PostgreSQL for all data...

**Michael Nygard**: What about read scalability? If we get 10x traffic...

[Expert dialogue continues...]

### Questions for Team:
1. Expected read/write ratio?
2. Caching strategy?
3. Eventual consistency acceptable?
```

### Example 3: Quick Check

```bash
/speckit.review --quick

# Output:
🔍 Quick Review: 000001-auth-system

Top 3 Issues:
1. 🔴 FR-5 untestable: "user-friendly" → specify metrics
2. 🟡 No security section → add auth/authz requirements
3. 🟡 Missing edge cases → define error scenarios

Overall: 6/10 (Fair - needs refinement)

Run full review: /speckit.review --mode critique
```

## Integration with Workflow

```yaml
Recommended Usage:
  After /speckit.specify:
    → /speckit.review --focus requirements --quick

  After /speckit.clarify:
    → /speckit.review --mode critique

  After /speckit.plan:
    → /speckit.review --focus architecture --mode discussion

  Before /speckit.implement:
    → /speckit.review --quick (final check)
```

## State Updates

Review results stored in state.json:

```json
{
  "review_scores": {
    "clarity": 8,
    "completeness": 7,
    "testability": 6,
    "consistency": 9,
    "overall": 7.5,
    "last_reviewed": "2025-01-26T14:30:00Z",
    "reviewer_mode": "critique",
    "top_issues": [
      "3 untestable requirements",
      "Missing error scenarios",
      "Needs concrete examples"
    ]
  }
}
```

## Success Criteria

- ✅ Expert frameworks accurately applied
- ✅ Quality scores consistent and meaningful
- ✅ Recommendations actionable and specific
- ✅ Mode switching works (critique/discussion/socratic)
- ✅ Focus areas properly filtered
- ✅ Integration with state management
- ✅ <1000 tokens for quick mode, <3000 for full critique

---

**Implementation**: Phase 3.5
**Dependencies**: Phase 1.2 (State Management)
**Pattern**: Expert panel simulation with quality metrics
**Token Budget**: 1000-3000 tokens depending on mode
