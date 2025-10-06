# 完整執行範例

## 場景 1：用戶想開發線上訂餐系統

### 1. 用戶輸入
```
「我想開發一個線上訂餐系統」
```

### 2. Orchestrator 分析
- 類型：產品開發流程
- 需求完整度：概念階段，需要深化
- 決策：從產品經理 Agent 開始

### 3. Orchestrator 執行
- Read .claude/agents/產品經理.md
- 組合 prompt（角色 + 用戶需求 + 輸出要求）
- Task tool 調用產品經理 Agent

### 4. 產品經理 Agent 回報
- 交付：docs/PROD.md
- 建議：下一步調用 UI/UX 設計師 Agent

### 5. Orchestrator 驗證並繼續
- 驗證 PROD.md 品質
- Read .claude/agents/UI-UX設計師.md
- 組合 prompt（角色 + PROD.md + 輸出要求）
- Task tool 調用 UI/UX 設計師 Agent

### 6. 依此類推，直到完整交付

---

## 場景 2：PM 提出需求，系統發現已有類似功能

### 1. 用戶輸入（PM）
```
「我們需要一個可以關閉 issue 並通知團隊的功能」
```

### 2. Orchestrator 分析
- 類型：產品開發流程
- 決策：從產品經理 Agent 開始

### 3. Orchestrator 調用產品經理 Agent
- 輸入：用戶需求
- 輸出：docs/PROD.md
- 內容包含：關閉 issue + 通知團隊成員的詳細需求

### 4. Orchestrator 執行功能重複檢查
- Read .claude/agents/後端開發-go.md
- 組合 prompt：
  * 角色定義 + 功能重複檢查任務
  * 輸入：PROD.md 的功能描述
  * 要求：搜尋類似功能
- Task tool 調用後端開發 Agent

### 5. 後端開發 Agent 回報
```
## 現有功能分析報告

**找到的類似功能：**

1. close issue API (src/api/issues.go:45)
   - 相似度：80%（高）
   - 差異：缺少通知功能

2. notification service (src/services/notification.go:20)
   - 相似度：50%（中）
   - 差異：需要整合到 issue 流程

**建議：** 增強現有 close issue API
```

### 6. Orchestrator 向用戶確認
```
「我在系統中找到以下類似功能：

📍 close issue API - 相似度 80%
差異：缺少通知團隊成員功能

💡 建議：增強現有 API（預估 2-3 小時）
   vs 全新開發（預估 1-2 天）

請選擇：
1. [ ] 增強現有 API（推薦）
2. [ ] 建立新 API
3. [ ] 直接使用現有功能
」
```

### 7. 用戶選擇
選項 1（增強現有 API）

### 8. Orchestrator 轉入現有專案增強流程
- 跳過 UI/UX 設計（已有基礎）
- 調用架構師 Agent：
  * 輸入：PROD.md + 現有實作分析
  * 任務：設計如何整合 notification service
  * 輸出：DESIGN.md
- 調用後端開發 Agent：
  * 輸入：DESIGN.md + 現有代碼分析
  * 任務：修改 close issue API，整合通知功能
- 調用 QA Agent（迴歸測試）
- 交付

### 關鍵改進
- ✅ 自動發現重複功能（PM 可能不知道已有 close issue API）
- ✅ 提供透明的分析和建議
- ✅ 節省開發時間（2-3 小時 vs 1-2 天）
- ✅ 避免技術債和重複代碼
