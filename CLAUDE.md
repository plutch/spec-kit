# Spec-Kit for Claude Code - Framework Distribution Project

## Project Purpose

This is a **framework distribution project**, not a regular software project. It contains template files and Claude Code commands that are meant to be copied into OTHER projects to enable Spec-Driven Development (SDD).

## Important: Do NOT Execute Commands from src/

The `src/.claude/commands/` directory contains TEMPLATE commands that will be copied to other projects. These are NOT active commands for this project.

**Why?** If `.claude/` were at the root of this repository, Claude Code would interpret them as commands for THIS project (the spec-kit project itself), not as distributable templates for other projects.

## Working on This Project

When making changes to this framework:

1. **Edit template commands** in `src/.claude/commands/`
   - Each command file is self-contained (no @include references)
   - Files: speckit.specify.md, speckit.clarify.md, speckit.analyze.md, speckit.analyze-ux.md, speckit.plan.md, speckit.tasks.md, speckit.implement.md, speckit.constitution.md, speckit.supplement.md, speckit.reconcile.md, speckit.validate-hierarchy.md, speckit.amend-technical.md, speckit.pm.md, speckit.next.md

2. **Update templates** in `src/.specify/templates/`
   - Specification templates
   - Planning templates
   - Task templates
   - Command templates (original sources before copying to .claude/)

3. **Modify documentation** in `src/.specify/docs/`
   - spec-driven.md - Core methodology
   - HIERARCHICAL-SPECS-GUIDE.md
   - RECONCILE-GUIDE.md
   - VALIDATION-GUIDE.md

4. **Review framework quality** with workflow-reviewer agent
   - Agent location: `src/.claude/agents/workflow-reviewer.md`
   - Purpose: Analyze workflow coherence, DX, quality framework integration
   - Use for: Pre-release reviews, post-feature reviews, quarterly health checks
   - See: `src/.claude/agents/README.md` for usage guide

5. **Test changes** by copying to a test project and running there
   ```bash
   cp -r src/{.claude,.specify} /path/to/test-project/
   cd /path/to/test-project/
   # Now test the commands
   ```

## Project Structure

```
spec-kit-claude/
├── CLAUDE.md                  # This file - explains project purpose
├── README.md                  # Framework overview and usage
├── INSTALL.md                 # Installation instructions
├── CODE_OF_CONDUCT.md         # Community guidelines
├── LICENSE                    # MIT License
└── src/                       # Distribution source (users copy this)
    ├── .claude/
    │   ├── agents/            # Framework review agents
    │   └── commands/          # 14 self-contained command files
    └── .specify/
        ├── config.example.yml     # Configuration template
        ├── business-rules/    # Business rule validation templates
        ├── memory/            # Project constitution templates
        ├── templates/         # All SDD templates
        └── docs/              # Methodology guides
```

## Development Guidelines

### Commands Are Self-Contained
All commands in `src/.claude/commands/` are self-contained markdown files. They do NOT use `@include` references. This ensures:
- Easy distribution (just copy files)
- No dependency resolution needed
- Immediate availability in target projects

### Templates Are Reference Material
The `src/.specify/templates/` directory contains:
- Original template sources (before copying to commands)
- Additional templates users can customize
- Reference documentation

### Maintain Claude Code-Specific Focus
- All commands designed for Claude Code
- No multi-LLM compatibility layers
- Claude-optimized workflows and patterns

### Installation Is Copy/Paste Only
No scripts, no CLI, no package managers required. Users simply:
1. Copy `src/.claude/` to their project
2. Copy `src/.specify/` to their project
3. Start using commands immediately

## Available Commands

Once users copy the framework to their project, they get access to:

**Core Workflow (5 commands)**:
- `/speckit.specify` - Create feature specifications
- `/speckit.clarify` - Clarify ambiguities in specs
- `/speckit.plan` - Generate implementation plans
- `/speckit.tasks` - Break down plans into tasks
- `/speckit.implement` - Execute implementation

**Advanced Workflow (5 commands)**:
- `/speckit.constitution` - Create project constitution
- `/speckit.supplement` - Create hierarchical supplementary specs
- `/speckit.reconcile` - Post-implementation gap closure
- `/speckit.validate-hierarchy` - Validate spec relationships
- `/speckit.amend-technical` - Amend architecture decisions via ADR

**Quality & Session Management (4 commands)**:
- `/speckit.analyze` - Fast quality analysis with actionable recommendations
- `/speckit.analyze-ux` - Fast UX quality analysis for interface-heavy features
- `/speckit.pm` - Session context and state management
- `/speckit.next` - Get next recommended action

## Quality Framework Enhancements (v2.1)

Spec-Kit v2.1 introduced comprehensive quality, security, and testing features throughout the workflow:

### Risk Assessment Framework (`/speckit.specify`)
- **6-Criteria Risk Scoring** (0-12 scale): Data sensitivity, access control, external integration, performance, complexity, business impact
- **Risk Classifications**: 🔴 HIGH (8-12), 🟠 MEDIUM (4-7), 🟢 LOW (0-3)
- **Output**: Risk assessment table in spec.md with mitigation requirements

### Enhanced Severity System (All Commands)
- **Standardized Color-Coded Severity**: 🔴 CRITICAL, 🟠 MAJOR, 🟡 MEDIUM, 🟢 LOW, ℹ️ INFO
- **Applied Across**: analyze, analyze-ux, clarify (all modes), reconcile
- **Severity Discipline**: Reserve CRITICAL for actual data/security risks

### Edge Case Detection Mode (`/speckit.clarify --edge-cases`)
- **6 Detection Categories**: Boundary conditions, error conditions, state transitions, data quality, race conditions, multi-tenant isolation
- **Production Resilience**: Timeout handling, retry logic, idempotency, crash recovery
- **Output**: Edge case questions + Production Readiness Assessment

### UX Quality Framework (`/speckit.specify`, `/speckit.clarify --ux`, `/speckit.analyze-ux`)
- **5-Dimensional UX Scoring**: Usability, Efficiency, Accessibility, Error Prevention, Delight (0-10 scale each)
- **Expert Methodologies**: Nielsen, Norman, Cooper, Wroblewski, Krug, WCAG 2.1 AA
- **UX Lens Mode**: 4-pass multi-perspective review (Usability, Task Efficiency, Interaction Patterns, Accessibility)
- **Quick Wins**: High-impact, low-effort improvement recommendations

### Test Strategy Planning (`/speckit.plan` - Phase 2.5)
- **Test Coverage by Layer**: Unit, integration, E2E, performance tests
- **Risk-Based Prioritization**: Maps to Risk Assessment scores (🔴/🟠/🟡)
- **Edge Case Test Planning**: Boundary, error, state, concurrency test scenarios
- **Mock/Stub Strategy**: External APIs, database, clock, file system
- **Coverage Thresholds**: Critical paths ≥90%, business logic ≥80%, overall ≥75%

### Security & Compliance Review (`/speckit.plan` - Phase 3)
- **OWASP Top 10 Validation**: Comprehensive checklist for all 10 vulnerability categories
- **Authentication/Authorization**: Token validation, RBAC/ABAC, principle of least privilege
- **Data Protection**: Encryption, PII handling, data retention, GDPR/CCPA/PCI-DSS compliance
- **Input Validation**: Schema validation, SQL injection prevention, XSS protection
- **Audit Logging**: Authentication, authorization, data access, financial transactions
- **Blocking Criteria**: Critical OWASP vulnerabilities, missing encryption, auth gaps

### Implementation Quality Gate (`/speckit.implement` - Step 9)
- **6-Layer Validation**: Code quality, security, test coverage, spec alignment, documentation, build readiness
- **Security Checks**: No hardcoded secrets, input validation, auth/authz, dependency audit
- **Test Coverage**: All tests pass, coverage thresholds met, high-risk requirements tested
- **Gate Statuses**: 🟢 READY TO COMMIT, 🟡 NEEDS ATTENTION, 🔴 BLOCKED
- **Output**: Quality Gate Report with findings and next steps

### Key Integration Points
1. **Risk Assessment (specify)** → **Test Priority (plan)** → **Test Coverage Validation (implement)**
2. **Edge Case Detection (clarify)** → **Edge Case Tests (plan)** → **Test Execution (implement)**
3. **Security Review (plan)** → **Security Validation (implement)**
4. **UX Scoring (specify)** → **UX Lens Mode (clarify)** → **UX Analysis (analyze-ux)**

### Benefits for Framework Users
- ✅ **Production Readiness**: Systematic quality gates prevent incomplete implementations
- ✅ **Security by Design**: OWASP Top 10 validation built into workflow
- ✅ **Risk-Driven Development**: High-risk features get extra scrutiny automatically
- ✅ **Comprehensive Testing**: Risk-based test prioritization from specification phase
- ✅ **Compliance Assurance**: GDPR, CCPA, PCI-DSS, HIPAA requirements mapped
- ✅ **UX Excellence**: Evidence-based usability validation with expert methodologies

## Testing Changes

### Manual Testing Workflow

1. **Make changes** to commands or templates
2. **Copy to test project**:
   ```bash
   # From spec-kit-claude directory
   rm -rf /path/to/test-project/.claude
   rm -rf /path/to/test-project/.specify
   cp -r src/.claude /path/to/test-project/
   cp -r src/.specify /path/to/test-project/
   ```
3. **Test in Claude Code**:
   - Open test project in Claude Code
   - Run commands to verify functionality
   - Check for errors or unexpected behavior
4. **Iterate** until satisfied

### What to Test

- **Command execution**: Do commands run without errors?
- **Output quality**: Are generated files correct and complete?
- **Workflow integration**: Do commands chain together properly?
- **Documentation clarity**: Are instructions clear and actionable?

## Contributing

When contributing to this framework:

1. **Maintain self-contained commands** - No @include references
2. **Test thoroughly** - Copy to test project and verify
3. **Update README.md** - Document any new features or changes
4. **Keep it Claude-only** - No multi-LLM references
5. **Preserve structure** - Don't break the src/ distribution pattern

## Distribution

Users receive this framework as:
- **GitHub repository** - Clone and copy `src/` contents
- **Zip/tarball** - Download and extract `src/` contents
- **Direct copy** - Copy `src/{.claude,.specify}` from existing project

No installation process, no dependencies, no configuration required.

## FAQ

### Q: Why is src/ at the root instead of distribution in root?

**A:** Because Claude Code interprets `.claude/` at the root as active commands for THIS project. By keeping distribution files in `src/`, we avoid confusion and make it clear these are TEMPLATES to be copied elsewhere.

### Q: Can I use these commands in the spec-kit project itself?

**A:** No, these commands are designed for target projects (where you copy them). This project doesn't need spec-driven development because it's a template distribution project.

### Q: How do I update commands after copying to my project?

**A:** Either:
1. **Manual edits**: Edit commands directly in your project's `.claude/` directory
2. **Re-copy**: Delete `.claude/` and `.specify/`, copy fresh from spec-kit updates

### Q: Why no @include references?

**A:** Self-contained commands are easier to:
- Distribute (no dependency resolution)
- Customize (users can edit without breaking includes)
- Understand (everything in one file)
- Version (clear what changed in each command)

### Q: What is the workflow-reviewer agent for?

**A:** The workflow-reviewer agent analyzes the spec-kit framework itself (not user projects) for:
- **Workflow coherence**: State machine integrity, review gate consistency, command chaining
- **Developer experience**: Token economy, session continuity, error handling
- **Quality framework integration**: Risk → Test → Security flow validation
- **Process clarity**: Documentation alignment, simplification opportunities
- **Resilience**: Error recovery, graceful degradation, operational robustness

Use it for:
- Pre-release reviews (before major/minor versions)
- Post-feature reviews (after adding new commands or enhancements)
- Quarterly health checks (continuous improvement)

See `src/.claude/agents/README.md` for detailed usage instructions.

---

**Spec-Kit Version**: 2.1.1
**Target**: Claude Code
**License**: MIT
**Compatibility**: Claude Code 0.7.0+
