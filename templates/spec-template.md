# Feature Specification: [FEATURE NAME]

**Feature Branch**: `[###-feature-name]`  
**Created**: [DATE]  
**Status**: Draft  
**Input**: User description: "$ARGUMENTS"

## User Scenarios & Testing *(mandatory)*

<!--
  IMPORTANT: User stories should be PRIORITIZED as user journeys ordered by importance.
  Each user story/journey must be INDEPENDENTLY TESTABLE - meaning if you implement just ONE of them,
  you should still have a viable MVP (Minimum Viable Product) that delivers value.
  
  Assign priorities (P1, P2, P3, etc.) to each story, where P1 is the most critical.
  Think of each story as a standalone slice of functionality that can be:
  - Developed independently
  - Tested independently
  - Deployed independently
  - Demonstrated to users independently
-->

### User Story 1 - [Brief Title] (Priority: P1)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently - e.g., "Can be fully tested by [specific action] and delivers [specific value]"]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]
2. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### User Story 2 - [Brief Title] (Priority: P2)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

### User Story 3 - [Brief Title] (Priority: P3)

[Describe this user journey in plain language]

**Why this priority**: [Explain the value and why it has this priority level]

**Independent Test**: [Describe how this can be tested independently]

**Acceptance Scenarios**:

1. **Given** [initial state], **When** [action], **Then** [expected outcome]

---

[Add more user stories as needed, each with an assigned priority]

### Edge Cases

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right edge cases.
-->

- What happens when [boundary condition]?
- How does system handle [error scenario]?

## Requirements *(mandatory)*

<!--
  ACTION REQUIRED: The content in this section represents placeholders.
  Fill them out with the right functional requirements.
-->

### Functional Requirements

- **FR-001**: System MUST [specific capability, e.g., "allow users to create accounts"]
- **FR-002**: System MUST [specific capability, e.g., "validate email addresses"]  
- **FR-003**: Users MUST be able to [key interaction, e.g., "reset their password"]
- **FR-004**: System MUST [data requirement, e.g., "persist user preferences"]
- **FR-005**: System MUST [behavior, e.g., "log all security events"]

*Example of marking unclear requirements:*

- **FR-006**: System MUST authenticate users via [NEEDS CLARIFICATION: auth method not specified - email/password, SSO, OAuth?]
- **FR-007**: System MUST retain user data for [NEEDS CLARIFICATION: retention period not specified]

### Key Entities *(include if feature involves data)*

- **[Entity 1]**: [What it represents, key attributes without implementation]
- **[Entity 2]**: [What it represents, relationships to other entities]

## Success Criteria *(mandatory)*

<!--
  ACTION REQUIRED: Define measurable success criteria.
  These must be technology-agnostic and measurable.
-->

### Measurable Outcomes

- **SC-001**: [Measurable metric, e.g., "Users can complete account creation in under 2 minutes"]
- **SC-002**: [Measurable metric, e.g., "System handles 1000 concurrent users without degradation"]
- **SC-003**: [User satisfaction metric, e.g., "90% of users successfully complete primary task on first attempt"]
- **SC-004**: [Business metric, e.g., "Reduce support tickets related to [X] by 50%"]

---

## Complex Features (optional - skip for CRUD features)

<!--
  ACTION REQUIRED: Only complete this section if this is a COMPLEX feature.

  WHEN TO USE THIS SECTION:
  - Multi-tenant systems (company isolation, data partitioning)
  - Performance-critical features (sub-100ms latency, high throughput)
  - Integration-heavy features (3+ external systems)
  - Security-sensitive features (PCI, HIPAA, SOC2 compliance)
  - Large-scale features (10,000+ entities, distributed systems)

  WHEN TO SKIP THIS SECTION:
  - Standard CRUD operations (create/read/update/delete)
  - Simple UI features with no backend complexity
  - Features with <5 functional requirements
  - Prototypes or proof-of-concepts

  If you complete this section, you will create a TECHNICAL.md file during the /plan phase.
-->

### Complexity Indicators

Check all that apply:

- [ ] **Multi-tenancy**: Requires data isolation, tenant-scoped queries, or schema-per-tenant
- [ ] **Performance-Critical**: Has latency requirements (e.g., p99 <100ms) or throughput requirements (e.g., 10K req/sec)
- [ ] **Integration-Heavy**: Integrates with 3+ external systems or APIs
- [ ] **Security-Sensitive**: Handles PII, payment data, or requires compliance (PCI, HIPAA, SOC2)
- [ ] **Large-Scale**: Manages 10,000+ entities, distributed data, or requires sharding
- [ ] **Real-Time**: Requires WebSockets, Server-Sent Events, or push notifications
- [ ] **Complex Business Rules**: Has 10+ business rules with interdependencies
- [ ] **Architectural Uncertainty**: Multiple viable approaches exist, choice impacts future features

### Technical Constraints (Preliminary)

If ANY complexity indicators checked, note any known constraints:

**Performance Constraints** (if applicable):
- API Latency: [e.g., "p99 < 100ms for all queries"]
- Throughput: [e.g., "Handle 5,000 concurrent users"]
- Data Volume: [e.g., "Support 100,000+ zones across 10,000 companies"]

**Security Constraints** (if applicable):
- Data Isolation: [e.g., "Zero cross-company data leakage"]
- Compliance: [e.g., "SOC2 Type II audit requirements"]
- Authentication: [e.g., "Support SSO via SAML 2.0"]

**Integration Constraints** (if applicable):
- External APIs: [e.g., "Stripe for payments, SendGrid for email, Auth0 for SSO"]
- Rate Limits: [e.g., "Stripe: 100 req/sec, SendGrid: 10K emails/day"]
- SLA Requirements: [e.g., "99.9% uptime"]

**Scalability Constraints** (if applicable):
- Tenant Count: [e.g., "Support up to 10,000 companies"]
- Data Growth: [e.g., "100GB/year data growth"]
- Geographic Distribution: [e.g., "Multi-region deployment (US, EU, APAC)"]

### Risk Flags

Use `[NEEDS VALIDATION]` for areas requiring investigation during planning:

- [NEEDS VALIDATION] **Multi-tenant scaling**: Unknown if current ORM supports 10,000+ tenant schemas
  - **Investigation**: Load test with 1,000 tenants during /plan phase
  - **Deadline**: Before implementation starts
  - **Fallback**: Evaluate alternative database strategies if ORM doesn't scale

- [NEEDS VALIDATION] **[Risk description]**: [What is unknown or uncertain]
  - **Investigation**: [How to validate]
  - **Deadline**: [When decision needed]
  - **Fallback**: [Backup plan if primary approach fails]

### TECHNICAL.md Creation

**If you completed this section**, during the `/plan` phase:

1. The planning agent will create `specs/###-feature-name/TECHNICAL.md`
2. TECHNICAL.md will capture:
   - Technical constraints (performance, security, scalability)
   - Risk flags with validation plans
   - Architectural decisions and rationale
   - Decision history (as pivots occur)
3. Business rules will reference TECHNICAL.md for domain/tenant context
4. If pivots occur (ADRs created), TECHNICAL.md will be amended with ~~old~~ → new

**If you skipped this section** (CRUD feature):
- No TECHNICAL.md will be created
- Standard CRUD patterns will be used
- Simpler business rules structure
- Pivots less likely (lower complexity)
