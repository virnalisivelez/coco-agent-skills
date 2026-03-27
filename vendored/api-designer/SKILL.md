---
name: api-designer
description: Design clean, consistent REST and GraphQL APIs. Use when building API endpoints, defining schemas, creating OpenAPI specs, or reviewing API architecture. Follows REST best practices and consistent naming conventions.
license: Apache-2.0
metadata:
  original-author: good-stories-llc
  vendored-by: good-stories-llc
  version: "1.0"
---

# API Designer

Design clean, consistent, and well-documented REST APIs. Covers resource modeling, HTTP method semantics, status codes, error handling, pagination, authentication, versioning, and OpenAPI specification.

## When to Use

- User wants to design a new API or set of endpoints
- User needs to review an existing API for consistency
- User asks to create an OpenAPI/Swagger specification
- User wants best practices for error handling, pagination, or auth

## REST Resource Design

### URL Structure

URLs represent resources (nouns), not actions (verbs):

```
GET    /users              List users
POST   /users              Create a user
GET    /users/{id}         Get a specific user
PUT    /users/{id}         Replace a user
PATCH  /users/{id}         Partially update a user
DELETE /users/{id}         Delete a user
```

Rules:
- Plural nouns for collections: `/users`, `/orders`, `/products`
- Lowercase with hyphens: `/order-items`, not `/orderItems` or `/order_items`
- Nest for clear ownership: `/users/{id}/orders` (orders belonging to a user)
- Maximum nesting depth: 2 levels. Beyond that, promote to a top-level resource with a filter
- No verbs in URLs: `/users/{id}/activate` is acceptable only for non-CRUD actions (RPC-style)

### HTTP Methods

| Method | Idempotent | Safe | Use For |
|---|---|---|---|
| GET | Yes | Yes | Read data. Never modify state. |
| POST | No | No | Create a resource or trigger an action. |
| PUT | Yes | No | Replace a resource entirely. |
| PATCH | No* | No | Partially update a resource. |
| DELETE | Yes | No | Remove a resource. |

*PATCH is idempotent if the same patch produces the same result, but the spec does not require it.

### Status Codes

Use the right code for every response:

**Success (2xx)**
| Code | When |
|---|---|
| 200 OK | Successful GET, PUT, PATCH, or DELETE |
| 201 Created | Successful POST that creates a resource (include `Location` header) |
| 204 No Content | Successful DELETE or action with no response body |

**Client Error (4xx)**
| Code | When |
|---|---|
| 400 Bad Request | Malformed request body, missing required fields |
| 401 Unauthorized | Missing or invalid authentication |
| 403 Forbidden | Authenticated but lacks permission |
| 404 Not Found | Resource does not exist |
| 409 Conflict | Duplicate resource, version conflict |
| 422 Unprocessable Entity | Valid JSON but fails business validation |
| 429 Too Many Requests | Rate limit exceeded (include `Retry-After` header) |

**Server Error (5xx)**
| Code | When |
|---|---|
| 500 Internal Server Error | Unexpected server failure |
| 502 Bad Gateway | Upstream service failure |
| 503 Service Unavailable | Planned maintenance or overload |

### Error Response Format

Use a consistent error envelope across all endpoints:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request body contains invalid fields.",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address.",
        "value": "not-an-email"
      }
    ]
  }
}
```

Rules:
- `code`: Machine-readable error identifier (UPPER_SNAKE_CASE)
- `message`: Human-readable description
- `details`: Optional array with field-level errors
- Never expose stack traces, SQL errors, or internal paths in production

## Pagination

### Cursor-Based (recommended for most APIs)

```
GET /users?limit=20&cursor=eyJpZCI6MTIzfQ
```

Response:
```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTQzfQ",
    "has_more": true
  }
}
```

Advantages: Stable under inserts/deletes, efficient on large datasets.

### Offset-Based (acceptable for small, stable datasets)

```
GET /users?limit=20&offset=40
```

Response:
```json
{
  "data": [...],
  "pagination": {
    "total": 250,
    "limit": 20,
    "offset": 40
  }
}
```

Disadvantage: Skips or duplicates items if data changes between page requests.

## Filtering, Sorting, and Search

### Filtering

Use query parameters matching field names:

```
GET /orders?status=shipped&customer_id=42
GET /orders?created_after=2025-01-01&created_before=2025-12-31
```

For complex filters, consider a filter parameter with a structured syntax:
```
GET /orders?filter=status:shipped,total>100
```

### Sorting

```
GET /users?sort=created_at:desc
GET /users?sort=last_name:asc,first_name:asc
```

### Search

Full-text search via a `q` parameter:
```
GET /products?q=wireless+keyboard
```

## Authentication Patterns

### API Key (simple, service-to-service)

```
X-API-Key: sk_live_abc123def456
```

- Send in header, never in URL query string
- Prefix keys by environment: `sk_live_`, `sk_test_`
- Store hashed in database, not plaintext

### Bearer Token (JWT or opaque)

```
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

- Access tokens: short-lived (15-60 minutes)
- Refresh tokens: long-lived, stored securely, rotated on use
- JWTs: always verify signature and expiration; do not trust claims without verification

### OAuth 2.0 (third-party access)

Use the Authorization Code flow with PKCE for public clients (SPAs, mobile apps). Never use the Implicit flow.

## Versioning

### URL Path Versioning (most common)

```
/v1/users
/v2/users
```

- Only increment major version for breaking changes
- Support the previous version for a deprecation period (minimum 6 months)
- Return `Sunset` header on deprecated versions

### Header Versioning (alternative)

```
Accept: application/vnd.myapi.v2+json
```

## Request/Response Design

### Request Body

```json
{
  "name": "Alice Smith",
  "email": "alice@example.com",
  "role": "admin"
}
```

- Use `snake_case` for field names (consistent across Python, Ruby, Go, databases)
- Include only writable fields (no `id`, `created_at` in POST/PUT bodies)
- Validate all input server-side, regardless of client-side validation

### Response Body

Wrap responses in a consistent envelope:

```json
{
  "data": {
    "id": "usr_abc123",
    "name": "Alice Smith",
    "email": "alice@example.com",
    "role": "admin",
    "created_at": "2025-03-15T10:30:00Z"
  }
}
```

For collections:
```json
{
  "data": [...],
  "pagination": { ... }
}
```

### Dates and Times

- Always use ISO 8601: `2025-03-15T10:30:00Z`
- Always include timezone (prefer UTC with `Z` suffix)
- Store as `TIMESTAMPTZ` in the database

### IDs

- Prefer prefixed string IDs for public APIs: `usr_abc123`, `ord_def456`
- Benefits: type-safe (you can tell a user ID from an order ID), not guessable
- Implementation: UUID with a prefix, or a short-ID library

## OpenAPI Specification Structure

```yaml
openapi: 3.1.0
info:
  title: My API
  version: 1.0.0
  description: Brief description of the API.

servers:
  - url: https://api.example.com/v1
    description: Production

paths:
  /users:
    get:
      summary: List users
      operationId: listUsers
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'

components:
  schemas:
    User:
      type: object
      required: [id, name, email]
      properties:
        id:
          type: string
          example: usr_abc123
        name:
          type: string
          example: Alice Smith
        email:
          type: string
          format: email
  securitySchemes:
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
```

## Rate Limiting

Include rate limit headers in every response:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 997
X-RateLimit-Reset: 1679616000
Retry-After: 30
```

Common rate limit tiers:
- Free tier: 100 requests/hour
- Standard: 1000 requests/hour
- Pro: 10000 requests/hour

Return `429 Too Many Requests` when exceeded with a `Retry-After` header.

## Output Expectations

When designing an API, deliver:

1. Resource model with URL patterns
2. Endpoint list with methods, parameters, and response shapes
3. Error response format
4. Authentication approach
5. Pagination strategy
6. OpenAPI spec (YAML) if the user needs documentation
