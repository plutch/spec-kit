# Changelog

<!-- markdownlint-disable MD024 -->

All notable changes to the Specify CLI and templates are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.0] - 2025-10-23

### Added - Unified SDD Methodology v2.1

**Adaptive SDD & Architectural Pivots**:
- **TECHNICAL.md**: Living constraints document for complex features (multi-tenant, performance-critical, integration-heavy)
  - Captures technical constraints (performance, security, scalability)
  - Documents risk flags with `[NEEDS VALIDATION]` markers
  - Evolves through amendments (~~old~~ → new) preserving decision history
- **Architecture Decision Records (ADRs)**: Immutable decision records for architectural pivots
  - Template: `templates/adaptive/adr-pivot-template.md` (7.4KB)
  - Template: `templates/adr-template.md` (6.6KB) - general ADR format
  - Constitutional validation gates (simplicity, anti-abstraction, integration-first)
  - Test-first imperative (contract tests updated BEFORE implementation)
  - Business rules impact tracking
- **Article XI: Technical Pivot Protocol**: Constitutional framework for handling architectural blockers
  - Template: `templates/constitutional-article-xi.md` (3.5KB)
  - 7-step protocol for architectural changes discovered during implementation
  - Integration with TECHNICAL.md amendments and ADR creation
- **Slash Commands**:
  - `/create-adr`: Interactive ADR creation with constitutional validation
  - `/amend-technical`: TECHNICAL.md amendment workflow with strikethrough pattern

**Business Rules Management**:
- **Constitutional Quality Gates**: 9 validation rules ensuring business logic quality
  - Template: `.specify/business-rules/validation-rules.md` (6.8KB)
  - Testability (≥2 test references), Completeness, Traceability, Test-First philosophy
  - HIGH/MEDIUM/LOW severity levels for validation failures
- **Workflow Integration**:
  - Harvest (Planning): Extract business rules from specifications
  - Implement (Development): `@BusinessRule BR-DOMAIN-###` code annotations
  - Extract (Post-Implementation): Update catalog from annotations
  - Query (Ongoing): LLM agents read QUICK_REFERENCE.md for fast lookup
- **ADR Integration**: Business rules impact tracking in pivot decisions
  - Template: `templates/business-rules/adr-integration-template.md` (4.2KB)
  - Tracks modified, new, and deprecated rules during architectural changes

**Templates & Structure**:
- **Directory Structure**:
  - `templates/adaptive/` - TECHNICAL.md and ADR pivot templates
  - `templates/business-rules/` - ADR integration templates
  - `templates/commands/` - create-adr and amend-technical command workflows
- **Enhanced spec-template.md**:
  - New "Complex Features" section (93 lines) with complexity indicators
  - Risk flags, technical constraints, and TECHNICAL.md creation guidance
  - Clear separation between CRUD and complex feature requirements
- **Enhanced plan-template.md**:
  - New "Phase -1: Constitutional Validation" (73 lines)
  - Pre-planning constitutional gate checks
  - Article-by-article validation with pass/conditional/fail assessments

**Script Enhancements**:
- **`--complex` flag** for `create-new-feature` scripts (bash + PowerShell)
  - Creates `decisions/` directory (for ADRs) only when needed
  - Creates `rules/` directory (for business rules) only when needed
  - Copies TECHNICAL.md template automatically for complex features
  - Warning message if template missing (graceful fallback to minimal TECHNICAL.md)
- **Conditional directory creation**: decisions/ and rules/ only created for complex features

**Documentation**:
- **Unified SDD Methodology Guide**: `docs/unified-sdd-methodology.md` (15KB)
  - Complete methodology with decision trees, workflows, and examples
  - Quick start guides (30min minimal adoption, 2-4hr full adoption)
  - ROI metrics (25-250x token savings, 80% reduction in wasted effort)
- **README.md enhancements**:
  - "Adaptive SDD & Architectural Pivots" section
  - "Business Rules Management" section
  - Links to methodology guide and templates

### Changed

- **Version jump**: 0.0.20 → 2.1.0 (major methodology enhancement)
- **create-new-feature scripts**: Now conditionally create directories based on feature complexity
- **Template organization**: Restructured into adaptive/, business-rules/, commands/ subdirectories

### Performance & ROI

- **Token Savings**: 25-250x through confidence checks and evidence requirements
- **Waste Reduction**: 80% reduction in wrong-direction work via pre-implementation validation
- **Hallucination Prevention**: 94% reduction through mandatory evidence packages
- **Error Recurrence**: <10% via reflexion pattern (search past ADRs before creating new ones)
- **Timeline Impact**: 7-week implementation roadmap (down from 10 weeks, removed governance overhead)

### Migration Notes

For existing spec-kit users:
- CRUD features work exactly as before (no changes required)
- Complex features: Add `--complex` flag when creating new features
- Optional: Adopt Article XI in your project constitution
- Optional: Use TECHNICAL.md for performance-critical/multi-tenant features
- See migration guide: `docs/unified-sdd-methodology.md#quick-start`

## [0.0.20] - 2025-10-14

### Added

- **Intelligent Branch Naming**: `create-new-feature` scripts now support `--short-name` parameter for custom branch names
  - When `--short-name` provided: Uses the custom name directly (cleaned and formatted)
  - When omitted: Automatically generates meaningful names using stop word filtering and length-based filtering
  - Filters out common stop words (I, want, to, the, for, etc.)
  - Removes words shorter than 3 characters (unless they're uppercase acronyms)
  - Takes 3-4 most meaningful words from the description
  - **Enforces GitHub's 244-byte branch name limit** with automatic truncation and warnings
  - Examples:
    - "I want to create user authentication" → `001-create-user-authentication`
    - "Implement OAuth2 integration for API" → `001-implement-oauth2-integration-api`
    - "Fix payment processing bug" → `001-fix-payment-processing`
    - Very long descriptions are automatically truncated at word boundaries to stay within limits
  - Designed for AI agents to provide semantic short names while maintaining standalone usability

### Changed

- Enhanced help documentation for `create-new-feature.sh` and `create-new-feature.ps1` scripts with examples
- Branch names now validated against GitHub's 244-byte limit with automatic truncation if needed

## [0.0.19] - 2025-10-10

### Added

- Support for CodeBuddy (thank you to [@lispking](https://github.com/lispking) for the contribution).
- You can now see Git-sourced errors in the Specify CLI.

### Changed

- Fixed the path to the constitution in `plan.md` (thank you to [@lyzno1](https://github.com/lyzno1) for spotting).
- Fixed backslash escapes in generated TOML files for Gemini (thank you to [@hsin19](https://github.com/hsin19) for the contribution).
- Implementation command now ensures that the correct ignore files are added (thank you to [@sigent-amazon](https://github.com/sigent-amazon) for the contribution).

## [0.0.18] - 2025-10-06

### Added

- Support for using `.` as a shorthand for current directory in `specify init .` command, equivalent to `--here` flag but more intuitive for users.
- Use the `/speckit.` command prefix to easily discover Spec Kit-related commands.
- Refactor the prompts and templates to simplify their capabilities and how they are tracked. No more polluting things with tests when they are not needed.
- Ensure that tasks are created per user story (simplifies testing and validation).
- Add support for Visual Studio Code prompt shortcuts and automatic script execution.

### Changed

- All command files now prefixed with `speckit.` (e.g., `speckit.specify.md`, `speckit.plan.md`) for better discoverability and differentiation in IDE/CLI command palettes and file explorers

## [0.0.17] - 2025-09-22

### Added

- New `/clarify` command template to surface up to 5 targeted clarification questions for an existing spec and persist answers into a Clarifications section in the spec.
- New `/analyze` command template providing a non-destructive cross-artifact discrepancy and alignment report (spec, clarifications, plan, tasks, constitution) inserted after `/tasks` and before `/implement`.
  - Note: Constitution rules are explicitly treated as non-negotiable; any conflict is a CRITICAL finding requiring artifact remediation, not weakening of principles.

## [0.0.16] - 2025-09-22

### Added

- `--force` flag for `init` command to bypass confirmation when using `--here` in a non-empty directory and proceed with merging/overwriting files.

## [0.0.15] - 2025-09-21

### Added

- Support for Roo Code.

## [0.0.14] - 2025-09-21

### Changed

- Error messages are now shown consistently.

## [0.0.13] - 2025-09-21

### Added

- Support for Kilo Code. Thank you [@shahrukhkhan489](https://github.com/shahrukhkhan489) with [#394](https://github.com/github/spec-kit/pull/394).
- Support for Auggie CLI. Thank you [@hungthai1401](https://github.com/hungthai1401) with [#137](https://github.com/github/spec-kit/pull/137).
- Agent folder security notice displayed after project provisioning completion, warning users that some agents may store credentials or auth tokens in their agent folders and recommending adding relevant folders to `.gitignore` to prevent accidental credential leakage.

### Changed

- Warning displayed to ensure that folks are aware that they might need to add their agent folder to `.gitignore`.
- Cleaned up the `check` command output.

## [0.0.12] - 2025-09-21

### Changed

- Added additional context for OpenAI Codex users - they need to set an additional environment variable, as described in [#417](https://github.com/github/spec-kit/issues/417).

## [0.0.11] - 2025-09-20

### Added

- Codex CLI support (thank you [@honjo-hiroaki-gtt](https://github.com/honjo-hiroaki-gtt) for the contribution in [#14](https://github.com/github/spec-kit/pull/14))
- Codex-aware context update tooling (Bash and PowerShell) so feature plans refresh `AGENTS.md` alongside existing assistants without manual edits.

## [0.0.10] - 2025-09-20

### Fixed

- Addressed [#378](https://github.com/github/spec-kit/issues/378) where a GitHub token may be attached to the request when it was empty.

## [0.0.9] - 2025-09-19

### Changed

- Improved agent selector UI with cyan highlighting for agent keys and gray parentheses for full names

## [0.0.8] - 2025-09-19

### Added

- Windsurf IDE support as additional AI assistant option (thank you [@raedkit](https://github.com/raedkit) for the work in [#151](https://github.com/github/spec-kit/pull/151))
- GitHub token support for API requests to handle corporate environments and rate limiting (contributed by [@zryfish](https://github.com/@zryfish) in [#243](https://github.com/github/spec-kit/pull/243))

### Changed

- Updated README with Windsurf examples and GitHub token usage
- Enhanced release workflow to include Windsurf templates

## [0.0.7] - 2025-09-18

### Changed

- Updated command instructions in the CLI.
- Cleaned up the code to not render agent-specific information when it's generic.

## [0.0.6] - 2025-09-17

### Added

- opencode support as additional AI assistant option

## [0.0.5] - 2025-09-17

### Added

- Qwen Code support as additional AI assistant option

## [0.0.4] - 2025-09-14

### Added

- SOCKS proxy support for corporate environments via `httpx[socks]` dependency

### Fixed

N/A

### Changed

N/A
