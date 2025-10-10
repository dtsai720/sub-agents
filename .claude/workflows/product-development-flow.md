# Product Development Workflow

**觸發條件：** 用戶描述產品概念或想法

---

## 工作流程

### 步驟 1: 產品需求分析

**調用 Agent:** Product Manager Agent

**輸入：** 用戶需求描述

**輸出：** `docs/PROD.md`

**內容包含：**
- 產品概述與價值主張
- 目標用戶與使用場景
- 功能需求清單（優先級排序）
- 非功能需求（效能、安全性）
- MVP 範圍定義
- **⭐ UI/UX 需求標記**（新增）

---

### 步驟 2: 功能重複檢查

**調用 Agent:** Backend Developer Agent（分析模式）

**Orchestrator 判斷：**
- 檢查專案中現有代碼語言（Go/Java/Python）
- → Read `.claude/agents/backend-developer-{go|java|python}.md`
- 若專案為新專案或無法判斷：詢問用戶技術棧

**任務：** 搜尋專案中是否已有類似功能

**輸出：** 現有功能分析報告

**決策分支：**
```
IF (找到類似功能):
  THEN:
    → 向用戶確認：
      - 選項 1：增強現有功能 → 轉「現有專案增強流程」
      - 選項 2：建立新功能 → 繼續此流程
      - 選項 3：直接使用 → 提供文件，結束
ELSE:
  → 繼續開發
ENDIF
```

詳細檢查邏輯：
→ Read `.claude/workflows/feature-duplication-check.md`

---

### 步驟 3: UI/UX 設計決策 ⭐ 新增可選邏輯

**Orchestrator 判斷邏輯：**

```
→ Read docs/PROD.md
→ 查找 UI/UX 需求標記：
  - "UI Type: API Backend Only"
  - "UI Type: Bootstrap/Tailwind Simple UI"
  - "UI Type: Custom Design"
  - "Frontend: None"
  - "Frontend: React/Vue/Angular"

IF (UI Type = "API Backend Only" OR Frontend = "None"):
  THEN:
    → 跳過 UI/UX 設計師 Agent
    → 記錄決策：「此專案為純 API Backend，使用 Swagger UI 作為開發者介面」
    → 進入步驟 4

ELSE IF (UI Type = "Bootstrap/Tailwind Simple UI"):
  THEN:
    → 跳過 UI/UX 設計師 Agent
    → 記錄決策：「使用現成 UI Framework，不需要自訂設計」
    → 進入步驟 4

ELSE IF (UI Type = "Custom Design" OR 用戶提供 Figma 連結):
  THEN:
    → 調用 UI/UX 設計師 Agent
    → 輸入：docs/PROD.md + Figma 連結（若有）
    → 輸出：docs/UI_UX_DESIGN.md (設計規範)
    → 進入步驟 4

ELSE (PROD.md 未明確標記):
  THEN:
    → 詢問用戶：
      「此專案的 UI 需求為何？

      **選項 A: 純 API Backend（推薦用於後端服務）**
      - 使用 Swagger UI 作為開發者介面
      - 前端團隊基於 OPENAPI.yaml 獨立開發
      - 適用：Microservices、Mobile Backend、B2B API

      **選項 B: 使用現成 UI Framework**
      - Bootstrap 5 / Tailwind CSS 快速介面
      - 專注功能而非視覺設計
      - 適用：內部工具、Admin Dashboard、MVP 驗證

      **選項 C: 需要完整視覺設計**
      - 需要提供 Figma 設計稿或詳細設計需求
      - 調用 UI/UX 設計師 Agent 產出設計規範
      - 適用：面向消費者的產品、品牌重視的專案

      請選擇 A、B 或 C，或直接描述您的 UI 需求。」

    → 根據用戶回答更新 PROD.md 的 UI Type 標記
    → 執行對應的分支邏輯
ENDIF
```

---

### 步驟 4: 雲端架構設計

**調用 Agent:** Cloud Architect Agent

**輸入：**
- `docs/PROD.md`
- `docs/UI_UX_DESIGN.md`（若存在）

**輸出：**
- `docs/CLOUD_ARCHITECTURE.md`
- `docs/API_ENDPOINTS.md`
- `docs/ER_DIAGRAM.md`

**重要：** CLOUD_ARCHITECTURE.md 必須包含明確的資料庫類型標記：
- Database Type: SQL (PostgreSQL/MySQL/Azure SQL 等)
- Database Type: NoSQL (MongoDB/DynamoDB/Cosmos DB 等)
- Database Type: Hybrid (SQL + NoSQL - 需分別調用兩個 DBA Agent)

---

### 步驟 5: API 與資料庫設計（可並行）

#### 5.1 API 設計

**調用 Agent:** API Designer Agent

**輸入：**
- `docs/CLOUD_ARCHITECTURE.md`
- `docs/API_ENDPOINTS.md`

**輸出：** `docs/OPENAPI.yaml`（擴展 API_ENDPOINTS.md）

#### 5.2 資料庫設計

**Orchestrator 根據 CLOUD_ARCHITECTURE.md 的 Database Type 決定**

**決策邏輯：**
```
→ Read docs/CLOUD_ARCHITECTURE.md
→ 查找 "Database Type:" 標記

IF Database Type = SQL:
  THEN:
    → Read .claude/agents/sql-dba.md
    → 調用 SQL DBA Agent
    → 預期輸出：SCHEMA.sql + Migration 腳本

ELSE IF Database Type = NoSQL:
  THEN:
    → Read .claude/agents/nosql-dba.md
    → 調用 NoSQL DBA Agent
    → 預期輸出：NOSQL_SCHEMA.md + INDEX_STRATEGY.md + DATA_MODEL.json

ELSE IF Database Type = Hybrid:
  THEN:
    → 並行調用 SQL DBA + NoSQL DBA
    → SQL DBA 處理關聯式資料庫部分
    → NoSQL DBA 處理 NoSQL 資料庫部分

ELSE:
  THEN:
    → 詢問用戶選擇資料庫類型
    → 根據用戶回答調用對應 DBA Agent
ENDIF
```

詳細決策邏輯：
→ Read `.claude/core/quality-control.md` (DBA Agent 調度決策)

---

### 步驟 5.5: 資料庫運維設計（若為生產環境）

**觸發條件：**
- Schema 設計完成（SCHEMA.sql 或 NOSQL_SCHEMA.md 已產出）
- 環境為生產環境（或用戶明確要求運維規劃）
- 用戶提到：備份、HA、災難復原、安全、監控等關鍵字

**DB Ops Agent 調用：**
```
→ Read .claude/agents/db-ops.md
→ 調用 DB Ops Agent
→ 輸入：
  - docs/CLOUD_ARCHITECTURE.md
  - docs/SCHEMA.sql 或 docs/NOSQL_SCHEMA.md
  - 業務要求（RPO、RTO、可用性目標）
→ 預期輸出：
  - docs/BACKUP_STRATEGY.md（備份策略）
  - docs/HA_DR_PLAN.md（高可用與災難復原）
  - docs/MONITORING_SETUP.md（監控告警）
  - docs/SECURITY_HARDENING.md（安全加固）
  - docs/DB_OPS_RUNBOOK.md（運維手冊）
```

**注意事項：**
- 開發環境可跳過此步驟
- 若用戶未提及運維需求，Orchestrator 應主動詢問：
  「此專案是否需要規劃生產環境的資料庫運維方案？（備份、HA、監控）」

---

### 步驟 5.6: 交付物完整性檢查（開發前強制驗證）

Orchestrator 必須在進入開發階段前檢查以下必要文件：

**必要文件清單：**
```
✅ 必須存在：
- docs/CLOUD_ARCHITECTURE.md (架構師產出)
- docs/OPENAPI.yaml 或 docs/API_ENDPOINTS.md (API Designer 產出)
- docs/SCHEMA.sql (SQL DBA) 或 docs/NOSQL_SCHEMA.md (NoSQL DBA)

⚠️ 生產環境必須：
- docs/BACKUP_STRATEGY.md (DB Ops 產出)
- docs/HA_DR_PLAN.md (DB Ops 產出)
```

**檢查邏輯：**
```
IF (缺少 OPENAPI.yaml AND 缺少 API_ENDPOINTS.md):
  THEN:
    → 暫停開發流程
    → 調用 API Designer Agent 產出 OPENAPI.yaml
    → 驗證檔案是否產出
    → 繼續開發流程

IF (缺少 SCHEMA.sql AND 缺少 NOSQL_SCHEMA.md):
  THEN:
    → 暫停開發流程
    → 根據 Database Type 調用對應 DBA Agent
    → 驗證檔案是否產出
    → 繼續開發流程

IF (生產環境 AND 缺少 DB Ops 文件):
  THEN:
    → 詢問用戶：「是否需要生產環境運維規劃？」
    → 若需要 → 調用 DB Ops Agent
ENDIF
```

---

### 步驟 6: 開發階段（分兩階段執行）

#### 階段 6.1: Implementation Plan 產出

**調用 Agent:** Backend Developer Agent（首次）

**任務：** 產出 IMPLEMENTATION_PLAN

**輸出：** `docs/IMPLEMENTATION_PLAN_{AGENT_NAME}.md`

**完成後：** 開發 Agent 回報 Orchestrator，STOP 執行

#### 階段 6.2: 用戶審查 Plan

**Orchestrator 行動：**
- 讀取 IMPLEMENTATION_PLAN 檔案
- 呈現給用戶審查（3-5 個開發階段、測試計畫、檔案清單）
- 等待用戶確認或修改

**決策點：**
- 批准 → 進入階段 6.3
- 修改 → Orchestrator 更新 Plan，再次確認
- 拒絕 → 調整需求，重新規劃

#### 階段 6.3: 實際開發（可並行）

**調用 Agent:** Backend Developer Agent（第二次）

**輸入：** 已批准的 IMPLEMENTATION_PLAN

**任務：**
- 前端開發 Agent 執行 IMPLEMENTATION_PLAN_FRONTEND.md（若需要）
- 後端開發 Agent 執行 IMPLEMENTATION_PLAN_BACKEND_{GO|JAVA|PYTHON}.md
- 開發 Agent 更新 Plan 中的 Stage Status
- 完成後清理 IMPLEMENTATION_PLAN 檔案

---

### 步驟 6.5: API 變更同步檢查（自動觸發）

**Orchestrator 行動：**
- 讀取 `docs/CHANGE_SUMMARY.md` 檢測 API 變更
- 若有 API 變更 → 調用 API Designer Agent 同步 OPENAPI.yaml
- 若無 API 變更 → 跳過此步驟

詳細規則：
→ Read `.claude/core/quality-control.md` (API 變更檢測與同步規則)

---

### 步驟 7: Backend Code Review

**調用 Agent:** Backend Code Reviewer Agent

**輸入：** `docs/CHANGE_SUMMARY.md`

**輸出：** `docs/CODE_REVIEW_REPORT.md`

**決策：**
- 若有 Critical Issues → 返回開發 Agent 修復
- 若通過 → 繼續下一步

---

### 步驟 8: QA 測試

**調用 Agent:** QA Agent

**任務：**
- 執行測試套件
- 驗證功能完整性

**輸出：** `docs/QA_TEST_REPORT.md`

**決策：**
- 失敗 → 返回開發 Agent 修改
- 通過 → 繼續下一步

---

### 步驟 9: DevOps 部署配置（若需要）

**觸發條件（滿足任一即調用）：**
- 用戶明確要求部署到生產環境
- 初始需求分析時選擇「生產環境」
- QA 測試通過且用戶詢問「如何部署」
- 用戶提到：Terraform、Kubernetes、ECS、CI/CD、部署

**調用 Agent:** DevOps Agent

**輸出：**
- `terraform/` directory (IaC for AWS resources)
- `.github/workflows/deploy.yml` (CI/CD pipeline)
- `k8s/` directory (若使用 Kubernetes)
- `docs/DEPLOYMENT_GUIDE.md`
- `docs/INFRASTRUCTURE.md`

**注意事項：**
- 開發環境可跳過此步驟（使用 docker-compose 即可）
- 若用戶未提及部署，Orchestrator 應在 QA 通過後詢問：
  「QA 測試已通過！您需要部署到生產環境嗎？
   - 選項 A: 是（我會調用 DevOps Agent 產出 Terraform + CI/CD）
   - 選項 B: 否（僅本地開發/測試）」

**⚠️ IaC 安全約束（DevOps Agent 必須遵守）：**
- ✅ Terraform: 僅允許 `terraform plan`（驗證配置）
- ✅ Helm: 僅允許 `helm install --dry-run --debug`（模擬部署）
- ✅ Kubernetes: 僅允許 `kubectl apply --dry-run=client`（客戶端驗證）
- ❌ 禁止: 任何實際部署指令（`terraform apply`, `helm install`, `kubectl apply`）

詳細安全規範：
→ Read `.claude/agents/devops.md` 的 `[IaC 測試安全約束]` 章節

---

### 步驟 10: 文件審查與交付

**調用 Agent:** 文件審查 Agent（若實作）

**任務：** 驗收所有交付物

**完成：** 專案交付

---

## UI/UX 決策範例

### 範例 1: 純 API Backend 專案

**PROD.md 標記：**
```markdown
## UI/UX Requirements

**UI Type:** API Backend Only

**Frontend:** None (API only, will be consumed by mobile apps and web clients)

**Developer Interface:** Swagger UI (auto-generated from OPENAPI.yaml)

**Rationale:** This is a microservice providing authentication functionality.
Frontend teams will develop their own UI based on the OpenAPI specification.
```

**Orchestrator 行為：**
- ✅ 跳過 UI/UX 設計師 Agent
- ✅ 專注 API 品質和文件完整性
- ✅ 確保 OPENAPI.yaml 包含完整的範例和說明

### 範例 2: Bootstrap 快速 MVP

**PROD.md 標記：**
```markdown
## UI/UX Requirements

**UI Type:** Bootstrap/Tailwind Simple UI

**Frontend:** React with Bootstrap 5

**Design Approach:** Use Bootstrap components for rapid MVP development

**Rationale:** This is an internal admin dashboard. Focus on functionality
over visual design. Use standard Bootstrap components to save time.
```

**Orchestrator 行為：**
- ✅ 跳過 UI/UX 設計師 Agent
- ✅ Frontend Developer 使用 Bootstrap 預設樣式
- ✅ 專注功能實作而非視覺設計

### 範例 3: 需要完整設計

**PROD.md 標記：**
```markdown
## UI/UX Requirements

**UI Type:** Custom Design

**Frontend:** React with custom design system

**Design Files:** https://www.figma.com/file/abc123...

**Rationale:** This is a customer-facing SaaS product. Brand consistency
and user experience are critical. Full UI/UX design required.
```

**Orchestrator 行為：**
- ✅ 調用 UI/UX 設計師 Agent
- ✅ 基於 Figma 設計稿產出設計規範
- ✅ Frontend Developer 嚴格遵守設計規範

---

## 工作流程視覺化

```
產品經理 (PROD.md)
  ↓
功能重複檢查
  ↓
UI/UX 決策 ⭐ 新增
  ├─ 選項 A: 跳過 (API Only)
  ├─ 選項 B: 跳過 (Bootstrap)
  └─ 選項 C: UI/UX 設計師 → UI_UX_DESIGN.md
  ↓
雲端架構師 (CLOUD_ARCHITECTURE.md)
  ↓
API Designer + DBA (並行)
  ↓
DB Ops（若生產環境）
  ↓
交付物檢查
  ↓
開發 (Plan → 審查 → 實作)
  ↓
API 同步檢查
  ↓
Backend Code Reviewer
  ↓
QA 測試
  ↓
DevOps（若生產環境）
  ↓
完成
```
