## 📋 任務完成報告

**Agent 身分：** API Designer Agent

**完成任務：**
為 [專案名稱] 設計完整 OpenAPI 3.x 規格，包含：
- **端點數量**：[N] 個端點（Authentication: [N], Users: [N], Admin: [N]）
- **Schema 數量**：[N] 個資料實體、[N] 個請求 Schema、[N] 個回應 Schema
- **認證機制**：[JWT/OAuth2/API Key]
- **錯誤處理**：標準化錯誤回應格式（RFC 7807）
- **範例數量**：每個端點至少 [N] 個範例
- [關鍵設計決策 1]
- [關鍵設計決策 2]

**交付文件：**
- openapi.yaml：完整 OpenAPI 3.x 規格（[N] 個端點、[N] 個 Schema）

**品質自檢：**
✅ 已完成項目：

**OpenAPI 規範：**
- openapi.yaml 符合 OpenAPI 3.0.3 規範
- 所有端點包含完整 paths 定義（URL、Method、描述、參數、請求、回應）
- 所有 Schema 定義驗證規則（required、format、pattern、minLength、maxLength）
- 所有端點包含錯誤回應（400、401、403、404、500）
- 所有端點至少 1 個 request/response example
- components/schemas 完整定義（[N] 個 Schema）
- components/securitySchemes 已定義（[認證方式]）
- 錯誤回應格式統一（RFC 7807 Problem Details）
- info 章節包含完整說明（認證、分頁、錯誤處理）
- 使用 tags 分組端點（[列出 tags]）
- 可直接用於 Swagger UI、Postman、程式碼生成

**RESTful 原則遵守：**
- 所有 CRUD 端點使用名詞複數（/users、/orders，非 /getUsers、/createOrder）
- HTTP 方法語意正確（GET 讀取、POST 建立、PUT 完整更新、PATCH 部分更新、DELETE 刪除）
- HTTP 狀態碼正確使用（201 Created + Location header、204 No Content、400/401/403/404/409/422/500）
- POST 成功建立資源回傳 201 Created 並包含 Location header
- DELETE 成功無需回傳資料使用 204 No Content
- 資源命名一致（全部小寫 + 連字號，如 /order-items）
- 階層式資源合理（/users/{id}/orders 表達所屬關係）
- GET 端點冪等且安全（無副作用、可快取）
- PUT、DELETE 端點冪等

**資料建模與抽象化：**
- Schema 設計清晰、精簡且符合業務邏輯
- 避免過度正規化（合理聚合資料，減少 API 呼叫次數）
- 避免過度巢狀（最多 3 層巢狀結構）
- 資料關聯合理（1:1、1:N、N:N 關聯清晰表達）
- 優先使用 optional 欄位（向前相容性，避免破壞性變更）
- 敏感資料（password、token、secret）已排除在回應 Schema
- 介面契約穩定（底層實作變化不影響 API 契約）

⚠️ 需注意事項：
- [假設或未確認的部分]
- [需要後端開發注意的事項]
- [API 使用限制或建議]
（若無則寫「無」）

**API 設計決策：**
- 認證機制：[JWT/OAuth2] - 理由：[說明] - 安全性考量：[...]
- 分頁策略：[offset/cursor-based] - 理由：[說明] - 效能考量：[...]
- 錯誤格式：RFC 7807 Problem Details - 理由：[標準化、易於除錯]
- Schema 驗證：[說明驗證規則設計邏輯]
- [其他關鍵設計決策]

**設計哲學應用：**
- ✅ 一致性：[統一命名、格式、錯誤處理]
- ✅ 開發者友善：[清晰文件、豐富範例、易於理解]
- ✅ 安全內建：[認證機制、輸入驗證、敏感資料保護]
- ✅ 可測試性：[明確 Schema、範例、錯誤定義]
- ✅ 標準遵循：[OpenAPI 3.x、RFC 7807、RESTful 最佳實踐]

**建議下一步：**

**情境：標準開發流程**
- 推薦 Agent 1：Backend Developer Agent (Go/Java/Python)
  - 原因：根據 openapi.yaml 實作 API 端點
  - 所需輸入：CLOUD_ARCHITECTURE.md、openapi.yaml、SCHEMA.sql（如有）
  - 完成後：提供 API 實作程式碼、單元測試
- 推薦 Agent 2：Frontend Agent（可並行）
  - 原因：根據 openapi.yaml 開發前端（使用 Mock API 或 API Client 生成）
  - 所需輸入：CLOUD_ARCHITECTURE.md、openapi.yaml
  - 完成後：提供前端程式碼、元件、路由

**開發工具建議：**
- Swagger UI：視覺化 API 文件（https://editor.swagger.io）
- Postman：匯入 openapi.yaml 進行 API 測試
- OpenAPI Generator：自動生成 Client SDK（Java/Python/JavaScript）
- Prism：Mock Server（基於 openapi.yaml 提供測試用 API）
