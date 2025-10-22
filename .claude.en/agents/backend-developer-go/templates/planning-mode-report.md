# Planning Mode 回報格式範本

## 📋 任務完成報告 - Planning Mode

**Agent 身分：** Backend Developer (Go) Agent

**執行階段：** Phase 1 - Planning Mode

**完成任務：**
為 [專案名稱] 設計 Go 後端實作計畫
- **API 端點數量**：[N] 個
- **資料表數量**：[N] 個
- **複雜度評估**：[Simple/Medium/Complex]
- **開發階段**：[N] 個 Stages
- **預估總時間**：[X] 分鐘

**交付文件：**
- IMPLEMENTATION_PLAN_BACKEND_GO.md：完整實作計畫（[N] 個 Stages）

**開發階段概覽：**

### Stage 1: [階段名稱]
- **Goal**: [目標]
- **Files**: [N] 個檔案
- **Estimated Time**: [X] 分鐘
- **Status**: Not Started

### Stage 2: [階段名稱]
- **Goal**: [目標]
- **Files**: [N] 個檔案
- **Estimated Time**: [X] 分鐘
- **Status**: Not Started

[... 其他 Stages]

**技術堆疊：**
- Go 版本：1.24+
- Web 框架：[Gin/Echo/Fiber]
- ORM：[GORM/sqlx/ent/sqlc]
- 測試：testify, uber-go/mock, testcontainers-go
- 日誌：[slog/logrus]
- 驗證：go-playground/validator

**測試策略：**
- 單元測試：Service、Repository 層（目標覆蓋率 > 80%）
- 整合測試：使用 testcontainers-go（真實資料庫）
- Mock：Repository 介面、外部服務

**品質自檢：**
✅ 已完成項目：
- 輸入文件已分析（OPENAPI.yaml, SCHEMA.sql, CLOUD_ARCHITECTURE.md）
- 複雜度已評估（[Simple/Medium/Complex]）
- [N] 個開發階段已定義（3-5 Stages）
- 每個 Stage 包含 Goal、Tasks、Files、Tests、Success Criteria
- 測試策略已定義
- 完整檔案清單已列出
- IMPLEMENTATION_PLAN_BACKEND_GO.md 已產出

⚠️ 需注意事項：
- [如有輸入文件缺失，列出缺失項目與假設]
- [如有技術選型未明確，列出預設選擇]
- [如有外部依賴，列出整合點]

**風險識別：**
- [技術風險]：[如：外部 API 整合、效能瓶頸]
- [資源風險]：[如：開發時間不足、測試環境]
- [依賴風險]：[如：第三方套件版本、雲端服務限制]

**建議下一步：**

**重要：此為 Planning Mode，已產出 IMPLEMENTATION_PLAN，請 Orchestrator：**

1. 將 IMPLEMENTATION_PLAN_BACKEND_GO.md 呈現給使用者審查
2. 等待使用者確認或修改
3. 使用者批准後，再次調用 Backend Developer (Go) Agent
4. 明確指定 "Phase 2: Development Mode"
5. 提供已批准的 IMPLEMENTATION_PLAN_BACKEND_GO.md

**Phase 2 將執行：**
- 實際代碼開發（按 Stage 順序）
- 測試撰寫
- 文件產出
- 完成後清理 IMPLEMENTATION_PLAN 檔案
