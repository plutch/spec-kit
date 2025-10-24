# Article XI: Technical Pivot Protocol

**Purpose**: Handle architectural blockers discovered during implementation
**Version**: 1.0
**Status**: Template for project constitutions

---

When implementation/testing reveals architectural blockers requiring pivots:

## Section 11.1: Blocker Classification

Engineers encountering potential blockers MUST distinguish:
- **Bugs** (code-level fixes) vs. **Architectural Limitations** (framework/design)
- **Workarounds** (maintain requirements) vs. **Fundamental Blockers** (violate NFRs)
- **Single-feature** impact vs. **Cross-cutting** concerns

Escalation required for architectural blockers affecting >1 feature or violating NFRs.

## Section 11.2: Pivot Documentation (ADRs)

Architectural pivots MUST be documented via Architecture Decision Records:
- **Feature-scoped**: `specs/###-feature/decisions/adr-###-title.md`
- **Cross-project**: `.specify/decisions/adr-###-title.md`
- **Required sections**: Context, Decision, Constitutional Compliance, Consequences
- **Immutability**: ADRs are historical records, never modified after approval

## Section 11.3: Constitutional Validation (NON-NEGOTIABLE)

ALL pivots MUST validate against your project's constitutional articles.

Minimum validation required:
- Simplicity: No unnecessary proliferation of technologies
- Framework Trust: Only trusted, well-maintained frameworks
- Integration Testing: Contract tests updated BEFORE implementation
- Violations require explicit documentation + architect approval

## Section 11.4: Test-First Imperative (Article III Extension)

Pivots MUST follow test-first development:
1. Update contract tests to reflect new architecture
2. Validate tests FAIL with current implementation (RED phase)
3. Obtain user approval of test changes
4. ONLY THEN proceed with implementation

## Section 11.5: Constraint Amendment

After ADR approval:
- Update `TECHNICAL.md` with amended constraints (~~old~~ → new)
- Link to ADR number with rationale
- Preserve historical decision trail
- Regenerate plan.md + tasks.md via `/plan --regenerate`

## Section 11.6: Business Rules Impact (If Applicable)

If pivot affects business logic:
- Document in ADR "Affected Business Rules" section
- List BR-* IDs requiring updates
- Run `/rules-extract` post-pivot to sync catalog
- Update rule annotations with new implementation references

## Section 11.7: Communication Protocol

Stakeholders MUST be informed:
- Timeline impact (original vs. adjusted)
- Rationale (blocker + decision)
- Traceability (ADR links)
- Sprint/release adjustments

---

## Integration Instructions

To add this article to your project's constitution:

1. **Add to `memory/constitution.md`**:
   - Copy sections 11.1-11.7 above
   - Adjust Section 11.3 to reference your specific constitutional articles
   - Update version and last amended date

2. **Update existing articles**:
   - Reference Article XI in your Test-First article (usually Article III)
   - Add cross-reference in architectural standards

3. **Create supporting templates**:
   - Copy `templates/adr-template.md` for Architecture Decision Records
   - Create `specs/[feature]/decisions/` directories

4. **Update workflow documentation**:
   - Add pivot protocol to development workflow
   - Include in onboarding materials

---

**Related Templates**:
- `templates/adr-template.md` - Architecture Decision Record format
- `templates/technical-constraints-template.md` - TECHNICAL.md structure
- `.specify/business-rules/validation-rules.md` - Business logic quality gates
