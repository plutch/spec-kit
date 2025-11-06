---
description: Create or update project memory files (constitution, API standards, testing approach, deployment runbook) for persistent AI context across sessions
allowed-tools: Bash, Glob, Grep, Read, Write, Edit
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

This command creates and maintains **Project Memory** files that provide persistent AI context across all sessions and commands. These memory files act as the "long-term memory" of the project, capturing knowledge that would otherwise need to be re-discovered in every session.

## Memory File Types & Inclusion Modes

### Always Loaded (Core Context)
**File**: `.specify/memory/constitution.md`
**Loaded**: Every command, every session
**Purpose**: Core project principles, technology stack, file organization, development standards
**Update Frequency**: Major project changes, new team members, tech stack evolution

### Conditionally Loaded (Context-Aware)
**File**: `.specify/memory/api-standards.md`
**Loaded**: When working in API-related files (`src/api/**/*`, `**/controllers/**/*`, `**/routes/**/*`)
**Purpose**: REST API conventions, request/response formats, error handling, versioning
**Update Frequency**: API pattern changes, new endpoints, standards evolution

**File**: `.specify/memory/testing-approach.md`
**Loaded**: When working in test files (`**/*.test.*`, `**/*.spec.*`, `**/tests/**/*`)
**Purpose**: Testing strategy, coverage requirements, TDD practices, mocking guidelines
**Update Frequency**: Testing tooling changes, new testing patterns, coverage policy updates

### Manual Loading (On-Demand)
**File**: `.specify/memory/deployment-runbook.md`
**Loaded**: Via `@deployment-runbook.md` syntax when explicitly needed
**Purpose**: Deployment procedures, infrastructure configuration, incident response
**Update Frequency**: Infrastructure changes, new deployment strategies, operational learnings

## Execution Flow

### Phase 1: Context Discovery & Analysis

**Step 1.1: Detect Project Structure**
- Get project overview (package.json, requirements.txt, pom.xml, etc.)
- Get directory structure (max 2-3 levels deep, exclude build dirs)
- Detect framework/technology from file extensions

**Step 1.2: Read Existing Documentation**
- Check for README.md, ARCHITECTURE.md, CONTRIBUTING.md
- Extract project description, technology stack, architecture patterns
- Identify existing conventions and standards

**Step 1.3: Check Existing Memory Files**
- List existing memory files with sizes
- Check git history for manual edits
- Determine update strategy based on git history

**IMPORTANT**: If git history shows manual edits (commits not by "spec-kit" or "memory command"), preserve user customizations. Only add/update factual information, never remove user-specific content.

### Phase 2: Memory File Selection

**Based on user input, determine which memory files to create/update:**

1. **No arguments or "all"** → Process all four memory files
2. **"constitution"** → Only constitution.md
3. **"api"** → Only api-standards.md
4. **"testing"** → Only testing-approach.md
5. **"deployment"** → Only deployment-runbook.md
6. **Multiple args** (e.g., "constitution api") → Process specified files

**If user provides specific content**, use it to populate relevant sections. Otherwise, extract from project context.

### Phase 3: Constitution.md (Core Project Memory)

**Template Location**: `.specify/memory/constitution.md`

**Information to Extract/Update:**

**3.1: Project Overview**
- **What**: Project purpose and domain (from README.md, package.json description)
- **Why**: Business value and user benefit (from docs, mission statements)
- **Who**: Target users and stakeholders (from docs, user stories)

**3.2: Technology Stack**
- **Language**: Detect from file extensions
- **Framework**: Detect from dependencies
- **Database**: Search for database client libraries
- **Infrastructure**: Check for Docker, Kubernetes configs, cloud provider files

**3.3: Project Structure**
- Generate directory tree (max 3 levels deep, exclude node_modules/build/dist)
- Document purpose of major directories
- Extract naming conventions from existing files

**3.4: Development Standards**
- **Style Guide**: Check for .eslintrc, .prettierrc, etc.
- **Testing Framework**: Detect Jest, Pytest, JUnit, etc.
- **Documentation**: Check for JSDoc, Sphinx, Javadoc patterns

**3.5: Quality & Security Baseline**
- Look for security dependencies
- Check for auth/authz patterns
- Identify performance monitoring tools

**3.6: Deployment Context**
- Check for CI/CD configs
- Identify deployment targets

**Preservation Rules:**
- If constitution.md exists and has custom sections, preserve them
- Only update factual information
- Mark unclear information as `[TODO: Verify]`
- Increment version appropriately

### Phase 4: API Standards (Conditional Memory)

**Template Location**: `.specify/memory/api-standards.md`

**When to Create**: If project has API endpoints

**Information to Extract/Update:**
- Detect REST API patterns
- Extract conventions (resource naming, HTTP methods, response formats)
- Check version strategy
- Identify auth patterns

### Phase 5: Testing Approach (Conditional Memory)

**Template Location**: `.specify/memory/testing-approach.md`

**When to Create**: If project has tests

**Information to Extract/Update:**
- Detect testing framework
- Extract testing patterns (AAA, Given-When-Then, etc.)
- Check coverage requirements
- Identify test types (unit, integration, E2E)

### Phase 6: Deployment Runbook (Manual Memory)

**Template Location**: `.specify/memory/deployment-runbook.md`

**When to Create**: If deployment infrastructure exists

**Information to Extract/Update:**
- Detect deployment strategy
- Extract deployment patterns
- Check for IaC tools
- Identify monitoring/logging setup

### Phase 7: Smart Update Strategy

**For Each Memory File:**

**7.1: Check Git History**
- Get commit count for file
- Get last commit author and message

**7.2: Determine Update Approach**

**Scenario A: File doesn't exist**
→ Create from template with extracted project information
→ Set initial version to 1.0.0

**Scenario B: File exists, no manual edits**
→ Safe to update factual information
→ Increment version (PATCH for factual updates)

**Scenario C: File exists, manual edits detected**
→ **Preserve all user content**
→ Only add NEW factual information in comments
→ Report: "File has manual edits - added updates as comments only"

**Scenario D: User requests overwrite ("force" or "reset")**
→ Backup existing file to `.specify/memory/<filename>.backup.md`
→ Overwrite with fresh template and extracted info
→ Report: "File backed up and reset"

### Phase 8: Validation & Quality Check

**For Each Updated Memory File:**

✅ **Completeness Check**
- No `[PLACEHOLDER]` tokens remain (or marked as `[TODO: Research]`)
- All major sections have content
- Version number updated
- Last Updated date is today

✅ **Accuracy Check**
- Technology stack matches actual dependencies
- File structure reflects current repository
- Standards match existing code patterns

✅ **Clarity Check**
- Instructions are project-specific
- Examples use actual project patterns
- Conditional files include trigger patterns

✅ **Git Awareness Check**
- User customizations preserved
- Update approach documented
- Backup created if overwriting

### Phase 9: Final Report

**Present Memory Update Report:**

```markdown
## Project Memory Update Report

### Files Processed
- [ ] constitution.md → [Created / Updated / Preserved]
- [ ] api-standards.md → [Created / Updated / Skipped / Preserved]
- [ ] testing-approach.md → [Created / Updated / Skipped / Preserved]
- [ ] deployment-runbook.md → [Created / Updated / Skipped / Preserved]

### Extracted Project Information
**Technology Stack:**
- Language: [Detected]
- Framework: [Detected]
- Database: [Detected]
- Infrastructure: [Detected]

**Project Patterns:**
- API Style: [REST / GraphQL / gRPC / N/A]
- Testing Framework: [Jest / Pytest / JUnit / N/A]
- Deployment: [Docker / Kubernetes / Serverless / N/A]

### Next Steps
1. Review generated memory files for accuracy
2. Fill in any [TODO: Verify] placeholders
3. Customize memory files for project-specific needs
4. Run `/speckit.status` to see memory file usage
```

### Phase 10: Memory Creation Review Gate

**Evidence-Based Self-Check** (Mandatory):

❓ **"Were memory files created/updated?"**
- List all files in `.specify/memory/` with sizes
- Verify each file has meaningful content (>1KB)

❓ **"Was project information extracted?"**
- Show detected technology stack
- Show detected project patterns

❓ **"Were user customizations preserved?"**
- Check git history for manual edits
- Verify no user content was removed

---

## Usage Examples

**Create all memory files:**
```
/speckit.memory
```

**Update only constitution:**
```
/speckit.memory constitution
```

**Update API and testing memory:**
```
/speckit.memory api testing
```

**Force reset constitution:**
```
/speckit.memory constitution force
```

---

**Last Updated**: 2025-11-05
**Version**: 1.0.0
