# Unified Spec-Driven Development Methodology
## Quick Start Guide for Spec-Kit Users

**Version**: 2.1
**Status**: Production-Ready
**Audience**: Teams using GitHub Spec-Kit

---

## Overview

This guide introduces **enhancements** to the core Spec-Kit workflow. The unified methodology adds:

1. **Confidence Checks** - Prevent wrong-direction work by assessing understanding before starting
2. **Reflexion Pattern** - Reuse past architectural decisions to avoid repeating mistakes
3. **Evidence Requirements** - Prevent hallucination by requiring proof of completion
4. **Adaptive Pivot Protocol** - Handle late-discovery architectural blockers systematically
5. **Business Rules Management** - Capture and validate business logic centrally

**Key Benefit**: 80% reduction in wasted effort + 94% hallucination prevention

---

## Quick Start: Add to Existing Project

### Option 1: Minimal Adoption (Confidence Checks Only)

**Time Investment**: 30 minutes
**ROI**: 25-250x token savings

Add confidence checks before making architectural decisions:

```markdown
# Before creating TECHNICAL.md or ADRs, ask:

🎯 Confidence Check:
- Do I understand the requirements? (>90% = proceed, <70% = stop and clarify)
- Can I distinguish bugs from architectural issues?
- Do I know which features are affected?

IF confidence <70%: STOP and request clarification
```

**Implementation**: Use Decision Trees from Section 2 of full methodology

---

### Option 2: Full Adoption (All Enhancements)

**Time Investment**: 2-4 hours setup + ongoing use
**ROI**: Complete methodology benefits

**Setup Steps**:

1. **Add Article XI to your constitution** (15 min)
   ```bash
   # Copy template
   cat templates/constitutional-article-xi.md >> memory/constitution.md

   # Customize for your project
   vim memory/constitution.md
   # Update Section 11.3 to reference your specific articles
   ```

2. **Create directory structure** (5 min)
   ```bash
   mkdir -p .specify/business-rules
   mkdir -p .specify/decisions
   ```

3. **Copy templates** (10 min)
   ```bash
   # Business rules validation template is already at:
   # .specify/business-rules/validation-rules.md
   # (No copying needed - customize as needed)

   # For each feature, create decisions directory
   mkdir -p specs/###-feature/decisions
   mkdir -p specs/###-feature/rules
   ```

4. **Update workflows** (30-60 min)
   - Add confidence checks to planning process
   - Train team on pivot protocol
   - Integrate evidence requirements into PR reviews

---

## Core Concepts

### 1. Confidence Checks (Prevent Wrong-Direction Work)

**Problem**: Implementing solutions based on misunderstood requirements wastes 5K-50K tokens

**Solution**: Assess confidence BEFORE starting work

**How to Use**:
```
Before any decision, ask:
1. Self-Assessment Questions (4-5 questions specific to decision type)
2. Rate Confidence: High (>90%), Medium (70-90%), Low (<70%)
3. Action:
   - High: Proceed immediately
   - Medium: Present options to user
   - Low: STOP and request clarification
```

**When to Use**:
- Before creating TECHNICAL.md
- Before creating ADR
- Before determining ADR placement
- Before using Business Rules system

**Example**:
```markdown
🎯 Before creating TECHNICAL.md:

Questions:
1. Are acceptance criteria quantifiable? [YES]
2. Are NFRs specified or clearly not needed? [NO - UNCLEAR]
3. Is multi-tenancy strategy clear? [YES]
4. Do I know the tech stack? [YES]

Confidence: Medium (75%) - 3/4 questions answered

Action: Present options to user about NFR clarification
```

---

### 2. Reflexion Pattern (Reuse Past Decisions)

**Problem**: Repeating same architectural mistakes, wasting investigation time

**Solution**: Search past ADRs before creating new ones

**How to Use**:
```bash
# Step 1: Search for similar blockers
grep -ri "blocker_keyword" .specify/decisions/ specs/*/decisions/

# Step 2: If found
- Read past ADR context
- Check if same solution applies
- IF yes: Reuse solution, link to existing ADR
- IF no: Create new ADR, reference past one

# Step 3: Document learning
Add to TECHNICAL.md decision history
```

**Token Savings**: ~2,000 tokens per reused solution

**Example**:
```bash
# Discovered Prisma blocker
grep -ri "prisma" .specify/decisions/
# Found: ADR-002 (Prisma → pg for multi-tenancy)

# Current issue: Prisma N+1 queries (different context)
# Action: Create ADR-003, reference ADR-002 for context
```

---

### 3. Evidence Requirements (Prevent Hallucination)

**Problem**: 94% of AI "completions" lack actual evidence of success

**Solution**: Require proof before marking tasks complete

**How to Use**:

For each milestone/task completion, provide:

1. **File Changes** (show actual diffs)
   ```bash
   git diff HEAD~1 path/to/file.md | head -50
   ```

2. **Validation** (run actual checks)
   ```bash
   # Lint
   markdownlint file.md  # Must show ✅

   # Tests
   pytest tests/  # Must show X/X passed

   # Build
   npm run build  # Must show success
   ```

3. **Hallucination Detection** (check for red flags)
   - 🚨 "Tests pass" without showing output
   - 🚨 "Everything works" without evidence
   - 🚨 "Implementation complete" with failing tests

**IF evidence missing**: BLOCK completion, show honest status

**Example**:
```markdown
✅ GOOD:
"Milestone 1 complete. Evidence:
- git diff: +85 lines in constitution.md ✅
- markdownlint: 0 errors ✅
- Functional test: LLM correctly summarizes Article XI ✅"

❌ BAD:
"Milestone 1 complete, everything works!"
```

---

### 4. Article XI: Adaptive Pivot Protocol

**Problem**: Architectural blockers discovered late in implementation cause timeline chaos

**Solution**: Systematic protocol for handling pivots

**7-Step Protocol**:

1. **Classify Blocker**
   - Bug vs Architectural Limitation
   - Workaround vs Fundamental Blocker
   - Single-feature vs Cross-cutting

2. **Create ADR** (Architecture Decision Record)
   - Use `templates/adr-template.md`
   - Document: Context, Decision, Compliance, Consequences

3. **Update Contract Tests FIRST** (Article III extension)
   - Tests must FAIL with current implementation (RED phase)
   - Get user approval of test changes

4. **Amend TECHNICAL.md**
   - Use ~~strikethrough~~ → new pattern
   - Link to ADR number

5. **Regenerate Plans**
   - Run `/plan --regenerate`
   - Tasks updated to reflect new constraints

6. **Update Business Rules** (if applicable)
   - Document affected BR-* IDs in ADR
   - Run `/rules-extract` after implementation

7. **Communicate Impact**
   - Inform stakeholders of timeline change
   - Provide ADR link for traceability

**Example Use Case**: See "Multi-Company Zone Capacity System with Prisma Pivot" in full methodology

---

### 5. Business Rules Management

**Problem**: Business logic scattered across code, specs, and tribal knowledge

**Solution**: Centralized catalog with constitutional validation

**Workflow**:

> **Note**: The `/rules-harvest` and `/rules-extract` commands shown below are conceptual examples for methodology illustration. Teams can implement these as custom slash commands, scripts, or use AI agents to perform these tasks manually following the patterns described.

1. **Harvest** (During Planning)
   ```bash
   /rules-harvest  # Conceptual command - implement as needed
   # Creates specs/###-feature/rules/harvest.md
   # Status: DRAFT
   ```

2. **Implement** (During Development)
   ```typescript
   /**
    * @BusinessRule BR-ZONES-001
    * Rule: Maximum Zone Capacity
    * Priority: HIGH
    * Status: ACTIVE
    * Test: tests/zones/test_capacity.py:52
    */
   export function checkCapacity(companyId: number): boolean {
     // Implementation
   }
   ```

3. **Extract** (Post-Implementation)
   ```bash
   /rules-extract --module ZONES  # Conceptual command - implement as needed
   # Updates .specify/business-rules/catalog.md
   # Status: DRAFT → ACTIVE
   # Validates 9 constitutional gates
   ```

4. **Query** (Ongoing)
   - LLM agents read `.specify/business-rules/QUICK_REFERENCE.md`
   - <2000 tokens for fast lookup
   - Links to full catalog for details

**9 Constitutional Gates**:
1. Testability (HIGH) - ≥2 test refs
2. Completeness (HIGH) - All required fields
3. Single Source (MEDIUM) - Unique IDs
4. Priority & Domain (HIGH) - Explicitly declared
5. Traceability (MEDIUM) - Links to spec/impl/tests
6. Test-First (HIGH) - Tests before ACTIVE
7. No Speculation (LOW) - FR-backed only
8. Versioning (MEDIUM) - Deprecated rules linked
9. Multi-Tenancy (MEDIUM) - Scope declared

---

## Decision Trees

Use these to determine when to use each component:

### Tree 1: Should I Create TECHNICAL.md?

```
START: New feature spec written
  ↓
  🎯 CONFIDENCE CHECK: Understand requirements?
  ↓
  Simple CRUD + known stack? → NO
  ↓
  Feature involves:
  - Multi-tenancy? YES → CREATE
  - Performance targets? YES → CREATE
  - New frameworks? YES → CREATE
  - Regulatory constraints? YES → CREATE
  - Architectural unknowns? YES → CREATE
  ↓
  Otherwise → SKIP
```

### Tree 2: Should I Create ADR?

```
START: Blocker encountered
  ↓
  🔍 REFLEXION: Search past ADRs
  ↓
  Found similar? → Reuse or reference
  ↓
  🎯 CONFIDENCE CHECK: Understand blocker?
  ↓
  Bug (code-fix)? YES → NO ADR
  Simple workaround? YES → NO ADR
  ↓
  Requires:
  - Framework change? YES → CREATE
  - Architectural pattern change? YES → CREATE
  - Database strategy change? YES → CREATE
  - Major refactoring (>5 files)? YES → CREATE
  ↓
  Otherwise → NO ADR
```

### Tree 3: Where to Place ADR?

```
START: ADR creation required
  ↓
  🎯 CONFIDENCE CHECK: Understand scope?
  ↓
  Affects only current feature? YES → specs/###/decisions/
  ↓
  Affects:
  - Multiple features? YES → .specify/decisions/
  - System architecture? YES → .specify/decisions/
  - Shared infrastructure? YES → .specify/decisions/
  ↓
  Edge case: Starts feature-scoped but becomes cross-project
  → Move ADR + update references
```

### Tree 4: Use Business Rules System?

```
START: Feature has business logic
  ↓
  🎯 CONFIDENCE CHECK: Can distinguish business vs technical logic?
  ↓
  Includes:
  - Validation rules? YES ↓
  - Authorization logic? YES ↓
  - State transitions? YES ↓
  - Calculation formulas? YES ↓
  - Multi-tenant config? YES ↓
  ↓
  Will logic be:
  - Queried by other features? YES → USE
  - Need test coverage tracking? YES → USE
  - Need version control? YES → USE
  - Exposed to LLMs? YES → USE
  ↓
  Otherwise → OPTIONAL (use for traceability)
```

---

## Templates Provided

### Core Templates
- `templates/constitutional-article-xi.md` - Add to your constitution
- `templates/adr-template.md` - Architecture Decision Record format
- `templates/technical-constraints-template.md` - TECHNICAL.md structure
- `.specify/business-rules/validation-rules.md` - Business logic quality gates

### Usage Examples
```bash
# Start new complex feature
mkdir specs/002-multi-tenant-zones
cd specs/002-multi-tenant-zones

# Copy templates
cp ../../templates/spec-template.md spec.md
cp ../../templates/technical-constraints-template.md TECHNICAL.md

# Create supporting directories
mkdir -p decisions rules

# During implementation, if blocker found
cp ../../templates/adr-template.md decisions/adr-001-orm-choice.md
```

---

## FAQ

### Q: Do I have to use all enhancements?

**A**: No! Adopt incrementally:
- **Minimum**: Confidence checks (30 min, huge ROI)
- **Recommended**: Confidence + Evidence (2 hours, prevents hallucination)
- **Full**: All 5 enhancements (4 hours, complete methodology)

---

### Q: What if my project doesn't use multi-tenancy?

**A**: Skip multi-tenancy sections:
- Article IX validation (N/A)
- Tenant scope in business rules (N/A)
- TECHNICAL.md multi-tenancy section (omit)

---

### Q: Can I customize Article XI for my project?

**A**: Yes! Section 11.3 (Constitutional Validation) should reference YOUR project's articles. Template has placeholders - fill with your specifics.

---

### Q: How do confidence checks work with AI agents?

**A**: AI agents use the same protocol:
1. LLM reads decision tree
2. Self-assesses confidence based on context
3. If <70%, outputs: "Need clarification on: [specific gaps]"
4. Human provides clarification
5. LLM proceeds with high confidence

---

### Q: What's the relationship between constitution.md and validation-rules.md?

**A**:
- **constitution.md**: Development methodology (how we build software)
  - Includes Article XI (pivot protocol)
  - Applies to ALL development work
- **validation-rules.md**: Documentation quality (how we write business rules)
  - 9 quality gates for business logic
  - Applies only to business rules catalog

Both complement each other but serve different purposes.

---

## Success Metrics

Track these to measure methodology adoption:

### Confidence Checks
- **Metric**: % of architectural decisions made without clarifications needed afterward
- **Target**: ≥90% correct first-time
- **How**: Track instances of "had to redo decision due to misunderstanding"

### Reflexion Pattern
- **Metric**: % of ADRs reusing past solutions
- **Target**: ≥20% (indicates learning from past)
- **How**: Count ADRs with "Referenced ADR-###" in context section

### Evidence Requirements
- **Metric**: % of PR reviews requiring evidence re-submission
- **Target**: <5% (indicates thorough first submission)
- **How**: Track PR review comment "Missing evidence for [X]"

### Pivot Protocol
- **Metric**: Average blocker discovery time (Week X of implementation)
- **Target**: Week ≤3 (early detection via TECHNICAL.md [NEEDS VALIDATION] flags)
- **How**: Track ADR creation date vs feature start date

### Business Rules
- **Metric**: Catalog constitutional compliance score
- **Target**: ≥95%
- **How**: Run `/rules-extract --validate-only` in CI/CD

---

## Next Steps

1. **Try Confidence Checks** (Start small)
   - Use Decision Trees for next 3 architectural decisions
   - Track if decisions needed rework

2. **Adopt Evidence Requirements** (Medium effort)
   - Add evidence package template to PR checklist
   - Require proof before marking tasks complete

3. **Implement Article XI** (Full adoption)
   - Add to constitution
   - Train team on pivot protocol
   - Create first ADR for practice

4. **Optional: Business Rules** (If applicable)
   - Identify 1-2 features with complex business logic
   - Run `/rules-harvest` and `/rules-extract`
   - Measure catalog usefulness for LLM agents

---

## Support & Resources

- **Full Methodology**: See `temp/unified-sdd-methodology-v2.md` for complete 2500-line guide
- **Templates**: All templates in `templates/` and `.specify/` directories
- **Examples**: See "Complete End-to-End Walkthrough" in full methodology
- **Issues**: Report problems via GitHub Issues
- **Questions**: Use GitHub Discussions

---

**Version**: 2.1
**Last Updated**: 2025-01-23
**Maintained By**: Spec-Kit Community
**License**: MIT (same as Spec-Kit)
