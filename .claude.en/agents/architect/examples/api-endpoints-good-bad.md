# API Endpoints - 高層次 vs 詳細範例對照

本文件說明 Cloud Architect 與 API Designer 在 API 端點定義上的職責邊界。

---

## 範例對照

### ✅ Architect 產出（高層次）- API_ENDPOINTS.md

```markdown
## User Management APIs

### GET /users
- **描述**：取得使用者列表
- **認證**：需要（x-user-id, x-tenant-id headers）
- **請求參數**：
  - page: 頁碼
  - limit: 每頁筆數
- **回應格式**：
  ```json
  {
    "users": [ { "id": "...", "name": "...", "email": "..." } ],
    "total": 100,
    "page": 1
  }
  ```

### GET /users/{id}
- **描述**：取得單一使用者資訊
- **認證**：需要（x-user-id, x-tenant-id headers）
- **回應格式**：
  ```json
  {
    "id": "uuid",
    "name": "string",
    "email": "string"
  }
  ```
```

---

### ❌ 不產出（交給 API Designer）- openapi.yaml

```yaml
# Architect 不需要產出以下內容：

paths:
  /users:
    get:
      summary: Get user list
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                required: [users, total, page]
                properties:
                  users:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  total:
                    type: integer
                    example: 100
                  page:
                    type: integer
                    example: 1

components:
  schemas:
    User:
      type: object
      required: [id, name, email]
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email
```

---

## 職責邊界清楚定義

| 項目 | Architect（高層次） | API Designer（詳細） |
|------|-------------------|-------------------|
| **端點定義** | ✅ URL + Method | ✅ 完整 OpenAPI path |
| **描述** | ✅ 一句話說明 | ✅ 詳細 summary + description |
| **認證** | ✅ 認證方式（JWT/Headers） | ✅ SecuritySchemes 定義 |
| **請求參數** | ✅ 參數名稱列表 | ✅ Schema（type, min, max, pattern） |
| **回應格式** | ✅ JSON 結構範例 | ✅ 完整 Schema + 多個 examples |
| **錯誤處理** | ✅ 列出錯誤類型 | ✅ 每個錯誤的 Schema |
| **驗證規則** | ❌ 不定義 | ✅ required, format, pattern |
| **OpenAPI YAML** | ❌ 不產出 | ✅ 完整可用的 openapi.yaml |

---

## 最低交付要求

### Architect 的 API_ENDPOINTS.md 必須包含：

- **[ ]** 每個端點有 URL + HTTP Method
- **[ ]** 每個端點有一句話描述（說明用途）
- **[ ]** 每個端點標註認證需求（Public / Protected）
- **[ ]** 每個端點有基本 JSON 結構範例（不含 schema 定義）
- **[ ]** 已分組端點（Authentication / CRUD / Business Logic）
- **[ ]** 已說明整體認證策略（JWT / OAuth2 / API Key / Custom Headers）

### Architect 不需要做（由 API Designer Agent 負責）：

- ❌ 詳細 Schema 定義（type, format, pattern, minLength, maxLength）
- ❌ 驗證規則設計（required, enum, default）
- ❌ 完整範例（multiple examples per endpoint）
- ❌ OpenAPI 3.x YAML 檔案
- ❌ SecuritySchemes 詳細定義
- ❌ Components/Schemas 定義

---

## 輸出要求總結

```
REQUIRED OUTPUT from STEP 3:
- API_ENDPOINTS.md（高層次端點清單，符合上述最低要求）
- 認證與授權策略說明
- **Note**: 完整的 OPENAPI.yaml 由 API Designer Agent 設計
```
