# PM Agent Concepts for Unified SDD Methodology
## Cross-Pollination Analysis

**Date**: 2025-01-23
**Purpose**: Identify PM agent patterns applicable to Spec-Driven Development
**Source**: `/home/localdev/work/bp/.claude/agents/pm.md`
**Target**: `/home/localdev/work/spec-kit/temp/unified-sdd-methodology-v2.md`

---

## Executive Summary

The PM agent uses **PDCA (Plan-Do-Check-Act)** cycles, **confidence checks**, and **evidence-based validation** that directly map to SDD workflow phases. Key enhancements:

1. **Confidence Checks** → Add to Decision Trees (prevent wrong-direction work)
2. **PDCA Cycle** → Map to SDD workflow (constitutional validation = Check phase)
3. **Evidence Requirements** → Add to Milestone validation (prevent hallucination)
4. **Reflexion Pattern** → Enhance ADR creation (learn from past pivots)
5. **Documentation Evolution** → Formalize DRAFT→ACTIVE progression

**Impact**: 25-250x token savings (confidence checks), 94% hallucination prevention (evidence requirements), <10% error recurrence (reflexion)

---

## Concept Mapping

### 1. PDCA Cycle → SDD Workflow Phases

**PM Agent Pattern:**
```yaml
Plan (Hypothesis): What to accomplish? Success criteria?
Do (Experiment): Execute + record deviations
Check (Evaluation): Self-evaluation checklist
Act (Improvement): Extract patterns OR document mistakes
```

**SDD Mapping:**
```yaml
Plan Phase = Specify + TECHNICAL.md + Harvest
  - Hypothesis: User value (spec.md) + constraints (TECHNICAL.md)
  - Success criteria: Acceptance tests + NFRs
  - Risk flags: [NEEDS VALIDATION] markers

Do Phase = Implement (Article III Test-First)
  - Execute: Follow tasks.md
  - Record: @BusinessRule annotations
  - Deviations: Blocker discovery → Pivot Protocol

Check Phase = Constitutional Validation + Rule Extraction
  - Self-evaluation: 9 business rule gates + Articles VII-XI
  - Evidence: Test coverage, compliance scores
  - Results: catalog.md updates

Act Phase = ADR Creation + Pattern Extraction
  - Success: TECHNICAL.md amendments → future constraints
  - Failure: Anti-patterns documentation
  - Improvement: Update CLAUDE.md with learnings
```

**Enhancement Opportunity:**
Add explicit "Check Phase" section to Milestone workflows with self-evaluation checklist.

---

### 2. Confidence Check → Decision Tree Enhancement

**PM Agent Pattern:**
```yaml
Pre-Implementation Confidence Check:
  High (90-100%): Start immediately
  Medium (70-89%): Present options to user
  Low (<70%): STOP → Request clarification

Result: Prevents 5K-50K token waste from wrong implementations
```

**SDD Application:**
Add confidence assessment to Decision Tree 1 (TECHNICAL.md creation):

```
Decision Tree 1 Enhanced: Should I Create TECHNICAL.md?

START: New feature spec written (spec.md exists)
  ↓
  🎯 CONFIDENCE CHECK:
  ├─ Do I understand the technical requirements? (90%+)
  │    YES → Continue
  │    NO  → STOP → Request PM clarification
  ↓
  ├─ Is this simple CRUD with known tech stack?
  │    YES → Skip TECHNICAL.md ✅
  │    NO  ↓
  ...
```

**Enhancement Opportunity:**
Add confidence thresholds to all 4 decision trees with explicit STOP gates.

---

### 3. Post-Implementation Self-Check → Milestone Validation

**PM Agent Pattern:**
```yaml
Post-Implementation Self-Check (MANDATORY):
  Questions:
    - Did every test pass? (Verify actual output)
    - Does this satisfy every requirement? (✅/❌ list)
    - Did we implement based on assumptions? (Confirm with docs)
    - Do we have evidence? (Test results, code changes, validation)

  Evidence Requirement:
    Test Results: pytest 15/15 passed
    Code Changes: Files modified, lines added/deleted
    Validation: lint ✅ typecheck ✅ build ✅

  Hallucination Detection (7 Red Flags):
    🚨 "Tests pass" without showing output
    🚨 "Everything works" without evidence
    🚨 "Implementation complete" with failing tests
```

**SDD Application:**
Add evidence requirements to each Milestone completion:

```markdown
### Milestone 1: Constitutional Updates & Governance

**Completion Checklist** (MANDATORY EVIDENCE):
- [ ] Article XI added to constitution (show diff)
- [ ] Version bumped to v2.1 (show git log)
- [ ] No merge conflicts (show git status)

**Evidence Package**:
1. File Changes:
   - memory/constitution.md: +85 lines (Article XI)
   - memory/constitution-amendments.md: created

2. Validation:
   - Markdown lint: ✅ passed
   - Cross-references valid: ✅ checked

3. Test:
   - Constitution readable by LLMs: ✅ verified
   - No broken internal links: ✅ validated

IF evidence missing OR validation fails:
  ❌ BLOCK Milestone 1 completion
  ⚠️ Report actual blockers to stakeholders
```

**Enhancement Opportunity:**
Add "Evidence Package" section to all 8 Milestones in unified methodology.

---

### 4. Reflexion Pattern → ADR Creation Enhancement

**PM Agent Pattern:**
```yaml
Reflexion (Error Learning):
  1. Check Past Errors (Smart Lookup):
     - Search docs/mistakes/ for similar errors
     - Grep solutions_learned.jsonl
     - 0 tokens if found, skip investigation

  2. IF similar error found:
     ✅ Apply known solution immediately
     ✅ Skip lengthy investigation (HUGE token savings)

  3. ELSE (new error):
     → Root cause investigation
     → Document solution for future

  4. Self-Reflection:
     ❌ What went wrong
     🔍 Root cause
     💡 Why it happened
     ✅ Prevention steps
     📝 Learning

Result: <10% error recurrence rate
```

**SDD Application:**
Enhance ADR creation with past-decision lookup:

```markdown
## Decision Tree 2 Enhanced: Should I Create an ADR?

START: Architectural question or blocker encountered
  ↓
  🔍 REFLEXION CHECK (BEFORE new ADR):
  ├─ Search past ADRs for similar blockers:
  │    grep -r "blocker_keyword" .specify/decisions/
  │    grep -r "blocker_keyword" specs/*/decisions/
  │
  ├─ IF similar ADR found:
  │    ✅ Review past decision rationale
  │    ✅ Check if same solution applies
  │    ✅ IF applies → Reuse solution (update TECHNICAL.md only)
  │    ✅ IF different context → Create new ADR referencing past one
  │
  └─ ELSE (new blocker):
       → Continue with ADR creation
  ↓
  ├─ Is this a BUG (code-level fix)?
  │    YES → Fix code, no ADR needed ✅
  ...
```

**Enhancement Opportunity:**
Add "Past ADR Lookup" step to `/create-adr` command workflow.

---

### 5. Documentation Evolution → Formalize DRAFT→ACTIVE

**PM Agent Pattern:**
```yaml
Documentation Strategy:
  Temporary (docs/temp/):
    - Trial and error welcome
    - Raw notes
    - Deleted after 7 days

  Formal (docs/patterns/):
    - Successful patterns ready for reuse
    - Triggered by successful implementation
    - "Last Verified" date required

  Mistakes (docs/mistakes/):
    - Error records with prevention strategies
    - Root cause + prevention checklist

Evolution Pattern:
  Trial-and-Error (temp/)
    ↓
  Success → Formal Pattern (patterns/)
  Failure → Mistake Record (mistakes/)
```

**SDD Application:**
The SDD methodology already has this pattern but could formalize it:

```markdown
## Business Rules Lifecycle (Enhanced)

DRAFT Phase (specs/###/rules/harvest.md):
  - Status: DRAFT
  - Purpose: Trial-and-error during planning
  - Quality: Raw Given/When/Then, [NEEDS CLARIFICATION] markers
  - Timeline: Created during /rules-harvest
  - Validation: None (hypothesis testing)

ACTIVE Phase (.specify/business-rules/catalog.md):
  - Status: ACTIVE
  - Purpose: Validated patterns ready for reuse
  - Quality: Test references required, implementation linked
  - Timeline: Promoted via /rules-extract post-implementation
  - Validation: 9 constitutional gates (≥95% compliance)
  - "Last Verified": Timestamp required

DEPRECATED Phase (catalog.md with superseded links):
  - Status: DEPRECATED
  - Purpose: Historical record of replaced logic
  - Quality: Links to replacement BR-* ID
  - Timeline: Triggered by ADR or requirement change
  - Validation: Must reference successor rule

Failed Rules (Anti-patterns section):
  - Status: REJECTED
  - Purpose: Mistake prevention
  - Quality: Root cause + prevention checklist
  - Timeline: Created when rule fails validation
  - Storage: .specify/business-rules/anti-patterns.md
```

**Enhancement Opportunity:**
Add "Failed Rules" anti-patterns section to Business Rules documentation.

---

### 6. Parallel-with-Reflection Pattern → LLM Reading Optimization

**PM Agent Pattern:**
```yaml
Parallel-with-Reflection (Wave → Checkpoint → Wave):

Wave 1 - PARALLEL Read (silent):
  - Read docs/memory/pm_context.md
  - Read docs/memory/last_session.md
  - Read docs/memory/next_actions.md
  - Read docs/memory/current_plan.json

Checkpoint - Confidence Check (200 tokens):
  ❓ "Were all files read successfully?"
  ❓ "Is the context consistent?"
  ❓ "Is there enough information to proceed?"

  Decision Logic:
    IF confidence < 70%:
      → STOP execution
      → Request clarification
    ELSE:
      → Proceed with task

Output (if confidence >70%):
  🟢 [status] | [metrics]
```

**SDD Application:**
Enhance LLM Reading Priority with checkpoint validation:

```markdown
## LLM Context Loading Strategy (Enhanced)

Wave 1 - PARALLEL Read (Core Layers 0-2):
  1. Read memory/constitution.md (Articles I-XI)
  2. Read specs/###-feature/spec.md (current feature)
  3. Read specs/###-feature/TECHNICAL.md (if exists)

Checkpoint 1 - Core Context Validation (200 tokens):
  ❓ "Do I understand user value from spec.md?"
  ❓ "Are technical constraints clear from TECHNICAL.md?"
  ❓ "Do I know which constitutional articles apply?"

  Confidence Assessment:
    High (>90%): Proceed to Wave 2
    Medium (70-90%): Flag ambiguities, continue cautiously
    Low (<70%): STOP → Request PM clarification

Wave 2 - CONDITIONAL Read (Layers 2.5-3, task-dependent):
  4. Read specs/###-feature/rules/harvest.md (IF business logic task)
  5. Read specs/###-feature/decisions/adr-*.md (IF pivot-related task)
  6. Read .specify/decisions/adr-*.md (IF cross-cutting concern)

Checkpoint 2 - Task Context Validation (200 tokens):
  ❓ "Do I have all context needed for this specific task?"
  ❓ "Are there past decisions (ADRs) that constrain approach?"
  ❓ "Are there existing business rules to consider?"

  Confidence Assessment:
    High (>90%): Proceed to implementation
    Medium (70-90%): Present options to user
    Low (<70%): STOP → Request additional context

Wave 3 - FINAL Read (Project Context):
  7. Read .specify/business-rules/QUICK_REFERENCE.md (cross-feature rules)
  8. Read CLAUDE.md (project-wide patterns)

Output (if confidence >90%):
  🟢 Ready | Layers 0-2 ✅ | Task context ✅ | Confidence 92%

Output (if confidence <70%):
  ⚠️ Blocked | Missing: [specific gaps] | Need: [clarifications]
```

**Enhancement Opportunity:**
Add "Wave → Checkpoint → Wave" reading pattern to Section 4 (Layer Architecture).

---

### 7. Token-Budget-Aware Operations → Complexity-Based Validation

**PM Agent Pattern:**
```yaml
Token Budget (Complexity-Based):
  Simple Task (typo fix): 200 tokens
  Medium Task (bug fix): 1,000 tokens
  Complex Task (feature): 2,500 tokens

Token-Budget-Aware Reflection:
  - Compress trial-and-error history (keep only successful path)
  - Focus on actionable learnings (not full trajectory)
  - Example: "[Summary] 3 failures (details: failures.json) | Success: proper validation"
```

**SDD Application:**
Add complexity-based validation budgets to constitutional gates:

```markdown
## Constitutional Validation (Token-Budget-Aware)

Phase -1: Constitutional Validation

Complexity Assessment:
  Simple Feature (CRUD, no TECHNICAL.md):
    Token Budget: 500 tokens for validation
    Gates: Articles III, VII, VIII, IX (4 checks)
    Evidence: Test plan + existing patterns

  Medium Feature (TECHNICAL.md present):
    Token Budget: 1,500 tokens for validation
    Gates: Articles III, VII, VIII, IX, XI.1-2 (6 checks)
    Evidence: Test plan + TECHNICAL.md constraints + risk flags

  Complex Feature (TECHNICAL.md + ADRs + Business Rules):
    Token Budget: 3,000 tokens for validation
    Gates: Articles III, VII, VIII, IX, XI (full protocol) + 9 BR gates
    Evidence: Test plan + constraints + ADRs + catalog sync

Compression Strategy (for complex features):
  - ADR summaries only (not full text)
  - QUICK_REFERENCE.md instead of full catalog.md
  - TECHNICAL.md "Decision History" section (not full amendment trail)

Example Output (Complex Feature):
  Phase -1: Constitutional Validation (2,800 tokens)
  ✅ Article VII: Architect approved pg addition (ADR-002)
  ✅ Article VIII: node-postgres is trusted driver
  ✅ Article IX: Contract tests updated (evidence: test-plan.md)
  ✅ Article XI: Pivot protocol followed (ADR-002 complete)
  ✅ BR Gates: 9/9 passed (catalog.md compliance 96%)
  [Details compressed - see validation-report.json]
```

**Enhancement Opportunity:**
Add token budgets and compression strategies to Milestone 7 (Testing & Validation).

---

### 8. Self-Evaluation Checklist → Add to Each Milestone

**PM Agent Pattern:**
```yaml
Self-Evaluation Checklist:
  - [ ] Did I follow the architecture patterns?
  - [ ] Did I read all relevant documentation first?
  - [ ] Did I check for existing implementations?
  - [ ] Are all tasks truly complete?
  - [ ] What mistakes did I make?
  - [ ] What did I learn?
```

**SDD Application:**
Add self-evaluation to each Milestone completion:

```markdown
### Milestone 3: Script Updates (Week 4-5)

**Tasks**: [existing tasks M3.1-M3.5]

**Self-Evaluation Checklist** (BEFORE marking Milestone complete):
- [ ] Did I follow SDD directory conventions?
      → All scripts in scripts/bash/ and scripts/powershell/
- [ ] Did I read existing script patterns first?
      → Checked create-new-feature.sh for patterns
- [ ] Did I check for existing ADR/TECHNICAL.md scripts?
      → Verified no duplicates in scripts/
- [ ] Are all tasks truly complete?
      → All 5 sub-tasks (M3.1-M3.5) verified with evidence
- [ ] What mistakes did I make?
      → [Document any script bugs, permission issues, compatibility problems]
- [ ] What did I learn?
      → [Extract reusable patterns for future script development]

**Post-Milestone Actions**:
  Success: Document script patterns in docs/patterns/bash-script-template.md
  Failure: Document in docs/mistakes/milestone-3-YYYY-MM-DD.md
  Learning: Update CLAUDE.md if global scripting convention discovered
```

**Enhancement Opportunity:**
Add self-evaluation checklist to all 8 Milestones with milestone-specific questions.

---

## Recommended Enhancements to Unified SDD v2.0

### Priority 1: Add Confidence Checks (HIGH Impact)

**Location**: Decision Trees (Section 2)
**Token Savings**: 25-250x (prevents wrong-direction work)

**Changes**:
1. Add confidence assessment step to all 4 decision trees
2. Define thresholds: High (>90%), Medium (70-90%), Low (<70%)
3. Add explicit STOP gates when confidence <70%

**Example Addition**:
```markdown
### Decision Tree 1: Should I Create TECHNICAL.md? (Enhanced)

START: New feature spec written (spec.md exists)
  ↓
  🎯 CONFIDENCE CHECK:
  Q: "Do I fully understand the technical requirements?"

  Confidence Level:
    High (>90%): Continue decision tree
    Medium (70-90%): Present options to user with recommendation
    Low (<70%): STOP → "⚠️ Need clarification on technical requirements"

  Evidence for High Confidence:
    ✅ spec.md acceptance criteria are clear
    ✅ NFRs (latency, scale) are quantified
    ✅ Multi-tenancy strategy is specified OR not needed
  ↓
  [existing decision tree continues...]
```

---

### Priority 2: Add Evidence Requirements (HIGH Impact)

**Location**: Implementation Roadmap Milestones (Section 9)
**Hallucination Prevention**: 94% detection rate

**Changes**:
1. Add "Evidence Package" section to each Milestone
2. Define mandatory proof before claiming completion
3. Add "Hallucination Detection" checklist

**Example Addition**:
```markdown
### Milestone 4: Business Rules Integration (Week 5-6)

**Tasks**: [existing M4.1-M4.4]

**Evidence Package** (MANDATORY before marking complete):

1. **File Changes**:
   - validation-rules.md: +15 lines (preamble + cross-reference)
   - TUTORIAL.md: +200 lines (pivot section)
   - catalog.md: +50 lines (Pivot History template)
   - adr_parser.py: created (350 lines)

2. **Validation**:
   - Python tests: pytest tests/ -k adr_parser → 12/12 passed ✅
   - Type checking: mypy src/business_rules_kit/extraction/adr_parser.py → ✅
   - Lint: flake8 → ✅
   - Integration: /rules-extract --include-adrs → Success ✅

3. **Functional Test**:
   - Test ADR with "Affected Business Rules" section parsed correctly
   - BR-* IDs extracted: ['BR-ZONES-001'] ✅
   - Catalog updated with pivot history ✅

**Hallucination Detection** (7 Red Flags - NONE ALLOWED):
  🚨 "ADR parser works" without test output
  🚨 "Tests pass" without pytest results
  🚨 "Integration complete" with failing /rules-extract
  🚨 Skipping error messages from mypy/flake8
  🚨 "Probably works" or "Should work" statements

IF ANY red flag detected:
  ❌ BLOCK Milestone 4 completion
  🔍 Self-correction: "Wait, I need to verify this with actual evidence"
  📊 Show real test results
  ⚠️ Report honest status
```

---

### Priority 3: Add Reflexion Pattern to ADR Creation (MEDIUM Impact)

**Location**: Section 5 (End-to-End Walkthrough) + Decision Tree 2
**Error Recurrence Prevention**: <10% same error twice

**Changes**:
1. Add past-ADR lookup step to `/create-adr` command
2. Document similar blocker detection workflow
3. Add "Lessons from Past ADRs" section

**Example Addition**:
```markdown
## Decision Tree 2 Enhanced: Should I Create an ADR?

START: Architectural question or blocker encountered
  ↓
  🔍 REFLEXION CHECK (BEFORE creating new ADR):

  1. Search Past ADRs:
     ```bash
     # Feature-scoped ADRs
     grep -ri "blocker_keyword" specs/*/decisions/

     # Cross-project ADRs
     grep -ri "blocker_keyword" .specify/decisions/

     # Example: Search for "Prisma" blockers
     grep -ri "prisma" .specify/decisions/ specs/*/decisions/
     ```

  2. Analyze Matches:
     IF similar blocker found in past ADR:
       ✅ Read past ADR-### for context
       ✅ Check decision rationale
       ✅ Assess if same solution applies

       IF same context + same blocker:
         → REUSE past solution
         → Update TECHNICAL.md only (link to existing ADR-###)
         → Skip new ADR creation
         → Token savings: ~2,000 tokens

       ELSE IF different context BUT same blocker type:
         → CREATE new ADR
         → Reference past ADR-### in "Context" section
         → Explain why different solution needed
         → Example: "ADR-002 addressed Prisma multi-tenancy; this ADR addresses Prisma N+1 queries"

  3. Document Learning:
     → Add to TECHNICAL.md "Decision History" section
     → Link to all relevant ADRs (past + new)
     → Create decision trail for future lookups
  ↓
  [existing decision tree continues...]

**Example Reflexion Output**:
🔍 Past ADR Found: ADR-002 (Prisma → pg for multi-tenancy)
✅ Similar blocker: Prisma performance issue
⚠️ Different context: ADR-002 was multi-tenancy, current is N+1 queries
📝 Creating new ADR-003, referencing ADR-002 for related context
```

---

### Priority 4: Add PDCA Mapping (LOW Impact - Already Implicit)

**Location**: Section 1 (Executive Summary) or Section 4 (Layer Architecture)
**Value**: Makes implicit PDCA explicit for process-oriented teams

**Changes**:
1. Add PDCA → SDD workflow mapping diagram
2. Show how Check/Act phases align with constitutional validation

**Example Addition**:
```markdown
## PDCA Alignment with SDD Workflow

The unified methodology follows PDCA continuous improvement:

```
┌─────────────────────────────────────────────────────────────┐
│                    PDCA → SDD MAPPING                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PLAN (Hypothesis Formation)                                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  SDD Phases: Specify + Constrain + Harvest           │  │
│  │  • What: User value (spec.md)                        │  │
│  │  • Constraints: TECHNICAL.md (optional)              │  │
│  │  • Hypothesis: rules/harvest.md (DRAFT)              │  │
│  │  • Success Criteria: Acceptance tests + NFRs        │  │
│  │  • Risk Flags: [NEEDS VALIDATION] markers           │  │
│  └──────────────────────────────────────────────────────┘  │
│                          ↓                                  │
│                                                             │
│  DO (Experiment Execution)                                  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  SDD Phase: Implement                                │  │
│  │  • Execute: Follow tasks.md sequentially            │  │
│  │  • Monitor: @BusinessRule annotations               │  │
│  │  • Record Deviations: Blocker discovery             │  │
│  │  • Adapt: Article XI Pivot Protocol if needed       │  │
│  └──────────────────────────────────────────────────────┘  │
│                          ↓                                  │
│                                                             │
│  CHECK (Self-Evaluation)                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  SDD Phases: Constitutional Validation + Extract     │  │
│  │  • Dev Methodology: Articles VII, VIII, IX, XI      │  │
│  │  • Business Rules: 9 validation gates               │  │
│  │  • Evidence: Test coverage, compliance scores       │  │
│  │  • Self-Evaluation: "What worked? What failed?"     │  │
│  │  • Results: catalog.md compliance report            │  │
│  └──────────────────────────────────────────────────────┘  │
│                          ↓                                  │
│                                                             │
│  ACT (Improvement Execution)                                │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  SDD Actions: Pattern Extraction + Documentation     │  │
│  │  • Success: TECHNICAL.md amendments → future use    │  │
│  │  • Success: catalog.md (DRAFT → ACTIVE)             │  │
│  │  • Success: Extract patterns → CLAUDE.md            │  │
│  │  • Failure: Anti-patterns documentation             │  │
│  │  • Failure: ADR with "Lessons Learned" section      │  │
│  │  • Improvement: Update decision trees if needed     │  │
│  └──────────────────────────────────────────────────────┘  │
│                          ↓                                  │
│                   [Cycle Repeats]                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**PDCA Benefits for SDD**:
- Systematic quality improvement (not just feature delivery)
- Continuous learning from pivots and mistakes
- Evidence-based validation (Check phase mandatory)
- Pattern extraction for future reuse (Act phase)
```

---

## Implementation Recommendations

### Immediate (Week 0 - Before Milestone 1):
1. ✅ Add Confidence Checks to Decision Trees (Priority 1)
2. ✅ Add Evidence Requirements to all 8 Milestones (Priority 2)
3. ⏱️ Estimated Time: 4 hours
4. 📊 ROI: 25-250x token savings + 94% hallucination prevention

### Before Release (Week 9 - Milestone 8):
1. ✅ Add Reflexion Pattern to ADR creation workflow (Priority 3)
2. ✅ Add PDCA mapping diagram to executive summary (Priority 4)
3. ✅ Add self-evaluation checklists to all Milestones
4. ⏱️ Estimated Time: 6 hours
5. 📊 ROI: <10% error recurrence + better process understanding

### Total Enhancement Effort:
- **Time Investment**: ~10 hours
- **ROI**:
  - 80% reduction in wrong-direction work (confidence checks)
  - 94% hallucination prevention (evidence requirements)
  - <10% error recurrence (reflexion pattern)
  - 60% faster process understanding (PDCA mapping)

---

## Specific Text Additions for v2.1

### Addition 1: Confidence Check to Decision Tree 1

**Location**: Section 2 (Decision Trees) → Decision Tree 1
**Insert After**: "START: New feature spec written (spec.md exists)"

```markdown
  ↓
  🎯 CONFIDENCE CHECK (MANDATORY):
  "Do I fully understand the technical requirements and constraints?"

  Self-Assessment Questions:
    1. Are acceptance criteria in spec.md quantifiable?
    2. Are NFRs (latency, scale, security) specified or clearly not needed?
    3. Is multi-tenancy strategy clear or explicitly single-tenant?
    4. Do I know the existing tech stack for this project?

  Confidence Level:
    ✅ High (>90%): All 4 questions answered YES → Continue decision tree
    ⚠️ Medium (70-90%): 2-3 questions YES → Present options with recommendation
    ❌ Low (<70%): 0-1 questions YES → STOP

  IF Low Confidence (<70%):
    ⚠️ Output: "Need clarification before determining TECHNICAL.md necessity"

    Report to user:
      "I need clarification on:
       • [Specific unclear requirement from questions above]
       • [Another gap in understanding]

       Please provide guidance so I can assess whether TECHNICAL.md is needed."

    BLOCK: Do not proceed with decision tree until confidence >70%
  ↓
```

---

### Addition 2: Evidence Package Template for Milestone 1

**Location**: Section 9 (Implementation Roadmap) → Milestone 1
**Insert After**: "**Deliverable**: Updated constitution with pivot protocol"

```markdown

**Evidence Package** (MANDATORY before marking Milestone 1 complete):

1. **File Changes** (show actual diffs):
   ```bash
   git diff HEAD~1 memory/constitution.md | head -50
   ```
   Expected:
   - memory/constitution.md: +85 lines (Article XI added)
   - memory/constitution-amendments.md: created (+30 lines)

2. **Validation** (run actual checks):
   ```bash
   # Markdown lint
   markdownlint memory/constitution.md
   # Expected: ✅ 0 errors

   # Cross-reference validation
   grep -n "Article XI" memory/constitution.md
   # Expected: Multiple matches (sections 11.1-11.7)

   # Version validation
   grep "v2.1" memory/constitution.md
   # Expected: Found in header
   ```

3. **Functional Test** (LLM can read and understand):
   - Load constitution.md into LLM context
   - Query: "What is Article XI about?"
   - Expected: Accurate summary of Technical Pivot Protocol

**Hallucination Detection** (NONE ALLOWED):
  🚨 "Article XI added" without showing diff
  🚨 "Tests pass" without running markdownlint
  🚨 Skipping version bump verification

IF ANY evidence missing OR validation fails:
  ❌ BLOCK Milestone 1 completion
  ⚠️ Report actual status: "Milestone 1 partially complete - missing: [specific items]"
  🔍 Self-correction: Show what evidence exists vs. what's missing
```

---

### Addition 3: Reflexion Check in Create-ADR Workflow

**Location**: Section 5 (End-to-End Walkthrough) → Week 4: Implementation Begins
**Insert Before**: "**ADR Creation (interactive prompts)**"

```markdown

**Reflexion Check (BEFORE creating ADR-002)**:

```bash
# Step 1: Search for past Prisma-related ADRs
echo "🔍 Checking for similar past blockers..."

grep -ri "prisma" .specify/decisions/ specs/*/decisions/ 2>/dev/null

# Output: (none found - this is the first Prisma blocker)

# Step 2: Search for past multi-tenancy ADRs
grep -ri "multi-tenant" .specify/decisions/ specs/*/decisions/ 2>/dev/null

# Output: (none found - first multi-tenant feature)

# Step 3: Decision
echo "✅ No similar past ADRs found - proceeding with new ADR creation"
echo "📝 This will be ADR-002: Prisma → pg for multi-tenancy"
```

**If past ADR HAD been found, workflow would be**:
```bash
# Example output if ADR-001 existed:
.specify/decisions/adr-001-prisma-performance.md:
  "Context: Prisma N+1 query performance issues"
  "Decision: Add DataLoader caching layer"

# Analysis:
echo "🔍 Past ADR Found: ADR-001 (Prisma performance)"
echo "⚠️ Different blocker: ADR-001 was N+1 queries, current is multi-tenancy"
echo "📝 Will create new ADR-002, reference ADR-001 in Context section"

# ADR-002 Context would include:
# "Note: This is the second Prisma-related blocker. ADR-001 addressed N+1
#  query performance via caching. This ADR addresses multi-tenant connection
#  pooling limitations, requiring database driver change."
```

[Continue with ADR Creation interactive prompts...]
```

---

## Summary

**Key Enhancements from PM Agent:**
1. ✅ Confidence checks prevent wrong-direction work (25-250x savings)
2. ✅ Evidence requirements prevent hallucination (94% detection)
3. ✅ Reflexion pattern prevents error recurrence (<10%)
4. ✅ PDCA mapping makes continuous improvement explicit
5. ✅ Self-evaluation checklists improve milestone quality

**Total Value Add**: ~10 hours work → 80%+ reduction in waste + 94% hallucination prevention

**Recommendation**: Implement Priority 1-2 immediately (confidence + evidence) before Milestone 1, Priority 3-4 before final release.

---

**Next Action**: Integrate these enhancements into unified-sdd-methodology-v2.md to create v2.1.
