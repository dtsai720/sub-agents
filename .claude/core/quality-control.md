# Quality Control & Validation

## 品質控制原則

遇到錯誤時，詳細處理流程請參考：
→ `.claude/workflows/error-handling.md`

---

## 驗證清單

### 交付物驗證

- 交付文件是否存在且完整
- 文件格式和內容是否符合規範
- 技術決策是否合理
- **測試覆蓋率是否達標**

---

## 測試覆蓋率驗證規則

當開發 Agent 完成開發階段後，Orchestrator 必須執行以下檢查：

### 1. 執行測試並檢查覆蓋率

```bash
# 執行完整測試套件並產生覆蓋率報告
go test ./... -v -coverprofile=coverage.out -timeout 600s
go tool cover -func=coverage.out | tail -1
```

### 2. 讀取 Implementation Plan 確認覆蓋率目標

從 `docs/IMPLEMENTATION_PLAN_BACKEND_{GO|JAVA|PYTHON}.md` 讀取各層的 Coverage Target

例如：
- Repository > 80%
- Service > 80%
- Handler > 75%

### 3. 比對實際覆蓋率與目標

```
IF (實際覆蓋率 < 目標覆蓋率):
  THEN:
    → 記錄未達標的模組和差距
    → 計算需要補充的測試數量
    → 重新調用開發 Agent，明確要求：
      - "當前 {模組} 覆蓋率為 {實際}%，未達目標 {目標}%"
      - "請補充測試案例，重點測試：{未覆蓋的函數/分支}"
      - "目標：將覆蓋率提升至 {目標}% 以上"
ELSE:
  → 覆蓋率達標，繼續下一階段 (Code Review)
ENDIF
```

### 4. 覆蓋率提升重試機制

- 最多重試 2 次
- 每次重試後重新驗證覆蓋率
- 若 2 次後仍未達標 → 向用戶報告，詢問是否：
  * 選項 A：繼續補充測試（第 3 次嘗試）
  * 選項 B：調整覆蓋率目標（需更新 Implementation Plan）
  * 選項 C：先進行 Code Review，標記覆蓋率問題

### 5. 覆蓋率檢查範例輸出

```
❌ 測試覆蓋率未達標

| 模組       | 當前覆蓋率 | 目標覆蓋率 | 差距   | 狀態 |
|-----------|-----------|-----------|--------|------|
| Repository | 72.3%     | >80%      | -7.7%  | ❌   |
| Service    | 13.9%     | >80%      | -66.1% | ❌   |
| Handler    | 55.8%     | >75%      | -19.2% | ❌   |

**決策：** 重新調用 Backend Developer Agent 補充測試
```

---

## 常見錯誤碼

- **AGENT_EXEC_FAIL** - Task 調用失敗
- **QUALITY_CHECK_FAIL** - 品質不足
- **REQUIREMENT_CHANGED** - 需求變更
- **TECHNICAL_CONFLICT** - 技術衝突
- **AGENT_NOT_FOUND** - 缺少 Sub-agent
- **COVERAGE_INSUFFICIENT** - 測試覆蓋率不足

---

## 迭代原則

最多 3 次改進，每次提供明確改進方向

---

## Sub-agent 輸出要求

每個 Sub-agent 必須在單次執行中完成任務，並以以下格式輸出：

```
## 📋 任務完成報告
**Agent 身分：** [Agent 名稱]

**完成任務：**
[具體完成的工作內容，包含關鍵決策和實作細節]

**交付文件：**
- 文件 1：{檔案路徑或內容摘要}
- 文件 2：{檔案路徑或內容摘要}
- ...

**品質自檢：**
✅ 已完成項目：
- [項目 1]
- [項目 2]

⚠️ 需注意事項：
- [注意事項 1]
- [注意事項 2]

**技術決策：**
- [重要的技術選型、架構決策、或設計考量]

**建議下一步：**
- 推薦 Agent：[Agent 名稱]
- 原因：[為什麼需要這個 Agent]
- 所需輸入：[需要哪些文件或資訊]
```

⚠️ 重要提醒：
- Sub-agent 無法等待或進行多輪互動
- 必須在單次回應中完成所有工作
- 若任務過於複雜，應拆分為多個獨立的 Sub-agent 任務

---

## DBA Agent 調度決策

### SQL DBA vs NoSQL DBA 選擇機制

Orchestrator 必須根據資料庫類型調用正確的 DBA Agent。

**步驟 1：讀取 CLOUD_ARCHITECTURE.md**
```
→ Read docs/CLOUD_ARCHITECTURE.md
→ 查找 "Database Type:" 標記或資料庫服務名稱
```

**步驟 2：識別資料庫類型**

**SQL 資料庫關鍵字：**
- PostgreSQL, MySQL, MariaDB
- AWS RDS, Aurora (PostgreSQL/MySQL)
- Azure SQL Database, Azure Database for PostgreSQL/MySQL
- Google Cloud SQL, Spanner

**NoSQL 資料庫關鍵字：**
- MongoDB, MongoDB Atlas
- AWS DynamoDB, DocumentDB
- Azure Cosmos DB (NoSQL API, MongoDB API)

**步驟 3：調用對應的 DBA Agent**

```
IF 識別到 SQL 資料庫:
  THEN:
    → Read .claude/agents/sql-dba.md
    → 組合 Task Prompt (runtime-core + sql-dba + 任務)
    → 調用 SQL DBA Agent
    → 預期輸出：
      - docs/SCHEMA.sql
      - docs/migrations/*.sql
      - docs/QUERY_OPTIMIZATION.md

ELSE IF 識別到 NoSQL 資料庫:
  THEN:
    → Read .claude/agents/nosql-dba.md
    → 組合 Task Prompt (runtime-core + nosql-dba + 任務)
    → 調用 NoSQL DBA Agent
    → 預期輸出：
      - docs/NOSQL_SCHEMA.md
      - docs/INDEX_STRATEGY.md
      - docs/DATA_MODEL.json

ELSE IF 同時識別到 SQL 和 NoSQL (Hybrid):
  THEN:
    → 並行調用 SQL DBA + NoSQL DBA
    → 在 Task Prompt 中明確告知各自負責的資料庫部分
    → 預期輸出：兩者的輸出文件都會產出

ELSE (無法識別):
  THEN:
    → 詢問用戶：
      "此專案使用的資料庫類型為何？
       - SQL 資料庫（PostgreSQL、MySQL、Azure SQL 等）
       - NoSQL 資料庫（MongoDB、DynamoDB、Cosmos DB 等）
       - 混合架構（SQL + NoSQL）"
    → 根據用戶回答調用對應 DBA Agent
ENDIF
```

**步驟 4：驗證輸出**

- SQL DBA → 檢查 SCHEMA.sql 是否可執行、Migration 腳本是否完整
- NoSQL DBA → 檢查 Document Schema、Partition Key 設計是否合理

**常見錯誤處理：**
- 若 CLOUD_ARCHITECTURE.md 未標記 Database Type → 從資料庫服務名稱推斷
- 若無法推斷 → 詢問用戶
- 若調用錯誤的 DBA Agent → 從錯誤訊息中識別，重新調用正確的 Agent

---

## 專案狀態管理

詳細的 PROJECT_STATUS.md 管理規則請參考：
→ `.claude/CLAUDE.md` 的 `[專案狀態管理]` 章節

**核心原則：**
- Orchestrator 負責維護 `docs/PROJECT_STATUS.md`
- 每次 Sub-Agent 執行完成後自動更新
- 支援中斷恢復功能
- 記錄當前階段、已完成階段、待執行階段
- 提供明確的下一步指令
