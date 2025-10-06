# Sub-Agent Design Checklist

## 已完成的 Agent

### ✅ Cloud Architect Agent
- **檔案位置**: `.claude/agents/architect.md`
- **核心職責**: 雲端架構設計、雲端服務選型、高層次系統架構
- **輸入**: PROD.md、現有程式碼、技術需求
- **輸出**: CLOUD_ARCHITECTURE.md、API_ENDPOINTS.md、ER_DIAGRAM.md
- **特色功能**:
  - 專注三大雲端平台 (AWS/Azure/GCP)
  - 雲端服務選型 (Compute/Database/Storage/Messaging)
  - 高可用性與災難恢復架構
  - 成本優化策略
  - 安全架構設計 (VPC/IAM/Security Groups)
  - 遵循架構設計哲學 10 條黃金法則
  - 從程式碼反向工程架構圖
  - 支援專案切分機制 (預估時間 > 60 分鐘自動切分)
  - 支援遞迴切分 (Divide & Conquer，最多 3 層)
  - 使用 Mermaid 繪製架構圖
- **時間**: 35-40 分鐘 ⚡ (優化後)

---

## 已完成的 Agent

### ✅ Product Manager Agent
- **檔案位置**: `.claude/agents/product-manager.md`
- **核心職責**: 需求分析、產品規劃、PRD 撰寫、Figma 設計稿分析
- **輸入**: 產品想法、目標用戶、Figma 設計稿（選填）
- **輸出**: PROD.md（產品需求文件）
- **特色功能**:
  - 支援 Figma 設計稿分析（頁面結構、資料實體、使用者流程）
  - 從 UI 推導功能需求與 API 端點
  - 用戶故事撰寫（標準格式：As a... I want... So that...）
  - MVP 定義與優先級排序（P0/P1/P2）
  - 非功能性需求定義（效能、安全性、可用性）
  - 風險識別與緩解策略
  - 遵循產品管理哲學 10 條原則
- **時間**: 30-40 分鐘

### 🔲 Product Manager Agent (原設計決策記錄)
- **重要設計決策**:

  #### Figma 設計稿處理 (重要！)
  - **問題**: 如果使用者提供 Figma 設計稿，應該由誰處理？
  - **決策**: 採用**方案 A（標準流程）**
    ```
    Figma 設計稿
      ↓
    Product Manager Agent (分析 Figma)
      ├─ 提取功能需求
      ├─ 定義資料模型（使用者、訂單、產品等）
      └─ 產出 PROD.md
      ↓
    Architect Agent (根據 PROD.md 設計架構)
      ├─ 不直接看 Figma
      ├─ 定義前後端技術選型
      └─ 產出 DESIGN.md + OPENAPI.yaml
      ↓
    Frontend Agent (參考 Figma + OPENAPI.yaml)
      └─ 實作 UI 元件
    ```

  - **理由**:
    1. ✅ 職責清晰（PM 負責需求、Architect 負責架構）
    2. ✅ Architect 專注技術決策，不需理解 UI 細節
    3. ✅ 符合實際開發流程

  - **Product Manager Agent 需要的能力**:
    - 分析 Figma 設計稿（頁面、流程、互動）
    - 從 UI 設計推導功能需求
    - 識別資料實體與關聯（例如：使用者、訂單、產品）
    - 定義 User Story 與驗收條件
    - 產出 PROD.md（給 Architect Agent 使用）

  - **輸入格式**:
    - Figma 設計稿 URL 或截圖
    - 使用者想法或業務需求

  - **輸出格式**:
    - PROD.md（產品需求文件）
      - 功能需求清單
      - 資料模型定義
      - 使用者流程
      - 非功能性需求

### ✅ API Designer Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/api-designer.md`
- **核心職責**: OpenAPI 規格設計、Schema 定義、API 文件撰寫
- **輸入**:
  - CLOUD_ARCHITECTURE.md (雲端架構師產出)
  - API_ENDPOINTS.md (高層次端點定義)
  - ER_DIAGRAM.md (資料模型定義)
- **輸出**: openapi.yaml (完整的 OpenAPI 3.x 規格)
- **特色功能**:
  - 符合 OpenAPI 3.x 規範（3.0.3 或 3.1.0）
  - 完整 Schema 定義（request、response、parameters）
  - 驗證規則設計（required、format、pattern、minLength、maxLength、minimum、maximum）
  - 認證與授權定義（JWT/OAuth2/API Key/Custom Headers）
  - 錯誤回應格式標準化（RFC 7807 Problem Details）
  - 每個端點至少 1 個 request/response example
  - 可直接用於 Swagger UI、Postman、OpenAPI Generator
  - 遵循 API 設計哲學 10 條原則
- **時間**: 20-30 分鐘
- **與 Cloud Architect 的關係**:
  - Cloud Architect 提供高層次 API 端點清單（API_ENDPOINTS.md）
  - Cloud Architect 提供高層次資料模型（ER_DIAGRAM.md）
  - API Designer 負責完整的 OpenAPI 規格設計（詳細 Schema、驗證規則、範例）
  - Backend Developer 根據 openapi.yaml 實作 API

### ✅ SQL DBA Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/sql-dba.md`
- **核心職責**: SQL 資料庫 Schema 設計、索引優化、Migration 腳本、雲端 SQL 資料庫優化
- **輸入**:
  - CLOUD_ARCHITECTURE.md (雲端架構師產出，含 SQL 資料庫服務選擇)
  - ER_DIAGRAM.md (高層次資料模型)
  - API_ENDPOINTS.md (了解查詢模式)
- **輸出**:
  - SCHEMA.sql (詳細 SQL Schema 定義)
  - Migration 腳本 (版本化的資料庫變更)
  - QUERY_OPTIMIZATION.md (SQL 查詢優化建議)
  - 雲端 SQL 資料庫特定配置 (RDS parameters, Azure SQL settings)
- **特色功能**:
  - 完整 SQL Schema 設計（欄位型別、約束、預設值、註解）
  - 智能索引策略（單欄位、複合、Partial、Covering、GIN）
  - 外鍵約束設計（ON DELETE/UPDATE 策略）
  - 雲端 SQL 資料庫優化（RDS Parameter Groups、Read Replicas、Aurora Serverless、Azure SQL Elastic Pools）
  - Migration 腳本管理（UP/DOWN、零停機部署）
  - SQL 查詢效能優化（EXPLAIN ANALYZE、Partition、Sharding）
  - 資料完整性保證（CHECK、UNIQUE、NOT NULL、觸發器）
  - 安全與合規（敏感資料加密、審計日誌、GDPR）
  - 遵循 SQL 資料庫設計哲學 10 條原則
- **時間**: 30-40 分鐘
- **支援的雲端 SQL 資料庫**:
  - AWS RDS (PostgreSQL/MySQL/MariaDB)
  - AWS Aurora (PostgreSQL/MySQL 相容)
  - Azure SQL Database
  - Azure Database for PostgreSQL/MySQL
  - Google Cloud SQL (PostgreSQL/MySQL/SQL Server)
  - Google Cloud Spanner

### ✅ NoSQL DBA Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/nosql-dba.md`
- **核心職責**: NoSQL 資料模型設計、索引策略、存取模式優化、雲端 NoSQL 服務優化
- **輸入**:
  - CLOUD_ARCHITECTURE.md (雲端架構師產出，含 NoSQL 資料庫服務選擇)
  - 資料結構範例 (Document 範例、實體定義)
  - API_ENDPOINTS.md (存取模式分析)
- **輸出**:
  - NOSQL_SCHEMA.md (完整 NoSQL Schema - Collection/Table 設計、Partition Key)
  - INDEX_STRATEGY.md (索引策略與查詢優化建議)
  - DATA_MODEL.json (JSON Schema 與資料驗證規則)
  - MIGRATION_GUIDE.md (Migration 腳本與版本管理)
- **特色功能**:
  - Document Schema 設計（Embedding vs Referencing 策略）
  - Partition Key & Sort Key 設計（DynamoDB）
  - 索引策略（Compound Index、Text Index、Geospatial Index、TTL Index）
  - Single-Table Design 模式（DynamoDB）
  - GSI/LSI 設計與優化（DynamoDB）
  - Aggregation Pipeline 優化（MongoDB）
  - 存取模式優先設計（Access Pattern First）
  - 資料驗證規則（JSON Schema、Document Validation）
  - 雲端 NoSQL 服務優化（MongoDB Atlas Cluster Tier、DynamoDB Capacity Mode、Cosmos DB RU 優化）
  - Sharding 策略與 Replica Set 配置
  - 遵循 NoSQL 資料庫設計哲學 10 條原則
- **時間**: 30-40 分鐘
- **支援的雲端 NoSQL 資料庫**:
  - MongoDB Atlas
  - AWS DocumentDB (MongoDB 相容)
  - AWS DynamoDB (Key-Value Store)
  - Azure Cosmos DB (NoSQL API / MongoDB API)

### ✅ DB Ops Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/db-ops.md`
- **核心職責**: 資料庫運維與安全、備份還原、高可用性、災難復原、監控告警、安全加固
- **輸入**:
  - CLOUD_ARCHITECTURE.md (雲端架構、資料庫服務選型)
  - SCHEMA.sql / NOSQL_SCHEMA.md (資料庫結構)
  - 業務要求 (RPO、RTO、可用性目標、合規要求)
- **輸出**:
  - BACKUP_STRATEGY.md (備份策略、還原流程、驗證機制)
  - HA_DR_PLAN.md (高可用架構、災難復原計畫、故障轉移流程)
  - MONITORING_SETUP.md (監控指標、告警規則、Dashboard 配置)
  - SECURITY_HARDENING.md (安全加固、權限管理、合規檢查清單)
  - DB_OPS_RUNBOOK.md (日常運維手冊、故障排除指南)
- **特色功能**:
  - 備份策略設計 (全量、增量、PITR、跨區域備份)
  - 高可用架構 (Multi-AZ、Replica Set、Global Tables)
  - 災難復原計畫 (RPO/RTO、故障轉移、DR 演練)
  - 監控與告警 (CloudWatch、Grafana、PagerDuty)
  - 安全加固 (RBAC、加密、VPC 隔離、審計日誌)
  - 合規檢查 (GDPR、PCI-DSS、SOC2)
  - 成本優化 (Glacier、壓縮、Lifecycle Policy)
  - 自動化腳本 (備份驗證、故障轉移、健康檢查)
- **時間**: 30-40 分鐘
- **調用時機**:
  - Schema 設計完成後
  - 生產部署前
  - 安全審計時
  - 災難復原演練時

### ✅ Backend Developer Agent (Go) ⭐ 已完成
- **檔案位置**: `.claude/agents/backend-developer-go.md`
- **核心職責**: Go 後端 API 實作、測試驅動開發、分層架構設計
- **輸入**:
  - CLOUD_ARCHITECTURE.md (雲端架構、Go 框架選擇、ORM 選擇)
  - OPENAPI.yaml (API 規格)
  - SCHEMA.sql / NOSQL_SCHEMA.md (資料庫 Schema)
- **輸出**:
  - Go Source Code (cmd/, internal/, pkg/, tests/)
  - Tests (*_test.go 單元測試、整合測試)
  - .env.example (環境變數範例)
  - IMPLEMENTATION_PLAN_BACKEND_GO.md (僅 Planning Mode)
- **特色功能**:
  - 兩階段執行模式 (Planning Mode → Development Mode)
  - 分層架構 (Handler → Service → Repository → Model)
  - 支援多種 ORM (GORM/sqlx/ent/sqlc)
  - 測試驅動開發 (gomock/testcontainers-go)
  - 錯誤處理標準化 (RFC 7807 Problem Details)
  - Graceful Shutdown 實作
  - 支援三種專案類型:
    - 新專案 (Full Stack)
    - 增量開發 (Feature Addition)
    - 重構 (Legacy Code Refactoring)
  - Legacy Code Refactoring 支援:
    - 無測試的代碼重構策略
    - 無 DI 到有 DI 的漸進式轉換
    - Seam-Based Refactoring (5 階段)
  - 遵循 Go 開發哲學 10 條原則
  - Migration 由 DBA Agent 專責管理 (嚴格禁止 Backend Developer 執行 Migration)
- **時間**:
  - Planning Mode: 10-15 分鐘
  - Development Mode: 60-120 分鐘 (依複雜度)
- **品質標準**:
  - 測試覆蓋率 > 80%
  - 代碼可編譯 (go build)
  - 無 lint 警告 (golangci-lint)
  - 所有測試通過 (go test ./...)
- **範例檔案**:
  - examples/gorm-example.md (GORM Repository 範例)
  - examples/handler-example.md (Handler & Middleware 範例)
  - examples/service-example.md (Service 層範例)
  - examples/testing-example.md (測試範例)
  - examples/legacy-refactoring-example.md (遺留代碼重構範例)

### ✅ Backend Developer Agent (Java) ⭐ 已完成
- **檔案位置**: `.claude/agents/backend-developer-java.md`
- **核心職責**: Java/Spring Boot 後端 API 實作、測試驅動開發、分層架構設計
- **輸入**:
  - CLOUD_ARCHITECTURE.md (雲端架構、Spring 版本選擇、ORM 選擇)
  - OPENAPI.yaml (API 規格)
  - SCHEMA.sql (資料庫 Schema)
- **輸出**:
  - Java Source Code (Controller, Service, Repository, Entity)
  - Tests (JUnit 5 單元測試、整合測試)
  - application.yml (Spring Boot 設定檔)
- **特色功能**:
  - 兩階段執行模式 (Planning Mode → Development Mode)
  - 分層架構 (Controller → Service → Repository → Entity)
  - Constructor Injection（避免 Field Injection）
  - Bean Validation（@Valid, @NotNull, @Size）
  - @RestControllerAdvice 統一錯誤處理
  - JUnit 5 + Mockito + Testcontainers 測試
  - Spring Boot Actuator 健康檢查
  - Migration 由 DBA Agent 專責管理（禁止使用 Hibernate auto-update）
- **支援 ORM**: Spring Data JPA / MyBatis / jOOQ
- **時間**:
  - Planning Mode: 10-15 分鐘
  - Development Mode: 60-120 分鐘 (依複雜度)
- **品質標準**:
  - 測試覆蓋率 > 80%
  - 代碼可編譯 (mvn compile 或 gradle build)
  - 無 lint 警告
  - 所有測試通過

### ✅ Backend Developer Agent (Python) ⭐ 已完成
- **檔案位置**: `.claude/agents/backend-developer-python.md`
- **核心職責**: Python/FastAPI 後端 API 實作、Python Script 撰寫、測試驅動開發
- **支援任務類型**:
  1. **API Development** (兩階段：Planning → Development)
  2. **Script Development** (單階段：直接開發)
- **輸入 (API Development)**:
  - CLOUD_ARCHITECTURE.md (雲端架構、Python 版本、ORM 選擇)
  - OPENAPI.yaml (API 規格)
  - SCHEMA.sql (資料庫 Schema)
- **輸出 (API Development)**:
  - Python Source Code (Router, Service, Repository, Model, Schema)
  - Tests (pytest 單元測試、整合測試)
  - .env.example (環境變數範例)
- **輸出 (Script Development)**:
  - Python Script (使用 uv 管理、async/await、typer CLI)
  - Tests (pytest，可重用 Script 必須，一次性 Script 可選)
  - README.md (使用說明)
- **特色功能**:
  - 兩階段執行模式 (Planning Mode → Development Mode，僅 API Development)
  - 分層架構 (Router → Service → Repository → Model)
  - **強制使用 Type Hints** (所有函數參數與回傳值，包含 `-> None`)
  - **禁止使用 dict** 傳遞結構化資料（必須使用 dataclass/Pydantic）
  - async/await 強制使用（所有 I/O 操作）
  - Pydantic 資料驗證（FastAPI 內建）
  - pytest + pytest-asyncio + Testcontainers 測試
  - Script Development 使用 uv 管理依賴（取代 pip/poetry）
  - Migration 由 DBA Agent 專責管理（禁止使用 SQLAlchemy metadata.create_all）
- **Script 測試規則**:
  - 可重用 Script（資料處理工具、API client）→ **MUST 撰寫測試**（覆蓋率 > 80%）
  - 一次性 Script（generate mock data、migration）→ **可選撰寫測試**
- **支援 ORM**: SQLAlchemy / Tortoise ORM / Piccolo
- **時間**:
  - API Development - Planning Mode: 10-15 分鐘
  - API Development - Development Mode: 60-120 分鐘
  - Script Development: 30-60 分鐘
- **品質標準**:
  - 所有函數使用 Type Hints
  - 禁止使用 dict 傳遞結構化資料
  - 所有 I/O 使用 async/await
  - 測試覆蓋率 > 80%（可重用 Script）
  - 代碼通過 black + ruff 檢查
  - 所有測試通過（pytest）

### ✅ Frontend Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/frontend-developer.md`
- **核心職責**: 前端 UI 實作、元件驅動開發、Accessibility
- **輸入**:
  - CLOUD_ARCHITECTURE.md (前端技術選型)
  - OPENAPI.yaml (API 規格)
  - PROD.md (功能需求)
  - Figma 設計稿 (UI 參考，選填)
- **輸出**:
  - Frontend Source Code (src/, components/, pages/, hooks/, stores/)
  - Tests (單元測試、整合測試，覆蓋率 > 80%)
  - .env.example (環境變數範例)
  - CHANGE_SUMMARY.md (變更追蹤)
- **特色功能**:
  - 支援多框架 (React 18+, Vue 3+, Angular 15+)
  - 兩階段執行模式 (Planning Mode → Development Mode)
  - TypeScript strict mode 強制
  - Accessibility (WCAG 2.1 AA) 標準
  - Container/Presentational pattern
  - Component-driven development
  - 狀態管理 (Zustand/Pinia/Services+RxJS)
  - API 整合 (基於 OPENAPI.yaml)
  - 測試覆蓋率 > 80%
  - 遵循 development-guide.md 原則
- **支援框架**:
  - React 18+ (Hooks, Context API, Suspense)
  - Vue 3+ (Composition API, Pinia)
  - Angular 15+ (Standalone Components, Signals)
- **Build Tools**: Vite (React/Vue), Angular CLI (Angular)
- **時間**:
  - Planning Mode: 10-15 分鐘
  - Development Mode: 60-120 分鐘 (依複雜度)
- **品質標準**:
  - 測試覆蓋率 > 80%
  - TypeScript strict mode 無錯誤
  - ESLint 無警告
  - Accessibility score > 90 (Lighthouse)
  - Build 成功 (npm run build)
- **範例檔案**:
  - examples/react-component-example.md (React 元件範例)
  - examples/state-management-example.md (狀態管理範例)
- **模板檔案**:
  - templates/IMPLEMENTATION_PLAN.md (實作計畫模板)

### 🔲 UI/UX Agent
- **核心職責**: 使用者體驗設計、Wireframe、視覺設計
- **輸入**: PROD.md、使用者需求
- **輸出**: Wireframe、User Flow、UI 設計稿

### 🔲 DevOps Agent
- **核心職責**: CI/CD、容器化、雲端部署、基礎設施即代碼
- **輸入**:
  - CLOUD_ARCHITECTURE.md (雲端架構與部署策略)
  - 程式碼
- **輸出**:
  - Dockerfile
  - Kubernetes/ECS YAML
  - CI/CD Pipeline (GitHub Actions/GitLab CI)
  - Terraform/CloudFormation (基礎設施即代碼)

### ✅ Backend Code Reviewer Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/backend-code-reviewer.md`
- **核心職責**: 後端代碼品質審查、安全漏洞檢測、最佳實踐驗證、測試覆蓋率評估
- **專業領域**: Go、Java、Python 後端應用程式（不涵蓋前端代碼審查）
- **輸入**:
  - CHANGE_SUMMARY.md (Backend Developer 產出的變更追蹤)
  - IMPLEMENTATION_PLAN_BACKEND_{LANGUAGE}.md (實作計畫)
  - OPENAPI.yaml (API 規格)
  - SCHEMA.sql / NOSQL_SCHEMA.md (資料庫 Schema)
  - CLOUD_ARCHITECTURE.md (架構設計，選填)
- **輸出**:
  - CODE_REVIEW_REPORT.md (完整審查報告)
    - Issues Summary (Critical/Major/Minor)
    - Detailed Analysis (Code Quality/Security/Performance/Testing)
    - API Compliance Report
    - Test Coverage Report
    - Recommendations (可執行的改進建議)
- **特色功能**:
  - 三級問題分類 (Critical/Major/Minor)
  - 安全漏洞檢測 (OWASP Top 10)
  - 效能問題識別 (N+1 查詢、索引缺失)
  - 測試覆蓋率分析 (目標 > 80%)
  - 語言特定審查 (Go/Java/Python 慣例)
  - API 規格一致性驗證
  - 資料庫存取審查 (ORM 使用、交易管理)
  - 所有問題提供修復建議與程式碼範例
- **調用時機**: Backend Developer Agent 完成後（自動觸發）
- **時間**: 15-90 分鐘 (依變更規模)
- **與 Backend Developer 的整合**:
  - Backend Developer 產出 CHANGE_SUMMARY.md（變更追蹤文件）
  - CHANGE_SUMMARY.md 包含：新增/修改/刪除檔案清單、技術決策、安全考量、效能考量、測試覆蓋率、已知問題
  - Backend Code Reviewer 根據 CHANGE_SUMMARY.md 進行審查
  - 若發現 Critical Issues → 回到 Backend Developer 修復
  - 若代碼品質良好 → 進入 QA Agent 測試
- **未來擴展**:
  - DevOps Code Reviewer Agent（審查 Terraform/Kubernetes 代碼）

### ✅ Frontend Code Reviewer Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/frontend-code-reviewer.md`
- **核心職責**: 前端代碼品質審查、安全漏洞檢測、Accessibility 審查、效能優化建議
- **專業領域**: React、Vue、Angular 前端應用程式（不涵蓋後端代碼審查）
- **輸入**:
  - CHANGE_SUMMARY.md (Frontend Developer 產出的變更追蹤)
  - IMPLEMENTATION_PLAN_FRONTEND.md (實作計畫)
  - OPENAPI.yaml (API 規格)
  - CLOUD_ARCHITECTURE.md (架構設計，選填)
  - Figma 設計稿 (選填)
- **輸出**:
  - CODE_REVIEW_REPORT_FRONTEND.md (完整審查報告)
    - Issues Summary (Critical/Major/Minor)
    - Detailed Analysis (Code Quality/Security/Accessibility/Performance/Testing/TypeScript/Architecture)
    - Strengths & Areas for Improvement
    - Action Items (按優先級)
    - Recommendations (可執行的改進建議)
- **特色功能**:
  - 三級問題分類 (Critical/Major/Minor)
  - 安全漏洞檢測 (XSS, 敏感資料暴露, 不安全套件)
  - Accessibility 審查 (WCAG 2.1 AA 標準)
  - 效能問題識別 (Bundle size, 渲染效能, 記憶體洩漏)
  - TypeScript 類型安全審查
  - Framework 特定審查 (React Hooks, Vue Composition API, Angular 慣例)
  - 測試覆蓋率分析 (目標 > 80%)
  - 所有問題提供修復建議與程式碼範例
  - 使用自動化工具驗證 (Lighthouse, ESLint, jest-axe)
- **支援框架**:
  - React 18+ (Hooks, Functional Components)
  - Vue 3+ (Composition API, Pinia)
  - Angular 15+ (Standalone Components, Signals)
- **審查面向** (7 個):
  1. Code Quality (代碼品質)
  2. Security (安全性)
  3. Accessibility (無障礙)
  4. Performance (效能)
  5. Testing (測試)
  6. TypeScript Type Safety (型別安全)
  7. Architecture & Design (架構與設計)
- **調用時機**: Frontend Developer Agent 完成後（自動觸發）
- **時間**: 15-90 分鐘 (依變更規模)
- **與 Frontend Developer 的整合**:
  - Frontend Developer 產出 CHANGE_SUMMARY.md（變更追蹤文件）
  - CHANGE_SUMMARY.md 包含：新增/修改/刪除檔案清單、技術決策、安全考量、效能考量、測試覆蓋率、已知問題
  - Frontend Code Reviewer 根據 CHANGE_SUMMARY.md 進行審查
  - 若發現 Critical Issues → 回到 Frontend Developer 修復
  - 若代碼品質良好 → 進入 QA Agent 測試

### ✅ QA Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/qa.md`
- **核心職責**: 測試策略設計、API 整合測試、E2E 測試、效能測試、安全測試、Bug 追蹤
- **專業領域**: 整合測試 (Integration Testing)、端到端測試 (E2E Testing)、非功能性測試 (Performance, Security)
- **輸入**:
  - PROD.md (功能需求、User Stories、驗收條件)
  - OPENAPI.yaml (API 規格)
  - CODE_REVIEW_REPORT.md (後端代碼審查報告)
  - CODE_REVIEW_REPORT_FRONTEND.md (前端代碼審查報告)
  - 應用程式存取 (Backend API endpoint, Frontend URL)
  - CLOUD_ARCHITECTURE.md (架構設計,選填)
  - SCHEMA.sql / NOSQL_SCHEMA.md (資料庫 Schema,選填)
- **輸出**:
  - QA_TEST_REPORT.md (完整測試報告)
    - Executive Summary (測試摘要、通過率、Bug 數量)
    - Test Coverage (測試覆蓋率)
    - Test Results (API/Frontend/E2E/Performance/Security 測試結果)
    - Bugs Found (按 Severity & Priority 分類)
    - Recommendations (改進建議)
  - BUG_REPORT.md (詳細 Bug 清單)
  - tests/ (測試腳本與測試資料)
- **特色功能**:
  - 測試金字塔策略 (專注於 Integration 30% + E2E 10%)
  - API 整合測試 (基於 OPENAPI.yaml,涵蓋正向/負向/認證/驗證/業務邏輯)
  - 前端整合測試 (元件整合、表單測試、導航測試、狀態管理)
  - E2E 測試 (關鍵使用者流程,如註冊登入、核心業務流程)
  - 效能測試 (API Response Time, Frontend Load Time, Lighthouse Score)
  - 安全測試 (SQL Injection, XSS, CSRF, 認證授權)
  - 相容性測試 (瀏覽器相容性、響應式設計)
  - Bug 追蹤與分類 (Severity: Critical/Major/Minor, Priority: P0/P1/P2/P3)
  - 回歸測試 (若為增量開發)
  - 需求符合度驗證 (基於 PROD.md User Stories)
- **測試工具建議**:
  - API 測試: Postman/Newman, Pytest+requests, Jest+supertest, RestAssured
  - E2E 測試: Cypress (推薦), Playwright, Selenium
  - 效能測試: k6, Apache JMeter, Lighthouse
  - 安全測試: OWASP ZAP, Burp Suite
  - 相容性測試: BrowserStack, LambdaTest
- **測試流程** (8 步驟):
  1. 測試計畫設計 (識別測試範圍、設計測試案例、優先級排序)
  2. API 整合測試 (基於 OPENAPI.yaml)
  3. 前端整合測試 (元件整合、表單、導航、狀態管理)
  4. E2E 測試 (關鍵使用者流程)
  5. 非功能性測試 (效能、安全、相容性、可用性)
  6. 回歸測試 (若為增量開發)
  7. Bug 追蹤與分類 (Severity & Priority)
  8. 測試報告產出
- **調用時機**: 開發與代碼審查完成後
- **時間**: 60-120 分鐘 (依測試範圍)
- **與其他 Agent 的整合**:
  - 輸入: Developer Agents 產出的代碼 + Code Reviewer Agents 產出的審查報告
  - 若發現 Critical/Major Bugs → 回到 Developer Agent 修復
  - 若測試通過 → 進入 DevOps Agent 部署
- **品質標準**:
  - 測試覆蓋率: 100% User Stories, 100% API endpoints
  - 通過率目標: > 95%
  - Bug 分類: Critical/Major/Minor 清楚分類
  - Bug 描述: 可重現 (Steps to Reproduce)
- **不涵蓋範圍**:
  - 單元測試 (由 Developer Agents 負責)
  - 代碼審查 (由 Code Reviewer Agents 負責)
  - 視覺設計驗收 (由 UI/UX Reviewer 負責)
  - 基礎設施測試 (由 DevOps Agent 負責)

### ✅ DevOps Agent ⭐ 已完成
- **檔案位置**: `.claude/agents/devops.md`
- **核心職責**: CI/CD 自動化、容器化、容器編排、基礎設施即代碼、雲端部署、監控告警
- **專業領域**: Docker、Kubernetes、ECS/Fargate、GitHub Actions、GitLab CI、Terraform、CloudWatch、Prometheus
- **輸入**:
  - CLOUD_ARCHITECTURE.md (雲端平台、服務選型、網路架構)
  - 應用程式代碼 (Backend/Frontend)
  - SCHEMA.sql / NOSQL_SCHEMA.md (資料庫 Schema)
  - QA_TEST_REPORT.md (測試已通過)
  - 部署目標環境 (Development/Staging/Production)
- **輸出**:
  - Dockerfile (Backend + Frontend multi-stage builds)
  - docker-compose.yml (本地開發環境)
  - kubernetes/*.yaml (Deployment, Service, Ingress, HPA, PDB, ConfigMap, Secret)
  - ecs/*.json (Task Definition, Service Definition for AWS ECS/Fargate)
  - .github/workflows/*.yml 或 .gitlab-ci.yml (CI/CD Pipeline)
  - terraform/ (Infrastructure as Code - VPC, ECS, RDS, ElastiCache, ALB modules)
  - DEPLOYMENT.md (部署指南、Rollback 流程、故障排除)
  - MONITORING_SETUP.md (監控配置、告警規則、Runbook)
  - docs/DEPLOYMENT_CHECKLIST.md (部署驗證清單)
- **特色功能**:
  - 容器化最佳實踐 (Multi-stage builds, non-root user, security scanning)
  - 多平台支援 (Kubernetes / AWS ECS / Azure Container Apps / Google Cloud Run)
  - CI/CD Pipeline (GitHub Actions + GitLab CI 雙支援)
  - Infrastructure as Code (Terraform 模組化設計)
  - 滾動更新策略 (Zero-downtime deployment)
  - 自動擴展配置 (HPA for K8s / Auto Scaling for ECS)
  - Health Check 與 Graceful Shutdown
  - 監控告警 (CloudWatch Alarms + Prometheus + Grafana)
  - 日誌聚合 (CloudWatch Logs / ELK Stack / Fluent Bit)
  - 秘密管理 (AWS Secrets Manager / Azure Key Vault / Kubernetes Secrets)
  - 安全掃描 (Trivy for containers, Dependabot for dependencies)
  - 12-Factor App 原則實踐
- **容器編排方案**:
  - Kubernetes: Deployment, Service, Ingress, HPA, PDB, ConfigMap, Secret
  - AWS ECS/Fargate: Task Definition, Service, Auto Scaling, ALB integration
  - 資源配置: CPU/Memory requests & limits
  - 探針配置: Liveness & Readiness probes
- **CI/CD 流程** (7 stages):
  1. Test (單元測試、程式碼覆蓋率)
  2. Lint (代碼風格檢查)
  3. Security Scan (Trivy vulnerability scanning)
  4. Build (Docker image build)
  5. Push (推送至 Container Registry)
  6. Deploy Staging (自動部署至 Staging)
  7. Deploy Production (手動審批後部署至 Production)
- **監控指標**:
  - Application: Request rate, error rate, latency (P50/P95/P99), active connections
  - Infrastructure: CPU, memory, network I/O, disk I/O
  - Database: Connections, query latency, throughput
  - Business: User registrations, active users, API usage
- **告警策略**:
  - P0 (Critical): Service down, high error rate (>5%), database connection failed
  - P1 (Warning): High CPU (>80%), high memory (>85%), elevated latency (>1s)
  - P2 (Info): Deployment notifications, scaling events
- **調用時機**: QA 測試通過後，準備部署至雲端環境
- **時間**: 30-60 分鐘 (依環境複雜度)
- **與其他 Agent 的整合**:
  - 輸入: QA Agent 產出的測試報告 (必須通過率 > 95%)
  - 輸入: Cloud Architect Agent 產出的架構設計
  - 輸出: 可運行的生產環境
  - 若部署失敗 → Rollback 至上一個穩定版本
- **品質標準**:
  - Health Check 必須通過
  - Zero-downtime deployment
  - Rollback time < 5 minutes
  - 所有 Critical alerts 已配置
  - Container security scan 通過 (無 Critical/High vulnerabilities)
- **不涵蓋範圍**:
  - 應用程式開發 (由 Developer Agents 負責)
  - 代碼審查 (由 Code Reviewer Agents 負責)
  - 測試執行 (由 QA Agent 負責)
  - 資料庫 Schema 設計 (由 DBA Agents 負責)
  - 資料庫運維 (由 DB Ops Agent 負責 - 備份、HA、DR)

---

## 工作流程定位

### 標準全端開發流程（優化版 - 含雲端架構）

```
1. Figma 設計稿 + 使用者想法
     ↓
2. Product Manager Agent (30-40 分鐘)
     ├─ 分析 Figma
     ├─ 提取功能需求
     └─ 產出 PROD.md
     ↓
3. Cloud Architect Agent (35-40 分鐘)
     ├─ 根據 PROD.md 設計雲端架構
     ├─ 選擇雲端平台與服務 (AWS/Azure/GCP)
     ├─ 評估資料庫選型 (SQL vs NoSQL)
     └─ 產出 CLOUD_ARCHITECTURE.md + API_ENDPOINTS.md + ER_DIAGRAM.md
     ↓
4. API 與資料庫設計階段（可並行）
     ├─ API Designer Agent (20-30 分鐘)
     │   └─ 產出 OPENAPI.yaml
     │
     ├─ SQL DBA Agent (30-40 分鐘，若選擇 SQL 資料庫)
     │   ├─ 設計詳細 SQL Schema
     │   ├─ 索引優化
     │   └─ 產出 SCHEMA.sql + Migration 腳本
     │
     └─ NoSQL DBA Agent (30-40 分鐘，若選擇 NoSQL 資料庫)
         ├─ 設計 Document/Table Schema
         ├─ Partition Key & 索引策略
         └─ 產出 NOSQL_SCHEMA.md + INDEX_STRATEGY.md
     ↓
5. 開發階段（可並行）
     ├─ Backend Developer Agent
     │   ├─ 實作 API
     │   └─ 單元測試
     └─ Frontend Agent
         ├─ 參考 Figma + OPENAPI.yaml
         └─ 實作 UI
     ↓
6. QA Agent
     ├─ 測試 API
     └─ 整合測試
     ↓
7. DevOps Agent
     ├─ 基於 CLOUD_ARCHITECTURE.md
     └─ 部署上線（雲端基礎設施）

總時間（最佳情況 - 並行）：
  PM (40) + Cloud Architect (40) + max(API Designer (30), SQL/NoSQL DBA (40)) + Backend (60) + QA (30) + DevOps (30)
  = 40 + 40 + 40 + 60 + 30 + 30 = 240 分鐘

總時間（串行）：
  40 + 40 + 30 + 40 + 60 + 30 + 30 = 270 分鐘

注意：SQL DBA 與 NoSQL DBA 為互斥選擇（根據 Cloud Architect 的資料庫選型決定）
```

---

## 設計原則

1. **單一職責**: 每個 Agent 專注一個領域
2. **標準化溝通**: 使用統一的輸入/輸出格式（PROD.md、CLOUD_ARCHITECTURE.md、OPENAPI.yaml）
3. **職責劃分清晰**:
   - PM: 需求分析（包含 Figma 分析）
   - Cloud Architect: 雲端架構設計（雲端服務選型、資料庫選型、高層次系統設計）
   - API Designer: API 規格設計（OpenAPI 詳細規格）
   - SQL DBA: SQL 資料庫 Schema 設計（關聯式資料庫表結構、索引優化、Migration）
   - NoSQL DBA: NoSQL 資料模型設計（Document/Table 設計、Partition Key、索引策略）
   - Developer: 程式實作
   - QA: 品質保證
4. **避免重複**:
   - Cloud Architect 不處理 UI 細節、不設計詳細 API Schema、不設計詳細 DB Schema（僅做資料庫選型）
   - API Designer 不做架構決策，專注 OpenAPI 規格
   - SQL DBA 不做架構決策，專注 SQL 資料庫設計與優化（不涉及 NoSQL）
   - NoSQL DBA 不做架構決策，專注 NoSQL 資料庫設計與優化（不涉及 SQL）
   - Frontend 不做架構決策
   - Backend 不做 UI 設計
5. **SQL vs NoSQL 選擇**:
   - Cloud Architect 根據資料特性選擇資料庫類型（強關聯 → SQL、高擴展性 → NoSQL）
   - SQL DBA 與 NoSQL DBA 為互斥選擇（單一專案通常僅需其中一種）
   - 混合架構場景（SQL + NoSQL）需分別呼叫兩個 Agent

---

## 下一步行動

### 優先級 1 (高)
- [x] ✅ 修改 architect.md → 重新定位為 Cloud Architect（已完成）
- [x] ✅ 設計 Product Manager Agent（已完成，含 Figma 分析）
- [x] ✅ 設計 API Designer Agent（已完成）
- [x] ✅ 設計 SQL DBA Agent（已完成 - 從 dba.md 重構）
- [x] ✅ 設計 NoSQL DBA Agent（已完成 - 新建）
- [x] ✅ 設計 DB Ops Agent（已完成 - 運維與安全）

### 優先級 2 (中)
- [x] ✅ 設計 Backend Developer Agent (Go)（已完成）
- [x] ✅ 設計 Backend Developer Agent (Java)（已完成）
- [x] ✅ 設計 Backend Developer Agent (Python)（已完成）
- [x] ✅ 設計 Frontend Developer Agent（已完成）
- [x] ✅ 設計 Frontend Code Reviewer Agent（已完成）

### 優先級 3 (低)
- [x] ✅ 設計 QA Agent（已完成）
- [x] ✅ 設計 DevOps Agent（已完成）
- [ ] 設計 UI/UX Agent
