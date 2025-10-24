# ADR-###: [Title - Short Description of Decision]

**Status**: [Proposed | Accepted | Deprecated | Superseded]
**Date**: YYYY-MM-DD
**Deciders**: [Names or roles of decision makers]
**Tags**: [architecture, database, framework, performance, security]

---

## Context

What is the issue that we're seeing that is motivating this decision or change?

### Problem Statement
[Describe the architectural blocker or challenge that triggered this ADR]

### Background
- Current implementation: [What exists today]
- Discovery point: [When/how was the blocker discovered - Week X of implementation, during testing, etc.]
- Impact: [Which features/components are affected]

### Blocker Classification (Article XI.1)
- **Type**: [ ] Bug | [X] Architectural Limitation | [ ] Performance Issue | [ ] Framework Limitation
- **Scope**: [ ] Single Feature | [X] Cross-Cutting Concern
- **NFR Violation**: [ ] Yes (specify) | [X] No

---

## Decision

We will [describe the architectural decision/change being made].

### From → To
- **FROM**: [Current technology/pattern/approach]
- **TO**: [New technology/pattern/approach]

### Rationale
1. [Primary reason for change]
2. [Secondary considerations]
3. [Alternative approaches considered and rejected]

### Key Changes
- [ ] Technology stack change (specify)
- [ ] Architectural pattern change (specify)
- [ ] Database strategy change (specify)
- [ ] Framework/library swap (specify)
- [ ] Major refactoring (>5 files affected)

---

## Constitutional Compliance (Article XI.3)

### Article VII: Simplicity Gate
**Assessment**: [PASS | CONDITIONAL PASS | FAIL]
- Concern: [Does this add new project/framework?]
- Justification: [Why additional complexity is warranted]
- Architect Approval: [ ] Required | [ ] Not Required
- Approval Status: [ ] Granted | [ ] Pending

### Article VIII: Anti-Abstraction Gate
**Assessment**: [PASS | FAIL]
- Framework Trust: [Is new technology well-maintained and trusted?]
- Custom Wrappers: [Are we introducing unnecessary abstractions?]
- Community Support: [Active development, documentation quality]

### Article IX: Integration-First Gate
**Assessment**: [PASS | FAIL]
- Contract Tests: [ ] Updated BEFORE implementation
- Real Dependencies: [ ] Using actual databases/services (no mocks)
- Test Status: [ ] RED phase validated

### [Article X: Project-Specific]
**Assessment**: [PASS | FAIL]
- [Add project-specific constitutional requirements]

---

## Consequences

### Positive
- [Benefit 1 - e.g., 90% latency reduction]
- [Benefit 2 - e.g., Supports 10,000+ tenant schemas]
- [Benefit 3 - e.g., Simpler mental model]

### Negative
- [Tradeoff 1 - e.g., Lose Prisma type safety]
- [Tradeoff 2 - e.g., Engineers must write SQL manually]
- [Tradeoff 3 - e.g., Two database access patterns in codebase]

### Mitigation
- [How to address Negative 1]
- [How to address Negative 2]
- [How to address Negative 3]

###  Risks
- [Risk 1 and mitigation plan]
- [Risk 2 and mitigation plan]

---

## Affected Business Rules (Article XI.6)

### Modified Rules
- **BR-[DOMAIN]-[NUMBER]**: [Rule Name]
  - **Implementation Change**:
    - FROM: `src/path/file.ts:line` (old tech)
    - TO: `src/path/new-file.ts:line` (new tech)
  - **Test Change**:
    - FROM: `tests/path/test_old.py:line`
    - TO: `tests/path/test_new.py:line`
  - **Action**: Update catalog via `/rules-extract --module [MODULE]`
  - **Constitutional Impact**: [Which article validation affected]

### New Rules Created
- **BR-[DOMAIN]-[NUMBER]**: [New Rule Name]
  - **Reason**: [Why pivot introduces new business logic]
  - **Status**: DRAFT (pending implementation)
  - **Domain**: [MODULE]

### No Business Rules Affected
[ ] This decision is purely architectural with no business logic impact

---

## Timeline Impact (Article XI.7)

### Original Estimate
- Start: Week [X]
- End: Week [Y]
- Duration: [Z] weeks

### Adjusted Estimate (Post-Pivot)
- Pivot Decision: Week [X]
- Pivot Implementation: Week [X+1] - Week [Y+2]
- New End Date: Week [Y+2]
- Timeline Adjustment: +[N] weeks

### Breakdown
- Week [X]: Create ADR + update contract tests (RED phase)
- Week [X+1]: [Pivot implementation tasks]
- Week [Y+1]: [Testing and validation]
- Week [Y+2]: [Integration and deployment]

---

## Implementation Plan

### Phase 1: Preparation (Article XI.4 - Test-First)
1. [ ] Update contract tests to reflect new architecture
2. [ ] Validate tests FAIL with current implementation (RED phase)
3. [ ] Obtain user approval of test changes
4. [ ] Update `TECHNICAL.md` with amended constraints

### Phase 2: Implementation
1. [ ] [Specific implementation task 1]
2. [ ] [Specific implementation task 2]
3. [ ] [@BusinessRule annotations updated with new references]
4. [ ] [Integration points addressed]

### Phase 3: Validation
1. [ ] All contract tests pass (GREEN phase)
2. [ ] Performance benchmarks meet targets
3. [ ] Business rules catalog updated (`/rules-extract`)
4. [ ] Integration tests pass

### Phase 4: Communication (Article XI.7)
1. [ ] Stakeholders informed of timeline impact
2. [ ] ADR added to decision log
3. [ ] Sprint/release plan adjusted
4. [ ] Documentation updated

---

## Rollback Plan

### Triggers
Rollback if:
- [Condition 1 - e.g., P95 latency >target in production]
- [Condition 2 - e.g., Critical bug discovered within 48 hours]
- [Condition 3 - e.g., Performance regression >20%]

### Rollback Steps
1. [Step 1 - e.g., Toggle feature flag OFF]
2. [Step 2 - e.g., Revert to previous library version]
3. [Step 3 - e.g., Deploy last known good release]

### Estimated Rollback Time
- Preparation: [X] hours
- Execution: [Y] hours
- Validation: [Z] hours
- **Total**: [X+Y+Z] hours

---

## References

### Documentation
- TECHNICAL.md: `specs/[feature]/TECHNICAL.md` (amended)
- Original Spec: `specs/[feature]/spec.md`
- Implementation: `src/[module]/[file].ts`
- Contract Tests: `tests/integration/test_[feature]_contracts.py`

### Related ADRs
- [ADR-###]: [Related decision - predecessor]
- [ADR-###]: [Related decision - similar context]

### External Resources
- [Technology documentation URL]
- [Performance benchmark results URL]
- [Community discussion URL]

### Stakeholder Communication
- [Email thread link or date]
- [Meeting notes date and participants]
- [Approval documentation reference]

---

## Decision Outcome

**Final Decision**: [Accepted | Rejected | Deferred]
**Approval Date**: YYYY-MM-DD
**Approved By**: [Name/Role]

### Post-Implementation Review
- **Review Date**: [3-6 months after implementation]
- **Success Metrics**: [How to measure if decision was correct]
- **Lessons Learned**: [To be filled after review]

---

**ADR Version**: 1.0
**Last Updated**: YYYY-MM-DD
**Status**: [Proposed | Accepted | Deprecated | Superseded by ADR-###]
