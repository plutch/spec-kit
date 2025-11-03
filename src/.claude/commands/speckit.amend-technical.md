---
description: Amend the TECHNICAL.md file to reflect architectural changes documented in an ADR, using strikethrough for old constraints and linking to decision rationale
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

You are amending a TECHNICAL.md file to reflect an architectural decision documented in an ADR. TECHNICAL.md is a living document that captures technical constraints and evolves through amendments (strikethrough ~~old~~ → new) while preserving decision history.

Follow this execution flow:

### 1. Determine TECHNICAL.md Location

**Ask the user if not clear from input**:
- Is this amendment **feature-scoped** or **cross-project**?
  - Feature-scoped: `specs/###-feature-name/TECHNICAL.md`
  - Cross-project: `.specify/TECHNICAL.md`

**Check if file exists**:
```bash
# Feature-scoped
ls specs/###-feature-name/TECHNICAL.md 2>/dev/null

# Cross-project
ls .specify/TECHNICAL.md 2>/dev/null
```

**If file does NOT exist**:
- Offer to create from template: `.specify/templates/adaptive/technical-constraints-template.md`
- OR inform user to run `/plan` which creates TECHNICAL.md for complex features
- STOP - do not proceed with amendment until file exists

### 2. Identify ADR Triggering Amendment

**Collect from user** (or check recent ADRs):
- ADR number (e.g., ADR-002)
- ADR title
- ADR path (e.g., `specs/000001-zones/decisions/adr-002-prisma-to-pg.md`)

**Verify ADR exists**:
```bash
ls [ADR_PATH] 2>/dev/null
```

**Read ADR** to understand the decision:
```bash
cat [ADR_PATH]
```

**Extract from ADR**:
- FROM → TO change
- Rationale for change
- Which section of TECHNICAL.md is affected

### 3. Load Current TECHNICAL.md

Read the existing file:
```bash
cat [TECHNICAL_MD_PATH]
```

**Parse structure**:
- Identify all ## sections (Tech Stack, Constraints, Risk Flags, Decision History)
- Locate the specific section being amended

### 4. Determine Amendment Type

**Ask user to classify** (or infer from ADR):

**Type 1: Technology Change**
- Example: Prisma ORM → node-postgres
- Section: Usually "Tech Stack" or "Data Persistence"

**Type 2: Constraint Modification**
- Example: Change latency requirement 100ms → 50ms
- Section: Usually "Performance Constraints" or "Non-Functional Requirements"

**Type 3: Risk Flag Resolution**
- Example: Resolve [NEEDS VALIDATION] flag with actual decision
- Section: Usually "Risk Flags" or "Unresolved Decisions"

**Type 4: Pattern Change**
- Example: Multi-tenancy strategy: RLS → Schema isolation
- Section: Usually "Architectural Patterns" or "Design Decisions"

### 5. Apply Amendment with Strikethrough

**Amendment Format**:
```markdown
## [Section Name]

~~[Old constraint text]~~ **DEPRECATED** via [ADR-NNN](decisions/adr-NNN-title.md) (YYYY-MM-DD)

**Updated Constraint**: [New constraint text]
```

**Examples**:

**Technology Change**:
```markdown
## Data Persistence

~~**ORM**: Prisma with connection pooling per tenant~~ **DEPRECATED** via [ADR-002](decisions/adr-002-prisma-to-pg.md) (2025-01-15)

**Database Driver**: node-postgres with `SET search_path` per request
**Multi-tenancy Strategy**: PostgreSQL schemas (one per company)
**Rationale**: Prisma connection pooling caused 500ms latency under load (see ADR-002)
```

**Constraint Modification**:
```markdown
## Performance Constraints

~~**API Latency**: p99 < 100ms~~ **AMENDED** via [ADR-003](decisions/adr-003-caching-strategy.md) (2025-01-20)

**API Latency**: p99 < 50ms (tightened requirement)
**Cache Layer**: Redis for hot-path queries
**Rationale**: User testing showed 100ms too slow for UX requirements (see ADR-003)
```

**Risk Flag Resolution**:
```markdown
## Risk Flags

~~[NEEDS VALIDATION] **Multi-tenant Scaling**: Current ORM may not support 10,000+ schemas~~ **RESOLVED** via [ADR-002](decisions/adr-002-prisma-to-pg.md) (2025-01-15)

**Resolution**: Load testing confirmed node-postgres handles 10,000+ schemas with <10ms overhead per schema switch
**Validation Evidence**: [Load test results](docs/load-test-2025-01-14.md)
```

### 6. Update Decision History Section

**Add chronological entry** to "Decision History" section:

```markdown
## Decision History

[Existing entries remain unchanged...]

#### YYYY-MM-DD: [Brief Change Description]
- **Trigger**: [What blocker or need triggered this change]
- **Change**: ~~[Old approach]~~ → [New approach]
- **ADR**: [ADR-NNN - Title](decisions/adr-NNN-title.md)
- **Impact**: [Timeline impact, files affected, business rules affected]
- **Business Rules**: [List BR-* IDs if applicable]
```

**Example**:
```markdown
#### 2025-01-15: Database Layer Pivot - Prisma to node-postgres
- **Trigger**: Prisma multi-tenant connection pooling caused 500ms latency (Week 3 testing)
- **Change**: ~~Prisma ORM~~ → node-postgres with schema-based isolation
- **ADR**: [ADR-002 - Migrate to node-postgres](decisions/adr-002-prisma-to-pg.md)
- **Impact**: +2 weeks timeline, 15 test files updated, 8 service files refactored
- **Business Rules**: BR-ZONES-001, BR-ZONES-003 (implementation paths changed)
```

### 7. Validate Amendment

**Checklist**:
- [ ] Original constraint uses strikethrough: ~~old text~~
- [ ] ADR link is valid and points to correct file
- [ ] Date is in ISO format: YYYY-MM-DD
- [ ] New constraint is clearly stated
- [ ] Rationale references ADR
- [ ] Decision History entry added chronologically
- [ ] No orphaned text (all old constraints marked deprecated)

**Run validation**:
```bash
# Check ADR link works
ls [ADR_PATH]
# Expected: File exists

# Check for any dangling [NEEDS VALIDATION] that should have been resolved
grep "\[NEEDS VALIDATION\]" [TECHNICAL_MD_PATH]
# Expected: Only unresolved flags remain (if any)
```

### 8. Cross-Reference Business Rules (If Applicable)

**If ADR affected business rules**:

Read ADR's "Affected Business Rules" section:
```bash
grep -A 20 "## Affected Business Rules" [ADR_PATH]
```

**Add business rules note to Decision History entry**:
```markdown
#### YYYY-MM-DD: [Change Description]
...
- **Business Rules**: BR-ZONES-001 (impl: src/zones/pg.ts:38), BR-ZONES-003 (impl: src/middleware/schema.ts:18)
- **Catalog Update**: Run `/rules-extract --module ZONES` after implementation
```

### 9. Determine Regeneration Scope

**After amending TECHNICAL.md, determine what needs regeneration**:

**If change affects implementation approach**:
- `plan.md` must be regenerated (reads TECHNICAL.md constraints)
- `tasks.md` must be regenerated (follows updated plan)

**If change only documents past decision**:
- No regeneration needed (already implemented)

**Output regeneration instructions**:
```markdown
## Regeneration Required

TECHNICAL.md has been amended. The following files MUST be regenerated:

1. **plan.md** (reads TECHNICAL.md constraints):
   ```bash
   /plan --regenerate
   ```
   Expected: plan.md updated to use node-postgres instead of Prisma

2. **tasks.md** (follows updated plan):
   ```bash
   /tasks
   ```
   Expected: tasks.md includes migration tasks for Prisma → pg
```

### 10. Write Amended TECHNICAL.md

**Save changes**:
- Overwrite existing TECHNICAL.md with amended version
- Preserve all sections and structure
- Maintain chronological order in Decision History

**Validate before writing**:
```bash
# Check markdown syntax
# Ensure no duplicate section headers
# Verify all internal links work
```

### 11. ADR Amendment Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate TECHNICAL.md amendment completeness before presenting results.

### Evidence Collection (Mandatory)

❓ **"Was ADR file created/verified?"**
Action Required:
  - Verify ADR file exists at specified path
  - Show ACTUAL ADR path and size
  - Report: ADR number, title, file location

Expected Evidence:
  ✓ ADR path: specs/###-feature/decisions/adr-NNN-*.md OR ADR/ADR-NNN-*.md
  ✓ ADR exists and readable
  ✓ ADR number: NNN

❓ **"Does ADR follow template structure?"**
Action Required:
  - Check ADR has required sections
  - Show ACTUAL sections present
  - Report: Context, Decision, Consequences sections verified

Expected Evidence:
  ✓ ADR has ## Context section
  ✓ ADR has ## Decision section
  ✓ ADR has ## Consequences section
  ✓ ADR documents FROM → TO change

❓ **"Was TECHNICAL.md amended correctly?"**
Action Required:
  - Verify strikethrough applied to old constraint
  - Show ACTUAL amendment diff
  - Report: Section amended, old → new change

Expected Evidence:
  ✓ Old constraint marked: ~~old text~~ **DEPRECATED** via [ADR-NNN]
  ✓ New constraint documented
  ✓ ADR link is valid markdown format
  ✓ Date in ISO format (YYYY-MM-DD)

❓ **"Was Decision History updated?"**
Action Required:
  - Verify new entry added to Decision History section
  - Show ACTUAL history entry
  - Report: Date, trigger, change, ADR link, impact

Expected Evidence:
  ✓ Decision History entry added
  ✓ Entry includes: Date, Trigger, Change, ADR link, Impact
  ✓ Chronological order maintained
  ✓ Business rules referenced (if applicable)

IF any evidence is MISSING:
  ❌ CANNOT report completion
  → Gather missing evidence first
  → Re-run this step with complete evidence

### Hallucination Prevention (7 Red Flags for ADR Amendment)

```yaml
Detect and BLOCK these patterns:

🚨 "ADR created" WITHOUT showing file path
   → Self-correction: "Wait, I need to verify ADR file exists"

🚨 "Template followed" WITHOUT checking required sections
   → Self-correction: "Let me verify Context, Decision, Consequences exist"

🚨 "TECHNICAL.md amended" WITHOUT showing actual diff
   → Self-correction: "I need to show old → new change"

🚨 "Decision documented" WITHOUT consequences section
   → Self-correction: "Must verify Consequences section exists in ADR"

🚨 Claiming "complete" WITHOUT state.json update
   → Self-correction: "Need to verify last_adr_number incremented"

🚨 Creating ADR WITHOUT unique number
   → Self-correction: "Must check existing ADRs to assign next number"

🚨 "File written" WITHOUT verifying content
   → Self-correction: "I need to read back file to confirm write succeeded"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

✅ **READY for Commit**:
```yaml
Criteria (ALL must be met):
  - ADR file created with valid template structure
  - TECHNICAL.md amended with strikethrough ~~old~~ → new
  - ADR link is valid and points to correct file
  - Decision History entry added with all required fields
  - Date in ISO format (YYYY-MM-DD)
  - State.json updated (last_adr_number incremented)

IF ALL criteria met:
  → Proceed to final summary output
```

⚠️ **NEEDS REVIEW** (can proceed with warnings):
```yaml
Criteria:
  - Missing optional sections in ADR (Status, Alternatives Considered)
  - Technical debt not tracked (if applicable)
  - Regeneration scope unclear (user decision needed)

IF criteria met:
  → Present warnings to user
  → Ask for confirmation before committing
```

❌ **NOT READY** (more work needed):
```yaml
Criteria (ANY triggers NOT READY):
  - ADR file creation failed
  - ADR missing required sections (Context, Decision, Consequences)
  - TECHNICAL.md not amended (old constraint not marked deprecated)
  - Decision History entry missing
  - Invalid ADR link or date format

IF NOT READY:
  → Present issues with evidence
  → Recommend: "Fix ADR/TECHNICAL.md issues before proceeding"
  → STOP workflow progression
```

### Output Format (Present to User - ONLY if evidence provided)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 TECHNICAL.md Amendment Review Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**ADR Created**: [ADR-NNN - Title]
**Path**: [path/to/adr-NNN.md]
**Template Sections**:
  ✓ Context
  ✓ Decision
  ✓ Consequences

**TECHNICAL.md Amended**:
**File**: [path/to/TECHNICAL.md]
**Section**: [Section Name]
**Change**: ~~Old constraint~~ → New constraint
**ADR Link**: [ADR-NNN](path/to/adr-NNN.md) (YYYY-MM-DD)

**Decision History Entry**:
```markdown
#### YYYY-MM-DD: [Brief Change Description]
- **Trigger**: [Blocker/need that triggered change]
- **Change**: ~~Old approach~~ → New approach
- **ADR**: [ADR-NNN - Title](decisions/adr-NNN.md)
- **Impact**: [Timeline/scope impact]
- **Business Rules**: [BR-* IDs if applicable]
```

**State Updated**:
  ✓ last_adr_number: [N-1] → [N]

**Regeneration Required**:
[IF change affects implementation]
  - plan.md: Run /plan --regenerate
  - tasks.md: Run /tasks

[IF change only documents past decision]
  - No regeneration needed (already implemented)

Next Action: [Commit with suggested message OR Regenerate plan/tasks]

**Suggested Commit Message**:
```
docs(technical): amend TECHNICAL.md per ADR-NNN

~~Old approach~~ → New approach

- Trigger: [Brief blocker description]
- ADR: ADR-NNN
- Impact: [Brief timeline/scope impact]
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 12. Update Constitutional Compliance (If Cross-Project)

**If this is cross-project TECHNICAL.md** (`.specify/TECHNICAL.md`):

Check if `memory/constitution.md` needs updating:
- If amendment changes a fundamental principle (e.g., "We never use ORMs")
- If amendment violates an existing constitutional article

**If constitutional conflict**:
- Flag for user review
- Suggest either:
  - Amend constitution to reflect new reality
  - Revert TECHNICAL.md change and re-evaluate ADR

### 12. Output Summary to User

**Provide**:
```markdown
## TECHNICAL.md Amended

**File**: `[path/to/TECHNICAL.md]`
**ADR**: [ADR-NNN - Title](path/to/adr-NNN.md)
**Date**: YYYY-MM-DD

### Changes Applied

**Section**: [Section name]
**Change**: ~~Old constraint~~ → New constraint
**Rationale**: [Brief summary from ADR]

### Decision History Entry Added

```markdown
[Show the added decision history entry]
```

### Next Steps

1. **Regenerate Planning Artifacts**:
   ```bash
   /plan --regenerate  # Reads amended TECHNICAL.md
   /tasks             # Uses regenerated plan
   ```

2. **Update Business Rules Catalog** (if applicable):
   ```bash
   /rules-extract --module [AFFECTED_MODULES] --validate
   ```

3. **Verify Compliance**:
   - [ ] plan.md reflects new constraints
   - [ ] tasks.md includes migration tasks
   - [ ] No constitutional conflicts introduced

### Suggested Commit Message

```
docs(technical): amend TECHNICAL.md per ADR-NNN

~~Old approach~~ → New approach

- Trigger: [Brief blocker description]
- ADR: ADR-NNN
- Impact: [Brief timeline/scope impact]
```
```

## Formatting & Style Requirements

- Use strikethrough for deprecated constraints: `~~old text~~`
- ADR links in markdown format: `[ADR-NNN - Title](path/to/adr-NNN.md)`
- Dates in ISO format: YYYY-MM-DD
- Decision History in reverse chronological order (newest first OR oldest first - be consistent)
- Business rule IDs in format: BR-DOMAIN-NUMBER
- Keep amendments inline (do not create separate "Amendments" section)

## Error Handling

**If TECHNICAL.md does not exist**:
```markdown
⚠️ TECHNICAL.md not found at `[expected/path]`

**Options**:
1. Create from template:
   ```bash
   /create-technical
   ```
2. Run `/plan` which creates TECHNICAL.md for complex features
3. This feature may not need TECHNICAL.md (CRUD features can skip)

**Cannot proceed with amendment until file exists.**
```

**If ADR not found**:
```markdown
⚠️ ADR not found at `[expected/path]`

Please create the ADR first:
```bash
/create-adr
```

Amendments MUST reference an approved ADR (Article XI.2).
```

**If section not found in TECHNICAL.md**:
- List all available sections
- Ask user which section to amend OR if new section needed
- Provide guidance on section naming conventions

## Notes

- TECHNICAL.md is a **living document** - amendments are expected and normal
- Original constraints are **never deleted** - always use strikethrough
- Every amendment **MUST** link to an ADR - no undocumented changes
- Decision History provides **audit trail** for future developers
- After amendment, `plan.md` and `tasks.md` **SHOULD** be regenerated
- Business rules catalog **MUST** be updated if rules are affected (Article XI.6)

## Integration with Article XI (Technical Pivot Protocol)

This command implements **Article XI.5: Constraint Amendment**:

> After ADR approval:
> - Update `TECHNICAL.md` with amended constraints (~~old~~ → new)
> - Link to ADR number with rationale
> - Preserve historical decision trail
> - Regenerate plan.md + tasks.md via `/plan --regenerate`

Ensure all amendments follow this protocol for constitutional compliance.
