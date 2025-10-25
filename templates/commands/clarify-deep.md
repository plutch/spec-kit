---
description: Identify underspecified areas in the current feature spec using AI-powered deep review analysis, then ask up to 5 highly targeted clarification questions and encode answers back into the spec.
scripts:
   sh: scripts/bash/check-prerequisites.sh --json --paths-only
   ps: scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Deep Review Mode

This enhanced clarify command uses **sequential thinking** to perform multi-pass requirements analysis before generating questions. This catches contradictions, suggests optimizations, and identifies high-impact ambiguities that taxonomy-based scanning might miss.

### Configuration Detection

Determine deep review mode from the following sources (in priority order):

1. **User Arguments** (highest priority):
   - If `$ARGUMENTS` contains `--deep` or `deep review` → Enable deep review (thorough mode)
   - If `$ARGUMENTS` contains `--fast` or `skip deep` → Disable deep review (taxonomy only)
   - If `$ARGUMENTS` contains `--balanced` → Enable deep review (balanced mode)

2. **Auto-Detection** (if no explicit flag):
   - Check if spec contains `## Complex Features` section → Enable (balanced mode)
   - Count requirements: if ≥15 functional requirements → Enable (balanced mode)
   - Check for keywords: "authentication", "authorization", "payment", "multi-tenant", "real-time", "integration", "external API" → Enable (balanced mode)
   - Check if spec contains `[NEEDS CLARIFICATION]` markers ≥3 → Enable (thorough mode)

3. **Default Fallback**:
   - If spec word count >3000 → Enable (balanced mode)
   - Otherwise → Enable (fast mode) - still beneficial with lower token cost

### Review Modes

- **Fast**: 8-12 thoughts, focus on contradictions and critical gaps (~5,000 tokens)
- **Balanced**: 15-20 thoughts, adds optimization suggestions (~10,000 tokens)
- **Thorough**: 25-30 thoughts, comprehensive analysis with alternatives (~15,000 tokens)

## Outline

Goal: Detect and reduce ambiguity or missing decision points in the active feature specification using advanced AI reasoning, then record clarifications directly in the spec file.

Note: This clarification workflow is expected to run (and be completed) BEFORE invoking `/speckit.plan`. If the user explicitly states they are skipping clarification (e.g., exploratory spike), you may proceed, but must warn that downstream rework risk increases.

Execution steps:

1. Run `{SCRIPT}` from repo root **once** (combined `--json --paths-only` mode / `-Json -PathsOnly`). Parse minimal JSON payload fields:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - (Optionally capture `IMPL_PLAN`, `TASKS` for future chained flows.)
   - If JSON parsing fails, abort and instruct user to re-run `/speckit.specify` or verify feature branch environment.
   - For single quotes in args like "I'm Groot", use escape syntax: e.g 'I'\''m Groot' (or double-quote if possible: "I'm Groot").

2. Load the current spec file. Perform a structured ambiguity & coverage scan using this taxonomy. For each category, mark status: Clear / Partial / Missing. Produce an internal coverage map used for deep review input (do not output raw map unless no questions will be asked).

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

   Edge Cases & Failure Handling:
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

   For each category with Partial or Missing status, add a candidate issue for deep review analysis.

2.5. **DEEP REVIEW ANALYSIS** (Sequential Thinking):

   **If deep review is enabled** (per configuration detection above), perform multi-pass requirements analysis using the `mcp__sequential-thinking__sequentialthinking` tool.

   **Input Context** for analysis (provide to first thought):
   - Full spec content
   - Coverage map from Step 2 (taxonomy scan results)
   - Project constitution (if available at `/memory/constitution.md`)
   - Feature type/domain detected from spec content

   **Analysis Passes** (adjust thought count by mode):

   **Pass 1: Logic Validation** (thoughts 1-5 for balanced mode)
   - Thought 1: Scan for contradictory requirements
     - Example: "Must be public" vs "Requires authentication"
     - Example: "Real-time updates" vs "Batch processing every hour"
     - Output: List of contradictions with section references

   - Thought 2: Identify circular dependencies
     - Example: Feature A requires Feature B, Feature B requires Feature A
     - Example: Data model dependencies that create impossible initialization order
     - Output: Dependency chains that form cycles

   - Thought 3: Check for impossible constraints
     - Example: "Sub-millisecond latency" with "External API call required"
     - Example: "Zero downtime deployment" with "In-memory session storage"
     - Output: Constraints that violate physical/technical limits

   - Thought 4: Validate success criteria measurability
     - Flag criteria like "fast", "intuitive", "robust" without metrics
     - Check if each user story has testable acceptance criteria
     - Output: Vague success criteria needing quantification

   - Thought 5: Synthesis of logic issues
     - Rank findings by severity (blocking vs warning)
     - Generate 1-2 clarification questions to resolve top contradictions
     - Output: Prioritized logic issues with question candidates

   **Pass 2: Completeness Analysis** (thoughts 6-11 for balanced mode)
   - Thought 6: Assess domain coverage against feature type
     - If authentication feature: Check for password policy, MFA, session management, audit logs
     - If payment feature: Check for idempotency, refund handling, webhook security
     - If multi-tenant feature: Check for data isolation, tenant scoping
     - If integration feature: Check for rate limiting, retry logic, failure modes
     - Output: Missing domain-specific requirements

   - Thought 7: Identify missing edge cases
     - What happens with empty data sets?
     - What happens with maximum scale (e.g., 10,000 items)?
     - What happens during external service failures?
     - What happens with concurrent user actions?
     - Output: Unspecified edge cases by category

   - Thought 8: Check non-functional coverage
     - Performance targets: Are latency/throughput specified?
     - Security: Are threat models and auth boundaries defined?
     - Observability: Are logging/monitoring requirements stated?
     - Reliability: Are error recovery and fallback strategies defined?
     - Output: Missing non-functional requirements

   - Thought 9: Validate data model completeness
     - Are all entities mentioned in requirements defined?
     - Are relationships (1:1, 1:N, M:N) specified?
     - Are identity/uniqueness constraints defined?
     - Are required vs optional fields distinguished?
     - Output: Data model gaps and ambiguities

   - Thought 10: Check integration completeness
     - Are all external dependencies listed?
     - Are API contracts/protocols specified?
     - Are authentication methods for external services defined?
     - Are failure modes and fallbacks documented?
     - Output: Integration specification gaps

   - Thought 11: Synthesis of completeness issues
     - Group gaps by impact (must-have vs nice-to-have)
     - Prioritize based on architecture/implementation blocking
     - Generate 1-2 clarification questions for critical gaps
     - Output: Top completeness issues with question candidates

   **Pass 3: Optimization Analysis** (thoughts 12-18 for balanced mode)
   - Thought 12: Identify over-engineering opportunities
     - Are there requirements that add complexity without clear value?
     - Example: "Support 17 date formats" when ISO-8601 would suffice
     - Example: "Real-time sync" when 5-second polling meets user needs
     - Output: Requirements that could be simplified

   - Thought 13: Suggest alternative approaches
     - For each complex requirement, propose simpler alternatives
     - Example: "Complex state machine" → "Event sourcing could simplify"
     - Example: "Custom authentication" → "OAuth 2.0 + JWT standard pattern"
     - Output: Alternative implementations with trade-offs

   - Thought 14: Analyze scope reduction opportunities
     - Identify requirements that could be deferred without losing MVP value
     - Flag P2/P3 user stories that overlap with P1 functionality
     - Suggest phased rollout options
     - Output: Scope reduction suggestions with retained value

   - Thought 15: Assess trade-off clarity
     - Are performance vs simplicity trade-offs explicit?
     - Are cost vs feature trade-offs documented?
     - Are security vs usability trade-offs acknowledged?
     - Output: Implicit trade-offs needing explicit discussion

   - Thought 16: Check for premature optimization
     - Flag performance requirements before scale requirements defined
     - Flag caching requirements before identifying bottlenecks
     - Flag distributed systems requirements for small-scale needs
     - Output: Requirements that may be premature

   - Thought 17: Identify technical debt risks
     - Flag requirements that create future maintenance burden
     - Flag requirements that limit future extensibility
     - Flag requirements that couple unrelated subsystems
     - Output: Technical debt indicators

   - Thought 18: Synthesis of optimization opportunities
     - Rank by impact (token savings, development time, maintenance)
     - For each, formulate as a trade-off question for user
     - Generate 1-2 clarification questions offering better alternatives
     - Output: Top optimization opportunities with question candidates

   **Pass 4: Final Synthesis** (thoughts 19-20 for balanced mode)
   - Thought 19: Consolidate all findings
     - Merge question candidates from all passes
     - Remove duplicate or overlapping questions
     - Assess each question's impact on spec quality
     - Score by: Impact × Uncertainty × Downstream Effect
     - Output: Ranked list of all potential clarification questions

   - Thought 20: Select top 5 questions
     - Choose questions with highest combined score
     - Ensure category diversity (don't ask 3 data model questions)
     - Balance between resolving contradictions and adding missing requirements
     - For each selected question:
       * Formulate as multiple-choice (preferred) or short-answer
       * Generate 2-5 answer options with clear trade-offs
       * Provide recommendation based on best practices
       * Include rationale for why this question matters
     - Output: Final 5 questions ready for sequential presentation

   **Sequential Thinking Format**:
   Use the `mcp__sequential-thinking__sequentialthinking` tool with these parameters:
   - `thought`: The current analysis step (see pass descriptions above)
   - `thoughtNumber`: Current thought number (1-20 for balanced mode)
   - `totalThoughts`: Initial estimate (adjust with `needsMoreThoughts` if needed)
   - `nextThoughtNeeded`: true until final thought (20)
   - `isRevision`: true if reconsidering previous thought
   - `revisesThought`: thought number being reconsidered (if isRevision=true)

   **Output Format** (after sequential thinking completes):
   Generate a structured Deep Review Report:

   ```markdown
   ## Deep Review Analysis

   **Mode**: Balanced (20 thoughts)
   **Analysis Duration**: [X thoughts completed]

   ### Key Findings

   #### Logic Issues (Severity: HIGH/MEDIUM/LOW)
   - [Issue 1]: Description with spec section reference
   - [Issue 2]: Description with spec section reference

   #### Completeness Gaps (Priority: CRITICAL/HIGH/MEDIUM)
   - [Gap 1]: Missing requirement area with impact explanation
   - [Gap 2]: Missing requirement area with impact explanation

   #### Optimization Opportunities (Impact: HIGH/MEDIUM/LOW)
   - [Opportunity 1]: Alternative approach with trade-off analysis
   - [Opportunity 2]: Alternative approach with trade-off analysis

   ### Questions Selected (5)

   The following questions were selected for maximum impact on specification quality:

   1. **[Category]**: Question text
      - **Impact**: Architecture/Data/UX/Security/Performance
      - **Rationale**: Why this matters for downstream success

   2. [repeat for all 5]

   ---
   ```

   **If deep review is disabled** (fast mode or user override):
   - Skip sequential thinking
   - Proceed directly to Step 3 with taxonomy-based question generation
   - Output: "Deep review skipped. Using fast taxonomy-based question generation."

3. Generate (internally) a prioritized queue of clarification questions (maximum 5).

   **If deep review was used**:
   - Use the 5 questions from Pass 4 synthesis (already prioritized)
   - Questions already have options and recommendations prepared

   **If deep review was skipped**:
   - Generate questions from taxonomy scan (original behavior)
   - For each category with Partial or Missing status, add a candidate question opportunity unless:
     - Clarification would not materially change implementation or validation strategy
     - Information is better deferred to planning phase (note internally)
   - Apply original constraints:
     * Each question must be answerable with EITHER:
        - A short multiple‑choice selection (2–5 distinct, mutually exclusive options), OR
        - A one-word / short‑phrase answer (explicitly constrain: "Answer in <=5 words").
     * Only include questions whose answers materially impact architecture, data modeling, task decomposition, test design, UX behavior, operational readiness, or compliance validation.
     * Ensure category coverage balance
     * Exclude questions already answered, trivial stylistic preferences, or plan-level execution details
     * Favor clarifications that reduce downstream rework risk or prevent misaligned acceptance tests
     * If more than 5 categories remain unresolved, select the top 5 by (Impact × Uncertainty) heuristic

   Do NOT output questions all at once - they will be presented sequentially.

4. Sequential questioning loop (interactive):
    - Present EXACTLY ONE question at a time.
    - For multiple‑choice questions:
       - **Analyze all options** and determine the **most suitable option** based on:
          - Best practices for the project type
          - Common patterns in similar implementations
          - Risk reduction (security, performance, maintainability)
          - Alignment with any explicit project goals or constraints visible in the spec
          - Deep review analysis findings (if available)
       - Present your **recommended option prominently** at the top with clear reasoning (1-2 sentences explaining why this is the best choice).
       - Format as: `**Recommended:** Option [X] - <reasoning>`
       - Then render all options as a Markdown table:

       | Option | Description | Trade-offs |
       |--------|-------------|------------|
       | A | <Option A description> | <Key trade-offs or implications> |
       | B | <Option B description> | <Key trade-offs or implications> |
       | C | <Option C description> (add D/E as needed up to 5) | <Key trade-offs or implications> |
       | Short | Provide a different short answer (<=5 words) (Include only if free-form alternative is appropriate) | N/A |

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
   - **Deep Review Summary** (if used):
     - Mode used (fast/balanced/thorough)
     - Thoughts completed
     - Key findings summary (contradictions, gaps, optimizations)
     - Questions derived from deep analysis
   - Number of questions asked & answered.
   - Path to updated spec.
   - Sections touched (list names).
   - Coverage summary table listing each taxonomy category with Status: Resolved (was Partial/Missing and addressed), Deferred (exceeds question quota or better suited for planning), Clear (already sufficient), Outstanding (still Partial/Missing but low impact).
   - **Optimization Opportunities** (from deep review, if any remain unaddressed)
   - If any Outstanding or Deferred remain, recommend whether to proceed to `/speckit.plan` or run `/speckit.clarify` again later post-plan.
   - **Token Usage Report** (if deep review used):
     - Estimated tokens used for sequential thinking
     - Estimated savings from prevented rework (based on issues caught)
     - ROI assessment
   - Suggested next command.

Behavior rules:

- If no meaningful ambiguities found (or all potential questions would be low-impact), respond: "No critical ambiguities detected worth formal clarification." and suggest proceeding.
- If spec file missing, instruct user to run `/speckit.specify` first (do not create a new spec here).
- Never exceed 5 total asked questions (clarification retries for a single question do not count as new questions).
- Avoid speculative tech stack questions unless the absence blocks functional clarity.
- Respect user early termination signals ("stop", "done", "proceed").
- If no questions asked due to full coverage, output a compact coverage summary (all categories Clear) then suggest advancing.
- If quota reached with unresolved high-impact categories remaining, explicitly flag them under Deferred with rationale.
- **Deep review auto-detection**: If not explicitly specified by user, intelligently enable based on feature complexity indicators.
- **Token consciousness**: In fast mode, limit thoughts to 8-12; in balanced mode, 15-20; in thorough mode, 25-30.
- **Incremental thinking**: Use `needsMoreThoughts` if analysis uncovers unexpected complexity requiring additional thoughts.
- **Revision capability**: Use `isRevision=true` if a later thought contradicts an earlier finding (mark with `revisesThought`).

Context for prioritization: {ARGS}
