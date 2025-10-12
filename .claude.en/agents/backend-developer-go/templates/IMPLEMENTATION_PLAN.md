# Implementation Plan - Backend Developer (Go)

**Project**: [專案名稱]
**Generated**: [日期時間]
**Complexity**: [Simple/Medium/Complex]
**Total Estimated Time**: [X] minutes

---

## 📊 專案概述

### 輸入分析
- **API 端點數量**: [N] 個
- **資料表數量**: [N] 個
- **業務邏輯複雜度**: [Simple/Medium/Complex]
- **外部整合**: [N] 個

### 技術堆疊
- **Go 版本**: 1.21+
- **Web 框架**: [Gin/Echo/Fiber]
- **資料庫存取工具**: [GORM/sqlx/ent/sqlc]
- **資料庫**: [PostgreSQL/MySQL/SQLite]
- **測試**: testify, uber-go/mock, testcontainers-go
- **日誌**: slog (Go 1.21+)
- **驗證**: go-playground/validator

---

## 🎯 開發階段

### Stage 1: 專案結構建立與基礎設定

**Goal**: 建立標準 Go 專案結構、初始化依賴、實作應用程式骨架

**Tasks**:
1. 建立專案目錄結構（cmd/, internal/, pkg/, tests/）
2. 初始化 go.mod 與依賴套件
3. 建立 main.go 骨架（DB 連線、路由、Graceful Shutdown）
4. 建立 Makefile（build, run, test, migrate）
5. 建立 .env.example（環境變數範例）
6. 建立 .gitignore

**Files to Create**:
```
project/
├── cmd/api/main.go
├── internal/
│   ├── config/config.go
│   ├── errors/errors.go
│   └── middleware/
│       ├── logger.go
│       └── error_handler.go
├── go.mod
├── go.sum
├── Makefile
├── .env.example
├── .gitignore
└── README.md
```

**Tests**:
- N/A（基礎設定階段）

**Success Criteria**:
- [ ] 專案目錄結構建立完成
- [ ] go.mod 正確配置所有依賴
- [ ] main.go 可編譯（go build）
- [ ] Makefile 指令可執行
- [ ] .env.example 包含所有必要環境變數

**Estimated Time**: 15 minutes

**Status**: Not Started

---

### Stage 2: 資料模型實作（Model Layer）

**Goal**: 根據 SCHEMA.sql 建立 GORM Models、定義關聯關係

**Tasks**:
1. 建立所有資料表對應的 Model struct
2. 定義 GORM tags（主鍵、外鍵、索引、約束）
3. 定義 JSON tags（API 序列化）
4. 定義驗證 tags（binding、validate）
5. 實作關聯關係（Has One、Has Many、Belongs To、Many to Many）
6. 實作 TableName() 方法
7. 實作 Hooks（BeforeCreate、BeforeUpdate）

**Files to Create**:
```
internal/model/
├── user.go
├── order.go
├── product.go
└── [其他資料表].go
```

**Tests**:
- N/A（Model 通常在 Repository 層測試）

**Success Criteria**:
- [ ] 所有資料表都有對應的 Model
- [ ] GORM tags 完整（主鍵、外鍵、索引）
- [ ] 關聯關係正確定義
- [ ] TableName() 方法已實作
- [ ] Hooks 已實作（如需要）

**Estimated Time**: 20 minutes

**Status**: Not Started

---

### Stage 3: Repository 層實作（Data Access Layer）

**Goal**: 實作資料存取層、定義 Repository 介面與實作

**Tasks**:
1. 定義 Repository 介面（CRUD + 業務查詢方法）
2. 實作 Repository 結構（依賴注入 DB、Logger）
3. 實作 CRUD 方法（Create、GetByID、List、Update、Delete）
4. 實作業務查詢方法（如 GetByEmail、ListActive）
5. 實作錯誤處理（區分 Not Found / Database Error）
6. 實作查詢優化（Preload、分頁、軟刪除）
7. 撰寫 Repository 單元測試（使用 testcontainers）

**Files to Create**:
```
internal/repository/
├── user_repository.go
├── user_repository_test.go
├── order_repository.go
├── order_repository_test.go
└── [其他 Repository].go
```

**Tests**:
```
internal/repository/
├── user_repository_test.go（單元測試 + testcontainers）
├── order_repository_test.go
└── [其他測試].go
```

**Success Criteria**:
- [ ] 所有 Repository 介面已定義
- [ ] 所有 Repository 實作完成
- [ ] CRUD 方法已實作
- [ ] 業務查詢方法已實作
- [ ] 錯誤處理正確（Not Found、Database Error）
- [ ] 單元測試通過（使用 testcontainers）
- [ ] Test Coverage > 80%

**Estimated Time**: 30 minutes

**Status**: Not Started

---

### Stage 4: Service 層實作（Business Logic Layer）

**Goal**: 實作業務邏輯層、定義 Service 介面與實作

**Tasks**:
1. 定義 Service 介面（業務方法）
2. 實作 Service 結構（依賴注入 Repository、Logger）
3. 實作業務邏輯（驗證、轉換、協調多個 Repository）
4. 定義請求/回應 DTO
5. 實作錯誤處理（轉換 Repository 錯誤為業務錯誤）
6. 實作交易處理（如需要）
7. 撰寫 Service 單元測試（使用 Mock Repository）

**Files to Create**:
```
internal/service/
├── user_service.go
├── user_service_test.go
├── order_service.go
├── order_service_test.go
└── [其他 Service].go
```

**Tests**:
```
internal/service/
├── user_service_test.go（使用 mockery）
├── order_service_test.go
└── [其他測試].go
```

**Success Criteria**:
- [ ] 所有 Service 介面已定義
- [ ] 所有 Service 實作完成
- [ ] 業務邏輯正確實作
- [ ] 請求/回應 DTO 已定義
- [ ] 錯誤處理正確
- [ ] 單元測試通過（使用 Mock）
- [ ] Test Coverage > 80%

**Estimated Time**: 35 minutes

**Status**: Not Started

---

### Stage 5: Handler 層實作與整合測試

**Goal**: 實作 HTTP Handlers、定義路由、撰寫整合測試

**Tasks**:
1. 定義 Handler 結構（依賴注入 Service、Logger）
2. 實作 HTTP Handlers（請求綁定、驗證、呼叫 Service、回應處理）
3. 定義路由（根據 OPENAPI.yaml）
4. 實作中介軟體（錯誤處理、日誌、認證）
5. 整合 main.go（初始化 Handlers、註冊路由）
6. 撰寫 Handler 測試（使用 httptest）
7. 撰寫整合測試（完整流程，使用 testcontainers）

**Files to Create**:
```
internal/handler/
├── user_handler.go
├── user_handler_test.go
├── order_handler.go
├── order_handler_test.go
└── [其他 Handler].go

internal/middleware/
├── logger.go
├── error_handler.go
└── auth.go（如需要）

tests/integration/
├── user_integration_test.go
├── order_integration_test.go
└── [其他整合測試].go
```

**Tests**:
```
internal/handler/
├── user_handler_test.go（httptest）
└── [其他測試].go

tests/integration/
├── user_integration_test.go（testcontainers）
└── [其他測試].go
```

**Success Criteria**:
- [ ] 所有 Handlers 已實作
- [ ] 路由定義正確（符合 OPENAPI.yaml）
- [ ] 中介軟體已實作（錯誤處理、日誌）
- [ ] main.go 整合完成
- [ ] Handler 測試通過
- [ ] 整合測試通過（testcontainers）
- [ ] Test Coverage > 80%
- [ ] 所有測試通過（go test ./...）
- [ ] 代碼可編譯且執行

**Estimated Time**: 40 minutes

**Status**: Not Started

---

## 🧪 測試策略

### 單元測試
- **範圍**: Service 層、Repository 層
- **工具**: testify/assert、testify/suite
- **Mock**: uber-go/mock（gomock + mockgen）
- **Mock 生成**: `go generate ./...`
- **覆蓋率目標**: > 80%

### 整合測試
- **範圍**: 完整流程（Handler → Service → Repository → DB）
- **工具**: testcontainers-go（啟動真實 PostgreSQL）
- **測試資料**: fixtures（測試資料檔案）

### Table-Driven Tests
- 使用 Table-Driven Tests 模式
- 涵蓋正常、異常、邊界案例

### Test Coverage
- 使用 `go test -cover ./...` 檢查覆蓋率
- 目標：整體覆蓋率 > 80%

---

## 📁 完整檔案清單

```
project/
├── cmd/
│   └── api/
│       └── main.go                          # 應用程式入口
├── internal/
│   ├── handler/
│   │   ├── user_handler.go                  # User HTTP Handler
│   │   ├── user_handler_test.go
│   │   ├── order_handler.go                 # Order HTTP Handler
│   │   └── order_handler_test.go
│   ├── service/
│   │   ├── user_service.go                  # User 業務邏輯
│   │   ├── user_service_test.go
│   │   ├── order_service.go                 # Order 業務邏輯
│   │   └── order_service_test.go
│   ├── repository/
│   │   ├── user_repository.go               # User 資料存取
│   │   ├── user_repository_test.go
│   │   ├── order_repository.go              # Order 資料存取
│   │   └── order_repository_test.go
│   ├── model/
│   │   ├── user.go                          # User Model
│   │   ├── order.go                         # Order Model
│   │   └── product.go                       # Product Model
│   ├── middleware/
│   │   ├── logger.go                        # 日誌中介軟體
│   │   ├── error_handler.go                 # 錯誤處理中介軟體
│   │   └── auth.go                          # 認證中介軟體（如需要）
│   ├── config/
│   │   └── config.go                        # 設定管理
│   └── errors/
│       └── errors.go                        # 自訂錯誤類型
├── pkg/
│   └── validator/
│       └── validator.go                     # 自訂驗證器
├── tests/
│   ├── integration/
│   │   ├── user_integration_test.go         # User 整合測試
│   │   └── order_integration_test.go        # Order 整合測試
│   └── fixtures/
│       └── testdata.sql                     # 測試資料
├── migrations/
│   ├── 20240115103000_create_users.sql      # 資料庫遷移
│   └── 20240115104000_create_orders.sql
├── go.mod
├── go.sum
├── Makefile
├── .env.example
├── .gitignore
└── README.md
```

---

## ⚠️ 風險識別

### 技術風險
- [ ] **外部 API 整合**：[如有外部 API，列出潛在問題]
- [ ] **效能瓶頸**：[如：複雜查詢、大量資料]
- [ ] **並發安全**：[如：goroutine、shared state]

### 資源風險
- [ ] **開發時間不足**：[估計總時間 vs 可用時間]
- [ ] **測試環境**：[testcontainers 需要 Docker]

### 依賴風險
- [ ] **第三方套件版本**：[列出關鍵依賴與版本限制]
- [ ] **雲端服務限制**：[如：資料庫連線數、API Rate Limit]

---

## 📝 備註

### 假設與前提
- 假設 PostgreSQL 14+ 可用
- 假設 Docker 可用（用於 testcontainers）
- 假設 OPENAPI.yaml 與 SCHEMA.sql 正確且完整

### 待確認項目
- [ ] Go 框架選擇已確認（Gin/Echo/Fiber）
- [ ] ORM 選擇已確認（GORM/sqlx/ent）
- [ ] 認證機制已確認（JWT/OAuth2/Custom Headers）

### 參考文件
- CLOUD_ARCHITECTURE.md
- OPENAPI.yaml
- SCHEMA.sql
- API_ENDPOINTS.md

---

## 🎯 下一步

**當使用者批准此 Implementation Plan 後：**

1. Orchestrator 再次調用 Backend Developer (Go) Agent
2. 明確指定 "Phase 2: Development Mode"
3. 提供已批准的 IMPLEMENTATION_PLAN_BACKEND_GO.md
4. Agent 將按 Stage 順序執行實作
5. 每個 Stage 完成後更新 Status
6. 所有 Stages 完成後清理此檔案

**預估總開發時間**: [Stage 1 + Stage 2 + Stage 3 + Stage 4 + Stage 5] = [總分鐘數] minutes
