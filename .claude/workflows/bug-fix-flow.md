# Bug Fix Flow - 按影響範圍分類的混合方案

**版本**: 1.0
**建立日期**: 2025-10-10
**適用場景**: Code Review 後發現的 bugs、技術債務、效能問題

---

## 流程概述

當 Code Review 或 Codebase Analysis 發現問題時，Orchestrator 根據**影響範圍**選擇不同的處理流程：

```
Code Review 發現問題 → Orchestrator 分類 →
  ├─ Critical/High (影響用戶) → PM 追蹤流程
  ├─ Medium/Low (技術債) → Orchestrator 快速修復流程
  └─ Security (安全漏洞) → Orchestrator 緊急修復流程
```

---

## 分類標準

### Critical/High Priority（影響用戶）→ PM 追蹤

**判斷標準**：
- ✅ 直接影響用戶體驗（UI bug、功能錯誤、數據錯誤）
- ✅ 導致服務不可用（timeout、crash、hang）
- ✅ 法律/合規風險（無障礙、GDPR、隱私）
- ✅ 數據完整性問題（數據遺失、不一致）

**範例**：
- HTTP timeout 導致用戶等待無回應
- UI 狀態管理錯誤導致顯示混亂
- 無障礙違規（WCAG 不符合）
- Goroutine leak 導致應用程式變慢

**處理流程**：
1. Orchestrator 建立 FIXES_TRACKING.md（臨時追蹤）
2. 直接調用 Developer Agents 快速修復
3. **修復完成後**，調用 PM Agent 更新 PROD.md（事後記錄）
4. PM 將技術問題轉換為產品改進描述

---

### Medium/Low Priority（技術債）→ Orchestrator 追蹤

**判斷標準**：
- ✅ 代碼品質問題（code smell、重複代碼）
- ✅ 效能優化（非緊急）
- ✅ 測試覆蓋率改進
- ✅ 文件補充
- ✅ 依賴更新（無安全風險）

**範例**：
- 增加測試覆蓋率從 35% → 70%
- 重構重複代碼
- 優化演算法效率（無用戶抱怨）
- 補充 API 文件

**處理流程**：
1. Orchestrator 建立 FIXES_TRACKING.md
2. 直接調用 Developer Agents 修復
3. **不需要** PM 介入，不更新 PROD.md
4. 在 CHANGELOG.md 記錄技術改進

---

### Security Issues（安全漏洞）→ Orchestrator 緊急流程

**判斷標準**：
- ✅ 安全漏洞（SQL injection、XSS、CSRF）
- ✅ 敏感資料洩漏
- ✅ 權限控制問題
- ✅ 依賴套件安全更新

**範例**：
- SQL injection 漏洞
- API 未驗證權限
- 密碼明文儲存
- 已知 CVE 的依賴套件

**處理流程**：
1. Orchestrator **立即**建立 SECURITY_FIX_TRACKING.md
2. 最高優先級調用 Developer Agents
3. **修復完成後**，調用 PM Agent 評估是否需要通知用戶
4. 在 SECURITY.md 記錄修復

---

## Orchestrator 決策樹

```
[Code Review 發現問題]
         ↓
[讀取問題清單和嚴重性評估]
         ↓
    ┌────┴────┐
    │ 分類問題 │
    └────┬────┘
         ↓
    是否影響用戶？
    ├─ YES → Critical/High Priority
    │         ├─ 建立 FIXES_TRACKING.md
    │         ├─ 調用 Developer Agents（快速修復）
    │         ├─ 調用 Code Reviewer（驗證）
    │         ├─ 調用 QA（測試）
    │         └─ 調用 PM Agent（事後記錄到 PROD.md）
    │
    ├─ NO → 是否為安全漏洞？
    │        ├─ YES → Security Issues
    │        │         ├─ 建立 SECURITY_FIX_TRACKING.md
    │        │         ├─ 最高優先級修復
    │        │         └─ PM 評估用戶通知
    │        │
    │        └─ NO → Medium/Low Priority
    │                  ├─ 建立 FIXES_TRACKING.md
    │                  ├─ 調用 Developer Agents
    │                  └─ 記錄到 CHANGELOG.md
    │
    └─ 完成
```

---

## FIXES_TRACKING.md 格式

Orchestrator 自動建立此文件追蹤修復進度：

```markdown
# Bug Fixes Tracking

**Created**: {YYYY-MM-DD}
**Source**: {CODE_REVIEW_REPORT.md | FRONTEND_CODE_REVIEW_REPORT.md}
**Branch**: {branch_name}
**Priority**: {Critical | High | Medium | Low | Security}

---

## Classification

**Type**: {Critical/High - 影響用戶 | Medium/Low - 技術債 | Security - 安全漏洞}

**PM Tracking Required**: {YES | NO}
- YES: 修復後需更新 PROD.md
- NO: 僅記錄到 CHANGELOG.md

---

## Issues to Fix

### Critical Priority (Must Fix)

1. **[Backend] HTTP Client Timeout Missing**
   - **Severity**: High
   - **Impact**: 服務可能 hang 導致用戶等待
   - **Location**: `internal/api/winfittsclient.go:33`
   - **Estimated Effort**: 2 hours
   - **Assigned Agent**: Backend Developer (Go)
   - **Status**: ⏳ Pending
   - **Fix**:
     ```go
     client.Timeout = 30 * time.Second
     ```

2. **[Backend] Goroutine Leak in Config Watcher**
   - **Severity**: High
   - **Impact**: 資源洩漏，應用程式變慢
   - **Location**: `internal/config/config.go:85`
   - **Estimated Effort**: 2 hours
   - **Assigned Agent**: Backend Developer (Go)
   - **Status**: ⏳ Pending

### High Priority (Should Fix Soon)

3. **[Frontend] Mode State Management Bug**
   - **Severity**: High
   - **Impact**: UI 可能顯示錯誤狀態
   - **Location**: `frontend/src/pages/HomePage.jsx`
   - **Estimated Effort**: 5 hours
   - **Assigned Agent**: Frontend Developer
   - **Status**: ⏳ Pending

4. **[Frontend] Accessibility Violations**
   - **Severity**: Critical
   - **Impact**: 無法符合 WCAG 2.1 Level AA，法律風險
   - **Location**: 全域 (缺少 ARIA 標籤)
   - **Estimated Effort**: 8 hours
   - **Assigned Agent**: Frontend Developer
   - **Status**: ⏳ Pending

---

## Progress Summary

- **Total Issues**: 4
- **Completed**: 0
- **In Progress**: 0
- **Pending**: 4
- **Total Estimated Effort**: 17 hours

---

## Testing Plan

- [ ] Backend: Run `make test` (target: maintain 89.2% coverage)
- [ ] Frontend: Run `npm test` (target: maintain 35% coverage, don't break existing)
- [ ] Integration: Manual testing of sync flow
- [ ] Accessibility: axe DevTools scan

---

## Post-Fix Actions

### If PM Tracking Required (YES):
1. Call PM Agent with context:
   - Input: FIXES_TRACKING.md + implementation details
   - Output: Updated PROD.md section "Recent Improvements"
   - PM converts technical fixes to product language

### If PM Tracking Not Required (NO):
1. Update CHANGELOG.md with technical details
2. No PROD.md update needed

---

## Completion Criteria

- [ ] All issues marked as ✅ Completed
- [ ] All tests passing
- [ ] Code review completed
- [ ] PROD.md updated (if PM tracking required)
- [ ] CHANGELOG.md updated
- [ ] Git branch merged to main
```

---

## PM Agent 事後記錄模式

當 `PM Tracking Required = YES` 時，Orchestrator 調用 PM Agent：

### 輸入 (Prompt)
```markdown
## Task: 將技術修復轉換為產品改進記錄

你是產品經理，需要將以下技術 bug 修復轉換為產品語言，更新 PROD.md。

### 輸入資料
- FIXES_TRACKING.md（技術問題清單）
- Implementation details（修復內容）

### 任務
1. 讀取 FIXES_TRACKING.md 了解修復內容
2. 讀取現有 PROD.md
3. 在 PROD.md 新增 "Recent Improvements" 章節
4. 用產品語言描述改進（不要技術術語）

### 輸出格式
更新後的 PROD.md，新增：

## Recent Improvements

**Version**: {version}
**Release Date**: {date}

### Stability Enhancements
- ✅ 提升應用程式穩定性，解決長時間運行後效能下降問題
- ✅ 改善網路連線逾時處理，減少用戶等待時間

### Accessibility Improvements
- ✅ 新增完整鍵盤導航支援，符合無障礙標準
- ✅ 改善螢幕閱讀器相容性（WCAG 2.1 Level AA）

### UI/UX Refinements
- ✅ 修復狀態顯示問題，提升介面一致性
```

### PM Agent 轉換範例

**技術描述** → **產品描述**

| 技術問題 | 產品改進描述 |
|---------|-------------|
| HTTP client timeout missing | 改善網路連線處理，避免用戶長時間等待 |
| Goroutine leak in config watcher | 提升應用程式穩定性，解決長時間運行效能問題 |
| Mode state boolean bug | 修復介面狀態顯示問題，提升使用體驗一致性 |
| WCAG accessibility violations | 新增完整無障礙支援，符合國際標準（WCAG 2.1 Level AA）|

---

## 完整流程範例

### 場景：Code Review 發現 4 個問題

#### Step 1: Orchestrator 分類

```
讀取 CODE_REVIEW_REPORT.md + FRONTEND_CODE_REVIEW_REPORT.md

分類結果：
- HTTP timeout → Critical (影響用戶)
- Goroutine leak → Critical (影響用戶)
- Mode state bug → High (影響用戶)
- Accessibility → Critical (法律風險 + 影響用戶)

決策：全部為 Critical/High → 使用 PM 追蹤流程
```

#### Step 2: Orchestrator 建立 FIXES_TRACKING.md

```
PM Tracking Required: YES
Total Issues: 4
Estimated Effort: 17 hours
```

#### Step 3: Orchestrator 調用 Developer Agents

```
並行調用：
1. Backend Developer (Go) → 修復 HTTP timeout + Goroutine leak
2. Frontend Developer → 修復 Mode state + Accessibility

等待完成...
```

#### Step 4: Orchestrator 調用 Code Reviewer

```
Backend Code Reviewer → 驗證後端修復
Frontend Code Reviewer → 驗證前端修復
```

#### Step 5: Orchestrator 調用 QA

```
QA Agent → 執行測試計畫
- 後端測試維持 89.2% 覆蓋率
- 前端測試不破壞現有功能
- 手動測試 accessibility
```

#### Step 6: Orchestrator 調用 PM Agent（事後記錄）

```
輸入：FIXES_TRACKING.md + implementation details
輸出：更新 PROD.md - "Recent Improvements" 章節

PM 轉換為產品語言：
- "提升應用程式穩定性"
- "改善網路連線處理"
- "新增完整無障礙支援"
- "修復介面狀態顯示問題"
```

#### Step 7: Git Manager 提交變更

```
Git Manager → 建立 commit + PR
Commit message: "fix: improve stability and accessibility (4 critical fixes)"
```

---

## Orchestrator 自動識別觸發條件

Orchestrator 在以下情況自動啟動 Bug Fix Flow：

1. **用戶明確說明**：
   - "請修復 Code Review 發現的問題"
   - "處理 CODE_REVIEW_REPORT.md 中的 bugs"
   - "修復這些技術債務"

2. **檔案存在觸發**：
   - 偵測到 `CODE_REVIEW_REPORT.md` 存在
   - 偵測到 `FRONTEND_CODE_REVIEW_REPORT.md` 存在
   - 偵測到 `FIXES_TRACKING.md` 存在（恢復中斷的修復）

3. **關鍵字觸發**：
   - "bug fix"、"修復"、"fix issues"
   - "technical debt"、"技術債"
   - "code review findings"、"審查發現"

---

## 與其他流程的整合

### 與 Product Development Flow 的關係

- **Product Development**: 新功能開發（從 PROD.md 開始）
- **Bug Fix Flow**: 現有功能改進（從 Code Review 開始）

**差異**：
- Product Development: PM → Architect → API Designer → Developer
- Bug Fix Flow: Developer → Code Reviewer → QA → PM（事後記錄）

### 與 Codebase Analysis Flow 的關係

- **Codebase Analysis**: 產生 Code Review Reports
- **Bug Fix Flow**: 修復 Code Review 發現的問題

**流程串接**：
```
Codebase Analysis → 產出 Reports →
Orchestrator 識別問題 → 啟動 Bug Fix Flow →
修復完成 → PM 記錄改進
```

---

## 檔案輸出總結

| 優先級 | 追蹤檔案 | PM 記錄 | 技術記錄 |
|-------|---------|---------|---------|
| Critical/High | FIXES_TRACKING.md | PROD.md (事後) | CHANGELOG.md |
| Medium/Low | FIXES_TRACKING.md | 不需要 | CHANGELOG.md |
| Security | SECURITY_FIX_TRACKING.md | PROD.md (視情況) | SECURITY.md |

---

## 流程檢查清單

Orchestrator 執行 Bug Fix Flow 時的檢查清單：

### 啟動階段
- [ ] 讀取 Code Review Reports
- [ ] 分類問題（Critical/Medium/Security）
- [ ] 判斷是否需要 PM 追蹤
- [ ] 建立 FIXES_TRACKING.md

### 執行階段
- [ ] 調用 Backend Developer（如有後端問題）
- [ ] 調用 Frontend Developer（如有前端問題）
- [ ] 調用 Code Reviewer（驗證修復）
- [ ] 調用 QA（測試）

### 完成階段
- [ ] 如需 PM 追蹤 → 調用 PM Agent 更新 PROD.md
- [ ] 如不需 PM 追蹤 → 更新 CHANGELOG.md
- [ ] 調用 Git Manager（提交 + PR）
- [ ] 更新 PROJECT_STATUS.md

---

## 恢復中斷的修復

如果 Bug Fix 過程中斷，恢復時：

```
用戶說：「從上次中斷處繼續」

Orchestrator 動作：
1. 讀取 FIXES_TRACKING.md
2. 檢查各問題的 Status
3. 找出 Status = "In Progress" 或 "Pending" 的項目
4. 繼續調用對應的 Agent
5. 完成後更新 Status
```

---

## 成功標準

Bug Fix Flow 成功完成的標準：

1. ✅ 所有問題 Status = "✅ Completed"
2. ✅ 測試全部通過
3. ✅ Code Review 通過
4. ✅ PROD.md 更新完成（如需要）
5. ✅ Git PR 已建立
6. ✅ PROJECT_STATUS.md 已更新

---

**流程版本**: 1.0
**建立日期**: 2025-10-10
**最後更新**: 2025-10-10
