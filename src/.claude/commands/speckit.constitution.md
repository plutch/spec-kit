---
description: Create or update the project constitution from interactive or provided principle inputs, ensuring all dependent templates stay in sync
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Outline

You are updating the project constitution at `.specify/memory/constitution.md`. This file is a TEMPLATE containing placeholder tokens in square brackets (e.g. `[PROJECT_NAME]`, `[PRINCIPLE_1_NAME]`). Your job is to (a) collect/derive concrete values, (b) fill the template precisely, and (c) propagate any amendments across dependent artifacts.

Follow this execution flow:

1. Load the existing constitution template at `.specify/memory/constitution.md`.
   - Identify every placeholder token of the form `[ALL_CAPS_IDENTIFIER]`.
   **IMPORTANT**: The user might require less or more principles than the ones used in the template. If a number is specified, respect that - follow the general template. You will update the doc accordingly.

2. Collect/derive values for placeholders:
   - If user input (conversation) supplies a value, use it.
   - Otherwise infer from existing repo context (README, docs, prior constitution versions if embedded).
   - For governance dates: `RATIFICATION_DATE` is the original adoption date (if unknown ask or mark TODO), `LAST_AMENDED_DATE` is today if changes are made, otherwise keep previous.
   - `CONSTITUTION_VERSION` must increment according to semantic versioning rules:
     - MAJOR: Backward incompatible governance/principle removals or redefinitions.
     - MINOR: New principle/section added or materially expanded guidance.
     - PATCH: Clarifications, wording, typo fixes, non-semantic refinements.
   - If version bump type ambiguous, propose reasoning before finalizing.

3. Draft the updated constitution content:
   - Replace every placeholder with concrete text (no bracketed tokens left except intentionally retained template slots that the project has chosen not to define yet—explicitly justify any left).
   - Preserve heading hierarchy and comments can be removed once replaced unless they still add clarifying guidance.
   - Ensure each Principle section: succinct name line, paragraph (or bullet list) capturing non‑negotiable rules, explicit rationale if not obvious.
   - Ensure Governance section lists amendment procedure, versioning policy, and compliance review expectations.

4. Consistency propagation checklist (convert prior checklist into active validations):
   - Read `.specify/templates/plan-template.md` and ensure any "Constitution Check" or rules align with updated principles.
   - Read `.specify/templates/spec-template.md` for scope/requirements alignment—update if constitution adds/removes mandatory sections or constraints.
   - Read `.specify/templates/tasks-template.md` and ensure task categorization reflects new or removed principle-driven task types (e.g., observability, versioning, testing discipline).
   - Read each command file in `.specify/templates/commands/*.md` (including this one) to verify no outdated references (agent-specific names like CLAUDE only) remain when generic guidance is required.
   - Read any runtime guidance docs (e.g., `README.md`, `docs/quickstart.md`, or agent-specific guidance files if present). Update references to principles changed.

5. Produce a Sync Impact Report (prepend as an HTML comment at top of the constitution file after update):
   - Version change: old → new
   - List of modified principles (old title → new title if renamed)
   - Added sections
   - Removed sections
   - Templates requiring updates (✅ updated / ⚠ pending) with file paths
   - Follow-up TODOs if any placeholders intentionally deferred.

6. Validation before final output:
   - No remaining unexplained bracket tokens.
   - Version line matches report.
   - Dates ISO format YYYY-MM-DD.
   - Principles are declarative, testable, and free of vague language ("should" → replace with MUST/SHOULD rationale where appropriate).

7. Write the completed constitution back to `.specify/memory/constitution.md` (overwrite).

8. **Constitution Creation Review Gate (Evidence-Based Self-Check)**

**Purpose**: Validate constitution creation completeness before presenting results.

### Evidence Collection (Mandatory)

❓ **"Was constitution file created?"**
Action Required:
  - Verify file exists at `.specify/memory/constitution.md`
  - Show ACTUAL file path and size
  - Report: File location and byte count

Expected Evidence:
  ✓ File path: `.specify/memory/constitution.md`
  ✓ File size: >1KB (meaningful content)
  ✓ File accessible: No permission errors

❓ **"Were all principles evaluated?"**
Action Required:
  - Count principle sections in constitution
  - Show ACTUAL principle count
  - Report: List of principle names

Expected Evidence:
  ✓ Principle count: [N] principles documented
  ✓ Each principle has: Name, rules, rationale
  ✓ No placeholder tokens remain (except intentionally deferred)

❓ **"Are governance sections complete?"**
Action Required:
  - Verify governance sections exist
  - Show ACTUAL section list
  - Report: Amendment procedure, versioning policy, compliance review

Expected Evidence:
  ✓ Amendment procedure documented
  ✓ Versioning policy defined
  ✓ Compliance review expectations specified

❓ **"Was version incremented correctly?"**
Action Required:
  - Show version change (old → new)
  - Justify version bump type (MAJOR/MINOR/PATCH)
  - Report: Version and rationale

Expected Evidence:
  ✓ Version: X.Y.Z → X'.Y'.Z'
  ✓ Bump type justified (MAJOR/MINOR/PATCH)
  ✓ Ratification/amendment dates in ISO format

IF any evidence is MISSING:
  ❌ CANNOT report completion
  → Gather missing evidence first
  → Re-run this step with complete evidence

### Hallucination Prevention (7 Red Flags for Constitution Creation)

```yaml
Detect and BLOCK these patterns:

🚨 "Constitution created" WITHOUT showing file path
   → Self-correction: "Wait, I need to verify file was actually written"

🚨 "All principles evaluated" WITHOUT showing principle count
   → Self-correction: "Let me count and list actual principles"

🚨 "Version incremented" WITHOUT justifying bump type
   → Self-correction: "I need to explain why MAJOR/MINOR/PATCH"

🚨 Claiming "complete" WITHOUT checking for placeholder tokens
   → Self-correction: "Must verify no unexplained [BRACKETS] remain"

🚨 "Governance sections added" WITHOUT listing what was added
   → Self-correction: "I need to show actual governance content"

🚨 "Sync Impact Report created" WITHOUT showing affected templates
   → Self-correction: "Must list templates requiring updates"

🚨 "Ready to commit" WITHOUT suggested commit message
   → Self-correction: "Need to provide commit message template"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

✅ **READY for Commit**:
```yaml
Criteria (ALL must be met):
  - Constitution file created with valid content (>1KB)
  - All principles documented with rationale
  - Governance sections complete
  - Version incremented with justification
  - No unexplained placeholder tokens
  - Sync Impact Report prepended to file
  - Suggested commit message provided

IF ALL criteria met:
  → Proceed to final output
```

⚠️ **NEEDS REVIEW** (can proceed with warnings):
```yaml
Criteria:
  - Some placeholders intentionally deferred (with TODO comments)
  - Templates flagged for manual follow-up
  - Version bump type ambiguous (user decision needed)

IF criteria met:
  → Present deferred items to user
  → Ask for confirmation before committing
```

❌ **NOT READY** (more work needed):
```yaml
Criteria (ANY triggers NOT READY):
  - File creation failed
  - No principles documented
  - Governance sections missing
  - Version not incremented
  - Unexplained placeholder tokens remain

IF NOT READY:
  → Present issues with evidence
  → Recommend: "Fix constitution content before proceeding"
  → STOP workflow progression
```

### Output Format (Present to User - ONLY if evidence provided)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Constitution Creation Review Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**File Created**: `.specify/memory/constitution.md` ([SIZE]KB)

**Version**: [OLD] → [NEW] ([BUMP_TYPE])
**Rationale**: [Why this version bump]

**Principles Documented**: [COUNT]
  - [Principle 1 Name]
  - [Principle 2 Name]
  - [...]

**Governance Sections**:
  ✓ Amendment Procedure
  ✓ Versioning Policy
  ✓ Compliance Review

**Templates Requiring Updates**:
  - [Template 1]: [Status: ✅ updated / ⚠ pending]
  - [Template 2]: [Status: ✅ updated / ⚠ pending]

**Deferred Items** (if any):
  - TODO([FIELD_NAME]): [Explanation]

Next Action: [Commit with suggested message OR Fix deferred items]

**Suggested Commit Message**:
```
docs: amend constitution to v[NEW_VERSION] ([brief description])

- [Change summary]
- [Affected sections]
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

9. Output a final summary to the user with:
   - New version and bump rationale.
   - Any files flagged for manual follow-up.
   - Suggested commit message (e.g., `docs: amend constitution to vX.Y.Z (principle additions + governance update)`).

Formatting & Style Requirements:

- Use Markdown headings exactly as in the template (do not demote/promote levels).
- Wrap long rationale lines to keep readability (<100 chars ideally) but do not hard enforce with awkward breaks.
- Keep a single blank line between sections.
- Avoid trailing whitespace.

If the user supplies partial updates (e.g., only one principle revision), still perform validation and version decision steps.

If critical info missing (e.g., ratification date truly unknown), insert `TODO(<FIELD_NAME>): explanation` and include in the Sync Impact Report under deferred items.

Do not create a new template; always operate on the existing `.specify/memory/constitution.md` file.
