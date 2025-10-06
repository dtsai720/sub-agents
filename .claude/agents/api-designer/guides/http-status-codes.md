# HTTP Status Codes Guide

## Quick Reference

### 2xx Success
- **200 OK**: GET, PUT, PATCH success (with response data)
- **201 Created**: POST success, resource created (with Location header + new resource)
- **202 Accepted**: Async operation accepted (with status URL)
- **204 No Content**: DELETE, PUT, PATCH success (no response data needed)

### 3xx Redirection
- **301 Moved Permanently**: Resource permanently moved
- **302 Found**: Resource temporarily moved
- **304 Not Modified**: Cache valid (with ETag/Last-Modified)

### 4xx Client Errors
- **400 Bad Request**: Invalid request format or validation failure
- **401 Unauthorized**: Not authenticated (missing/invalid token)
- **403 Forbidden**: Authenticated but insufficient permissions
- **404 Not Found**: Resource not found
- **405 Method Not Allowed**: HTTP method not allowed (return Allow header)
- **406 Not Acceptable**: Cannot provide requested Content-Type
- **409 Conflict**: Resource conflict (e.g., email exists, optimistic lock)
- **410 Gone**: Resource permanently deleted (vs 404 temporary)
- **412 Precondition Failed**: Precondition failed (If-Match/If-None-Match)
- **413 Payload Too Large**: Request body too large
- **415 Unsupported Media Type**: Content-Type not supported
- **422 Unprocessable Entity**: Semantic error (valid format but invalid logic)
- **429 Too Many Requests**: Rate limit exceeded (return Retry-After header)

### 5xx Server Errors
- **500 Internal Server Error**: Unexpected server error
- **501 Not Implemented**: Feature not implemented
- **502 Bad Gateway**: Upstream service error
- **503 Service Unavailable**: Service temporarily unavailable (maintenance, overload)
- **504 Gateway Timeout**: Upstream service timeout

## Decision Tree

```
Operation successful?
  └─ YES → Need to return data?
      ├─ YES → 200 OK
      ├─ NO → 204 No Content
      └─ Created new resource? → 201 Created (with Location header)

Operation failed?
  └─ Client issue?
      ├─ Not authenticated → 401 Unauthorized
      ├─ Insufficient permissions → 403 Forbidden
      ├─ Resource not found → 404 Not Found
      ├─ Validation error → 400 Bad Request
      ├─ Resource conflict → 409 Conflict
      ├─ Rate limit → 429 Too Many Requests
      └─ Semantic error → 422 Unprocessable Entity

  └─ Server issue?
      ├─ Upstream error → 502 Bad Gateway
      ├─ Timeout → 504 Gateway Timeout
      ├─ Maintenance → 503 Service Unavailable
      └─ Other → 500 Internal Server Error
```

## Common Patterns

### POST - Create Resource
```
Success: 201 Created + Location header + resource in body
Validation error: 400 Bad Request
Conflict (duplicate): 409 Conflict
Unauthorized: 401 Unauthorized
Server error: 500 Internal Server Error
```

### GET - Read Resource
```
Success: 200 OK + resource in body
Not found: 404 Not Found
Unauthorized: 401 Unauthorized
Forbidden: 403 Forbidden
Server error: 500 Internal Server Error
```

### PUT - Full Update
```
Success with data: 200 OK + updated resource
Success without data: 204 No Content
Not found: 404 Not Found
Validation error: 400 Bad Request
Conflict: 409 Conflict (e.g., optimistic lock)
Unauthorized: 401 Unauthorized
Server error: 500 Internal Server Error
```

### PATCH - Partial Update
```
Success with data: 200 OK + updated resource
Success without data: 204 No Content
Not found: 404 Not Found
Validation error: 400 Bad Request
Unauthorized: 401 Unauthorized
Server error: 500 Internal Server Error
```

### DELETE - Delete Resource
```
Success (no content): 204 No Content
Success (with confirmation): 200 OK + deletion info
Not found: 404 Not Found
Conflict: 409 Conflict (e.g., resource in use)
Unauthorized: 401 Unauthorized
Server error: 500 Internal Server Error
```

## Important Notes

- **201 Created MUST include Location header** pointing to the new resource
- **204 No Content MUST NOT include response body**
- **409 Conflict** for business-level conflicts (duplicate email, version mismatch)
- **422 Unprocessable Entity** for semantic errors (valid JSON but invalid business logic)
- **401 vs 403**: 401 = "who are you?", 403 = "I know who you are, but you can't do this"
