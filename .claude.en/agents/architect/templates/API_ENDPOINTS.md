# API Endpoints Definition

## Authentication
- **POST /auth/login** - 用戶登入
  - Request: `{ "email": "string", "password": "string" }`
  - Response: `{ "token": "string", "user": {...} }`
  - Auth: Public

- **POST /auth/refresh** - 刷新 Token
  - Request: `{ "refresh_token": "string" }`
  - Response: `{ "token": "string" }`
  - Auth: JWT

- **POST /auth/logout** - 用戶登出
  - Request: Empty
  - Response: `{ "message": "Success" }`
  - Auth: JWT

## Users
- **GET /users** - 獲取用戶列表
  - Query: `?page=1&limit=20`
  - Response: `{ "data": [...], "meta": { "total": 100 } }`
  - Auth: JWT (Admin only)

- **POST /users** - 建立用戶
  - Request: `{ "name": "string", "email": "string" }`
  - Response: `{ "id": "uuid", "name": "string", "email": "string" }`
  - Auth: JWT (Admin only)

- **GET /users/{id}** - 獲取單一用戶
  - Response: `{ "id": "uuid", "name": "string", "email": "string" }`
  - Auth: JWT

- **PUT /users/{id}** - 更新用戶
  - Request: `{ "name": "string" }`
  - Response: `{ "id": "uuid", "name": "string" }`
  - Auth: JWT (Owner or Admin)

- **DELETE /users/{id}** - 刪除用戶
  - Response: `{ "message": "Deleted" }`
  - Auth: JWT (Admin only)

## Orders (範例)
- **GET /orders** - 獲取訂單列表
- **POST /orders** - 建立訂單
- **GET /orders/{id}** - 獲取訂單詳情
- **PUT /orders/{id}/status** - 更新訂單狀態

---

**Note**: 完整的 OpenAPI 規格（含 Schema、驗證規則、錯誤處理）由 API Designer Agent 設計。
