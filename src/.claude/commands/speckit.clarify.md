---
description: Clarify ambiguities and edge cases through Q&A (default) or multi-perspective expert review (--deep)
scripts:
   sh: .specify/scripts/bash/check-prerequisites.sh --json --paths-only
   ps: .specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

Goal: Detect and reduce ambiguity or missing decision points in the active feature specification and record the clarifications directly in the spec file.

Note: This clarification workflow is expected to run (and be completed) BEFORE invoking `/speckit.plan`. If the user explicitly states they are skipping clarification (e.g., exploratory spike), you may proceed, but must warn that downstream rework risk increases.

## Mode Selection

**Available Modes**:

1. **Standard Mode** (Default): Iterative Q&A with integrated edge case detection
   - Auto-activates Challenge questions for high-risk features (Risk Score ≥8)
   - 5 standard questions + 3-5 Challenge questions (if high-risk)
   - Use for: ALL features (most common)

2. **Expert Lens Mode** (`--expert`, `--deep`, `--review`): Multi-perspective expert review
   - 4-lens analysis: Requirements, Architecture, UX, Resilience
   - Severity-prioritized findings (🔴 CRITICAL, 🟠 MAJOR)
   - WCAG 2.1 AA compliance check
   - Use for: Complex features, regulatory requirements, pre-implementation gate

**Mode Detection**:

```yaml
IF $ARGUMENTS contains "--expert" OR "--deep" OR "--review":
  → Activate Expert Lens Mode (multi-expert review)
  → Follow Expert Lens execution steps
ELSE:
  → Standard Mode (iterative Q&A + edge case detection)
  → Auto-detect high-risk features and add Challenge questions
  → Follow standard execution steps
```

## Usage Examples

```bash
# Standard clarification (auto-detects high-risk features)
/speckit.clarify

# Comprehensive expert review (requirements, architecture, UX, resilience)
/speckit.clarify --expert
```

**When to Use Each Mode**:

- **Standard Mode** (default): Use for ALL features
  - Identifies ambiguities through iterative Q&A (max 5 questions)
  - Scans for boundary conditions, error handling, state transitions
  - **Auto-activates Challenge questions** for high-risk features (Risk Score ≥8)
  - Best for: Regular clarification needs, most features (90%+ of use cases)

- **Expert Lens Mode** (`--expert`): Use for complex or interface-heavy features
  - 4-lens review: Requirements (Wiegers), Architecture (Fowler/Cockburn), UX (Nielsen/Norman), Resilience (Nygard)
  - Comprehensive quality assessment including WCAG 2.1 AA accessibility
  - Severity-prioritized findings (🔴 CRITICAL, 🟠 MAJOR, 🟡 MEDIUM)
  - Best for: Complex workflows, regulatory requirements, pre-implementation quality gate

**Deprecated Modes** (v2.1.1):
- `--challenge`: Merged into Standard Mode with auto-activation for high-risk features
- `--edge-cases`: Merged into Standard Mode (always active)
- `--ux`: Use `/speckit.analyze-ux` instead for UX-specific analysis

---

## Context Loading (v2.7 - Context Optimization)

**Purpose**: Load relevant memory content for the clarification phase, optimized for token efficiency.

**Current Phase**: `specification` (clarification is part of specification refinement)

**Context Loading Strategy**:

a. **Read Configuration** (if exists):
   - Check for `.specify/config.yml` or `.specify/config.example.yml`
   - Extract `context_budget.specification` (default: 15KB)
   - Extract `memory.strict_phase_loading` (default: true)
   - Extract `budget_enforcement.mode` (default: strict)

b. **Load Memory Files** (phase-aware):

   For each memory file in `.specify/memory/`:

   i. **Read YAML Frontmatter**:
      - Extract `inclusion_mode`, `context_level`, `load_phases`, `exclude_phases`
      - If metadata missing: Default to `inclusion_mode: always, context_level: strategic, load_phases: all`

   ii. **Determine if file should be loaded**:
      ```yaml
      IF current_phase IN exclude_phases:
        → SKIP this file entirely

      ELSE IF inclusion_mode = "always":
        → Load strategic sections only (constitution core principles)

      ELSE IF inclusion_mode = "conditional":
        IF current_phase IN load_phases OR "specification" IN load_phases:
          → Load strategic sections (e.g., requirements patterns, domain knowledge)
        ELSE:
          → SKIP (e.g., tactical implementation details excluded)

      ELSE IF inclusion_mode = "manual":
        → SKIP (manual loading via @filename only)
      ```

   iii. **Filter Sections** (if loading file):
      - Scan for `<!-- SECTION_META: context_level=X, load_phases=Y -->` comments
      - For specification/clarification phase: Load sections where `specification` IN load_phases
      - Load strategic sections (principles, patterns, domain knowledge)
      - Skip tactical sections (code examples, procedures)

   iv. **Track Budget**:
      - Sum loaded content size
      - If exceeds `context_budget.specification` (15KB):
        - **strict mode**: Warn user, prioritize core strategic content
        - **warn mode**: Continue loading, warn about budget exceeded
        - **adaptive mode**: Intelligently filter content to fit budget

c. **Context Loading Report** (verbose mode):

   IF `verbose_context_loading: true` in config:
   ```markdown
   📊 Context Loading Report

   Phase: specification (clarification)
   Budget: 15KB

   Loaded:
   - constitution.md (strategic: core principles): 3.2KB
   - domain-patterns.md (strategic: requirements patterns): 2.5KB
   - Skipped: api-standards.md (excluded phase: specification)
   - Skipped: testing-approach.md (tactical: code examples not needed)
   - Skipped: deployment-runbook.md (manual only)

   Total Loaded: 5.7KB / 15KB (38% budget used)
   Remaining: 9.3KB available

   Status: ✅ Within budget
   ```

d. **Backward Compatibility**:
   - If `.specify/memory/` doesn't exist → Skip context loading (no error)
   - If memory files lack metadata → Load all content (v2.6 behavior)
   - Never fail command due to missing memory files

**Expected Token Savings**: ~60% (20-30KB → 8-12KB for clarification phase)

**Key Difference from Implementation Phase**:
- Clarification loads STRATEGIC content (principles, patterns, domain knowledge)
- Implementation loads TACTICAL content (code examples, procedures)
- This prevents showing code examples during requirements clarification (wasted tokens)

---

### Expert Lens Mode Execution (Multi-Perspective Review)

**Purpose**: Apply systematic multi-expert review methodology to identify gaps across requirements quality, architecture, UX/usability, and production resilience dimensions.

**When to Use Expert Lens Mode**:
- Complex features requiring high quality standards
- Features with regulatory, security, or compliance requirements
- Interface-heavy features with significant UX impact
- After significant spec revisions
- Before committing to implementation planning
- When comprehensive quality assessment is needed

**Expert Lens Steps**:

1. Run `{SCRIPT}` to load FEATURE_SPEC (same as normal mode)

2. **Execute Multi-Expert Review** across four lenses:

   **Lens 1: Requirements Quality (Karl Wiegers)**
   Focus: SMART criteria, measurability, stakeholder validation, testability

   Review questions:
   - "How would you validate compliance in production?"
   - "Are acceptance criteria specific and measurable?"
   - "Can automated tests verify the requirement?"
   - "What stakeholder needs are unaddressed?"
   - "Can this be interpreted multiple ways?"
   - "Can you provide concrete examples demonstrating this?"
   - "How would the testing team validate this requirement?"
   - "Are boundary conditions tested (empty, zero, max, null)?"

   Flag when:
   - Requirements use vague terms ("gracefully", "efficiently")
   - No quantifiable acceptance criteria
   - Ambiguous stakeholder or scope
   - Missing validation criteria
   - Generic scenarios without specific values
   - Missing error handling scenarios
   - No boundary condition testing
   - Untestable requirements (no observable behavior)

   **Lens 2: Architecture & Design (Martin Fowler + Alistair Cockburn)**
   Focus: Goal alignment, design coherence, SOLID principles, simplification

   Review questions:
   - "Who is the primary stakeholder and what goal are they achieving?"
   - "Does this requirement directly support a business goal?"
   - "What breaks if we remove this requirement?"
   - "Does this violate separation of concerns or SOLID principles?"
   - "Can we achieve the same outcome more simply?"
   - "Are design patterns used appropriately?"
   - "Is this over-engineered for the problem at hand?"

   Flag when:
   - Requirements lack clear business justification
   - Overcomplicated solutions for simple problems
   - Contradictions with established patterns
   - Violations of design principles (SOLID, DRY, YAGNI)
   - Inconsistent handling of similar scenarios
   - Premature abstraction or gold-plating

   **Lens 3: UX & Usability (Nielsen + Norman + Cooper + Wroblewski)**
   Focus: Usability heuristics, cognitive load, task efficiency, accessibility

   Review questions:
   - **Nielsen's Heuristics**: System status visibility, error prevention, recognition over recall, consistency?
   - **Cognitive Load**: Can users complete tasks without memorizing state?
   - **Task Efficiency**: Are primary tasks achievable in ≤3 clicks? Smart defaults provided?
   - **Affordances**: Do interactive elements look clickable/tappable?
   - **Form Design**: Mobile-optimized keyboards? Inline validation? Clear labels?
   - **Touch Targets**: Buttons ≥44px on mobile? Primary actions in thumb zones?
   - **Accessibility**: Keyboard navigation? Screen reader support? Color contrast (4.5:1)? WCAG 2.1 AA?
   - **Error Recovery**: Can users fix errors inline without losing data?

   Flag when:
   - Critical actions lack confirmation dialogs
   - System status invisible (loading states missing)
   - Users must remember information across screens
   - Forms have >7 fields without grouping
   - Touch targets <44px on mobile
   - Color is only indicator (no text/icon)
   - No keyboard shortcuts or tab order
   - Error messages blame user
   - Workflows optimized for edge cases, not common tasks
   - No defaults for frequently-set fields

   **Lens 4: Production Resilience (Michael Nygard)**
   Focus: Failure modes, operational concerns, resilience patterns, scalability

   Review questions:
   - "What happens when this component fails?"
   - "Where are timeout specifications?"
   - "How does this behave under load (10x, 100x scale)?"
   - "What are the retry and fallback mechanisms?"
   - "How will this be monitored in production?"
   - "Are there race conditions or concurrency issues?"
   - "What about error recovery and graceful degradation?"

   Flag when:
   - No error handling for external dependencies
   - Missing timeout specifications
   - No recovery or fallback mechanisms
   - Missing operational monitoring requirements
   - No consideration of distributed system challenges
   - No scale/performance characteristics defined
   - No handling of concurrent access

3. **Synthesize Findings**: For each lens, identify top 2-3 issues (max 10 total across all lenses)

4. **Prioritize by Severity**:
   - 🔴 CRITICAL: Security, data loss, compliance, payment integrity
   - 🟠 MAJOR: Missing error handling, UX issues, untestable requirements
   - 🟡 MEDIUM: Edge cases, tech debt, documentation gaps
   - 🟢 LOW: Optimizations, simplification suggestions
   - ℹ️ INFO: Observations, pattern recommendations

5. **Generate Expert Review Questions** (3-5 questions, prioritized by severity):

   ```markdown
   ## Expert Review Question [N]: [Title]

   **Expert Lens**: [Wiegers/Adzic/Nygard/Fowler] - [Focus Area]
   **Severity**: [🔴 CRITICAL | 🟠 MAJOR | 🟡 MEDIUM | 🟢 LOW | ℹ️ INFO]
   **Location**: [Section reference from spec]

   **Issue**:
   > [Quote relevant spec section]

   [Specific description of the gap or problem]

   **Socratic Validation** (Evidence-Based Verification):
   - "What user goal is blocked or delayed?" → [Specific workflow/task affected by this gap]
   - "How many users are affected?" → [All users, specific cohort, % estimate with rationale]
   - "What is the cost of this issue?" → [Time waste, revenue impact, support cost, compliance risk, data loss]
   - "Why this severity level?" → [Justification by frequency × impact × criticality to business]

   **Why This Matters**:
   - **User Impact**: [How does this affect end users?]
   - **Technical Risk**: [What could break or go wrong?]
   - **Business Impact**: [Cost, compliance, reputation implications]

   **Question**: [Specific clarification needed]

   **Suggested Answers**:

   | Option | Answer | Implications |
   |--------|--------|--------------|
   | A      | [First option] | [Impact and tradeoffs] |
   | B      | [Second option] | [Impact and tradeoffs] |
   | C      | [Third option] | [Impact and tradeoffs] |
   | Custom | Provide your own answer | [How to provide custom input] |

   **Your choice**: _[Wait for user response]_
   ```

   **Socratic Filter for Expert Questions**: Before including any finding, verify all 4 Socratic questions can be answered with specific, evidence-based responses. If any question cannot be answered concretely, the finding may be a false positive or low-impact issue.

6. **Wait for User Responses**: Collect answers for all questions

7. **Identify Cross-Lens Conflicts**: After collecting answers, detect tensions where recommendations from different lenses conflict

   **Purpose**: Reveal tradeoffs where optimizing for one quality dimension may negatively impact another, requiring conscious decision-making.

   **Conflict Detection Logic**:
   - Compare recommendations across all 4 lenses
   - Identify where Lens A suggests approach X, but Lens B suggests contradictory approach Y
   - Common conflict patterns:
     - Requirements clarity vs. Architecture simplicity (detailed specs vs. emergent design)
     - UX efficiency vs. Architecture performance (fewer clicks vs. system load)
     - Production resilience vs. UX simplicity (error handling complexity vs. user confusion)
     - Architecture patterns vs. Requirements constraints (ideal design vs. business rules)

   **Output Format for Each Conflict**:
   ```markdown
   ## Cross-Lens Conflict #[N]: [Descriptive Title]

   **Lens A** ([Expert Name] - [Focus Area]): "[Recommendation from Lens A]"
   **Lens B** ([Expert Name] - [Focus Area]): "[Conflicting recommendation from Lens B]"

   **Tension**: [What's the fundamental conflict? What cannot be optimized simultaneously?]
   **Affected**: [Which workflows/components/requirements are impacted?]

   **Tradeoff Analysis**:
   - **Lens A Perspective**: [Benefits + costs of this approach]
   - **Lens B Perspective**: [Benefits + costs of alternative approach]

   **Resolution Options**:
   - **Option A**: [Compromise approach that balances both]
     - Pros: [Advantages]
     - Cons: [Disadvantages]
   - **Option B**: [Alternative approach]
     - Pros: [Advantages]
     - Cons: [Disadvantages]
   - **Option C**: [Another alternative if applicable]
     - Pros: [Advantages]
     - Cons: [Disadvantages]

   **Recommended Resolution**: Option [X]
   **Rationale**: [Why this balances both perspectives best, with evidence]

   **Question for User**: Which resolution do you prefer? [A/B/C/Custom]
   ```

   **Note**: Not all expert reviews will have cross-lens conflicts. Only document when genuine tensions exist between expert recommendations. Typical range: 0-3 conflicts per feature.

8. **Update Spec**: Apply answers and improvements to spec.md, documenting expert review findings AND cross-lens conflict resolutions

9. **Output Expert Review Summary**:
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   🔍 Expert Review Complete
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Expert Lenses Applied:
   ✅ Requirements Quality (Wiegers) - measurability, testability
   ✅ Architecture & Design (Fowler/Cockburn) - SOLID, simplification
   ✅ UX & Usability (Nielsen/Norman/Cooper) - heuristics, accessibility
   ✅ Production Resilience (Nygard) - failure modes, scalability

   Findings: [N] issues ([X] critical, [Y] major, [Z] minor)
   Questions Asked: [N]
   Cross-Lens Conflicts Identified: [M] (requiring tradeoff decisions)
   Spec Updates: [N] sections improved

   Quality Impact:
   - Requirements Clarity: +[X]% improvement
   - Design Simplification: +[Y]% improvement
   - UX Quality: +[Z]% improvement (WCAG compliance assessed)
   - Production Readiness: +[W]% improvement

   Tradeoff Decisions Made: [M] cross-lens conflicts resolved

   Status: ✅ Ready for Planning
   ```

**Benefits of Expert Lens Mode**:
- Systematic quality assessment across requirements, architecture, UX, and resilience
- Evidence-based findings with clear impact
- Prioritized issues by severity
- Multi-expert validation prevents blind spots
- Usability and accessibility (WCAG 2.1 AA) validation included
- Actionable recommendations with examples
- Comprehensive coverage of quality aspects (technical + user-facing)

---

### Standard Execution Steps (Normal Mode with Edge Case Detection)

**Purpose**: Identify and clarify ambiguities in the specification through targeted Q&A, with integrated edge case detection to ensure production resilience.

**Enhancement**: Standard Mode now automatically includes edge case detection (previously a separate mode). Every clarification session systematically scans for boundary conditions, error handling, and state transitions.

Execution steps:

1. Run `{SCRIPT}` from repo root **once** (combined `--json --paths-only` mode / `-Json -PathsOnly`). Parse minimal JSON payload fields:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - (Optionally capture `IMPL_PLAN`, `TASKS` for future chained flows.)
   - If JSON parsing fails, abort and instruct user to re-run `/speckit.specify` or verify feature branch environment.
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. Load the current spec file. Perform a structured ambiguity & coverage scan using this taxonomy. For each category, mark status: Clear / Partial / Missing. Produce an internal coverage map used for prioritization (do not output raw map unless no questions will be asked).

2.5. **Auto-Detect High-Risk Features (Challenge Mode Integration)**

   **Purpose**: Automatically activate adversarial Challenge questions for high-risk features to surface hidden risks and validate assumptions.

   **Risk Detection**:

   Check if spec.md contains a "## Risk Assessment" section:

   ```bash
   # Pseudo-code for risk detection logic
   if grep -q "## Risk Assessment" "${FEATURE_SPEC}"; then
     # Extract risk scores (look for "Score: X/12" or "Score: X")
     high_risk_criteria=$(grep -E "Score: ([8-9]|1[0-2])" "${FEATURE_SPEC}" | wc -l)

     if [ "$high_risk_criteria" -gt 0 ]; then
       echo "⚠️ HIGH-RISK FEATURE DETECTED"
       echo "Risk criteria with score ≥8: ${high_risk_criteria}"
       echo "Auto-activating Challenge Mode questions for extra scrutiny"
       CHALLENGE_MODE_ACTIVE=true
     else
       echo "Standard risk profile detected"
       CHALLENGE_MODE_ACTIVE=false
     fi
   else
     echo "No Risk Assessment found - using Standard Mode only"
     CHALLENGE_MODE_ACTIVE=false
   fi
   ```

   **IF CHALLENGE_MODE_ACTIVE = true**:

   After standard Q&A questions (max 5), generate 3-5 **Challenge Questions** (adversarial stress-testing):

   **Challenge Question Categories**:

   a. **Assumption Challenge**:
      ```
      ❓ Assumption Validation (Challenge)

      The spec assumes [stated assumption from high-risk area]. But have we validated this?

      Challenge: [Why this assumption might be wrong in high-risk context]
      Alternative: [What if the opposite is true]
      Question: How confident are we? What's the evidence?

      Options:
      A) High confidence - validated with users/data
      B) Medium confidence - based on best practices
      C) Low confidence - educated guess
      D) Should validate before proceeding
      ```

   b. **Alternative Approach**:
      ```
      ❓ Alternative Approach (Challenge)

      The spec proposes [approach from high-risk requirement]. But are there safer alternatives?

      Challenge: [What could go wrong with current approach]
      Alternative: [Lower-risk approach, trade-offs]
      Question: Have we considered [safer alternative]? Why not?

      Options:
      A) Current approach is safest
      B) Alternative is worth considering
      C) Should prototype both approaches
      D) Risk mitigation strategy needed
      ```

   c. **Scope Challenge**:
      ```
      ❓ Scope Validation (Challenge)

      The spec includes [high-risk feature]. Is this scope justified given the risk?

      Challenge: [Could we ship without this high-risk element]
      Question: What's the minimum viable version that reduces risk?

      Options:
      A) Full scope necessary for MVP
      B) Could phase implementation
      C) Should descope high-risk parts
      D) Need risk mitigation controls
      ```

   **Output**: Challenge Q&A section appended to standard clarifications in spec.md

   **Note**: Challenge questions are ADDITIONAL to standard questions. Standard questions focus on ambiguities; Challenge questions focus on validating high-risk assumptions.

   Functional Scope & Behavior:
   - Core user goals & success criteria
   - Explicit out-of-scope declarations
   - User roles / personas differentiation

   Domain & Data Model:
   - Entities, attributes, relationships
   - Identity & uniqueness rules
   - Lifecycle/state transitions
   - Data volume / scale assumptions

   Interaction & UX Flow:
   - Critical user journeys / sequences
   - Error/empty/loading states
   - Accessibility or localization notes

   Non-Functional Quality Attributes:
   - Performance (latency, throughput targets)
   - Scalability (horizontal/vertical, limits)
   - Reliability & availability (uptime, recovery expectations)
   - Observability (logging, metrics, tracing signals)
   - Security & privacy (authN/Z, data protection, threat assumptions)
   - Compliance / regulatory constraints (if any)

   Integration & External Dependencies:
   - External services/APIs and failure modes
   - Data import/export formats
   - Protocol/versioning assumptions

   Edge Cases & Failure Handling (ENHANCED):
   - **Boundary Conditions**: Empty/zero/null handling, max/min values, scale limits
   - **Error Conditions**: External failures (timeout, network), validation failures, authorization failures, resource exhaustion
   - **State Transitions**: Invalid transitions, terminal states, crash recovery, idempotency
   - **Data Quality**: Duplicate data, orphaned data, stale data, inconsistent data, corrupted data
   - **Race Conditions**: Read-modify-write races, TOCTOU, double processing, distributed locks, eventual consistency
   - **Multi-Tenant Isolation**: Cross-tenant leakage, quota bypass, tenant context loss (if applicable)
   - Negative scenarios
   - Rate limiting / throttling
   - Conflict resolution (e.g., concurrent edits)

   Constraints & Tradeoffs:
   - Technical constraints (language, storage, hosting)
   - Explicit tradeoffs or rejected alternatives

   Terminology & Consistency:
   - Canonical glossary terms
   - Avoided synonyms / deprecated terms

   Completion Signals:
   - Acceptance criteria testability
   - Measurable Definition of Done style indicators

   Misc / Placeholders:
   - TODO markers / unresolved decisions
   - Ambiguous adjectives ("robust", "intuitive") lacking quantification

   For each category with Partial or Missing status, add a candidate question opportunity unless:
   - Clarification would not materially change implementation or validation strategy
   - Information is better deferred to planning phase (note internally)

   **Edge Case Detection Priority**: When generating questions, prioritize edge case categories (Boundary, Error, State, Data Quality, Race Conditions, Multi-Tenant) for high-risk features (payment, security, financial, multi-user collaboration, external integrations).

3. Generate (internally) a prioritized queue of candidate clarification questions (maximum 5). Do NOT output them all at once. Apply these constraints:
    - Maximum of 10 total questions across the whole session.
    - Each question must be answerable with EITHER:
       - A short multiple‑choice selection (2–5 distinct, mutually exclusive options), OR
       - A one-word / short‑phrase answer (explicitly constrain: "Answer in <=5 words").
    - Only include questions whose answers materially impact architecture, data modeling, task decomposition, test design, UX behavior, operational readiness, or compliance validation.
    - Ensure category coverage balance: attempt to cover the highest impact unresolved categories first; avoid asking two low-impact questions when a single high-impact area (e.g., security posture) is unresolved.
    - Exclude questions already answered, trivial stylistic preferences, or plan-level execution details (unless blocking correctness).
    - Favor clarifications that reduce downstream rework risk or prevent misaligned acceptance tests.
    - If more than 5 categories remain unresolved, select the top 5 by (Impact * Uncertainty) heuristic.

   **ENHANCEMENT: Context-Aware Question Selection** (Smart Questioning):

   For EACH ambiguity identified, generate question VARIANTS from different perspectives, then select the BEST one based on context:

   **Question Perspectives**:

   - **Technical** (How/What): Specific implementation details, numbers, formats
     - Example: "What's the target API response time? (<100ms / <500ms / <1s / <5s)"
     - Best when: Spec mentions technical requirements but lacks specifics

   - **User** (Who/Why): User priorities, workflows, experience impacts
     - Example: "Which user workflows need the fastest response time? (search/filter / data export / all operations)"
     - Best when: Multiple use cases exist with potentially different needs

   - **Business** (Value/Tradeoff): Cost-benefit analysis, priority decisions, ROI
     - Example: "What's the user impact of 500ms vs 2s response time? (critical / moderate / minimal)"
     - Best when: Trade-offs exist between complexity and value

   - **Risk** (What-if): Scale considerations, failure modes, edge cases
     - Example: "What happens if API response time degrades under 10x load? (acceptable / degraded UX / critical failure)"
     - Best when: Scalability or reliability concerns exist

   **Context-Based Selection Algorithm**:

   For each ambiguity, analyze the spec context to choose the best perspective:

   ```yaml
   IF spec mentions "real-time", "critical path", or "performance-sensitive":
     → Choose Technical perspective (need specific numbers)
     → Rationale: Precision required for technical implementation

   ELSE IF spec has vague priorities ("fast", "good UX", "responsive"):
     → Choose User perspective (understand what matters most)
     → Rationale: Priorities unclear, need user-centric clarification

   ELSE IF spec has competing concerns (speed vs cost, features vs time):
     → Choose Business perspective (understand trade-offs)
     → Rationale: Decision requires cost-benefit analysis

   ELSE IF spec lacks error handling, scale considerations, or edge cases:
     → Choose Risk perspective (surface hidden issues)
     → Rationale: Resilience and scale not addressed

   ELSE (default):
     → Choose Technical perspective (most direct clarification)
   ```

   **Example: Performance Ambiguity**

   Ambiguity detected: "The API should respond quickly"

   Generated variants:
   - Technical: "What's the target response time? (<100ms / <500ms / <1s / <5s)"
   - User: "Which operations need fastest response? (search / export / all)"
   - Business: "What's the user impact of slower response? (critical / moderate / minimal)"
   - Risk: "What happens if response degrades under load?"

   Context analysis:
   - Spec mentions "search functionality" and "user experience"
   - No "real-time" or "critical" keywords
   - Multiple user workflows described

   Selected: **User perspective**
   Rationale: Multiple workflows exist, need to understand which ones prioritize speed

   Final question:
   ```
   ❓ Performance Priorities (User Perspective)

   Which user workflows require the fastest API response times?

   This helps prioritize optimization efforts and set appropriate targets for different operations.

   **Recommended:** Search/filter operations (typically interactive, users expect <500ms)

   | Option | Description |
   |--------|-------------|
   | A | Search and filter operations (interactive queries) |
   | B | Data export and report generation (background operations) |
   | C | All operations equally (uniform performance target) |
   | D | Real-time dashboard updates (continuous data flow) |
   | Short | Specify other workflows (<=5 words) |

   You can reply with the option letter, accept the recommendation ("yes"), or specify your own answer.
   ```

   **Question Annotation Format**:

   When presenting questions, include perspective label to help user understand the question's purpose:

   - `❓ [Topic] (Technical)` - For implementation specifics
   - `❓ [Topic] (User Perspective)` - For user priorities and workflows
   - `❓ [Topic] (Business Perspective)` - For value and trade-off decisions
   - `❓ [Topic] (Risk Perspective)` - For scale and failure considerations

   **Benefits of Context-Aware Selection**:
   - Same 5-question limit (no workflow change)
   - Questions reveal deeper insights (not just technical gaps)
   - Smart selection based on context (not random perspective choice)
   - Annotations help user understand WHY each question matters
   - Reduces follow-up clarification rounds (get it right first time)

4. Sequential questioning loop (interactive):
    - Present EXACTLY ONE question at a time.
    - For multiple‑choice questions:
       - **Analyze all options** and determine the **most suitable option** based on:
          - Best practices for the project type
          - Common patterns in similar implementations
          - Risk reduction (security, performance, maintainability)
          - Alignment with any explicit project goals or constraints visible in the spec
       - Present your **recommended option prominently** at the top with clear reasoning (1-2 sentences explaining why this is the best choice).
       - Format as: `**Recommended:** Option [X] - <reasoning>`
       - Then render all options as a Markdown table:

       | Option | Description |
       |--------|-------------|
       | A | <Option A description> |
       | B | <Option B description> |
       | C | <Option C description> (add D/E as needed up to 5) |
       | Short | Provide a different short answer (<=5 words) (Include only if free-form alternative is appropriate) |

       - After the table, add: `You can reply with the option letter (e.g., "A"), accept the recommendation by saying "yes" or "recommended", or provide your own short answer.`
    - For short‑answer style (no meaningful discrete options):
       - Provide your **suggested answer** based on best practices and context.
       - Format as: `**Suggested:** <your proposed answer> - <brief reasoning>`
       - Then output: `Format: Short answer (<=5 words). You can accept the suggestion by saying "yes" or "suggested", or provide your own answer.`
    - After the user answers:
       - If the user replies with "yes", "recommended", or "suggested", use your previously stated recommendation/suggestion as the answer.
       - Otherwise, validate the answer maps to one option or fits the <=5 word constraint.
       - If ambiguous, ask for a quick disambiguation (count still belongs to same question; do not advance).
       - Once satisfactory, record it in working memory (do not yet write to disk) and move to the next queued question.
    - Stop asking further questions when:
       - All critical ambiguities resolved early (remaining queued items become unnecessary), OR
       - User signals completion ("done", "good", "no more"), OR
       - You reach 5 asked questions.
    - Never reveal future queued questions in advance.
    - If no valid questions exist at start, immediately report no critical ambiguities.

5. Integration after EACH accepted answer (incremental update approach):
    - Maintain in-memory representation of the spec (loaded once at start) plus the raw file contents.
    - For the first integrated answer in this session:
       - Ensure a `## Clarifications` section exists (create it just after the highest-level contextual/overview section per the spec template if missing).
       - Under it, create (if not present) a `### Session YYYY-MM-DD` subheading for today.
    - Append a bullet line immediately after acceptance: `- Q: <question> → A: <final answer>`.
    - Then immediately apply the clarification to the most appropriate section(s):
       - Functional ambiguity → Update or add a bullet in Functional Requirements.
       - User interaction / actor distinction → Update User Stories or Actors subsection (if present) with clarified role, constraint, or scenario.
       - Data shape / entities → Update Data Model (add fields, types, relationships) preserving ordering; note added constraints succinctly.
       - Non-functional constraint → Add/modify measurable criteria in Non-Functional / Quality Attributes section (convert vague adjective to metric or explicit target).
       - Edge case / negative flow → Add a new bullet under Edge Cases / Error Handling (or create such subsection if template provides placeholder for it).
       - Terminology conflict → Normalize term across spec; retain original only if necessary by adding `(formerly referred to as "X")` once.
    - If the clarification invalidates an earlier ambiguous statement, replace that statement instead of duplicating; leave no obsolete contradictory text.
    - Save the spec file AFTER each integration to minimize risk of context loss (atomic overwrite).
    - Preserve formatting: do not reorder unrelated sections; keep heading hierarchy intact.
    - Keep each inserted clarification minimal and testable (avoid narrative drift).

6. Validation (performed after EACH write plus final pass):
   - Clarifications session contains exactly one bullet per accepted answer (no duplicates).
   - Total asked (accepted) questions ≤ 5.
   - Updated sections contain no lingering vague placeholders the new answer was meant to resolve.
   - No contradictory earlier statement remains (scan for now-invalid alternative choices removed).
   - Markdown structure valid; only allowed new headings: `## Clarifications`, `### Session YYYY-MM-DD`.
   - Terminology consistency: same canonical term used across all updated sections.

7. Write the updated spec back to `FEATURE_SPEC`.

8. Report completion (after questioning loop ends or early termination):
   - Number of questions asked & answered.
   - Path to updated spec.
   - Sections touched (list names).
   - Coverage summary table listing each taxonomy category with Status: Resolved (was Partial/Missing and addressed), Deferred (exceeds question quota or better suited for planning), Clear (already sufficient), Outstanding (still Partial/Missing but low impact).
   - If any Outstanding or Deferred remain, recommend whether to proceed to `/speckit.plan` or run `/speckit.clarify` again later post-plan.
   - Suggested next command.

9. **Clarification Review Gate** (Evidence-Based Self-Check):

   **Purpose**: Validate clarification effectiveness and assess readiness to proceed to planning phase.

   **MANDATORY: The Four Clarification Questions** (MUST answer with ACTUAL evidence):

   ❓ **"Were all critical ambiguities addressed?"**
      ```yaml
      Action Required:
        - Review coverage summary table from Step 8
        - Count: Resolved vs Outstanding vs Deferred categories
        - IF any category remains "Outstanding" with HIGH impact: NOT ready ❌
        - Show ACTUAL category status counts, not summaries

      Expected Evidence Format:
        Coverage Status:
        ✓ Resolved: [X] categories (was Partial/Missing, now Clear)
        ⚠️ Deferred: [Y] categories (better suited for planning phase)
        ✓ Clear: [Z] categories (already sufficient)
        ❌ Outstanding: [N] categories (still Partial/Missing)

        Outstanding Categories (if any):
        - [Category Name]: [Impact Level] - [Why still unresolved]

      Readiness Determination:
        IF Outstanding = 0 AND (Resolved + Clear) > 80% of taxonomy:
          → ✅ READY to proceed to planning
        IF Outstanding ≤ 2 AND all Outstanding are LOW impact:
          → ⚠️ NEEDS REVIEW (with noted risks)
        IF Outstanding > 2 OR any HIGH impact Outstanding:
          → ❌ NOT READY (recommend another clarify pass)
      ```

   ❓ **"Are clarifications properly integrated into spec?"**
      ```yaml
      Action Required:
        - Verify ## Clarifications section exists with Session YYYY-MM-DD
        - Confirm all [X] Q&A bullets recorded (count from Step 4)
        - Check each answer integrated into appropriate spec section
        - Validate no contradictory statements remain

      Expected Evidence Format:
        Integration Status:
        ✓ Clarifications section: [Present/Missing]
        ✓ Q&A bullets recorded: [X]/[X] (all questions answered)
        ✓ Sections updated: [list section names]
        ✓ Contradictions removed: [Yes/No - if No, list them]

      Validation Checklist:
        - [ ] Each accepted answer appears in Clarifications section
        - [ ] Each answer also integrated into relevant spec section
        - [ ] No obsolete/conflicting text remains
        - [ ] Terminology normalized across all updated sections
        - [ ] Markdown structure valid (no broken headings)

      IF any checklist item fails:
        → ❌ Integration incomplete
        → Fix integration issues before proceeding
      ```

   ❓ **"Did clarifications improve spec measurability?"**
      ```yaml
      Action Required:
        - Compare spec BEFORE vs AFTER clarification
        - Count vague adjectives remaining ("robust", "intuitive", "fast")
        - Check if [NEEDS CLARIFICATION] markers reduced
        - Validate acceptance criteria are more testable

      Expected Evidence Format:
        Measurability Improvements:
        Before: [N] vague adjectives, [M] [NEEDS CLARIFICATION] markers
        After: [N'] vague adjectives (-[N-N']), [M'] markers (-[M-M'])

        Specific Improvements:
        - Functional Requirements: [Example of vague → specific]
        - Non-Functional: [Example of added measurable target]
        - Data Model: [Example of clarified constraint/type]
        - Edge Cases: [Example of added failure scenario]

      Quality Check:
        IF (vague adjectives reduced by ≥50%) AND (markers reduced to ≤3):
          → ✅ Significant improvement
        IF some improvement but markers still >3:
          → ⚠️ Partial improvement (may need another pass)
        IF no measurable change:
          → ❌ Questions asked but spec not clarified
      ```

   ❓ **"Is there evidence of clarification impact?"**
      ```yaml
      Required Evidence (ALL must be provided):

      1. Spec File Comparison (MANDATORY):
         Before clarification line count: [N] lines
         After clarification line count: [M] lines
         Net change: [M-N] lines ([describe: additions in Clarifications + section updates])

      2. Category Coverage Change (MANDATORY):
         Run: Compare taxonomy status before/after
         Output:
           Categories changed from Partial→Clear: [list]
           Categories changed from Missing→Clear: [list]
           Categories still Partial/Missing: [list with reasons]

      3. Questions Efficiency (MANDATORY):
         Questions asked: [X]/5 quota used
         Categories resolved per question: [Y]/[X] = [Y/X] ratio
         High-impact categories resolved: [list]

         Efficiency Validation:
           IF ratio ≥ 1.5 (each question resolved 1-2 categories):
             → ✅ Efficient questioning
           IF ratio < 1.0:
             → ⚠️ Low impact questions (review question selection)

      4. Spec Diff Summary (MANDATORY):
         Show actual file changes:
         ```
         git diff HEAD -- [FEATURE_SPEC path]
         ```
         Output: [paste relevant diff sections showing Q&A + integration]

      IF any evidence is MISSING:
        ❌ CANNOT report completion
        → Gather missing evidence first
        → Re-run this step with complete evidence
      ```

   **Hallucination Prevention (7 Red Flags for Clarification):**
   ```yaml
   Detect and BLOCK these patterns:

   🚨 "All ambiguities resolved" WITHOUT showing coverage table
      → Self-correction: "Wait, I need to show actual coverage status"

   🚨 "Spec updated" WITHOUT showing what changed
      → Self-correction: "Let me show the actual diff and integration"

   🚨 "Questions answered" WITH Outstanding high-impact categories
      → Self-correction: "High-impact gaps remain, not ready yet"

   🚨 Claiming clarification WITHOUT reducing vague terms
      → Self-correction: "Spec measurability hasn't improved"

   🚨 Skipping integration validation
      → Self-correction: "I need to verify all answers are in spec"

   🚨 Hiding remaining ambiguities
      → Self-correction: "I must report Outstanding categories honestly"

   🚨 "Ready for planning" statements WITHOUT evidence
      → Self-correction: "Need to verify readiness criteria first"

   IF detected: STOP → Gather evidence → Report honestly
   ```

   **Determine Status**:

   ✅ **READY for Planning**:
   ```yaml
   Criteria (ALL must be met):
     - Outstanding categories = 0
     - (Resolved + Clear) ≥ 80% of taxonomy categories
     - All Q&A bullets recorded and integrated
     - Measurability improved (markers ≤3, vague terms reduced)
     - No contradictions in spec
     - Git diff shows actual content changes

   IF ALL criteria met:
     → Proceed to Step 10 (present completion report)
   ```

   ⚠️ **NEEDS REVIEW** (with risks noted):
   ```yaml
   Criteria:
     - Outstanding categories ≤ 2
     - All Outstanding are LOW impact
     - Most Q&A integrated (≥80%)
     - Some measurability improvement visible

   IF criteria met:
     → Present risks to user
     → Ask: "Low-impact gaps remain. Proceed to planning or run another clarify pass?"
     → Wait for user decision before Step 10
   ```

   ❌ **NOT READY** (more clarification needed):
   ```yaml
   Criteria (ANY triggers NOT READY):
     - Outstanding categories > 2
     - Any Outstanding category is HIGH impact
     - Q&A integration incomplete (<80%)
     - No measurability improvement
     - Contradictions remain in spec

   IF NOT READY:
     → Present issues to user with evidence
     → Recommend: "Run /speckit.clarify again to address [specific categories]"
     → STOP before Step 10 (do not proceed to planning)
   ```

   **Output Format** (Present to User - ONLY if evidence provided):
   ```markdown
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   📋 Clarification Review Complete
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

   Branch: [branch-name]
   Spec: [path/to/spec.md]

   **Challenge Mode**: [✅ ACTIVATED (Risk Score ≥8) | ⚪ NOT ACTIVATED (standard risk)]

   [IF Challenge Mode activated:]
   High-Risk Criteria Detected: [N] items with Risk Score ≥8
   Challenge Questions Asked: [N]
   Assumptions Validated: [N/N]
   Risk Mitigation Strategies: [Added to spec]

   ## Coverage Status

   ✓ Resolved: [X] categories (Partial/Missing → Clear)
   ✓ Clear: [Y] categories (already sufficient)
   ⚠️ Deferred: [Z] categories (planning phase)
   ❌ Outstanding: [N] categories

   [IF Outstanding > 0:]
   Outstanding Categories (need attention):
   - [Category]: [Impact] - [Why unresolved]

   ## Clarification Effectiveness

   Questions Asked: [X]/5 quota used
   Categories per Question: [Y]/[X] = [ratio] (target: ≥1.5)

   Spec Changes:
   - Lines added: +[N] (Clarifications + integrations)
   - Sections updated: [list names]
   - Vague terms reduced: [N] → [M] (-[N-M])
   - [NEEDS CLARIFICATION] markers: [X] → [Y] (-[X-Y])

   Integration Status:
   ✅ All Q&A recorded in Clarifications section
   ✅ All answers integrated into spec sections
   ✅ No contradictions remaining
   ✅ Terminology normalized

   ## Measurability Improvements

   [Show before/after examples:]
   - Functional: "[vague statement]" → "[specific statement]"
   - Non-Functional: "[missing target]" → "[measurable target: X ms]"
   - Data Model: "[unclear type]" → "[explicit type with constraints]"

   ## Readiness Assessment

   [IF ✅ READY:]
   ✅ All critical ambiguities resolved
   ✅ Spec measurability significantly improved
   ✅ No high-impact gaps remaining
   ✅ Ready to proceed to planning phase

   [IF ⚠️ NEEDS REVIEW:]
   ⚠️ Minor gaps remain (low impact):
      - [Gap 1]: [Why low impact]
      - [Gap 2]: [Why low impact]

   Risk: These gaps may require clarification during planning.

   [IF ❌ NOT READY:]
   ❌ Critical gaps remain:
      - [Gap 1]: [HIGH impact] - [Why blocks planning]
      - [Gap 2]: [HIGH impact] - [Why blocks planning]

   Recommendation: Run /speckit.clarify again focusing on:
      - [Specific category 1]
      - [Specific category 2]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Evidence: Spec Changes
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   ```diff
   [Paste relevant git diff output showing:]
   + ## Clarifications
   + ### Session YYYY-MM-DD
   + - Q: [question 1] → A: [answer 1]
   + ... [and integration into spec sections]
   ```

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   ## Next Steps
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [IF ✅ READY:]
   1. Proceed to planning phase:
      /speckit.plan

   2. Or review spec changes:
      git diff HEAD -- [FEATURE_SPEC]

   [IF ⚠️ NEEDS REVIEW:]
   Choose one:
   1. Proceed with noted risks:
      /speckit.plan

   2. Address remaining gaps:
      /speckit.clarify [focus on specific categories]

   3. Review current spec state:
      git diff HEAD -- [FEATURE_SPEC]

   [IF ❌ NOT READY:]
   REQUIRED: Address critical gaps before planning:
   1. Run another clarify pass:
      /speckit.clarify [targeting high-impact categories]

   2. Review what needs clarification:
      [List specific sections/questions to focus on]

   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   Ready to proceed? (yes/review spec/run clarify again/address gaps)
   ```

9. **Update spec-metadata.json** (NEW v2.3)

   After clarification session completes, update `specs/[FEATURE]/spec-metadata.json`:

   ```json
   {
     "version": "2.3.0",
     "phase": "clarifying",
     "approvals": {
       "specification": {
         "generated": true,
         "approved": false,
         "timestamp": "[ISO 8601 timestamp]"
       }
     },
     "metadata": {
       "clarifications_added": [count],
       "markers_remaining": [count of [NEEDS CLARIFICATION] markers],
       "last_clarify_mode": "[standard|expert]",
       "last_updated": "[ISO 8601 timestamp]"
     }
   }
   ```

   **Dual State Tracking (v2.3 Transition)**:
   - IF both spec-metadata.json AND state.json exist: Update both
   - IF only spec-metadata.json exists: Update spec-metadata.json only
   - IF only state.json exists: Update state.json only (v2.1 compatibility)

10. Present Next Steps based on review gate status

   **User Interaction:**
   ```yaml
   IF user says "yes" or "proceed":
     IF status = READY:
       → Suggest: /speckit.plan
     IF status = NEEDS REVIEW:
       → Warn about risks, then suggest: /speckit.plan
     IF status = NOT READY:
       → Block: "Cannot proceed - critical gaps remain"
       → Suggest: /speckit.clarify [with focus areas]

   IF user says "review" or "review spec":
     → Run: git diff HEAD -- [FEATURE_SPEC]
     → Show detailed spec changes

   IF user says "clarify again" or "run clarify":
     → Suggest: /speckit.clarify [focus on Outstanding categories]
     → List specific questions to target

   IF user says "address gaps":
     → List Outstanding categories with:
        - Why they're important (impact)
        - Suggested questions to ask
        - Which spec sections need updating
     → Ask: "Ready to run /speckit.clarify to address these?"

   IF EVIDENCE MISSING:
     ❌ "Cannot complete clarification review without evidence."
     → List missing evidence
     → Gather evidence automatically where possible
     → Re-run review gate
   ```

   **Benefits** (from PM Agent Reflexion pattern):
   - ✅ Prevents premature planning with unresolved ambiguities
   - ✅ Evidence-based readiness assessment
   - ✅ No false "ready" claims without coverage proof
   - ✅ Transparent validation of clarification effectiveness
   - ✅ User confidence in spec quality before planning

10. Report final completion summary (after review gate passes).

Behavior rules:

- If no meaningful ambiguities found (or all potential questions would be low-impact), respond: "No critical ambiguities detected worth formal clarification." and suggest proceeding.
- If spec file missing, instruct user to run `/speckit.specify` first (do not create a new spec here).
- Never exceed 5 total asked questions (clarification retries for a single question do not count as new questions).
- Avoid speculative tech stack questions unless the absence blocks functional clarity.
- Respect user early termination signals ("stop", "done", "proceed").
- If no questions asked due to full coverage, output a compact coverage summary (all categories Clear) then suggest advancing.
- If quota reached with unresolved high-impact categories remaining, explicitly flag them under Deferred with rationale.

Context for prioritization: {ARGS}

---

**Command Version**: 2.3.0
**Last Updated**: 2025-11-05
**Compatibility**: SpecKit v2.3+
**New in v2.3**: spec-metadata.json integration, dual state tracking, clarification metadata tracking
