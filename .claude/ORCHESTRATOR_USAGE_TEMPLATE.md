# Orchestrator Sub-Agent 調用模板

## 概述

此文件提供 Orchestrator 調用 Sub-Agent 的標準模板與最佳實踐。

**重要概念說明:**
- `sub-agent-runtime-core.md` 不是 Sub-Agent，是**執行規則文件**
- 調用時將其內容**載入**到 prompt 中，而非「調用」它
- 真正被調用的是 `general-purpose` agent，配合載入的規則和角色定義執行任務

---

## 標準調用模板

### 基本結構

```javascript
Task(
  subagent_type: "general-purpose",  // 真正調用的 agent 類型
  description: "[簡短描述任務 3-5 字]",
  prompt: `
    // 步驟 1: 載入執行規則 (非 Sub-Agent，是規則文件)
    ${readFile('.claude/templates/sub-agent-runtime-core.md')}

    // 步驟 2: 載入角色定義 (真正的 Sub-Agent 定義)
    ${readFile('.claude/agents/{agent-name}.md')}

    [當前任務]
    {具體任務描述}

    [輸入資料]
    {提供給 Sub-Agent 的所有必要資訊}

    [輸出要求]
    {期望的交付物}

    [額外上下文]（選填）
    {其他相關資訊}
  `
)
```

### 關鍵要素

1. **sub-agent-runtime-core.md** - 必須第一個**載入**（非調用），提供核心約束與標準回報格式
2. **{agent-name}.md** - **載入**具體的 Sub-Agent 定義，提供專業領域知識
3. **[當前任務]** - 明確說明要完成的任務
4. **[輸入資料]** - 所有必要的上下文資訊（Agent 無法存取歷史）
5. **[輸出要求]** - 期望的交付物格式

---

## 實際範例

### 範例 1: 調用 Architect Agent 設計系統架構

```javascript
Task(
  subagent_type: "general-purpose",
  description: "設計電商系統架構",
  prompt: `
    ${readFile('.claude/templates/sub-agent-runtime-core.md')}

    ${readFile('.claude/agents/architect.md')}

    [當前任務]
    設計一個電商平台的系統架構,支持以下功能:
    - 商品瀏覽與搜尋
    - 購物車管理
    - 訂單處理
    - 支付整合
    - 用戶管理

    [輸入資料]
    - 預期用戶規模: 10萬 DAU
    - 技術偏好: 雲原生架構,優先考慮 AWS
    - 團隊規模: 5 人後端團隊
    - 預算限制: 中等預算

    [輸出要求]
    - DESIGN.md: 完整的架構設計文件
    - OPENAPI.yaml: API 規格定義
    - 架構圖使用 Mermaid 格式

    [額外上下文]
    這是一個新創項目,需要快速上線,但也要考慮未來擴展性。
  `
)
```

### 範例 2: 調用 Agent Designer 設計新的 Sub-Agent

```javascript
Task(
  subagent_type: "general-purpose",
  description: "設計 Database Architect Agent",
  prompt: `
    ${readFile('.claude/templates/sub-agent-runtime-core.md')}

    ${readFile('.claude/templates/agent-designer.md')}

    [當前任務]
    設計一個專門負責資料庫架構設計的 Sub-Agent

    [輸入資料]
    需求:
    - 能根據 DESIGN.md 設計資料庫 schema
    - 產出 SQL migration 腳本
    - 產出 ER diagram (Mermaid 格式)
    - 考慮性能優化與索引策略
    - 支持 PostgreSQL, MySQL, MongoDB

    定位:
    - 在 Architect Agent 之後執行
    - 將高層次資料模型轉換為詳細的資料庫設計
    - 為後端開發團隊提供資料庫實作指引

    [輸出要求]
    - 完整的 Agent 定義檔案 (.claude/agents/database-architect.md)
    - 包含 frontmatter (name, description, model, color)
    - 遵循 agent-designer.md 的模板結構

    [額外上下文]
    此 Agent 將成為標準開發流程的一部分:
    Product Manager → Architect → Database Architect → Backend Developer
  `
)
```

### 範例 3: 調用 Prompt Agent 設計提示詞

```javascript
Task(
  subagent_type: "general-purpose",
  description: "設計代碼審查提示詞",
  prompt: `
    ${readFile('.claude/templates/sub-agent-runtime-core.md')}

    ${readFile('.claude/agents/prompt-agent.md')}

    [當前任務]
    設計一個用於代碼審查的 AI 提示詞

    [輸入資料]
    使用場景:
    - 自動審查 Pull Request
    - 檢查代碼品質、安全性、性能問題
    - 提供改進建議
    - 支持多種程式語言 (Go, Python, JavaScript)

    目標受眾:
    - 後端開發團隊
    - 中高級工程師

    期望輸出:
    - 結構化的審查報告
    - 問題嚴重程度分級 (Critical/High/Medium/Low)
    - 具體的改進建議與範例代碼

    [輸出要求]
    - 完整的提示詞文本 (英文)
    - 使用說明 (繁體中文)
    - 範例輸入與輸出

    [額外上下文]
    此提示詞將整合到 CI/CD pipeline 中,作為自動化代碼審查工具。
  `
)
```

---

## 最佳實踐

### ✅ 應該做的

1. **永遠載入 runtime-core**
   ```javascript
   ${readFile('.claude/templates/sub-agent-runtime-core.md')}  // 第一行
   ```

2. **提供完整的上下文**
   - Sub-Agent 無法存取對話歷史
   - 所有必要資訊都必須在 prompt 中
   - 包含相關的文件內容、需求、限制

3. **明確的輸出要求**
   - 具體說明期望的交付物
   - 指定文件格式與路徑
   - 列出必要的品質標準

4. **合理的任務範圍**
   - 確保任務能在單次執行中完成
   - 如果太複雜,考慮拆分為多個 Agent 調用

5. **提供足夠資訊以建議下一步**
   - Agent 需要知道整個工作流程
   - 這樣才能提供有意義的「建議下一步」

### ❌ 不應該做的

1. **不要假設 Agent 有記憶**
   ```javascript
   // ❌ 錯誤
   prompt: "繼續剛才的架構設計"

   // ✅ 正確
   prompt: `
     ${previousDesignContent}
     [當前任務]
     基於以上設計,補充 API 規格...
   `
   ```

2. **不要遺漏 runtime-core**
   ```javascript
   // ❌ 錯誤 - 缺少核心約束
   prompt: `
     ${readFile('.claude/agents/architect.md')}
     設計系統架構...
   `

   // ✅ 正確
   prompt: `
     ${readFile('.claude/templates/sub-agent-runtime-core.md')}
     ${readFile('.claude/agents/architect.md')}
     設計系統架構...
   `
   ```

3. **不要設定需要互動的任務**
   ```javascript
   // ❌ 錯誤 - Agent 無法等待回應
   prompt: "分析需求並詢問我缺少的資訊"

   // ✅ 正確
   prompt: "基於以下需求分析,若資訊不足請在回報中說明"
   ```

4. **不要讓 Agent 調用其他 Agent**
   ```javascript
   // ❌ 錯誤 - Agent 無法直接調用其他 Agent
   prompt: "完成後自動調用 Backend Developer Agent"

   // ✅ 正確
   prompt: "完成後在「建議下一步」中推薦 Backend Developer Agent"
   ```

---

## 工作流程範例

### 完整開發流程的 Agent 調用序列

```javascript
// Step 1: Product Manager Agent
Task(
  subagent_type: "general-purpose",
  description: "分析產品需求",
  prompt: `
    ${readFile('.claude/templates/sub-agent-runtime-core.md')}
    ${readFile('.claude/agents/product-manager.md')}

    [當前任務]
    分析以下用戶需求並產出 PROD.md

    [輸入資料]
    用戶想法: 我想要建立一個任務管理系統...
  `
)

// 等待 Product Manager 完成,獲得 PROD.md

// Step 2: Architect Agent
Task(
  subagent_type: "general-purpose",
  description: "設計系統架構",
  prompt: `
    ${readFile('.claude/templates/sub-agent-runtime-core.md')}
    ${readFile('.claude/agents/architect.md')}

    [當前任務]
    基於產品需求設計系統架構

    [輸入資料]
    ${readFile('docs/PROD.md')}  // 從 Step 1 獲得的產品需求
  `
)

// 等待 Architect 完成,獲得 DESIGN.md 和 OPENAPI.yaml

// Step 3: Backend Developer Agent
Task(
  subagent_type: "general-purpose",
  description: "實作後端 API",
  prompt: `
    ${readFile('.claude/templates/sub-agent-runtime-core.md')}
    ${readFile('.claude/agents/backend-developer.md')}

    [當前任務]
    實作用戶管理模組的 API

    [輸入資料]
    架構設計:
    ${readFile('docs/DESIGN.md')}

    API 規格:
    ${readFile('docs/OPENAPI.yaml')}
  `
)

// 繼續後續的 Agent 調用...
```

---

## 錯誤處理

### 當 Sub-Agent 回報問題時

Sub-Agent 可能在回報中說明:

```markdown
⚠️ 需注意事項:
- 缺少資料庫選型的偏好資訊
- 未提供非功能需求 (性能、安全性)
- 建議補充用戶認證方式的具體需求
```

**Orchestrator 應該:**

1. 補充缺失的資訊
2. 重新調用 Agent 或調用其他專業 Agent
3. 更新工作流程

### 範例: 處理資訊不足

```javascript
// 第一次調用 - 資訊不足
Task(...)
// 回報: "缺少資料庫選型資訊"

// 第二次調用 - 補充資訊
Task(
  subagent_type: "general-purpose",
  description: "重新設計架構",
  prompt: `
    ${readFile('.claude/templates/sub-agent-runtime-core.md')}
    ${readFile('.claude/agents/architect.md')}

    [當前任務]
    基於補充資訊重新設計架構

    [輸入資料]
    ${previousDesignContent}

    補充資訊:
    - 資料庫: PostgreSQL (團隊熟悉度高)
    - 非功能需求: 支持 1000 concurrent users
    - 認證方式: OAuth 2.0 + JWT
  `
)
```

---

## 性能優化建議

### Token 使用優化

1. **只載入必要的 Agent**
   ```javascript
   // ✅ 好 - 只載入需要的 Agent
   ${readFile('.claude/agents/architect.md')}

   // ❌ 差 - 載入多個不需要的 Agent
   ${readFile('.claude/agents/architect.md')}
   ${readFile('.claude/agents/product-manager.md')}
   ${readFile('.claude/agents/backend-developer.md')}
   ```

2. **使用 runtime-core 統一管理標準**
   - 不要在每個 Agent 定義中重複標準回報格式
   - runtime-core 只有 2.5KB,比重複內容省很多

3. **精簡上下文資訊**
   - 只提供當前任務必需的資訊
   - 避免載入完整的大型文件,可以提供摘要

---

## 維護指南

### 定期檢查

- **每月**: 檢查 runtime-core 是否需要更新
- **每次新增 Agent**: 驗證調用模式是否正確
- **發現問題時**: 更新此模板的最佳實踐

### 版本控制建議

在 main.md / CLAUDE.md 中追蹤使用的 Agent 版本:

```markdown
<!-- Agent Versions -->
<!-- sub-agent-runtime-core: v1.0.0 -->
<!-- architect: v2.1.0 -->
<!-- agent-designer: v1.0.0 -->
```

---

## 總結

**核心原則:**
1. 永遠載入 `sub-agent-runtime-core.md`
2. 提供完整上下文（無記憶假設）
3. 明確的輸出要求
4. 合理的任務範圍（單次可完成）
5. 從 Sub-Agent 回報中學習並調整

**標準流程:**
```
載入 runtime-core → 載入 Agent 定義 → 提供任務與資料 → 獲得標準回報 → 根據建議下一步繼續
```
