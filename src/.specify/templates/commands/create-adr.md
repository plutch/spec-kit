---
description: Create an Architecture Decision Record (ADR) documenting an architectural pivot or decision, with constitutional validation and business rules impact assessment
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

You are creating an Architecture Decision Record (ADR) to document an architectural decision or pivot. ADRs are immutable historical records that capture the context, decision, rationale, and consequences of significant architectural choices.

Follow this execution flow:

### 1. Determine ADR Scope and Location

**Ask the user if not clear from input**:
- Is this decision **feature-scoped** or **cross-project**?
  - Feature-scoped: Affects only one feature (e.g., `specs/001-photo-albums/`)
  - Cross-project: Affects multiple features or the entire project

**Set ADR location**:
- Feature-scoped: `specs/###-feature-name/decisions/adr-NNN-title.md`
- Cross-project: `.specify/decisions/adr-NNN-title.md`

**Determine ADR number** (NNN):
```bash
# Count existing ADRs in target directory
ls [TARGET_DIR]/adr-*.md 2>/dev/null | wc -l
# Next number = count + 1 (zero-padded to 3 digits: 001, 002, etc.)
```

### 2. Classify the Decision (Article XI.1)

**Ask the user to classify** (or infer from context):
- **Trigger**: [ ] Implementation blocker | [ ] Performance issue | [ ] Security concern | [ ] Proactive decision
- **Type**: [ ] Technology swap | [ ] Pattern change | [ ] Architecture refactor | [ ] Framework addition
- **Scope**: [ ] Single feature | [ ] Cross-cutting concern
- **NFR Impact**: [ ] Violates NFR | [ ] No NFR impact

**If this is a pivot** (blocker discovered during implementation):
- Use template: `.specify/templates/adaptive/adr-pivot-template.md`

**Otherwise** (planned decision):
- Use template: `.specify/templates/adr-template.md`

### 3. Gather Required Information

**Collect from user** (interactive prompts if missing):

**Context Section**:
- Problem statement: What issue or need triggered this decision?
- Current state: What exists today?
- Discovery point: When was this identified? (e.g., "Week 3 of implementation during testing")
- Impact: Which features/modules are affected?

**Decision Section**:
- FROM → TO: What is changing? (technology, pattern, approach)
- Rationale: Why is this the best choice?
- Alternatives considered: What other options were evaluated and why rejected?

**Constitutional Compliance** (Article XI.3):
For EACH constitutional article in the project's `memory/constitution.md`:
- Does this decision comply with [Article Name]?
- Assessment: PASS | CONDITIONAL PASS | FAIL
- If CONDITIONAL or FAIL: What mitigation or approval is needed?

**Business Rules Impact** (Article XI.6):
- Are any existing business rules affected?
- If YES: For each rule, collect:
  - Rule ID (BR-DOMAIN-NUMBER)
  - Implementation path changes (FROM file:line → TO file:line)
  - Test reference changes (FROM test:line → TO test:line)
- Are any new business rules introduced by this decision?
- Are any rules being deprecated?

**Test-First Requirements** (Article XI.4):
- Have contract tests been updated to reflect this decision?
- Do tests show RED phase with current implementation?
- Has user approved test changes?

**Timeline Impact** (Article XI.7):
- Original timeline: [weeks/dates]
- Impact: [+/- time]
- New timeline: [weeks/dates]
- Rationale for impact

### 4. Load and Fill Template

Read the appropriate template:
```bash
# For pivots:
cat .specify/templates/adaptive/adr-pivot-template.md

# For planned decisions:
cat .specify/templates/adr-template.md
```

**Fill all sections**:
- Replace placeholders with collected information
- Complete ALL checklists (check boxes: `[X]` for done, `[ ]` for pending)
- Ensure no `[PLACEHOLDER]` tokens remain
- Add concrete code examples, file paths, commands

### 5. Constitutional Validation (Article XI.3)

Read `memory/constitution.md` and validate decision against each article:

**Common Articles to Check**:
- Article VII: Simplicity Gate (does this add unnecessary complexity?)
- Article VIII: Anti-Abstraction Gate (are we creating unnecessary wrappers?)
- Article IX: Integration-First Gate (have contract tests been updated FIRST?)
- Article III: Test-First Philosophy (RED → GREEN → REFACTOR cycle)

**For each article**:
```markdown
### [Article Name]
**Assessment**: [PASS | CONDITIONAL PASS | FAIL]
- [Explain compliance or required mitigation]
- [If architect approval needed, note status]
```

**If ANY article fails**:
- Mark ADR status as "Proposed" (not "Accepted")
- Document required approvals in ADR
- Block implementation until approvals obtained

### 6. Business Rules Integration (Article XI.6)

**If business rules are affected**:

Read existing business rules catalog:
```bash
cat .specify/business-rules/catalog.md 2>/dev/null
```

**For each affected rule** (BR-DOMAIN-NUMBER):
1. Verify rule exists in catalog
2. Document implementation path change (FROM → TO)
3. Document test reference change (FROM → TO)
4. Add reminder to run `/rules-extract --module DOMAIN` after implementation

**Add to ADR**:
```markdown
## Affected Business Rules (Article XI.6)

### Modified Rules
- **BR-ZONES-001**: Maximum Zone Capacity
  - Implementation: FROM `src/zones/prisma.ts:45` → TO `src/zones/pg.ts:38`
  - Tests: FROM `tests/zones_prisma.py:52` → TO `tests/zones_pg.py:48`
  - Action: `/rules-extract --module ZONES` after implementation

### Post-Implementation Checklist
- [ ] Run `/rules-extract --module ZONES --validate`
- [ ] Verify catalog updated with new implementation paths
- [ ] Verify all constitutional gates pass for affected rules
```

### 7. TECHNICAL.md Amendment (If Applicable)

**If this decision changes technical constraints**:

Check if `TECHNICAL.md` exists:
```bash
# Feature-scoped
ls specs/###-feature-name/TECHNICAL.md 2>/dev/null

# Cross-project
ls .specify/TECHNICAL.md 2>/dev/null
```

**If exists**, add amendment instructions to ADR:
```markdown
## TECHNICAL.md Amendment (Article XI.5)

After ADR acceptance, update TECHNICAL.md:

**Section**: [Section name, e.g., "Database Access"]

**Amendment**:
```diff
- Current text about old approach
+ ~~Current text about old approach~~ [DEPRECATED via ADR-NNN]
+ New text about new approach
```

**Decision History Entry**:
```markdown
#### YYYY-MM-DD: [Change description]
- Trigger: [Blocker description]
- Change: ~~Old~~ → New
- ADR: [ADR-NNN](decisions/adr-NNN-title.md)
- Impact: [Timeline/files affected]
```

**Regeneration Required**:
```bash
/plan --regenerate  # Reads amended TECHNICAL.md
/tasks             # Uses regenerated plan
```

### 8. Create Implementation Checklist

**Add pre-implementation checklist to ADR**:
```markdown
## Implementation Checklist

### Pre-Implementation (MUST complete before coding)
- [ ] Contract tests updated with new expectations
- [ ] Tests run and show RED phase (current impl fails new tests)
- [ ] User approved test changes
- [ ] TECHNICAL.md amended (if applicable)
- [ ] plan.md regenerated (if TECHNICAL.md changed)
- [ ] tasks.md regenerated (if plan changed)

### During Implementation
- [ ] Implement changes following tasks.md
- [ ] Tests transition RED → GREEN
- [ ] Business rule code annotations updated
- [ ] New implementation paths verified

### Post-Implementation
- [ ] Run /rules-extract for affected modules
- [ ] Verify catalog updates successful
- [ ] Verify all constitutional gates pass
- [ ] Update timeline if needed
```

### 9. Write ADR File

**Create the ADR file**:
```bash
# Determine path based on scope
TARGET_PATH="[specs/###-feature/decisions/ OR .specify/decisions/]adr-NNN-title.md"

# Write filled template to file
```

**Validate**:
- No `[PLACEHOLDER]` tokens remaining
- All checklists have concrete items
- File paths are valid and specific
- Commands shown are runnable
- Constitutional compliance documented
- Business rules impact documented

### 10. Create Directories if Needed

**Ensure directories exist**:
```bash
# Feature-scoped
mkdir -p specs/###-feature-name/decisions/

# Cross-project
mkdir -p .specify/decisions/
```

### 11. Output Summary to User

**Provide**:
```markdown
## ADR Created: ADR-NNN - [Title]

**Location**: `[full/path/to/adr-NNN-title.md]`
**Status**: [Proposed | Accepted]
**Scope**: [Feature-scoped | Cross-project]

### Constitutional Compliance
- [List articles assessed]
- [Any CONDITIONAL/FAIL assessments requiring approvals]

### Business Rules Impact
- Modified: [count] rules
- New: [count] rules
- Deprecated: [count] rules

### Next Steps
1. [ ] Obtain required approvals (if status=Proposed)
2. [ ] Update contract tests (Article XI.4)
3. [ ] Amend TECHNICAL.md (if applicable)
4. [ ] Regenerate plan.md and tasks.md
5. [ ] Implement following tasks.md
6. [ ] Run /rules-extract after implementation

### Suggested Commit Message
```
docs(adr): add ADR-NNN [title]

[Brief 1-2 sentence description of decision]

- Scope: [feature/cross-project]
- Impact: [brief timeline/component impact]
- Status: [Proposed/Accepted]
```
```

## Formatting & Style Requirements

- Use Markdown headings exactly as in the template
- Code blocks must have language specifiers (```bash, ```typescript, etc.)
- File paths must be absolute from repo root (no relative paths)
- Checkboxes: `[ ]` for incomplete, `[X]` for complete
- Dates in ISO format: YYYY-MM-DD
- Rule IDs in format: BR-DOMAIN-NUMBER (e.g., BR-ZONES-001)

## Error Handling

**If missing critical information**:
- Ask user for clarification (do not guess)
- Provide examples to guide user response
- Explain why the information is needed

**If constitutional article fails**:
- Mark ADR status as "Proposed"
- Document required approvals
- Provide guidance on approval process

**If template not found**:
- Use general adr-template.md as fallback
- Inform user which template was expected vs. used

## Notes

- ADRs are **immutable** after acceptance - never edit, only supersede with new ADR
- All architectural pivots **MUST** be documented via ADR (Article XI.2)
- Test-first is **non-negotiable** - tests update BEFORE implementation (Article XI.4)
- Business rules catalog **MUST** be updated post-implementation (Article XI.6)
