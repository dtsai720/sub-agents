# 雲端架構設計檢查清單

此檢查清單供 Cloud Architect Agent 使用，確保設計完整性與品質。

## 使用時機

- **STEP 5 執行前**：產出交付物之前，進行最終自檢
- **需求不完整時**：判斷是否需要觸發 STEP 0 詢問使用者

---

## STEP 0: 需求完整性檢查

### 必要資訊檢查
- [ ] 已明確指定雲端平台（AWS / Azure / GCP）
- [ ] 已定義主要功能與業務目標
- [ ] 已定義資料實體（至少有高層次描述）
- [ ] 已定義目標使用者與規模

### 觸發需求補充的條件（符合任一即觸發）
- [ ] 未明確指定雲端平台
- [ ] 需求過於發散（例如：「設計一個系統」但無具體功能）
- [ ] 缺少關鍵資訊（無資料實體、無使用者規模、無效能需求）
- [ ] 技術選型不明確且可能影響架構決策
- [ ] 安全需求模糊（涉及敏感資料但無認證授權說明）
- [ ] 複雜專案但無優先級或分階段資訊

### 不需要觸發的情況
- [ ] 使用者已提供 PROD.md 且內容完整
- [ ] 使用者已明確提供雲端平台、資料實體、主要功能、非功能性需求
- [ ] 簡單專案且需求清晰（例如：CRUD API + PostgreSQL + AWS）

**決策：**
- 如果符合「觸發條件」→ 執行 STEP 0（讀取 requirement-questions.md，產生問題清單）
- 如果屬於「不需要觸發」→ 直接進入 STEP 1

---

## STEP 1: 需求分析與現狀評估

### 程式碼分析（如適用）
- [ ] 已使用 Glob 工具找出專案結構
- [ ] 已使用 Read 工具讀取依賴檔案（go.mod / pom.xml / requirements.txt）
- [ ] 已使用 Grep 工具搜尋路由定義、資料模型
- [ ] 已繪製反向工程架構圖（Mermaid 格式）
- [ ] 已識別現有技術堆疊

### 需求分析（如有 PROD.md）
- [ ] 已讀取並分析 PROD.md
- [ ] 已提取功能需求清單
- [ ] 已識別資料實體與關聯
- [ ] 已確認非功能性需求

### 輸出檢查
- [ ] 業務需求清單已產出
- [ ] 資料實體定義已產出
- [ ] 非功能性需求已記錄（效能、安全性、擴展性）
- [ ] 現有架構分析已完成（如適用）

---

## STEP 1.5: 專案複雜度評估

### 複雜度指標評估
- [ ] 已計算功能模組數量
- [ ] 已計算微服務/主要元件數量
- [ ] 已計算資料實體數量
- [ ] 已計算外部整合數量
- [ ] 已預估完整設計時間

### 切分決策
- [ ] 如符合切分條件 → 執行 STEP 1.6（專案切分）
- [ ] 如不符合切分條件 → 繼續 STEP 2（完整架構設計）

**切分條件（符合任一即切分）：**
- [ ] 功能模組 > 3 個
- [ ] 微服務數量 > 2 個
- [ ] 資料實體 > 7 個
- [ ] 外部整合 > 2 個
- [ ] 預估設計時間 > 60 分鐘
- [ ] 跨多個業務領域

---

## STEP 2: 雲端架構設計

### 核心原則應用
- [ ] 已說明架構如何支持業務目標（業務價值導向）
- [ ] 已避免過度設計，遵循 KISS、YAGNI（簡單至上）
- [ ] 已說明架構如何支持未來擴展（演進式架構）
- [ ] 已明確說明架構取捨（權衡思維：CAP、效能 vs 成本）

### 雲端平台確認
- [ ] 已確認雲端平台（AWS / Azure / GCP）
- [ ] 如使用者未提供，已詢問使用者選擇
- [ ] 已說明選擇此平台的理由

### 系統邊界定義
- [ ] 已繪製系統上下文圖（Mermaid System Context Diagram）
- [ ] 已標註使用者、系統、外部服務的關係
- [ ] 已標註主要互動方式（HTTP、Message Queue、gRPC）

### 雲端服務選型（MUST 完整）
每個服務選型都必須包含：
- [ ] **Compute Services**（Lambda/ECS/EKS / Cloud Functions/Cloud Run / Azure Functions/AKS）
  - [ ] 已選擇服務
  - [ ] 已說明選擇理由（Serverless vs Container vs VM 的權衡）
  - [ ] 已說明成本考量

- [ ] **Database Services**（RDS/DynamoDB / Cloud SQL/Firestore / Azure SQL/Cosmos DB）
  - [ ] 已選擇服務
  - [ ] 已說明選擇理由（關聯式 vs NoSQL, 資料量級, 查詢模式）
  - [ ] 已說明權衡（一致性 vs 延遲 vs 成本）

- [ ] **Storage Services**（S3/EFS / Cloud Storage / Blob Storage）
  - [ ] 已選擇服務
  - [ ] 已說明選擇理由（物件儲存 vs 檔案系統）
  - [ ] 已提供成本優化策略（Lifecycle Policies）

- [ ] **Messaging Services**（SQS/SNS/EventBridge / Pub/Sub / Service Bus）
  - [ ] 已選擇服務（如適用）
  - [ ] 已說明選擇理由（Queue vs Pub/Sub, 事件驅動需求）

- [ ] **API Gateway & Networking**（API Gateway/ALB/CloudFront/VPC）
  - [ ] 已選擇服務
  - [ ] 已說明選擇理由（API Gateway 功能、CDN 需求、網路隔離）

### 架構圖繪製（MUST use Mermaid）
- [ ] 已繪製系統上下文圖（System Context Diagram）
- [ ] 已繪製雲端服務元件圖（Component Diagram）
- [ ] 已繪製至少 1 個序列圖（Sequence Diagram，展示關鍵流程）
- [ ] 所有圖表使用 Mermaid 格式（❌ 禁止文字描述）

### 安全架構設計
- [ ] 已設計 API Gateway 與認證機制（JWT/OAuth2/API Key）
- [ ] 已設計授權策略（RBAC/ABAC，使用雲端 IAM）
- [ ] 已設計 Rate Limiting & Throttling
- [ ] 已設計網路安全（VPC、Security Groups）
- [ ] 已設計資料加密策略（傳輸加密、儲存加密）

### 可觀測性設計
- [ ] 已選擇雲端監控服務（CloudWatch / Azure Monitor / Cloud Monitoring）
- [ ] 已選擇日誌服務（CloudWatch Logs / Log Analytics / Cloud Logging）
- [ ] 已選擇追蹤服務（X-Ray / Application Insights / Cloud Trace）
- [ ] 已設計告警策略與閾值

### 成本優化策略
- [ ] 已預估每月成本（Compute/Database/Storage/Network）
- [ ] 已提供成本優化建議（Reserved Instances/Spot Instances/Auto Scaling）
- [ ] 已設計資源監控與成本告警

### 高可用性與災難恢復
- [ ] 已設計高可用性策略（Multi-AZ / Multi-Region）
- [ ] 已定義 RTO/RPO（如適用）
- [ ] 已設計備份與恢復策略

---

## STEP 3: 高層次 API 端點定義

### API 端點清單
- [ ] 已定義所有 API 端點（URL + Method）
- [ ] 已提供簡要描述（用途與功能）
- [ ] 已提供基本請求/回應格式（JSON object 結構）
- [ ] 已標註認證需求（Public/JWT/OAuth2/API Key）

### API 分組
- [ ] 已將端點分組（Authentication / User Management / Business Logic / Admin）

### 認證策略
- [ ] 已指定認證方式（JWT/OAuth2/API Key）
- [ ] 已定義權限模型（RBAC/ABAC，角色與權限定義）
- [ ] 已區分 Public endpoints vs Protected endpoints

### 產出檔案
- [ ] 已產出 API_ENDPOINTS.md（高層次端點清單）
- [ ] **❌ 未產出詳細 OpenAPI Schema**（交給 API Designer Agent）

---

## STEP 4: 資料模型複雜度評估

### 資料模型分析
- [ ] 已計算表（Table/Collection）數量
- [ ] 已分析關聯複雜度（1-1, 1-N, N-N）
- [ ] 已分析查詢需求（簡單 CRUD vs 複雜聚合）
- [ ] 已分析效能需求（QPS、資料量）

### 複雜度判斷
- [ ] 已判定複雜度等級（High / Medium / Low）
  - **High**: 表 > 10 OR 多層關聯 OR 複雜查詢 OR 高效能需求
  - **Medium**: 表 5-10 OR 標準關聯 OR 使用 ORM
  - **Low**: 表 < 5 OR 簡單關聯

### 開發順序建議
- [ ] 已根據複雜度建議開發順序
  - **High**: Cloud Architect → [API Designer || DBA Agent] 並行 → Backend → Frontend
  - **Medium**: Cloud Architect → [API Designer || DBA Agent] 並行 → Backend → Frontend
  - **Low**: Cloud Architect → API Designer → Backend → (可選) DBA Agent
- [ ] 已說明選擇此順序的理由

### ER Diagram 產出
- [ ] 已繪製高層次 ER Diagram（Mermaid 格式）
- [ ] 已包含實體名稱與關聯關係（1-1, 1-N, N-N）
- [ ] 已包含關鍵欄位（id, foreign keys）
- [ ] **❌ 未包含詳細欄位型別、索引、約束條件**（交給 DBA Agent）
- [ ] 已產出 ER_DIAGRAM.md

---

## STEP 5: 交付物產出與最終自檢

### 交付文件完整性
- [ ] 已產出 **CLOUD_ARCHITECTURE.md**（13 個章節完整）
  - [ ] 系統概覽與目標
  - [ ] 雲端平台選擇與理由
  - [ ] 雲端服務架構（Compute/Database/Storage/Messaging/API Gateway）
  - [ ] 高層次系統架構圖（Mermaid）
  - [ ] 微服務邊界（如適用）
  - [ ] 高層次資料模型（ER Diagram）
  - [ ] 成本優化策略
  - [ ] 高可用性與災難恢復
  - [ ] 安全架構設計（VPC/IAM/Security Groups）
  - [ ] 非功能性需求規劃
  - [ ] 部署架構
  - [ ] 監控與日誌策略
  - [ ] 演進路徑（如有現有系統）

- [ ] 已產出 **API_ENDPOINTS.md**（高層次端點清單）
  - [ ] API 端點列表
  - [ ] 簡要描述
  - [ ] 基本請求/回應格式
  - [ ] 認證需求
  - [ ] **Note**: 完整 OpenAPI 規格由 API Designer Agent 負責

- [ ] 已產出 **ER_DIAGRAM.md**（高層次資料模型）
  - [ ] 實體關聯圖（Mermaid ER Diagram）
  - [ ] 實體名稱與關聯關係
  - [ ] 關鍵欄位（id, foreign keys）
  - [ ] **Note**: 詳細 Schema 由 DBA Agent 負責

### 雲端服務選型檢查
- [ ] 所有雲端服務選型都有理由說明
- [ ] 所有雲端服務選型都有權衡分析
- [ ] 所有雲端服務選型都有成本考量
- [ ] 所有雲端服務選型考慮長期可持續性（3-5 年維運）

### 架構圖檢查
- [ ] 所有架構圖使用 Mermaid 格式
- [ ] 至少包含 3 個圖表（Context + Component + Sequence）
- [ ] 圖表清晰標註服務名稱與互動關係

### 安全性檢查
- [ ] 認證與授權策略已明確定義
- [ ] 資料加密策略已定義（如涉及敏感資料）
- [ ] 網路安全策略已定義（VPC、Security Groups）

### 非功能性需求檢查
- [ ] 效能需求已處理（QPS、回應時間）
- [ ] 可用性需求已處理（SLA、Multi-AZ）
- [ ] 擴展性需求已處理（Auto Scaling、Load Balancing）

### 成本優化檢查
- [ ] 已提供成本預估（每月約 USD）
- [ ] 已提供成本優化建議（Reserved Instances、Spot Instances）

### 資料模型檢查
- [ ] 資料模型複雜度已評估（High/Medium/Low）
- [ ] 開發順序建議已提供（API Designer + DBA + Backend）
- [ ] 已說明為何推薦此開發順序

### 標準回報格式檢查
- [ ] 已使用標準回報格式（一般模式 OR 切分模式）
- [ ] 已提供品質自檢（✅ 已完成項目、⚠️ 需注意事項）
- [ ] 已提供雲端服務決策說明（遵循設計哲學）
- [ ] 已提供建議下一步（推薦 API Designer/DBA/Backend Agent）

---

## 禁止事項檢查（❌ MUST NOT）

- [ ] ❌ 未追逐技術潮流而忽略業務價值
- [ ] ❌ 未過度設計與過早優化
- [ ] ❌ 未設計詳細 OpenAPI Schema（交給 API Designer Agent）
- [ ] ❌ 未設計詳細資料庫 Schema（交給 DBA Agent）
- [ ] ❌ 未選擇程式語言與框架（交給 Backend Developer）
- [ ] ❌ 未選擇 AWS/Azure/GCP 以外的雲端平台
- [ ] ❌ 未使用非 Mermaid 格式的架構圖
- [ ] ❌ 未假設無限資源或完美網路條件
- [ ] ❌ 未忽略維運需求（日誌、監控、告警）

---

## 最終確認

**在產出交付物之前，再次確認：**

1. [ ] 所有 STEP 都已完成（STEP 0 → 1 → 1.5 → 2 → 3 → 4 → 5）
2. [ ] 雲端平台已確認（AWS / Azure / GCP）
3. [ ] 雲端服務選型已完成（Compute/Database/Storage/Messaging/API Gateway）
4. [ ] 至少 3 個 Mermaid 架構圖已繪製
5. [ ] 資料模型複雜度已評估（High/Medium/Low）
6. [ ] 標準回報格式已使用
7. [ ] 所有交付文件已產出（CLOUD_ARCHITECTURE.md、API_ENDPOINTS.md、ER_DIAGRAM.md）
8. [ ] 已提供建議下一步（推薦 API Designer/DBA/Backend Agent）

**如有任何未完成項目 → MUST 完成後再產出交付物**

---

## 使用範例

### 範例 1：簡單 CRUD API
**STEP 0 檢查結果**：
- ✅ 雲端平台：AWS
- ✅ 主要功能：使用者 CRUD、認證已由上游處理
- ✅ 資料實體：users 表
- ✅ 需求清晰，不需觸發 STEP 0

**STEP 5 最終檢查**：
- ✅ CLOUD_ARCHITECTURE.md 已產出（AWS + Lambda + RDS PostgreSQL）
- ✅ API_ENDPOINTS.md 已產出（2 個端點：GET /users, GET /users/{id}）
- ✅ ER_DIAGRAM.md 已產出（users 實體）
- ✅ 資料模型複雜度：Low
- ✅ 建議順序：Architect → API Designer → Backend

### 範例 2：複雜微服務系統
**STEP 0 檢查結果**：
- ❌ 雲端平台未提供
- ❌ 功能過於發散（「設計一個電商平台」）
- ❌ 無使用者規模、無效能需求
- ⚠️ **觸發 STEP 0**：產生 10 個問題詢問使用者

**等待使用者回答後，再繼續 STEP 1-5**

---

## 總結

使用此檢查清單，確保：
1. **需求完整** - STEP 0 避免資訊不足導致設計偏差
2. **設計完整** - STEP 1-4 涵蓋所有架構面向
3. **品質保證** - STEP 5 自檢確保交付物符合標準
4. **職責清晰** - 明確區分 Architect / API Designer / DBA / Backend 的職責
