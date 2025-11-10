# Spec-Kit Migration Guide

This document provides upgrade paths for migrating between Spec-Kit versions.

---

## Upgrading from v2.3 to v2.4 (Constitutional Enforcement Bridge)

**Release Date**: 2025-11-07
**Breaking Changes**: None (backward compatible with graceful degradation)
**New Features**: Constitutional enforcement at planning, implementation, and reconciliation phases

---

### What's New in v2.4

Spec-Kit v2.4 introduces the **Constitutional Enforcement Bridge** - three enforcement points that ensure constitutional compliance throughout the workflow:

1. **Phase -1 (Planning)**: Constitutional validation before implementation planning
2. **Step 2.5 (Implementation Pre-Flight)**: Blocks unconstitutional implementations before code execution
3. **Step 10.4 (Implementation Quality Gate)**: Constitutional Reviewer validates TDD, simplicity, patterns
4. **Step 5 Q5 (Reconciliation)**: Constitutional re-validation during gap closure

**Benefits**:
- ✅ Catch violations at earliest possible point (planning phase)
- ✅ Automated enforcement via git history analysis, pattern scanning
- ✅ Prevents 10K-20K token rework from late-stage architectural pivots
- ✅ Marginal cost (+2-3% tokens) with 300-500% ROI

---

### Migration Scenarios

#### Scenario A: New Projects (Clean Start)

**If you're starting a new project with v2.4**:

1. Copy framework to project:
   ```bash
   cp -r spec-kit/src/.claude /path/to/project/
   cp -r spec-kit/src/.specify /path/to/project/
   ```

2. Create project constitution (optional but recommended):
   ```bash
   /speckit.constitution
   ```

3. Start your first feature:
   ```bash
   /speckit.specify
   /speckit.plan  # Automatically includes Phase -1 constitutional validation
   /speckit.implement
   ```

**No migration needed** - v2.4 works out of the box.

---

#### Scenario B: Existing Projects WITHOUT Constitution

**If you're upgrading from v2.3 and DON'T have constitution.md**:

1. **Update framework files**:
   ```bash
   # Backup current files
   cp -r .claude .claude.backup
   cp -r .specify .specify.backup

   # Copy v2.4 files
   cp -r spec-kit/src/.claude ./
   cp -r spec-kit/src/.specify ./
   ```

2. **Your existing features will work with graceful degradation**:
   - Plans without Phase -1: `/speckit.implement` Step 2.5 will prompt, but Step 10.4 still validates
   - No constitution.md: Constitutional Reviewer runs in **PARTIAL mode** (TDD + simplicity checks, no pattern scanning)
   - **Result**: ⚠️ NEEDS REVIEW status (reduced validation, not blocking)

3. **Optional: Create constitution for future features**:
   ```bash
   /speckit.constitution
   ```
   - Future features get full constitutional validation
   - Existing features continue with graceful degradation

**Migration effort**: ~5 minutes (copy files + optional constitution creation)

---

#### Scenario C: Existing Projects WITH Constitution (v2.3 format)

**If you're upgrading from v2.3 and HAVE constitution.md**:

1. **Update framework files** (same as Scenario B, step 1)

2. **Update constitution.md to v2.4 format**:

   Open `.specify/memory/constitution.md` and add the new section before "## Governance":

   ```markdown
   ## Prohibited Patterns (Machine-Readable - v2.4)

   **Purpose**: Define code patterns that violate project principles. Used by Constitutional Reviewer (Step 10.4.5) to scan codebase automatically.

   **Format**: Each pattern includes detection command, reason, and remediation.

   ---

   ### Pattern 1: Synchronous File Operations

   ```yaml
   Pattern: fs.readFileSync, fs.writeFileSync
   Reason: Blocks event loop, violates non-blocking principle
   Detection: grep -rn "fs\.\(read\|write\)FileSync" src/
   Remediation: Use fs.promises.readFile() / fs.promises.writeFile()
   ```

   ### Pattern 2: Direct Database Access

   ```yaml
   Pattern: db.query(), db.execute() (bypassing repository layer)
   Reason: Violates repository abstraction, reduces testability
   Detection: grep -rn "db\.\(query\|execute\)" src/ --exclude-dir=repositories
   Remediation: Use repository methods (e.g., userRepo.findById())
   ```

   ### Pattern 3: Global State Mutations

   ```yaml
   Pattern: Global variable assignments in modules
   Reason: Breaks functional purity, creates hidden dependencies
   Detection: grep -rn "global\.\w\+\s*=" src/
   Remediation: Pass state as function arguments or use dependency injection
   ```

   ---

   **Custom Patterns**: Add project-specific patterns as needed. Detection commands should work in your project environment.
   ```

3. **Update existing plans** (optional - for features in progress):

   For each feature with a `plan.md` file, you have 3 options:

   **Option 1: Continue with existing plans** (simplest)
   - `/speckit.implement` Step 2.5 will prompt: "No Phase -1 validation. Proceed? (yes/no)"
   - Answer "yes" to continue
   - Step 10.4 Constitutional Reviewer still validates (TDD, simplicity, patterns)
   - **Effort**: 1 prompt per feature

   **Option 2: Regenerate plans** (cleanest)
   - Backup customizations from `plan.md`
   - Delete `plan.md` and `research.md`
   - Run `/speckit.plan` (generates Phase -1 automatically)
   - Restore customizations
   - **Effort**: 5-10 minutes per feature

   **Option 3: Manual Phase -1 injection** (advanced)
   - Open `plan.md` in editor
   - Add Phase -1 section after Phase 0 (Discovery):

   ```markdown
   ## Phase -1: Constitutional Validation (Pre-Implementation Checkpoint) 🔴 MANDATORY

   **Purpose**: Verify feature design aligns with project constitution before implementation begins.

   ### Gate 1: Test-Driven Development (TDD) Feasibility
   - [ ] Feature has clear acceptance criteria suitable for test-first development
   - [ ] Integration points identified for contract testing
   - [ ] Testability assessed and documented

   **Result**: ✅ PASS (TDD feasible for this feature)

   ### Gate 2: Simplicity Assessment
   - [ ] Framework count check: No new frameworks required
   - [ ] Project count check: No new services/projects required
   - [ ] Complexity tracking: Feature fits within simplicity constraints

   **Result**: ✅ PASS (simplicity maintained)

   ### Gate 3: Abstractions Justification
   - [ ] No custom wrapper patterns (Repository, Factory, Adapter) planned
   - [ ] OR: Abstractions justified with problem statement

   **Result**: ✅ PASS (no unjustified abstractions)

   ### Gate 4: Integration-First Validation
   - [ ] External dependencies identified
   - [ ] Contract tests planned for integrations
   - [ ] Real dependencies (not mocks) planned for integration tests

   **Result**: ✅ PASS (integration-first approach)

   ### Constitutional Validation Result
   - **Overall Status**: ✅ PASS
   - **Conditional Items**: None
   - **Blocking Issues**: None
   - **Proceed to Implementation**: Yes

   ---
   ```

   - Update each gate result based on your feature
   - **Effort**: 10-15 minutes per feature

**Migration effort**: ~30 minutes (constitution update + choose plan migration strategy)

---

### Graceful Degradation Behavior

v2.4 is designed to work with partial constitutional infrastructure:

| Condition | Behavior | Impact |
|-----------|----------|--------|
| **No constitution.md** | Constitutional Reviewer runs in PARTIAL mode (TDD + simplicity checks, no pattern scanning) | ⚠️ NEEDS REVIEW |
| **constitution.md without Prohibited Patterns section** | Pattern scanning skipped, other checks run | ⚠️ PARTIAL |
| **Plans without Phase -1** | Step 2.5 prompts user, Step 10.4 validates anyway | User decision required |

**Result**: v2.4 never breaks existing workflows - it degrades gracefully and still provides value.

---

### Rollback Instructions

If you encounter issues with v2.4 and need to rollback to v2.3:

1. **Restore backup files**:
   ```bash
   rm -rf .claude .specify
   mv .claude.backup .claude
   mv .specify.backup .specify
   ```

2. **Remove Phase -1 sections from plans** (if manually added):
   - Edit each `plan.md` file
   - Delete "Phase -1: Constitutional Validation" section
   - Continue with v2.3 workflow

3. **Report issue**: [https://github.com/anthropics/spec-kit/issues](https://github.com/anthropics/spec-kit/issues)

**Rollback effort**: ~5 minutes

---

### Testing Your Migration

After migrating to v2.4, test with a simple feature:

1. **Create test feature**:
   ```bash
   /speckit.specify
   # Specify: "Add a simple utility function that formats dates"
   ```

2. **Generate plan with Phase -1**:
   ```bash
   /speckit.plan
   # Verify Phase -1 section appears in plan.md
   ```

3. **Implement with constitutional validation**:
   ```bash
   /speckit.implement
   # Step 2.5 should show: "✅ Constitutional validation passed"
   # Step 10.4 should show: "Constitutional Reviewer Status: ✅ READY"
   ```

4. **Verify graceful degradation** (optional):
   - Temporarily rename `.specify/memory/constitution.md` to `constitution.md.bak`
   - Run `/speckit.implement` on a feature
   - Step 10.4 should show: "ℹ️ PARTIAL (no constitution defined)"
   - Restore `constitution.md`

If all steps succeed, migration is complete! ✅

---

### Frequently Asked Questions

**Q: Do I need to update all existing features immediately?**
A: No. Existing features without Phase -1 will continue to work with graceful degradation. You can update plans incrementally or leave them as-is.

**Q: What if I don't want constitutional enforcement?**
A: Don't create `constitution.md`. The Constitutional Reviewer will run in PARTIAL mode (TDD + simplicity checks only) and report ℹ️ PARTIAL status. This is non-blocking.

**Q: Can I customize the Prohibited Patterns?**
A: Yes! The patterns in `constitution.md` are fully customizable. Add, remove, or modify patterns to match your project's needs. Detection commands can be any shell command (grep, rg, custom scripts).

**Q: What's the token cost of v2.4?**
A: +2-3% for implementation (~500-1,000 tokens per feature). ROI is 300-500% if it catches 1 violation per 20 features (prevents 10K-20K token rework).

**Q: Will v2.4 break my CI/CD pipelines?**
A: No. Step 2.5 prompts can be bypassed by answering "yes" when prompted. Step 10.4 validation is non-interactive and provides clear pass/fail status for automation.

**Q: What if I find a bug or have questions?**
A: File an issue: [https://github.com/anthropics/spec-kit/issues](https://github.com/anthropics/spec-kit/issues)

---

## Upgrading from v2.4 to v2.5/v2.6 (Constitutional Test Strategy Enforcement)

**v2.5.0 Release Date**: 2025-11-10 (Patch)
**v2.6.0 Release Date**: 2025-11-10 (Minor)
**Breaking Changes**: None (backward compatible with graceful degradation)
**New Features**: Phase 2.6 test strategy validation, Article V mock pattern enforcement, corrected Phase 2.5 guidance

---

### What's New in v2.5/v2.6

Spec-Kit v2.5/v2.6 closes the **"Test Strategy Guidance Violation"** gap discovered via root cause analysis. v2.4 had Phase -1 constitutional validation, but Phase 2.5 (Test Strategy Planning) **actively recommended** mocking auth/database/multi-tenancy dependencies, which violated Article V: Integration-First Testing.

**v2.5.0 (Patch) Fixes**:
1. **Phase 2.5 Guidance Corrections**: Removed violation-inducing recommendations (no more "mock Auth0 with jose library")
2. **Phase -1 Enhanced Article IX Gate**: Added Phase 2.5 test strategy validation checkpoint

**v2.6.0 (Minor) Features**:
3. **Phase 2.6 Constitutional Test Strategy Validation**: New validation phase between Phase 2.5 and Phase 3
4. **Article V Mock Patterns**: Added 3 patterns to constitution.md (mocked auth, mocked DB, mocked tenant context)
5. **Enhanced Step 2.5 Pre-Flight Check**: Blocks if Phase 2.6 validation failed

**Benefits**:
- ✅ Phase 2.5 now guides toward Article V-compliant test strategies
- ✅ Phase 2.6 validates test strategy BEFORE implementation begins (not at Step 10.4)
- ✅ Step 10.4.5 can scan for Article V mock patterns automatically
- ✅ Defense-in-depth: 3 layers of validation (Phase -1, Phase 2.6, Step 2.5 pre-flight)
- ✅ Marginal cost (+1.5-2% tokens) with 400-600% ROI

---

### Migration Scenarios

#### Scenario A: New Projects (Clean Start)

**If you're starting a new project with v2.5/v2.6**:

1. Copy framework to project:
   ```bash
   cp -r spec-kit/src/.claude /path/to/project/
   cp -r spec-kit/src/.specify /path/to/project/
   ```

2. Create project constitution (recommended for Article V patterns):
   ```bash
   /speckit.constitution
   ```

3. Start your first feature:
   ```bash
   /speckit.specify
   /speckit.plan  # Phase 2.5 has corrected guidance, Phase 2.6 validates automatically
   /speckit.implement
   ```

**No migration needed** - v2.6 works out of the box with corrected guidance.

---

#### Scenario B: Existing Projects Upgrading from v2.4 (No Constitution Changes Needed)

**If you're upgrading from v2.4**:

1. **Update framework files**:
   ```bash
   # Backup current files
   cp -r .claude .claude.backup
   cp -r .specify .specify.backup

   # Copy v2.5/v2.6 files
   cp -r spec-kit/src/.claude ./
   cp -r spec-kit/src/.specify ./
   ```

2. **Your existing features will work with graceful degradation**:
   - Plans without Phase 2.6: `/speckit.implement` Step 2.5 continues, Step 10.4.4 validates Article V
   - Plans with old Phase 2.5 guidance: Phase 2.6 not required (Step 10.4 catches violations)
   - **Result**: Works immediately, no breaking changes

3. **Optional: Update constitution.md with Article V mock patterns**:

   If you already have `constitution.md`, add these patterns to the **Prohibited Patterns** section:

   ```markdown
   ### Pattern 4: Mocked Auth in Integration Tests (Article V - v2.6)

   - **Pattern**: `jest.mock('auth0')`, `mockJWT()`, `jest.spyOn(authService, 'validate')`
   - **Reason**: Violates Article V - auth MUST use real Auth0 test tenant in integration tests
   - **Detection**: `grep -rn "jest\.mock.*auth\|mockJWT" tests/integration/`
   - **Remediation**: Use real Auth0 test tenant with test credentials

   ---

   ### Pattern 5: Mocked Database in Integration Tests (Article V - v2.6)

   - **Pattern**: `MockDatabase()`, `InMemoryDB()`, `createMockRepository()` in integration tests
   - **Reason**: Violates Article V - integration tests MUST use real database (Docker/Testcontainers)
   - **Detection**: `grep -rn "InMemory\|MockDB\|MockRepository" tests/integration/`
   - **Remediation**: Use Docker/Testcontainers for integration tests

   ---

   ### Pattern 6: Mocked Tenant Context (Multi-Tenant Violation - Article V - v2.6)

   - **Pattern**: `mockTenantContext()`, `setCurrentTenant(fakeId)`, `jest.mock('tenant-middleware')`
   - **Reason**: Violates Article V - tenant isolation MUST be tested with real tenant filtering
   - **Detection**: `grep -rn "mockTenant\|fakeTenant" tests/integration/`
   - **Remediation**: Use real tenant IDs in test database, verify tenant_id filtering
   ```

4. **Optional: Regenerate plans for in-progress features**:

   For features with test strategy planning in progress:

   **Option 1: Continue with existing plans** (simplest)
   - Phase 2.6 is optional - Step 10.4.4 still validates Article V
   - **Effort**: 0 minutes

   **Option 2: Regenerate Phase 2.5** (recommended)
   - Backup customizations from `plan.md`
   - Delete Phase 2.5 section from plan.md
   - Run `/speckit.plan` (only Phase 2.5, or regenerate entire plan)
   - Phase 2.5 now has corrected guidance + Phase 2.6 validation runs automatically
   - Restore customizations if needed
   - **Effort**: 5 minutes per feature

**Migration effort**: ~5-10 minutes (copy files + optional pattern additions)

---

#### Scenario C: Projects with Custom Test Strategy Needs

**If you have non-standard test requirements** (e.g., need to mock auth for specific reasons):

1. **Update framework files** (same as Scenario B, step 1)

2. **Document justifications in Phase -1 → Phase 2.5 Test Strategy Compliance**:

   When running `/speckit.plan`, Phase 2.6 will detect mocks. If you have valid reasons:

   - Check the "⚠️ CONDITIONAL" box in Phase -1 Article IX gate
   - Add justification: "Email/SMS mocks: No test mode available, external service"
   - Document in plan.md Complexity Tracking section

3. **Phase 2.6 will report ⚠️ CONDITIONAL**:
   - Step 2.5 Pre-Flight Check asks: "Proceed with conditional test strategy? (yes/no)"
   - Answer "yes" after verifying justifications
   - Step 10.4 validates actual implementation matches justified approach

**Migration effort**: ~10-15 minutes (justification documentation)

---

### Graceful Degradation Behavior

v2.5/v2.6 is designed to work with partial infrastructure:

| Condition | Behavior | Impact |
|-----------|----------|--------|
| **Plans without Phase 2.6** | Step 2.5 skips Phase 2.6 check, Step 10.4.4 validates Article V anyway | ℹ️ INFO (legacy plan) |
| **constitution.md without Article V patterns** | Phase 2.6 uses generic Article V principles, Step 10.4.5 skips pattern scanning | ⚠️ PARTIAL |
| **No constitution.md** | Phase 2.6 validates with generic principles, Constitutional Reviewer PARTIAL mode | ⚠️ NEEDS REVIEW |
| **Phase -1 without Test Strategy Compliance check** | Phase 2.6 runs independently, reports to plan.md | ✅ Works normally |

**Result**: v2.5/v2.6 never breaks existing workflows - graceful degradation maintained.

---

### What Changed in Files

#### Files Modified (v2.5.0 Patch):
- `src/.claude/commands/speckit.plan.md` - Phase 2.5 Mock & Stub Strategy guidance (lines 821-941)
- `src/.specify/templates/plan-template.md` - Phase -1 Article IX gate enhancement (lines 69-76)

#### Files Added/Modified (v2.6.0 Minor):
- `src/.claude/commands/speckit.plan.md` - New Phase 2.6 section (lines 974-1070)
- `src/.specify/memory/constitution.md` - Article V mock patterns (lines 106-177)
- `src/.claude/commands/speckit.implement.md` - Enhanced Step 2.5 Pre-Flight Check (lines 99-131)

**Backward Compatibility**: All changes are additive. Old plans work with graceful degradation.

---

### Testing Your Migration

After migrating to v2.5/v2.6, test with a feature that uses auth/database:

1. **Create test feature**:
   ```bash
   /speckit.specify
   # Specify: "Add user authentication with JWT tokens"
   ```

2. **Generate plan with Phase 2.6**:
   ```bash
   /speckit.plan
   # Verify Phase 2.5 guidance says "Real Auth0 test tenant" (NOT "mock JWT with jose")
   # Verify Phase 2.6 validation section appears after Phase 2.5
   # Check Phase 2.6 status: ✅ PASS (no mock violations)
   ```

3. **Implement with enhanced pre-flight check**:
   ```bash
   /speckit.implement
   # Step 2.5 should show: "✅ Phase -1 constitutional validation passed"
   # Step 2.5 should show: "✅ Test strategy complies with Article V"
   # Step 10.4.4 should validate integration tests use real Auth0
   ```

4. **Verify pattern scanning** (if constitution.md has Article V patterns):
   - Step 10.4.5 scans for `jest.mock('auth0')`, `MockDatabase()`, `mockTenantContext()`
   - Reports violations: `file:line - Pattern: Mocked Auth in Integration Tests - Reason: ...`

If all steps succeed, migration is complete! ✅

---

### Rollback Instructions

If you encounter issues with v2.5/v2.6 and need to rollback to v2.4:

1. **Restore backup files**:
   ```bash
   rm -rf .claude .specify
   mv .claude.backup .claude
   mv .specify.backup .specify
   ```

2. **Remove Phase 2.6 sections from plans** (if generated with v2.6):
   - Edit each `plan.md` file
   - Delete "Phase 2.6: Constitutional Test Strategy Validation" or "Constitutional Test Strategy Validation (Article V)" section
   - Continue with v2.4 workflow

3. **Remove Article V patterns from constitution.md** (if added manually):
   - Edit `.specify/memory/constitution.md`
   - Remove Pattern 4, 5, 6 (mocked auth, mocked DB, mocked tenant context)
   - Keep other patterns

4. **Report issue**: [https://github.com/anthropics/spec-kit/issues](https://github.com/anthropics/spec-kit/issues)

**Rollback effort**: ~5 minutes

---

### Frequently Asked Questions

**Q: Do I need to update all existing plans immediately?**
A: No. Plans without Phase 2.6 continue to work. Step 10.4.4 still validates Article V compliance during implementation.

**Q: What if I'm already mocking auth/database in integration tests?**
A: Phase 2.6 will report ⚠️ CONDITIONAL or ❌ FAIL. You can either:
1. **Recommended**: Switch to real dependencies (Docker, Auth0 test tenant)
2. **Document justification**: Add to plan.md Complexity Tracking, proceed with Step 2.5 warning

**Q: Can I skip Phase 2.6 validation?**
A: Yes. Phase 2.6 is optional. If your plan doesn't have it, Step 10.4 still validates (same as v2.4 behavior).

**Q: What's the token cost of v2.6?**
A: +1.5-2% for planning (~200-300 tokens per feature). Implementation cost unchanged (Step 2.5 reuses Phase 2.6 result). ROI is 400-600%.

**Q: Will v2.6 work with my non-JavaScript project?**
A: Yes! Phase 2.6 principles (real dependencies in integration tests) apply to any language. Article V mock patterns in constitution.md are examples - customize detection commands for your language (e.g., Python: `grep -rn "unittest\.mock.*auth"`, Go: `grep -rn "mockAuth"`).

**Q: What if I disagree with Article V requirements?**
A: Don't create constitution.md or skip Article V sections. Phase 2.6 degrades gracefully. You can also customize constitution.md to match your project's principles.

---

## Future Migration Guides

### v2.7 (Planned)

- **Performance Optimization**: Reduced token overhead for Phase 2.6 validation
- **Enhanced Pattern Library**: Pre-defined patterns for React, Angular, Vue, Python, Go
- **CI/CD Integration**: Auto-approval flags, non-interactive modes

### v3.0 (Planned)

- **Multi-Repository Support**: Mono-repo constitutional hierarchies
- **Custom Quality Gate Plugins**: User-defined reviewers and validators
- **Enhanced State Machine**: Parallel workflow branches, conditional phases

Stay tuned for updates at: [https://github.com/anthropics/spec-kit](https://github.com/anthropics/spec-kit)

---

**Last Updated**: 2025-11-10
**Applies To**: Spec-Kit v2.4.0+ (includes v2.5.0 and v2.6.0 migration guides)
**Maintained By**: Spec-Kit Core Team
