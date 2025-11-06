---
description: Analyze implementation feasibility by identifying gaps between specification requirements and existing codebase, providing implementation approach recommendations
allowed-tools: Bash, Glob, Grep, Read, Write, Task, WebFetch
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

This command performs **Gap Analysis** between your specification and existing codebase to:
- **Assess feasibility**: Can requirements be implemented with current architecture?
- **Identify reusable components**: What existing code can be leveraged?
- **Map implementation paths**: Should we extend existing code or create new components?
- **Surface technical risks**: What unknowns need research before planning?
- **Estimate complexity**: How much work is required?

Gap analysis is **critical for HIGH-risk features** and **recommended for all features** before planning begins.

## Execution Flow

### Phase 1: Load Specification & Requirements

**Step 1.1: Locate and Read Specification**
- Find spec.md in `.specify/specs/current/` or specified path
- Load spec-metadata.json (if exists) to get risk level
- Extract feature name, objectives, and requirements

**Step 1.2: Parse Requirements**

**If EARS format detected:**
```markdown
1. WHEN a user submits valid credentials THEN the Authentication Service SHALL grant access within 2 seconds
2. IF credentials are invalid THEN the Authentication Service SHALL reject access and log the attempt
```
→ Extract structured requirements with WHEN/IF/WHILE/WHERE patterns

**If traditional format:**
```markdown
- User must be able to log in with email and password
- System must validate credentials against database
- Failed login attempts must be logged
```
→ Extract requirements as numbered list

**Step 1.3: Categorize Requirements**

**Functional Requirements:**
- User interactions and business logic
- Data transformations and validations
- External integrations

**Non-Functional Requirements:**
- Performance (response times, throughput)
- Security (auth, encryption, compliance)
- Scalability and reliability
- Accessibility and usability

**Step 1.4: Extract Technical Components**

From requirements, identify needed components:
- **Services**: Business logic, external APIs
- **Data Models**: Entities, schemas, migrations
- **UI Components**: Forms, displays, interactions
- **Infrastructure**: Caching, queuing, monitoring

### Phase 2: Codebase Investigation

**Step 2.1: Detect Project Architecture**
- Identify framework (React, Angular, Vue, Express, Django, Spring, etc.)
- Detect architecture pattern (MVC, layered, microservices, serverless)
- Map directory structure to architecture layers

**Step 2.2: Search for Related Existing Code**

**For each technical component, search for:**

**Services/Business Logic:**
```text
# Search for service files
find src -name "*Service.*" -o -name "*service.*" -o -name "services/" 2>/dev/null

# Search for existing API clients
grep -r "axios\|fetch\|http" --include="*.js" --include="*.ts" src/ | head -20
```

**Data Models:**
```text
# Search for models/entities
find src -name "*Model.*" -o -name "*model.*" -o -name "models/" -o -name "entities/" 2>/dev/null

# Search for database schemas
find . -name "schema.*" -o -name "*migration*" -o -name "*.sql" 2>/dev/null
```

**UI Components:**
```text
# Search for components
find src -name "*Component.*" -o -name "components/" 2>/dev/null

# Search for existing forms
grep -r "form\|input\|button" --include="*.jsx" --include="*.tsx" src/components/ | head -20
```

**Authentication/Authorization:**
```text
# Search for auth patterns
grep -r "auth\|token\|jwt\|session" --include="*.js" --include="*.ts" src/ | head -30
```

**Step 2.3: Analyze Existing Patterns**

**For each found component:**
- **Naming conventions**: How are similar components named?
- **File structure**: Where are similar components located?
- **Dependencies**: What libraries are used?
- **Patterns**: How is similar logic implemented?

**Example Analysis:**
```markdown
**Existing Authentication Pattern:**
- Location: `src/services/AuthService.ts`
- Method: JWT tokens stored in localStorage
- Validation: Bearer token in Authorization header
- Libraries: `jsonwebtoken`, `bcrypt`

→ New OAuth implementation should follow same pattern and integrate with existing AuthService
```

### Phase 3: Gap Identification

**Step 3.1: Component Mapping**

**For each requirement, determine:**

**✅ Exists and Sufficient**
- Component exists and meets requirement fully
- Example: "User login with email/password" → AuthService.login() exists

**⚠️ Exists but Needs Extension**
- Component exists but needs modification
- Example: "OAuth login" → AuthService exists but only supports email/password

**🔴 Missing - Needs Creation**
- Component doesn't exist, must be built from scratch
- Example: "Two-factor authentication" → No 2FA code exists

**❓ Unclear - Needs Research**
- Unknown if implementation is possible or how to approach
- Example: "Integrate with Okta" → Need to research Okta SDK and compatibility

**Step 3.2: Dependency Analysis**

**Identify external dependencies needed:**
- **New libraries**: What npm packages, Python libraries, etc. are required?
- **External APIs**: What third-party services need integration?
- **Infrastructure**: New databases, caches, message queues needed?

**Check if dependencies are compatible:**
- Version compatibility with existing dependencies
- License compatibility
- Security vulnerabilities (check with npm audit, Snyk, etc.)

**Step 3.3: Data Model Impact**

**Assess database changes:**
- **New tables/collections**: What entities need to be created?
- **Schema changes**: Modifications to existing tables
- **Migrations**: How complex are migrations? (zero-downtime possible?)
- **Data volume**: Expected data growth and storage requirements

### Phase 4: Implementation Approach Options

**Present 3 implementation approaches with trade-offs:**

**Option A: Extend Existing Components** (Recommended if possible)

**Pros:**
- Lower risk (reuses tested code)
- Maintains architectural consistency
- Faster development
- Lower maintenance burden

**Cons:**
- May require refactoring existing code
- Could introduce coupling
- Backwards compatibility concerns

**Complexity**: Medium
**Timeline**: 2-3 weeks
**Risk**: Low

**Option B: Create New Components** (Greenfield)

**Pros:**
- Clean architecture
- No legacy constraints
- Easier to test in isolation
- Can use latest patterns/libraries

**Cons:**
- Higher initial effort
- May duplicate logic
- Integration complexity
- More code to maintain

**Complexity**: High
**Timeline**: 4-6 weeks
**Risk**: Medium

**Option C: Hybrid Approach** (Combination)

**Pros:**
- Balances reuse and clean design
- Minimizes risk
- Incremental migration path
- Flexibility

**Cons:**
- More planning required
- Potential for inconsistency
- Complex testing scenarios

**Complexity**: Medium-High
**Timeline**: 3-5 weeks
**Risk**: Medium

**Recommendation**: [Choose based on risk, timeline, and architectural fit]

**Rationale**: [Justify recommendation with specific analysis]

### Phase 5: Technical Research Requirements

**Identify knowledge gaps that need research before planning:**

**5.1: External API Integration**
- API documentation review needed
- Authentication/authorization mechanism
- Rate limits and quotas
- Error handling patterns

**5.2: Library/Framework Research**
- Best practices and gotchas
- Community support and maintenance
- Performance characteristics
- Security considerations

**5.3: Architectural Patterns**
- Design pattern suitability (Repository, Factory, Observer, etc.)
- State management approach
- Error boundary strategy
- Testing strategy

**5.4: Performance & Scalability**
- Load testing requirements
- Caching strategy
- Database indexing needs
- CDN or edge caching

**5.5: Security & Compliance**
- OWASP vulnerability assessment
- Compliance requirements (GDPR, HIPAA, PCI-DSS)
- Pen testing scope
- Audit logging requirements

### Phase 6: Complexity & Effort Estimation

**Assign T-shirt sizing based on analysis:**

**🟢 Small (S)**: 1-3 days
- Extend existing component with minor changes
- No new dependencies
- No database changes
- Low risk

**🟡 Medium (M)**: 1-2 weeks
- Create new component or significant extension
- 1-2 new dependencies
- Minor database changes
- Medium risk

**🟠 Large (L)**: 2-4 weeks
- Multiple new components
- Several new dependencies
- Significant database changes
- High complexity or risk

**🔴 Extra-Large (XL)**: 1-3 months
- Major architecture changes
- Many new components and dependencies
- Complex migrations
- Very high risk or unknowns

**Estimation Factors:**
- Codebase familiarity (low familiarity = +50% effort)
- Testing requirements (high coverage = +30% effort)
- Documentation needs (+10-20% effort)
- Code review and iteration (+20% effort)

### Phase 7: Generate Gap Analysis Report

**Create gap-analysis.md with structure:**

```markdown
# Gap Analysis: [Feature Name]

**Date**: [YYYY-MM-DD]
**Risk Level**: [HIGH/MEDIUM/LOW]
**Complexity**: [S/M/L/XL]
**Estimated Effort**: [Timeline]

---

## Executive Summary

[2-3 paragraph summary of feasibility, approach, and key findings]

---

## Requirements Analysis

### Functional Requirements (N total)
1. [Requirement 1] → ✅ Exists | ⚠️ Extend | 🔴 Create | ❓ Research
2. [Requirement 2] → ...

### Non-Functional Requirements (N total)
1. [Performance requirement] → Status
2. [Security requirement] → Status

---

## Existing Codebase Analysis

### Reusable Components
- **[Component Name]**: [Location] - [How it can be leveraged]
- ...

### Architecture Patterns
- **Pattern**: [MVC / Layered / Microservices / etc.]
- **Naming Conventions**: [kebab-case / camelCase / PascalCase]
- **File Organization**: [Description]

### Technology Stack
- **Language/Framework**: [Detected stack]
- **Key Libraries**: [Relevant dependencies]
- **Database**: [PostgreSQL / MySQL / MongoDB / etc.]

---

## Gap Identification

### ✅ Exists and Sufficient (N items)
- [Component]: [Why it's sufficient]

### ⚠️ Needs Extension (N items)
- [Component]: [What needs to change]
  - **Impact**: [Low / Medium / High]
  - **Effort**: [1-3 days / 1-2 weeks / etc.]

### 🔴 Missing - Needs Creation (N items)
- [Component]: [What needs to be built]
  - **Dependencies**: [Libraries / APIs needed]
  - **Complexity**: [Low / Medium / High]
  - **Effort**: [Estimate]

### ❓ Needs Research (N items)
- [Topic]: [What's unknown]
  - **Research Questions**: [Specific questions to answer]
  - **Time to Research**: [Hours / Days]

---

## Implementation Approach Options

### Option A: Extend Existing Components ⭐ RECOMMENDED
**Approach**: [Description]
**Pros**: [List]
**Cons**: [List]
**Complexity**: [S/M/L/XL]
**Timeline**: [Estimate]
**Risk**: [Low/Medium/High]

### Option B: Create New Components
**Approach**: [Description]
**Pros**: [List]
**Cons**: [List]
**Complexity**: [S/M/L/XL]
**Timeline**: [Estimate]
**Risk**: [Low/Medium/High]

### Option C: Hybrid Approach
**Approach**: [Description]
**Pros**: [List]
**Cons**: [List]
**Complexity**: [S/M/L/XL]
**Timeline**: [Estimate]
**Risk**: [Low/Medium/High]

### Recommendation
**Selected Option**: [A/B/C]
**Rationale**: [Detailed justification based on analysis]

---

## Technical Research Requirements

### High Priority Research
1. **[Topic]**: [Research questions and expected outcome]
   - **Resources**: [Documentation links, examples]
   - **Time Estimate**: [Hours/Days]

### Medium Priority Research
[Similar structure]

---

## Dependencies & External Integrations

### New Dependencies Required
| Package | Version | Purpose | License | Security |
|---------|---------|---------|---------|----------|
| oauth2-lib | ^2.0.0 | OAuth 2.0 client | MIT | No vulnerabilities |

### External APIs/Services
| Service | Purpose | Rate Limits | Cost | SLA |
|---------|---------|-------------|------|-----|
| Okta | Identity provider | 1000 req/min | $X/month | 99.9% |

---

## Database Impact

### New Tables/Collections
- **[table_name]**: [Purpose and schema overview]

### Schema Changes
- **[existing_table]**: [Modifications needed]
  - Migration complexity: [Low/Medium/High]
  - Zero-downtime possible: [Yes/No]

### Data Volume Estimates
- **Expected rows**: [Number]
- **Growth rate**: [Per day/month]
- **Storage requirements**: [GB/TB]

---

## Risk Assessment

### Technical Risks
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| [Risk description] | High/Med/Low | High/Med/Low | [Strategy] |

### Dependencies Risks
- **[Dependency]**: [Risk and mitigation]

### Timeline Risks
- [Risk factors that could extend timeline]

---

## Effort Estimation

**Overall Complexity**: 🟡 Medium (M)
**Estimated Timeline**: 2-3 weeks
**Confidence Level**: Medium (±30%)

**Breakdown**:
- Requirements analysis: 0.5 days
- Component extension: 3-4 days
- New component creation: 5-7 days
- Testing: 2-3 days
- Documentation: 1 day
- Buffer: 2-3 days

---

## Next Steps

1. ✅ Gap analysis complete
2. ⏭️ [High-priority research] (if needed)
3. ⏭️ Run `/speckit.plan` to generate implementation plan
4. ⏭️ Review and approve plan
5. ⏭️ Run `/speckit.tasks` or `/speckit.implement`

---

## Approval

**Gap Analysis Status**: 🟢 Complete
**Ready for Planning**: ✅ Yes / 🔴 No (reason: [blocker])

**Approver**: [Name] | **Date**: [YYYY-MM-DD]
```

### Phase 8: Update Metadata

**Update spec-metadata.json:**
```json
{
  "approvals": {
    "gap_analysis": {
      "generated": true,
      "approved": false,  # User must approve
      "timestamp": "2025-11-05T14:00:00Z"
    }
  }
}
```

### Phase 9: Gap Analysis Review Gate

**Evidence-Based Self-Check** (Mandatory):

❓ **"Were all requirements categorized?"**
- Count requirements vs categorized items
- Report: % coverage (should be 100%)

❓ **"Was codebase investigation performed?"**
- Show search commands executed
- Report: Number of relevant files found

❓ **"Were implementation options evaluated?"**
- Show all 3 options with pros/cons
- Report: Recommended option with rationale

❓ **"Were risks and unknowns identified?"**
- List technical research requirements
- Report: High-priority research items

---

## Usage Examples

**Analyze current specification:**
```
/speckit.validate-gap
```

**Analyze specific feature:**
```
/speckit.validate-gap oauth-integration
/speckit.validate-gap .specify/specs/user-dashboard/
```

**With auto-approval:**
```
/speckit.validate-gap -y
```

---

## Integration with Workflow

**Gap Analysis fits between Specification and Planning:**

```
/speckit.specify          # Create specification
    ↓
/speckit.clarify          # Clarify ambiguities (optional)
    ↓
/speckit.validate-gap     # ← ASSESS FEASIBILITY
    ↓
/speckit.plan             # Generate implementation plan
    ↓
/speckit.tasks            # Break down into tasks (optional)
    ↓
/speckit.implement        # Execute implementation
```

**When to use Gap Analysis:**
- **Always**: HIGH-risk features (risk score ≥ 8)
- **Recommended**: MEDIUM-risk features (risk score 4-7)
- **Optional**: LOW-risk features (risk score 0-3), simple extensions
- **Always**: When integrating with unfamiliar external APIs or libraries
- **Always**: When significant architecture changes are anticipated

---

**Last Updated**: 2025-11-05
**Version**: 1.0.0
