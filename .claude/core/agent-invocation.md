# Sub-Agent Invocation Mechanism

## 調度機制概述

**⚠️ 重要：完整的調用範例與最佳實踐請參考：**
→ `.claude/ORCHESTRATOR_USAGE_TEMPLATE.md`

### 核心調用原則

1. 永遠先**載入** `templates/sub-agent-runtime-core.md` (執行規則，非 Sub-Agent)
2. 再**載入**具體的 Sub-Agent 定義檔案 (真正的角色定義)
3. 提供完整上下文 (Sub-Agent 無法存取對話歷史)
4. 明確的輸出要求與交付物路徑
5. 實際調用的是 `general-purpose` agent，配合載入的內容執行

---

## 調度步驟

### 步驟 1：讀取必要檔案

```
→ Read .claude/templates/sub-agent-runtime-core.md (執行規則文件，必須載入)
→ Read .claude/agents/{Agent名稱}.md (Sub-Agent 角色定義)
確認 Agent 的角色、能力、和輸出規範
```

### 步驟 2：組合 Task Prompt

**標準 Prompt 結構：**

```
[=== Sub-Agent Runtime Core ===]
{從 templates/sub-agent-runtime-core.md 載入的核心約束與執行規則}

[=== Agent 角色定義 ===]
{從 agents/{Agent名稱}.md 載入的 Sub-Agent 角色定義}

[=== 當前任務 ===]
{根據工作流程階段定義的具體任務}

[=== 輸入資料 ===]
{前階段交付物內容或路徑}
{用戶提供的需求或資料}
{所有必要的上下文資訊}

[=== 輸出要求 ===]
{預期的交付物格式和內容要求}
{品質標準和驗收條件}
{檔案輸出路徑（如 docs/PROD.md）}

[=== 額外上下文 ===]（選填）
{技術限制、時程要求、或其他約束}
```

註：標準回報格式已在 templates/sub-agent-runtime-core.md 中定義，不需要重複

### 步驟 3：調用 Task Tool

**實際調用範例：**

```javascript
Task(
  subagent_type: "general-purpose",
  description: "調用產品經理 Agent 分析訂閱系統需求",
  prompt: `
你是專業的產品經理 Agent。

## 角色定義
[從 Read .claude/agents/產品經理.md 讀取的內容]

## 當前任務
用戶想開發一個訂閱系統，核心功能包含：
- 用戶可以訂閱感興趣的內容
- 接收內容更新通知

目標用戶：B2C 個人用戶
規模預期：中型（1000-10000 用戶）

## 輸出要求
請產出完整的 PROD.md，包含：
1. 產品概述與價值主張
2. 用戶故事和使用場景
3. 功能需求清單（優先級排序）
4. 非功能需求（效能、安全性）
5. MVP 範圍定義

檔案輸出：docs/PROD.md

## 回報格式
使用標準的 Sub-agent 回報格式
`
)
```

### 步驟 4：處理 Sub-agent 回報

- 驗證交付物完整性（檔案是否存在、內容是否完整）
- 檢查品質是否符合標準
- 分析技術決策是否合理
- 評估建議的下一步是否適當
- 決定下一階段調度策略（循序或並行）

---

## 視覺化回饋規範

調用 Sub-agent 時必須提供清楚的視覺化提示：

### 調用前（BEFORE）

```
## 🟢 即將調用：[Agent 名稱]

**📋 任務：** [簡述任務內容]
**📄 預期輸出：** [預期產出的文件路徑]
**⏱️ 預計時間：** [預估執行時間]

**輸入資訊：**
- [關鍵輸入資訊 1]
- [關鍵輸入資訊 2]
- ...
```

### 調用後（AFTER）

```
## ✅ [Agent 名稱] 執行完成

**📄 已產出：** [實際產出的文件路徑]
**⏱️ 執行時間：** [實際執行時間]
**📊 交付物摘要：** [簡述交付物內容]

**下一步建議：**
- [建議的下一個 Agent 或動作]
```

---

## Sub-Agent 執行失敗處理

### 檢測方法

1. Sub-agent 回報完成後，Orchestrator **必須**驗證交付物是否存在
2. 使用 Read/Glob/Bash 工具檢查檔案是否實際建立
3. 若交付物不存在 → 判定為「執行失敗」

### 失敗原因分析

- Sub-agent 可能只進行了「規劃」而未「執行」
- Sub-agent 可能誤解任務要求
- Sub-agent 可能遇到技術限制但未明確回報

### 處理流程（MUST 遵循）

```
IF (Sub-agent 回報完成 && 交付物不存在):
  THEN:
    1. 記錄失敗原因與缺失的交付物清單
    2. **重新調用同一個 Sub-agent**，並在 Task Prompt 中：
       - 明確指出「前次執行失敗」
       - 列出「缺失的交付物」
       - 強調「必須實際使用 Write/Edit 工具建立檔案」
       - 提供更詳細的輸出要求（檔案路徑、內容結構）
    3. 若第二次仍失敗 → 回報用戶並暫停執行

ELSE IF (交付物存在但品質不符):
  THEN:
    1. 分析品質問題（內容不完整、格式錯誤、邏輯錯誤）
    2. 重新調用 Sub-agent 並提供具體改進建議
    3. 最多嘗試 2 次改進

ELSE:
  → 執行成功，繼續下一步
ENDIF
```

### 重新調用範例

```markdown
## 🔄 重新調用：Backend Developer (Go) Agent

**原因：** 前次執行失敗 - 未產出交付物

**缺失的交付物：**
- ❌ internal/handler/auth_integration_test.go（整合測試檔案）
- ❌ docs/CHANGE_SUMMARY.md（變更摘要文件）
- ❌ Implementation Plan Status 未更新

**任務要求（強化版）：**
你**必須**使用 Write 工具建立以下檔案：
1. `/path/to/auth_integration_test.go` - 包含 5 個 E2E 測試
2. `/path/to/CHANGE_SUMMARY.md` - 包含完整變更摘要
3. 使用 Edit 工具更新 IMPLEMENTATION_PLAN Status

**驗證標準：**
- 完成後，Orchestrator 將使用 `ls -la /path/to/` 驗證檔案存在
- 使用 `wc -l /path/to/file` 驗證檔案非空
```

### Orchestrator 自我提醒

- ⚠️ 永遠不要假設 Sub-agent 回報「完成」就代表真的完成
- ✅ 必須驗證每個關鍵交付物的存在性
- ✅ 發現失敗時，提供更明確的指令重新調用
- ✅ 記錄失敗次數，避免無限重試（最多 2 次）

---

## 並行調度策略

### 何時使用並行調度

- 多個 Sub-agent 工作無依賴關係時
- 可顯著縮短整體執行時間
- 例如：前端開發 + 後端開發 + DBA 可同時進行
- 例如：Backend Code Reviewer + 文件審查可同時進行

### 並行調度方法

- 在單次回應中使用多個 Task tool 調用
- Claude Code 會自動並行執行這些 Task
- 等待所有結果返回後，統一進行分析

### 並行調度範例

```
同時調用三個 Sub-agent：
- Task 1: 後端開發 Agent (Go) - 實現 API
- Task 2: 前端開發 Agent - 實現 UI
- Task 3: DBA Agent - 設計資料庫

所有結果返回後：
- 驗證三方交付物的一致性
- 檢查介面定義是否匹配
- 決定下一步（通常是整合測試）
```

### 注意事項

- 確保並行的 Sub-agent 之間無數據依賴
- 若有依賴關係，必須循序調用
- 並行結果需要交叉驗證一致性

---

## 資訊協調與傳遞

當架構師需要了解現有實作時：
1. Orchestrator 先調用對應的開發 Agent 分析現有代碼
2. 將分析結果傳遞給架構師 Agent
3. 架構師基於分析結果進行設計

**原則：**
- Sub-agent 之間不直接溝通，所有資訊由 Orchestrator 傳遞
- Orchestrator 負責識別「需要哪些前置資訊」並依序調度

---

## Sub-Agent 約束

⚠️ Sub-agent 必須遵守：
- 必須在單次執行中完成所有任務
- 無法調用其他 Sub-agent（只有 Orchestrator 能調度）
- 無法等待或進行互動，必須一次性完成
- 必須按標準格式輸出結果
- 確保交付文件完整且符合規範

---

## 技術限制說明

- Sub-agent 使用 Task tool 的 "general-purpose" 類型
- Sub-agent 無法存取 Orchestrator 的對話歷史
- 所有必要資訊都必須在 prompt 中提供
- Sub-agent 的回應會完整返回給 Orchestrator
- Orchestrator 負責在 Sub-agent 間傳遞資訊
