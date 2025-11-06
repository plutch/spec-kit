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
   - Files: speckit.specify.md, speckit.clarify.md, speckit.analyze.md, speckit.analyze-ux.md, speckit.plan.md, speckit.tasks.md, speckit.implement.md, speckit.constitution.md, speckit.supplement.md, speckit.reconcile.md, speckit.validate-hierarchy.md, speckit.amend-technical.md, speckit.pm.md, speckit.next.md, speckit.memory.md, speckit.status.md, speckit.validate-gap.md

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
    │   └── commands/          # 17 self-contained command files
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

**Core Workflow (6 commands)**:
- `/speckit.specify` - Create feature specifications with EARS format support (v2.3+)
- `/speckit.clarify` - Clarify ambiguities in specs
- `/speckit.validate-gap` - Analyze implementation feasibility (v2.3+)
- `/speckit.plan` - Generate implementation plans with requirements traceability (v2.3+)
- `/speckit.tasks` - Break down plans into tasks with EARS references (v2.3+)
- `/speckit.implement` - Execute implementation

**Advanced Workflow (5 commands)**:
- `/speckit.constitution` - Create project constitution
- `/speckit.supplement` - Create hierarchical supplementary specs
- `/speckit.reconcile` - Post-implementation gap closure
- `/speckit.validate-hierarchy` - Validate spec relationships
- `/speckit.amend-technical` - Amend architecture decisions via ADR

**Quality & Session Management (6 commands)**:
- `/speckit.analyze` - Fast quality analysis with Socratic Validation and Quick Wins (v2.2+)
- `/speckit.analyze-ux` - Fast UX quality analysis with Component System Audit (v2.2+)
- `/speckit.memory` - Create/update project memory files (v2.3+)
- `/speckit.status` - View workflow status and phase progression (v2.3+)
- `/speckit.pm` - Session context and state management
- `/speckit.next` - Smart workflow navigator with quality/risk-aware recommendations (v2.2+)

## Memory System & Phase Tracking (v2.3.0)

Spec-Kit v2.3.0 introduces persistent project memory and workflow phase tracking inspired by [cc-sdd-Superclaude](https://github.com/symbioquine/cc-sdd-Superclaude):

### Project Memory System (`/speckit.memory`)
- **Persistent AI Context**: Memory files survive session resets and provide long-term project knowledge
- **Three Inclusion Modes**:
  - **Always**: Core project context loaded in every command ([constitution.md](../src/.specify/memory/constitution.md))
  - **Conditional**: Pattern-based loading (e.g., [api-standards.md](../src/.specify/memory/api-standards.md) when in `src/api/**/*`)
  - **Manual**: On-demand via `@filename` syntax ([deployment-runbook.md](../src/.specify/memory/deployment-runbook.md))
- **Memory Files Created**:
  1. `constitution.md` - Core project principles, tech stack, file organization, standards
  2. `api-standards.md` - REST API conventions, request/response formats, versioning
  3. `testing-approach.md` - Testing strategy, coverage requirements, TDD practices
  4. `deployment-runbook.md` - Deployment procedures, infrastructure, incident response
- **Git-Aware Updates**: Preserves user customizations during updates
- **Benefit**: Eliminates repetitive context gathering, maintains consistency across sessions

### Phase Tracking & Approval Gates (`spec-metadata.json`)
- **Workflow State Machine**: Tracks progression through phases (specification → gap_analysis → planning → tasks → implementation → reconciliation → complete)
- **Approval Gates**: Commands check if previous phases are approved before proceeding
- **Auto-Approval Support**: Use `-y` flag to skip approval prompts (trusted workflows)
- **Metadata Tracking**:
  ```json
  {
    "version": "2.3.0",
    "feature_name": "oauth-integration",
    "phase": "planning",
    "approvals": {
      "specification": { "generated": true, "approved": true, "timestamp": "2025-11-05T10:30:00Z" },
      "gap_analysis": { "generated": true, "approved": true, "timestamp": "2025-11-05T11:00:00Z" },
      "planning": { "generated": true, "approved": false, "timestamp": "2025-11-05T12:00:00Z" }
    },
    "metadata": {
      "risk_level": "MEDIUM",
      "overall_quality": 82
    }
  }
  ```
- **Status Visibility**: Use `/speckit.status` to view current phase and approval status
- **Benefit**: Prevents phase skipping, enforces workflow discipline, tracks progress

### Gap Analysis Command (`/speckit.validate-gap`)
- **Feasibility Assessment**: Analyzes existing codebase vs new requirements before planning
- **Reusable Component Identification**: Maps what can be leveraged vs what needs creation
- **Implementation Options**: Presents 3 approaches (Extend, Create, Hybrid) with trade-offs
- **Technical Research Requirements**: Identifies unknowns that need investigation
- **Complexity Estimation**: T-shirt sizing (S/M/L/XL) with effort estimates
- **Workflow Position**: Between specification and planning (critical for HIGH-risk features)
- **Benefit**: Reduces surprises during implementation, better planning accuracy

### EARS Requirements Format
- **Structured Acceptance Criteria**: Easy Approach to Requirements Syntax
- **Four Core Patterns**:
  - `WHEN [event] THEN [system] SHALL [response]` - Event-driven
  - `IF [condition] THEN [system] SHALL [response]` - State-driven
  - `WHILE [duration] THE [system] SHALL [behavior]` - Continuous
  - `WHERE [context] THE [system] SHALL [behavior]` - Contextual
- **Benefits**: Testable, unambiguous, measurable requirements
- **Requirements Traceability**: Link requirements (REQ-XX-1.1) through spec → plan → tasks → tests
- **Guide**: See [requirements-ears.md](../src/.specify/templates/requirements-ears.md)
- **Example**:
  ```markdown
  REQ-AUTH-1.1: WHEN a user submits valid credentials THEN the Authentication Service SHALL grant access within 2 seconds
  REQ-AUTH-1.2: IF credentials are invalid THEN the Authentication Service SHALL reject access and log the attempt
  ```

### Status Dashboard (`/speckit.status`)
- **Phase Progression Visualization**: See where you are in the workflow with emoji indicators (✅/⏳/⬜/🔴)
- **Approval Status**: Track which phases are complete and approved
- **Quality Metrics**: View overall quality score and critical issues count
- **Workflow Readiness**: Determine if you can proceed to next phase
- **File Detection**: Lists all spec files with status and modification dates
- **Memory Files Active**: Shows which memory files are loaded
- **Smart Recommendations**: Suggests next command based on current state and quality
- **Benefit**: Complete visibility into workflow status at a glance

### Enhanced Workflow (v2.3)
```
/speckit.memory              # ← NEW: Create project memory (one-time setup)
    ↓
/speckit.specify              # Create specification → spec-metadata.json created
    ↓
/speckit.status               # ← NEW: Check workflow status
    ↓
/speckit.clarify              # Clarify ambiguities (optional)
    ↓
/speckit.validate-gap         # ← NEW: Assess feasibility (critical for HIGH-risk)
    ↓
/speckit.plan                 # Generate implementation plan with req traceability
    ↓
/speckit.status               # Check approval status before tasks
    ↓
/speckit.tasks                # Break down into tasks (optional) with EARS refs
    ↓
/speckit.implement            # Execute implementation (checks approval gates)
    ↓
/speckit.reconcile            # Post-implementation gap closure
    ↓
/speckit.status               # Verify feature complete
```

### Key Integration Points (v2.3)
1. **Memory System** → **All Commands**: Provides persistent context across sessions
2. **Gap Analysis** → **Planning**: Informs implementation approach and complexity
3. **EARS Requirements** → **Tasks** → **Tests**: Full traceability from spec to code
4. **Phase Tracking** → **Approval Gates**: Prevents proceeding with incomplete work
5. **Status Dashboard** → **Smart Recommendations**: Context-aware next steps

### Token Economy Impact (v2.3.0)
- **Cost Increase**: 10-15% for memory loading and metadata tracking
- **ROI**: 300-500% from persistent context (eliminates repetitive codebase analysis)
- **Justification**: Gap analysis prevents 1-2 week implementation delays from surprises

### Benefits for Framework Users (v2.3)
- ✅ **Persistent Knowledge**: AI "remembers" project context across sessions
- ✅ **Workflow Discipline**: Phase gates prevent incomplete implementations
- ✅ **Better Planning**: Gap analysis reduces surprises and improves estimates
- ✅ **Requirements Clarity**: EARS format ensures testable, unambiguous specs
- ✅ **Full Visibility**: Status dashboard shows exactly where you are in workflow
- ✅ **Smart Navigation**: Context-aware recommendations for next steps

## Quality Framework Enhancements (v2.2.0)

Spec-Kit v2.2.0 introduces evidence-based validation and component system auditing:

### Socratic Validation Questions (All Quality Commands)
- **4-Question Filter**: Every finding must answer "What user goal blocked?", "How many users affected?", "What is cost?", "Why this severity?"
- **Applied To**: `/speckit.analyze`, `/speckit.analyze-ux`, `/speckit.clarify --deep`
- **Benefit**: 40-60% reduction in false positives - only report issues with clear user impact

### Quick Wins Identification (High-Impact, Low-Effort Improvements)
- **Criteria**: High impact + 1-2 days effort + actionable + independent
- **Applied To**: `/speckit.analyze`, `/speckit.analyze-ux`, `/speckit.reconcile`
- **Output**: 3-5 quick wins per command with impact/effort estimates
- **Benefit**: Builds momentum, prioritizes by ROI, demonstrates immediate value

### Component System Audit (`/speckit.analyze-ux`)
- **Framework Support**: React/MUI, React/Chakra, Angular/Kendo (with MCP), Angular/Material, Vue/Vuetify, Generic
- **Analyzes**: Component consistency, design token adherence, reuse score, framework best practices
- **Templates**: 7 framework-specific audit templates in `.specify/templates/component-audit-templates/`
- **MCP Integration**: Kendo UI queries `kendo-angular-assistant` for official documentation (graceful fallback)
- **UI Design Quality**: 5 new dimensions (Visual Clarity, Component Consistency, Visual Hierarchy, Responsive Design, Design System Adherence)
- **Benefit**: Prevents custom component proliferation, enforces design system consistency

### Cross-Lens Conflicts Detection (`/speckit.clarify --deep`)
- **Detects**: Conflicts where expert recommendations contradict (Requirements vs Architecture, UX vs Performance, etc.)
- **Output**: Conflict description + tension + resolution options + recommended resolution
- **Benefit**: Reveals critical tradeoffs early, prevents late-stage architectural pivots

### Smart Recommendations (`/speckit.next`)
- **Quality-Driven**: Checks quality.json for overall_quality, critical_issues count
- **Risk-Aware**: Parses Risk Assessment from spec.md (HIGH/MEDIUM/LOW)
- **Context-Aware**: Detects UI-SPEC.md, hierarchical specs, gap reports
- **Quality-Gated Transitions**: Blocks/warns/suggests based on quality scores and risk levels
- **Benefit**: Prevents proceeding with low-quality or incomplete specs

### Token Economy Impact (v2.2.0)
- **Cost Increase**: 15-25% across enhanced commands
- **ROI**: 200-400% from prevented false positives, avoided rework, and architectural pivots
- **Justification**: Socratic validation prevents 5K-10K token waste per false positive, component audit saves 20K-50K in maintenance

## Workflow Integration & Phase Tracking (v2.3.0)

Spec-Kit v2.3.0 introduces unified state management, EARS requirements format, and comprehensive phase tracking:

### Memory System & State Management (`/speckit.memory`, `/speckit.status`, `/speckit.pm`, `/speckit.next`)
- **Three Inclusion Modes**: Always (constitution.md), Conditional (api-standards.md, testing-approach.md), Manual (deployment-runbook.md)
- **Persistent AI Context**: Project memory files loaded across sessions
- **Unified State Management**: spec-metadata.json as primary source, state.json as fallback
- **Phase Mapping**: Automatic conversion between lowercase (v2.3) and uppercase (v2.1) phase formats
- **Dual State Tracking**: Maintains both formats during v2.3 migration period

### EARS Requirements Format (`/speckit.specify`, `/speckit.plan`, `/speckit.tasks`)
- **Four Core Patterns**: WHEN [event] THEN [system] SHALL [response], IF [condition], WHILE [duration], WHERE [context]
- **Requirement IDs**: REQ-{CATEGORY}-{NUMBER}.{SUB} format (e.g., REQ-AUTH-1.1, REQ-DASH-2.1)
- **Full Traceability**: spec.md → plan.md → tasks.md → tests
- **Unambiguous Requirements**: Structured patterns eliminate interpretation ambiguity

### Phase Tracking & Approval Gates (`/speckit.specify`, `/speckit.plan`, `/speckit.tasks`, `/speckit.implement`, `/speckit.reconcile`)
- **Six Workflow Phases**: specification → gap_analysis → planning → tasks → implementation → reconciliation
- **Approval Gates**: Commands check spec-metadata.json before proceeding to prevent skipping steps
- **Phase Metadata**: Timestamps, approval status, quality scores tracked per phase
- **Workflow Validation**: /speckit.next validates prerequisites before recommending next command

### Gap Analysis & Feasibility (`/speckit.validate-gap`)
- **Feasibility Analysis**: Analyzes codebase before planning for reusable components
- **Three Implementation Options**: Recommended, Alternative, Minimal (with T-shirt sizing: S/M/L/XL)
- **Risk-Based Activation**: HIGH-risk features (score ≥8) automatically trigger gap analysis
- **Codebase Context**: Identifies existing patterns, services, utilities before planning

### Live Context Gathering (`/speckit.reconcile`, `/speckit.validate-hierarchy`)
- **Git Diff Execution**: Bash tool used for live context (not static examples)
- **Actual Implementation Evidence**: Shows what was ACTUALLY implemented vs specified
- **Silent Gap Detection**: Identifies gaps not in user's gap report via git diff
- **Auto-Detection**: Feature directory auto-detected from git branch

### Token Economy Impact (v2.3.0)
- **Cost Increase**: 10-20% across workflow commands (metadata operations, state management)
- **ROI**: 300-500% from prevented workflow skips, improved traceability, reduced rework
- **Justification**: Phase tracking prevents 10K-20K token waste from incorrect workflow sequencing, EARS format eliminates 5K-10K ambiguity clarification cycles

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

**Spec-Kit Version**: 2.3.0
**Target**: Claude Code
**License**: MIT
**Compatibility**: Claude Code 0.7.0+
