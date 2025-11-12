---
# Context Optimization Metadata (v2.7+)
# These settings control when and how this memory file is loaded

inclusion_mode: conditional              # always | conditional | manual
context_level: tactical                  # strategic | tactical | reference
load_phases:                             # Phases where this content is loaded
  - implementation                       # Load during implementation (API coding)
  - reconciliation                       # Load during reconciliation (gap closure)
exclude_phases:                          # Phases where this content is skipped
  - specification                        # Skip during spec writing (too detailed)
  - gap_analysis                         # Skip during gap analysis
  - planning                             # Skip during planning (load strategic only)

# Pattern-based conditional loading
include_when:
  - path_pattern: "src/api/**/*"
  - path_pattern: "**/controllers/**/*"
  - path_pattern: "**/routes/**/*"
  - path_pattern: "**/*-api.js"
  - path_pattern: "**/*-api.ts"
  - path_pattern: "**/handlers/**/*"
  - path_pattern: "**/endpoints/**/*"

# Version and update tracking
memory_version: "1.0.0"
last_updated: "[YYYY-MM-DD]"
created_by: "speckit.memory"
---

# API Standards & Guidelines

**Purpose**: API design patterns, RESTful conventions, and endpoint standards
**Context**: This file is conditionally loaded when working in API-related directories
**Loaded During**: Implementation and reconciliation phases only

---

<!-- SECTION_META: context_level=strategic, load_phases=planning,implementation,reconciliation -->
## REST API Design Principles
<!-- Strategic section: High-level API design principles -->

### Resource Naming
<!-- URL path conventions -->
- **Collections**: Plural nouns (`/users`, `/products`, `/orders`)
- **Specific Resources**: ID-based (`/users/123`, `/products/456`)
- **Nested Resources**: Logical hierarchy (`/users/123/orders`, `/orders/456/items`)
- **Actions**: Verbs only when RESTful methods don't fit (`/orders/123/cancel`, `/users/123/reset-password`)

### HTTP Methods
<!-- Standard CRUD operations -->
- `GET` - Retrieve resource(s), idempotent, no side effects
- `POST` - Create new resource, non-idempotent
- `PUT` - Replace entire resource, idempotent
- `PATCH` - Partial update, idempotent
- `DELETE` - Remove resource, idempotent

### Status Codes
<!-- Standard HTTP response codes -->
- `200 OK` - Successful GET, PUT, PATCH, DELETE
- `201 Created` - Successful POST with resource creation
- `204 No Content` - Successful DELETE or update with no body
- `400 Bad Request` - Client error (validation, malformed request)
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Authenticated but insufficient permissions
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - Resource state conflict
- `422 Unprocessable Entity` - Validation errors
- `500 Internal Server Error` - Server-side failure
- `503 Service Unavailable` - Temporary downtime

<!-- SECTION_META: context_level=tactical, load_phases=implementation,reconciliation -->
## Request/Response Format
<!-- Tactical section: Code examples and templates for API implementation -->

### Request Structure
```json
{
  "data": { /* Payload */ },
  "metadata": {
    "request_id": "uuid",
    "timestamp": "ISO 8601",
    "client_version": "1.2.3"
  }
}
```

### Response Structure (Success)
```json
{
  "data": { /* Resource or collection */ },
  "metadata": {
    "request_id": "uuid",
    "timestamp": "ISO 8601",
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 150,
      "total_pages": 8
    }
  }
}
```

### Response Structure (Error)
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "User-friendly error message",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format",
        "code": "INVALID_FORMAT"
      }
    ],
    "request_id": "uuid",
    "timestamp": "ISO 8601"
  }
}
```

## Pagination

### Offset-Based (Default)
```
GET /users?page=1&per_page=20
```
**Response includes**: `page`, `per_page`, `total`, `total_pages`

### Cursor-Based (Large Datasets)
```
GET /users?cursor=abc123&limit=20
```
**Response includes**: `next_cursor`, `previous_cursor`, `has_more`

## Filtering & Sorting

### Query Parameters
- **Filtering**: `?status=active&role=admin`
- **Sorting**: `?sort_by=created_at&order=desc`
- **Search**: `?q=search+term`
- **Field Selection**: `?fields=id,name,email`

### Complex Filters (if needed)
```
?filter[created_at][gte]=2025-01-01&filter[status][in]=active,pending
```

## Versioning

### Strategy
<!-- Choose one: URL path, header, query parameter -->
- **URL Path** (Recommended): `/api/v1/users`, `/api/v2/users`
- **Header**: `Accept: application/vnd.api+json; version=1`
- **Query**: `/api/users?version=1`

### Deprecation Policy
1. Announce deprecation 6 months before removal
2. Return `Deprecation` header: `Deprecation: Sun, 01 Jun 2025 00:00:00 GMT`
3. Include `Link` header to migration docs
4. Maintain backwards compatibility during transition

## Authentication & Authorization

### Authentication Method
<!-- Choose based on project requirements -->
- **JWT Bearer Token**: `Authorization: Bearer <token>`
- **API Key**: `X-API-Key: <key>`
- **OAuth 2.0**: Standard OAuth flow
- **Session Cookie**: Stateful authentication

### Authorization
- Use **RBAC** (Role-Based Access Control) or **ABAC** (Attribute-Based)
- Return `403 Forbidden` for insufficient permissions (not `404` to avoid info leakage)
- Document required permissions per endpoint

## Rate Limiting

### Headers
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1609459200
```

### Response (429 Too Many Requests)
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Retry after 60 seconds.",
    "retry_after": 60
  }
}
```

## Validation

### Input Validation Rules
- Validate **all** inputs (path params, query params, body, headers)
- Use schema validation libraries (JSON Schema, Joi, Yup, Zod)
- Return `422 Unprocessable Entity` with field-level errors
- Sanitize inputs to prevent XSS, SQL injection

### Example Validation Response
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address",
        "value": "invalid-email"
      },
      {
        "field": "age",
        "message": "Must be between 18 and 120",
        "value": 200
      }
    ]
  }
}
```

## Error Handling

### Error Codes
<!-- Project-specific error codes -->
- `VALIDATION_ERROR` - Input validation failed
- `AUTHENTICATION_REQUIRED` - Missing or invalid auth
- `PERMISSION_DENIED` - Insufficient permissions
- `RESOURCE_NOT_FOUND` - Resource doesn't exist
- `RESOURCE_CONFLICT` - State conflict (e.g., duplicate)
- `RATE_LIMIT_EXCEEDED` - Too many requests
- `INTERNAL_ERROR` - Unexpected server error
- `SERVICE_UNAVAILABLE` - Temporary downtime

### Error Logging
- Log all 5xx errors with full stack trace
- Log 4xx errors with request context (no sensitive data)
- Include `request_id` for traceability

## Performance

### Caching
- Use `Cache-Control`, `ETag`, `Last-Modified` headers
- Cache GET requests when appropriate
- Invalidate cache on mutations (POST, PUT, PATCH, DELETE)

### Compression
- Enable Gzip/Brotli compression for responses
- Check `Accept-Encoding` header

### Response Time SLAs
<!-- Define based on project requirements -->
- **Simple GET**: < 200ms (p95)
- **Complex Query**: < 500ms (p95)
- **Mutations**: < 1000ms (p95)

## Documentation

### OpenAPI/Swagger
- Maintain OpenAPI 3.0 specification
- Auto-generate from code annotations where possible
- Include examples for all endpoints

### Required Documentation per Endpoint
- Description and purpose
- Authentication/authorization requirements
- Request schema with examples
- Response schema with examples (success + errors)
- Rate limits
- Deprecation status

<!-- SECTION_META: context_level=tactical, load_phases=implementation,reconciliation -->
## Testing
<!-- Tactical section: Testing patterns and examples for API implementation -->

### API Test Coverage
- **Contract Tests**: Verify request/response schemas
- **Integration Tests**: Test full request lifecycle
- **Error Scenarios**: Test all error codes (400, 401, 403, 404, 422, 500)
- **Edge Cases**: Empty lists, pagination boundaries, rate limits

### Test Fixtures
- Use consistent test data
- Include valid and invalid request examples
- Test boundary conditions

---

## Usage Instructions

This file is **conditionally loaded** when working in API-related directories or files. It provides AI assistants with project-specific API conventions to ensure consistency across all endpoints.

**Last Updated**: [YYYY-MM-DD]
**Next Review**: [YYYY-MM-DD]
