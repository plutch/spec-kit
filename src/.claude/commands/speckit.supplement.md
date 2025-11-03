# Supplement - Create Supplementary Specifications

**Command**: `/speckit.supplement`
**Phase**: SPECIFYING (0-20%)
**Purpose**: Create domain-specific supplementary specifications for large, multi-domain features
**Pattern**: Manual creation with structured templates

---

## Overview

The supplement command creates hierarchical specifications by generating supplementary spec files (UI-SPEC.md, API-SPEC.md, TECHNICAL-SPEC.md) that extend the parent spec.md with detailed implementation guidance.

**Use Case**: Large features (>150KB, >80 FRs) spanning multiple domains benefit from splitting concerns:
- **Parent spec.md**: WHAT and WHY (functional requirements, user stories)
- **Supplementary specs**: HOW (domain-specific implementation details)

**Benefits**:
- Token efficiency: Agents load only relevant specs (~50KB vs ~279KB)
- Clear ownership: UI team owns UI-SPEC.md, Backend team owns API-SPEC.md
- Maintainability: Update domain specs without touching parent

---

## When to Use

Use `/speckit.supplement` when:
- ✅ Parent spec.md is large (>150KB or >80 FRs)
- ✅ Feature spans multiple domains (UI + API + Architecture)
- ✅ Specialized agents need focused context (frontend-architect, backend-architect)
- ✅ Implementation details would clutter parent spec

Do NOT use supplement when:
- ❌ Feature is small (<100KB, <50 FRs)
- ❌ Single domain (backend-only or UI-only)
- ❌ Simple CRUD application

**Decision Rule**:
```
if (specSizeKB > 150 || frCount > 80 || hasMultipleDomains) {
  USE: /speckit.supplement
} else {
  USE: Single spec.md (traditional approach)
}
```

---

## Usage

```bash
/speckit.supplement <scope> "<description>"
```

**Scopes**:
- `ui-ux` → Creates UI-SPEC.md (component designs, layouts, accessibility)
- `api-contracts` → Creates API-SPEC.md (endpoint contracts, schemas, error codes)
- `technical` → Creates TECHNICAL-SPEC.md (architecture, database, observability)

**Examples**:
```bash
# Create UI supplementary spec
/speckit.supplement ui-ux "Complete UI/UX specifications for billing console with Angular 20 + Kendo UI"

# Create API supplementary spec
/speckit.supplement api-contracts "REST API contracts for payment processing and webhook handling"

# Create Technical supplementary spec
/speckit.supplement technical "Architecture decisions, database schema, and multi-tenant isolation design"
```

---

## Execution Protocol

### Step 1: Validate Parent Spec Exists

**Check**: Ensure spec.md exists in current feature directory

```bash
if [ ! -f "specs/${FEATURE_ID}/spec.md" ]; then
  echo "❌ ERROR: Parent spec.md not found"
  echo "Run /speckit.specify first to create parent specification"
  exit 1
fi
```

---

### Step 2: Check for Existing Supplementary Spec

**Check**: Prevent overwriting existing supplementary specs

```bash
SCOPE_TO_FILENAME = {
  "ui-ux": "UI-SPEC.md",
  "api-contracts": "API-SPEC.md",
  "technical": "TECHNICAL-SPEC.md"
}

FILENAME = SCOPE_TO_FILENAME[scope]
FILEPATH = "specs/${FEATURE_ID}/${FILENAME}"

if [ -f "${FILEPATH}" ]; then
  echo "❌ ERROR: ${FILENAME} already exists at ${FILEPATH}"
  echo "Use /speckit.amend-technical to modify existing supplementary spec"
  exit 1
fi
```

---

### Step 3: Extract Parent Spec Metadata

**Parse** parent spec.md to extract:
- Feature name and number
- Functional requirements (FR-001, FR-002, etc.)
- User stories
- Success criteria

**Example Extraction**:
```bash
# Extract FRs from parent spec
FRS=$(grep -oP 'FR-\d{3}' specs/${FEATURE_ID}/spec.md | sort -u)
FR_LIST=$(echo "$FRS" | tr '\n' ', ' | sed 's/, $//')

# Extract feature metadata
FEATURE_NAME=$(grep '^# ' specs/${FEATURE_ID}/spec.md | head -1 | sed 's/^# //')
```

---

### Step 4: Load Supplementary Spec Template

**Template**: `.specify/templates/supplementary-spec-template.md`

**Placeholder Replacement**:
```yaml
{SCOPE}               → "ui-ux-implementation" | "api-contracts" | "technical-architecture"
{SCOPE_DISPLAY}       → "UI/UX Implementation" | "API Contracts" | "Technical Architecture"
{FEATURE_ID}          → "000003"
{FEATURE_NAME}        → "billing-console"
{DATE}                → "2025-11-01"
{FR_LIST}             → "FR-001, FR-002, ..., FR-080"
{FR_RANGE}            → "FR-001 through FR-080"
{DESCRIPTION}         → User-provided description
{TARGET_AGENTS}       → Scope-specific agents (frontend-architect, backend-architect, etc.)
```

---

### Step 5: Generate Frontmatter

**YAML Frontmatter** (added at top of supplementary spec):

```yaml
---
parent: spec.md
scope: ui-ux-implementation
functional_requirements: [FR-001 through FR-080]
target_agents:
  - frontend-architect
version: 1.0
last_updated: 2025-11-01
---
```

**Scope-Specific Target Agents**:
```yaml
ui-ux:
  target_agents:
    - frontend-architect

api-contracts:
  target_agents:
    - backend-architect
    - system-architect

technical:
  target_agents:
    - system-architect
    - postgres-entity-architect
    - devops-architect
```

---

### Step 6: Populate Template Sections

**For UI-SPEC.md** (ui-ux scope):
```markdown
# UI/UX Implementation Specification
# Feature: {FEATURE_NAME}

{FRONTMATTER}

## 1. Overview
{DESCRIPTION}

This supplementary specification provides detailed UI/UX implementation guidance for the {FEATURE_NAME} feature. It extends the parent [spec.md](spec.md) with component designs, responsive layouts, accessibility patterns, and user interaction flows.

**Implements**: {FR_RANGE}

## 2. Component Architecture

### 2.1 Component Hierarchy
[PLACEHOLDER: Define Angular component tree]

### 2.2 State Management
[PLACEHOLDER: NgRx/Signals/Service-based state]

### 2.3 Routing
[PLACEHOLDER: Route definitions and guards]

## 3. Page Layouts

### 3.1 Desktop Layout (≥1200px)
[PLACEHOLDER: Desktop wireframes and specifications]

### 3.2 Tablet Layout (768px - 1199px)
[PLACEHOLDER: Tablet responsive behavior]

### 3.3 Mobile Layout (<768px)
[PLACEHOLDER: Mobile optimizations]

## 4. UI Components

### 4.1 [Component Name]
**Implements**: FR-XXX
**Description**: [Component purpose]
**Props/Inputs**: [List]
**Events/Outputs**: [List]
**Kendo UI Components**: [If applicable]

[Repeat for each major component]

## 5. User Interactions

### 5.1 Navigation Flows
[PLACEHOLDER: User journey maps]

### 5.2 Form Validation
[PLACEHOLDER: Validation rules and error messages]

### 5.3 Loading States
[PLACEHOLDER: Skeleton screens, spinners]

## 6. Accessibility (WCAG 2.1 AA)

### 6.1 Keyboard Navigation
[PLACEHOLDER: Tab order, shortcuts]

### 6.2 Screen Reader Support
[PLACEHOLDER: ARIA labels, roles]

### 6.3 Color Contrast
[PLACEHOLDER: Color palette compliance]

## 7. Performance Targets

- **Initial Load**: <3s (4G connection)
- **Time to Interactive**: <5s
- **Bundle Size**: <500KB (gzipped)
- **Lighthouse Score**: >90

## 8. Browser Compatibility

- Chrome 100+
- Firefox 100+
- Safari 15+
- Edge 100+

## 9. References

- **Parent Spec**: [spec.md](spec.md)
- **Implementation Plan**: [plan.md](plan.md) (generated after /speckit.plan)
- **Task Checklist**: [tasks.md](tasks.md) (generated after /speckit.tasks)
```

**For API-SPEC.md** (api-contracts scope):
```markdown
# API Contracts Specification
# Feature: {FEATURE_NAME}

{FRONTMATTER}

## 1. Overview
{DESCRIPTION}

This supplementary specification defines REST API contracts, request/response schemas, error codes, and integration patterns for the {FEATURE_NAME} feature.

**Implements**: {FR_RANGE}

## 2. API Endpoints

### 2.1 Endpoint: [METHOD /path]
**Implements**: FR-XXX
**Description**: [Endpoint purpose]

#### Request
```json
{
  "param1": "string",
  "param2": "number"
}
```

#### Response (200 OK)
```json
{
  "data": {...},
  "meta": {...}
}
```

#### Error Responses
- `400 Bad Request`: Invalid parameters
- `401 Unauthorized`: Authentication failed
- `403 Forbidden`: Insufficient permissions
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

[Repeat for each endpoint]

## 3. Data Schemas

### 3.1 Schema: [SchemaName]
```typescript
interface SchemaName {
  id: string;
  name: string;
  // ... fields
}
```

## 4. Authentication & Authorization

### 4.1 Authentication
- **Method**: JWT Bearer Token
- **Header**: `Authorization: Bearer <token>`

### 4.2 Authorization
- **Permissions**: [List required permissions]

## 5. Rate Limiting

- **Limit**: 100 requests per minute per API key
- **Headers**: `X-RateLimit-Limit`, `X-RateLimit-Remaining`

## 6. Webhooks

### 6.1 Webhook: [event.type]
**Triggers**: [When this webhook fires]
**Payload**: [JSON schema]

## 7. Error Handling

### 7.1 Error Response Format
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {...}
  }
}
```

## 8. Versioning

- **Current Version**: v1
- **Header**: `Accept: application/vnd.api+json; version=1`

## 9. References

- **Parent Spec**: [spec.md](spec.md)
- **OpenAPI Spec**: [swagger.yaml](swagger.yaml) (if applicable)
```

**For TECHNICAL-SPEC.md** (technical scope):
```markdown
# Technical Architecture Specification
# Feature: {FEATURE_NAME}

{FRONTMATTER}

## 1. Overview
{DESCRIPTION}

This supplementary specification defines architecture decisions, database design, infrastructure requirements, and observability patterns for the {FEATURE_NAME} feature.

**Implements**: {FR_RANGE}

## 2. Architecture Overview

### 2.1 System Components
[PLACEHOLDER: Component diagram]

### 2.2 Data Flow
[PLACEHOLDER: Data flow diagram]

### 2.3 Technology Stack
- **Frontend**: Angular 20, Kendo UI
- **Backend**: Cloudflare Workers, Hono
- **Database**: Neon Postgres
- **Cache**: [If applicable]

## 3. Database Design

### 3.1 Schema
```sql
CREATE TABLE table_name (
  id UUID PRIMARY KEY,
  -- columns
);
```

### 3.2 Indexes
[PLACEHOLDER: Index definitions]

### 3.3 Migrations
[PLACEHOLDER: Migration strategy]

## 4. Multi-Tenant Isolation

### 4.1 Isolation Strategy
- **Method**: [Row-level | Schema-level | Database-level | Shard-level]
- **Tenant Identifier**: [How tenants are identified]

### 4.2 Data Access Patterns
[PLACEHOLDER: RLS policies, tenant filtering]

## 5. Caching Strategy

### 5.1 Cache Layers
- **CDN**: [Cloudflare cache rules]
- **Application**: [Worker cache API]
- **Database**: [Postgres query cache]

### 5.2 Invalidation
[PLACEHOLDER: Cache invalidation triggers]

## 6. Security

### 6.1 Threat Model
[PLACEHOLDER: STRIDE analysis]

### 6.2 Mitigations
[PLACEHOLDER: Security controls]

### 6.3 Compliance
[PLACEHOLDER: GDPR, SOC2, etc.]

## 7. Observability

### 7.1 Logging
- **Level**: INFO, WARN, ERROR
- **Format**: JSON structured logs
- **Destination**: Cloudflare Logpush → ClickHouse

### 7.2 Metrics
[PLACEHOLDER: Key metrics to track]

### 7.3 Alerting
[PLACEHOLDER: Alert conditions and thresholds]

## 8. Performance Requirements

- **Response Time**: <200ms (p95)
- **Throughput**: 1000 req/s
- **Availability**: 99.9% uptime

## 9. Disaster Recovery

### 9.1 Backup Strategy
[PLACEHOLDER: Backup frequency and retention]

### 9.2 Recovery Procedures
[PLACEHOLDER: RTO/RPO targets]

## 10. References

- **Parent Spec**: [spec.md](spec.md)
- **ADRs**: [Architecture Decision Records](../ADR/)
```

---

### Step 7: Write Supplementary Spec File

**Action**: Create file at `specs/{FEATURE_ID}/{FILENAME}`

```bash
# Example: Create UI-SPEC.md
cat > specs/000003-billing-console/UI-SPEC.md << 'EOF'
---
parent: spec.md
scope: ui-ux-implementation
functional_requirements: [FR-001 through FR-080]
target_agents:
  - frontend-architect
version: 1.0
last_updated: 2025-11-01
---

# UI/UX Implementation Specification
# Feature: billing-console

[... full template content ...]
EOF
```

---

### Step 8: Update State

**State File**: `.specify/memory/features/{FEATURE_ID}/state.json`

```json
{
  "feature_num": "000003",
  "phase": "SPECIFYING",
  "supplementary_specs": {
    "enabled": true,
    "specs": [
      {
        "filename": "UI-SPEC.md",
        "scope": "ui-ux-implementation",
        "created_at": "2025-11-01T10:00:00Z",
        "target_agents": ["frontend-architect"],
        "size_kb": 182,
        "fr_coverage": "FR-001 through FR-080"
      }
    ]
  }
}
```

---

### Step 9: Supplementary Spec Creation Review Gate (Evidence-Based Self-Check)

**Purpose**: Validate supplementary spec creation completeness before presenting results.

### Evidence Collection (Mandatory)

❓ **"Was supplementary spec file created?"**
Action Required:
  - Verify file exists at specified path
  - Show ACTUAL file path and size
  - Report: Filename, location, file size

Expected Evidence:
  ✓ File path: specs/[FEATURE_ID]/[UI-SPEC.md|API-SPEC.md|TECHNICAL-SPEC.md]
  ✓ File exists and readable
  ✓ File size: >1KB (meaningful content with template structure)

❓ **"Does frontmatter reference parent spec?"**
Action Required:
  - Read YAML frontmatter from created file
  - Verify parent field = "spec.md"
  - Report: Frontmatter fields present

Expected Evidence:
  ✓ Frontmatter exists (YAML between --- markers)
  ✓ parent: spec.md
  ✓ scope: [ui-ux-implementation|api-contracts|technical-architecture]
  ✓ functional_requirements: [FR range or list]
  ✓ target_agents: [Array of agents]
  ✓ version: [Version number]
  ✓ last_updated: [ISO date YYYY-MM-DD]

❓ **"Were sections populated from template?"**
Action Required:
  - Count sections in created file
  - Verify template structure present
  - Report: Section count and PLACEHOLDER status

Expected Evidence:
  ✓ Section count: ≥5 major sections
  ✓ Template structure preserved (## headings)
  ✓ PLACEHOLDER sections present (to be filled by user)
  ✓ Content summary: [Brief description of what was generated]

❓ **"Was state.json updated?"**
Action Required:
  - Read state.json supplementary_specs array
  - Verify new spec added to array
  - Report: State update confirmation

Expected Evidence:
  ✓ state.json supplementary_specs.enabled = true
  ✓ state.json supplementary_specs.specs array updated
  ✓ New spec entry includes: filename, scope, created_at, target_agents

IF any evidence is MISSING:
  ❌ CANNOT report completion
  → Gather missing evidence first
  → Re-run this step with complete evidence

### Hallucination Prevention (7 Red Flags for Supplementary Spec Creation)

```yaml
Detect and BLOCK these patterns:

🚨 "Supplementary spec created" WITHOUT showing file path
   → Self-correction: "Wait, I need to verify file was written to disk"

🚨 "Frontmatter valid" WITHOUT showing parent: field value
   → Self-correction: "Let me read back frontmatter to confirm parent: spec.md"

🚨 "Content migrated" WITHOUT showing what sections were generated
   → Self-correction: "I need to show section count and template structure"

🚨 Claiming "ready for validation" WITHOUT state.json update
   → Self-correction: "Must verify state.json supplementary_specs array updated"

🚨 Creating spec WITHOUT checking parent spec exists
   → Self-correction: "Need to verify parent spec.md exists first"

🚨 "File written" WITHOUT verifying file size >0
   → Self-correction: "Must check file size to ensure content was written"

🚨 Missing scope validation (ui-ux, api-contracts, technical)
   → Self-correction: "Scope must be one of 3 valid values, let me verify"

IF detected: STOP → Gather evidence → Report honestly
```

### Determine Status

✅ **READY for Validation**:
```yaml
Criteria (ALL must be met):
  - Supplementary spec file created (size >1KB)
  - Frontmatter valid with all required fields
  - parent: spec.md field present
  - Template structure preserved with PLACEHOLDER sections
  - state.json updated with new spec entry
  - scope is valid (ui-ux-implementation, api-contracts, technical-architecture)

IF ALL criteria met:
  → Proceed with next steps (edit PLACEHOLDER sections, validate-hierarchy)
```

⚠️ **NEEDS REVIEW** (manual fix needed):
```yaml
Criteria:
  - File created but frontmatter has minor issues (date format, version)
  - Some PLACEHOLDER sections missing (user can add manually)
  - state.json update partial (manual correction needed)

IF criteria met:
  → Present issues to user
  → Recommend manual fixes before validation
```

❌ **NOT READY** (more work needed):
```yaml
Criteria (ANY triggers NOT READY):
  - File creation failed (permissions, disk space)
  - Frontmatter missing or invalid (no parent: field)
  - Invalid scope (not one of 3 valid scopes)
  - state.json update failed
  - Parent spec.md does not exist

IF NOT READY:
  → Present issues with evidence
  → Recommend: "Fix file creation issues before proceeding"
  → STOP workflow progression
```

### Output Format (Present to User - ONLY if evidence provided)

```markdown
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 Supplementary Spec Creation Review
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status: [✅ READY | ⚠️ NEEDS REVIEW | ❌ NOT READY]

**File Created**: specs/[FEATURE_ID]/[FILENAME]
**File Size**: [SIZE]KB
**Scope**: [ui-ux-implementation|api-contracts|technical-architecture]

**Frontmatter Validation**:
  ✓ parent: spec.md
  ✓ scope: [SCOPE]
  ✓ functional_requirements: [FR_RANGE]
  ✓ target_agents: [AGENT_LIST]
  ✓ version: [VERSION]
  ✓ last_updated: [DATE]

**Template Structure**:
  - Section count: [N] major sections
  - PLACEHOLDER sections: [N] (to be filled by user)
  - Template type: [UI-SPEC|API-SPEC|TECHNICAL-SPEC]

**State Updated**:
  ✓ .specify/memory/features/[FEATURE_ID]/state.json
  ✓ supplementary_specs.enabled = true
  ✓ New spec added to specs array

**Next Steps**:
  1. Edit [FILENAME] to fill in PLACEHOLDER sections
  2. Run /speckit.plan to generate plan.md (auto-discovers supplementary specs)
  3. Run /speckit.validate-hierarchy to verify integration

**PLACEHOLDER Sections to Complete**:
  - [Section 1 name]
  - [Section 2 name]
  - [...]

💡 TIP: Use PLACEHOLDER sections as guide for what to document

Next Action: [Edit PLACEHOLDER sections OR Validate hierarchy]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 10: Output Confirmation

```bash
✓ Created UI-SPEC.md at specs/000003-billing-console/UI-SPEC.md

📋 Supplementary Spec Details:
  Scope:         ui-ux-implementation
  Target Agents: frontend-architect
  FR Coverage:   FR-001 through FR-080
  Size:          182 KB (estimated after completion)

📝 Next Steps:
  1. Edit UI-SPEC.md to add implementation details
  2. Run /speckit.plan to generate plan.md (will auto-discover UI-SPEC.md)
  3. Run /speckit.validate-hierarchy to verify integration

💡 TIP: Use PLACEHOLDER sections as guide for what to document
```

---

## Supplementary Spec Types

### 1. UI-SPEC.md (ui-ux scope)

**Purpose**: Component designs, layouts, accessibility

**Target Agents**:
- `frontend-architect`

**Typical Contents**:
- Component hierarchy
- State management (NgRx, Signals)
- Routing and navigation
- Responsive layouts (desktop, tablet, mobile)
- UI component specifications
- Form validation rules
- Loading states and error handling
- Accessibility (WCAG 2.1 AA)
- Performance targets
- Browser compatibility

**When to Use**:
- Angular/React/Vue applications
- Complex UI with >10 components
- Responsive design requirements
- Accessibility compliance needed

---

### 2. API-SPEC.md (api-contracts scope)

**Purpose**: REST API contracts, schemas, webhooks

**Target Agents**:
- `backend-architect`
- `system-architect`

**Typical Contents**:
- API endpoints (method, path, params, response)
- Request/response schemas (JSON, TypeScript interfaces)
- Error codes and handling
- Authentication & authorization
- Rate limiting
- Webhooks
- API versioning
- OpenAPI/Swagger integration

**When to Use**:
- RESTful APIs with >10 endpoints
- Integration with external services
- Webhook handling
- Complex request/response schemas

---

### 3. TECHNICAL-SPEC.md (technical scope)

**Purpose**: Architecture, database, infrastructure

**Target Agents**:
- `system-architect`
- `postgres-entity-architect`
- `devops-architect`

**Typical Contents**:
- System architecture diagrams
- Database schema and indexes
- Multi-tenant isolation strategy
- Caching layers
- Security threat model
- Observability (logging, metrics, alerts)
- Performance requirements
- Disaster recovery

**When to Use**:
- Complex architectures (microservices, multi-tenant)
- Database design with >10 tables
- Performance-critical systems
- Compliance requirements (GDPR, SOC2)

---

## Integration with Workflow

### Before Supplement
1. `/speckit.specify` → Create parent spec.md
2. Recognize spec is large or multi-domain

### During Supplement
1. `/speckit.supplement ui-ux "..."` → Create UI-SPEC.md
2. `/speckit.supplement api-contracts "..."` → Create API-SPEC.md
3. `/speckit.supplement technical "..."` → Create TECHNICAL-SPEC.md
4. Edit supplementary specs to fill in PLACEHOLDER sections

### After Supplement
1. `/speckit.plan` → Auto-discovers supplementary specs, references in plan.md
2. `/speckit.validate-hierarchy` → Strict validation (errors block workflow)
3. `/speckit.tasks` → Tasks reference supplementary spec sections
4. `/speckit.implement` → Agents auto-load relevant supplementary specs

---

## Best Practices

### 1. Proactive Recommendations with Manual Control (v2.1.1)

The `/speckit.specify` command automatically **recommends** hierarchical specs when specs exceed thresholds (>100KB or >60 requirements), but supplement creation remains **manual only**.

**How It Works**:
- **Proactive Detection**: `/speckit.specify` detects large specs during creation and displays recommendation
- **Informational Only**: Recommendation shows examples and benefits, but creates no files automatically
- **User Control**: Developer decides whether to create supplementary specs (now, later, or never)
- **No Workflow Blocking**: Declining the recommendation has no penalty - workflow continues normally

**Rationale for Manual Control**:
- Developer control (explicit decision to split specs)
- No surprises (no unexpected file creation)
- Learning curve (teams decide when complexity justifies splitting)
- Flexibility (can defer until later if needed)

### 2. One Supplementary Spec Per Domain
Stick to the naming convention:
- `UI-SPEC.md` (one file for all UI concerns)
- `API-SPEC.md` (one file for all API concerns)
- `TECHNICAL-SPEC.md` (one file for all technical concerns)

**Don't**:
```
UI-SPEC-dashboard.md
UI-SPEC-settings.md
UI-SPEC-billing.md  ❌ Too granular
```

**Do**:
```markdown
# UI-SPEC.md

## 5. Pages

### 5.1 Dashboard Page
[Dashboard UI details]

### 5.2 Settings Page
[Settings UI details]

### 5.3 Billing Page
[Billing UI details]
```

### 3. Maintain FR Traceability
Every section in supplementary spec should map to parent FRs.

**Example**:
```markdown
### 4.2 Company Management Component
**Implements**: FR-008, FR-015, FR-019
**Description**: Allows users to manage company profiles
[... component details ...]
```

### 4. Reference Parent Spec
Include links back to parent spec.md for context.

```markdown
This component implements the "Company Profile Management" user story from [spec.md Section 3.2](spec.md#32-company-profile-management).
```

### 5. Use Frontmatter Strictly
Always include frontmatter with required fields:
- `parent`: Must be "spec.md"
- `scope`: One of predefined scopes
- `functional_requirements`: FR range or list
- `target_agents`: Array of agent identifiers

**Validation** (strict mode) will block workflow if frontmatter is missing or invalid.

---

## Error Handling

### Scenario 1: Parent Spec Not Found
```bash
❌ ERROR: Parent spec.md not found at specs/{FEATURE_ID}/spec.md

Run /speckit.specify first to create parent specification.
```

### Scenario 2: Supplementary Spec Already Exists
```bash
❌ ERROR: UI-SPEC.md already exists at specs/000003-billing-console/UI-SPEC.md

Options:
1. Delete existing UI-SPEC.md if you want to recreate
2. Use /speckit.amend-technical to modify existing spec
3. Edit UI-SPEC.md manually
```

### Scenario 3: Invalid Scope
```bash
❌ ERROR: Invalid scope "frontend" provided

Valid scopes:
- ui-ux (creates UI-SPEC.md)
- api-contracts (creates API-SPEC.md)
- technical (creates TECHNICAL-SPEC.md)

Usage: /speckit.supplement <scope> "<description>"
```

### Scenario 4: Empty Description
```bash
⚠️ WARNING: Empty description provided

Supplementary spec created with generic description.
Edit the spec to provide detailed implementation guidance.
```

---

## Exit Criteria

Supplement command completes successfully when:
1. ✅ Parent spec.md exists
2. ✅ Supplementary spec file created with frontmatter
3. ✅ Template populated with feature metadata
4. ✅ State updated (supplementary_specs.enabled = true)
5. ✅ Confirmation message displayed with next steps

---

## Related Commands

- `/speckit.specify` - Create parent specification (run before supplement)
- `/speckit.plan` - Auto-discovers supplementary specs
- `/speckit.validate-hierarchy` - Strict validation of supplementary specs
- `/speckit.reconcile` - Can update supplementary specs post-implementation
- `/speckit.amend-technical` - Modify existing supplementary specs

---

## Example Workflow

```bash
# 1. Create parent spec (traditional specify)
/speckit.specify "Build billing console for multi-tenant SaaS"
# Output: specs/000003-billing-console/spec.md (97KB, 80 FRs)

# 2. Recognize spec is large, create supplementary specs
/speckit.supplement ui-ux "Complete UI/UX specifications with Angular 20 + Kendo UI"
# Output: specs/000003-billing-console/UI-SPEC.md

/speckit.supplement api-contracts "REST API contracts for billing and payments"
# Output: specs/000003-billing-console/API-SPEC.md

/speckit.supplement technical "Multi-tenant architecture with Neon Postgres sharding"
# Output: specs/000003-billing-console/TECHNICAL-SPEC.md

# 3. Edit supplementary specs to fill in PLACEHOLDER sections
# (Manual editing of UI-SPEC.md, API-SPEC.md, TECHNICAL-SPEC.md)

# 4. Generate plan (auto-discovers supplementary specs)
/speckit.plan
# Output: plan.md with "Supplementary Specifications" section

# 5. Validate hierarchy (strict mode)
/speckit.validate-hierarchy
# Output: All checks pass, workflow continues

# 6. Generate tasks
/speckit.tasks
# Output: tasks.md with references to UI-SPEC.md sections

# 7. Implement
/speckit.implement
# Agents auto-load relevant supplementary specs (frontend-architect → UI-SPEC.md)
```

**Token Savings**:
- Before: 279KB loaded by all agents (5 sessions × 279KB = ~350K tokens)
- After: 97KB parent + 50KB domain-specific (5 sessions × 147KB = ~184K tokens)
- **Savings**: 166K tokens (47% reduction)

---

**Command Version**: 2.1.0
**Last Updated**: 2025-11-01
**Compatibility**: SpecKit v2.1+
