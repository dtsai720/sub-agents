# RESTful API Design Principles

## HTTP Methods

### GET - Retrieve Resource
- **Purpose**: Read resource (idempotent, safe, cacheable)
- **Example**: `GET /users`, `GET /users/{id}`
- **Success**: 200 OK
- **Rules**:
  - MUST be idempotent (same result every time)
  - MUST be safe (no side effects)
  - SHOULD be cacheable

### POST - Create Resource / Non-idempotent Action
- **Purpose**: Create new resource or trigger non-idempotent action
- **Example**: `POST /users`, `POST /orders/{id}/submit`
- **Success**: 201 Created (with Location header)
- **Rules**:
  - NOT idempotent
  - MUST return 201 Created for resource creation
  - MUST include Location header pointing to new resource

### PUT - Full Update
- **Purpose**: Complete replacement of resource (idempotent)
- **Example**: `PUT /users/{id}`
- **Success**: 200 OK or 204 No Content
- **Rules**:
  - MUST be idempotent
  - All fields required (complete replacement)
  - Can create resource if it doesn't exist (optional)

### PATCH - Partial Update
- **Purpose**: Partial modification of resource
- **Example**: `PATCH /users/{id}`
- **Success**: 200 OK or 204 No Content
- **Rules**:
  - Generally NOT idempotent (can be designed as idempotent)
  - Only modified fields in request

### DELETE - Remove Resource
- **Purpose**: Delete resource (idempotent)
- **Example**: `DELETE /users/{id}`
- **Success**: 204 No Content or 200 OK
- **Rules**:
  - MUST be idempotent
  - Deleting non-existent resource can return 404 or 204

### HEAD - Retrieve Headers
- **Purpose**: Same as GET but without response body
- **Example**: `HEAD /users/{id}`
- **Success**: 200 OK (no body)

### OPTIONS - Retrieve Allowed Methods
- **Purpose**: Get allowed HTTP methods (for CORS)
- **Example**: `OPTIONS /users`
- **Success**: 200 OK with Allow header

## Resource Naming

### ✅ Correct Patterns

```
# CRUD operations - use plural nouns
GET    /users              # List users
POST   /users              # Create user
GET    /users/{id}         # Get single user
PUT    /users/{id}         # Full update user
PATCH  /users/{id}         # Partial update user
DELETE /users/{id}         # Delete user

# Nested resources - express ownership
GET    /users/{id}/orders           # Get user's orders
POST   /users/{id}/orders           # Create order for user
GET    /orders/{id}/items           # Get order items
POST   /orders/{id}/items           # Add item to order

# Business actions - use verbs for non-CRUD operations
POST   /orders/{id}/submit          # Submit order
POST   /orders/{id}/cancel          # Cancel order
POST   /payments/{id}/refund        # Refund payment

# Use kebab-case for multi-word resources
GET    /order-items
GET    /shipping-addresses
```

### ❌ Wrong Patterns

```
# Don't use verbs for CRUD
GET    /getUsers           # Wrong - use GET /users
POST   /createUser         # Wrong - use POST /users
POST   /deleteUser         # Wrong - use DELETE /users/{id}

# Don't use singular for collections
GET    /user               # Wrong - use /users
GET    /order              # Wrong - use /orders

# Don't mix naming conventions
GET    /OrderItems         # Wrong - use /order-items
GET    /order_items        # Wrong - use /order-items (kebab-case)

# Don't use query params for resource ID
GET    /users?id=123       # Wrong - use /users/123
DELETE /users?id=123       # Wrong - use DELETE /users/123

# Don't use GET for operations with side effects
GET    /users/delete?id=123    # Wrong - use DELETE /users/123
GET    /orders/submit?id=123   # Wrong - use POST /orders/123/submit
```

## Idempotency

### Idempotent Methods (Must produce same result)
- **GET**: Always returns same resource
- **PUT**: Repeated updates produce same state
- **DELETE**: Deleting same resource multiple times has same effect
- **HEAD**: Same as GET
- **OPTIONS**: Same allowed methods

### Non-Idempotent Methods
- **POST**: Creating resource multiple times creates duplicates
- **PATCH**: Depends on implementation (can be designed as idempotent)

### Idempotency Keys (for POST)
For critical POST operations (payments, orders), use idempotency keys:

```
POST /payments
Headers:
  Idempotency-Key: unique-key-123

# Server stores key and result
# Duplicate requests with same key return same result
```

## HATEOAS (Optional - Level 3 REST)

Hypermedia as the Engine of Application State - include links to related resources:

```json
{
  "id": "123",
  "name": "張三",
  "email": "zhang@example.com",
  "_links": {
    "self": { "href": "/users/123" },
    "orders": { "href": "/users/123/orders" },
    "avatar": { "href": "/users/123/avatar" }
  }
}
```

## Richardson Maturity Model

- **Level 0**: Single URI, single HTTP method (RPC style)
- **Level 1**: Multiple URIs (resources), single HTTP method
- **Level 2**: Multiple URIs + HTTP methods (most REST APIs)
- **Level 3**: Level 2 + HATEOAS (hypermedia controls)

Most APIs aim for **Level 2** (resources + HTTP methods + status codes).

## Content Negotiation

Use `Accept` and `Content-Type` headers:

```
Request:
  Accept: application/json
  Content-Type: application/json

Response:
  Content-Type: application/json

# Support multiple formats
Accept: application/json, application/xml
```

## Versioning Strategies

### 1. URI Versioning (Recommended)
```
/v1/users
/v2/users
```
**Pros**: Simple, clear, cacheable
**Cons**: URL changes

### 2. Header Versioning
```
GET /users
Accept: application/vnd.api.v1+json
```
**Pros**: Clean URLs
**Cons**: Harder to test, not browser-friendly

### 3. Query Parameter
```
/users?version=1
```
**Pros**: Simple
**Cons**: Not RESTful, affects caching

## Best Practices Summary

✅ **DO**:
- Use plural nouns for resources (`/users`, `/orders`)
- Use HTTP methods correctly (GET=read, POST=create, PUT=full update, PATCH=partial update, DELETE=delete)
- Use correct status codes (201 Created, 204 No Content, 400/401/403/404/409/422/500)
- Return Location header for 201 Created
- Use kebab-case for multi-word resources (`/order-items`)
- Express ownership with nested resources (`/users/{id}/orders`)
- Make GET, PUT, DELETE idempotent
- Use consistent naming across all endpoints

❌ **DON'T**:
- Use verbs for CRUD operations (`/getUsers`, `/createOrder`)
- Use GET for operations with side effects
- Use POST for read operations
- Mix singular and plural (`/user` and `/users`)
- Mix naming conventions (camelCase, snake_case, kebab-case)
- Return 200 OK for DELETE (use 204 No Content)
- Break idempotency for GET, PUT, DELETE
- Expose internal implementation details in URLs
