# Go 資料庫存取工具對比與選擇指南

## 前提說明

⚠️ **重要前提：資料庫存取工具通常已由架構師/技術選型確定，Backend Developer 直接使用指定工具即可**

本指南僅供參考，幫助理解不同工具的特性與適用場景。實際開發時，請依照 CLOUD_ARCHITECTURE.md 中的技術選型執行。

---

## 支援的資料庫存取工具

### 1. GORM（ORM - Object Relational Mapping）

**類型：** Full-featured ORM

**特點：**
- ✅ 功能完整，社群最大
- ✅ 自動關聯管理（Has One, Has Many, Belongs To, Many to Many）
- ✅ Hook 支援（BeforeCreate, AfterUpdate）
- ✅ 軟刪除、樂觀鎖內建
- ✅ 豐富的查詢 API
- ⚠️ 效能略低於原生 SQL
- ⚠️ 複雜查詢可能產生 N+1 問題
- ⚠️ 學習曲線中等

**適用場景：**
- CRUD 為主的業務系統
- 需要複雜關聯管理
- 快速開發、原型驗證
- 團隊熟悉 ORM 概念

**範例：**
```go
// GORM Model 定義
type User struct {
    ID        uint           `gorm:"primaryKey"`
    Email     string         `gorm:"uniqueIndex;not null"`
    Name      string         `gorm:"size:100"`
    Posts     []Post         `gorm:"foreignKey:UserID"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`
}

// GORM 查詢
var users []User
db.Preload("Posts").Where("age > ?", 18).Find(&users)
```

**完整範例：** 見 `examples/gorm-example.md`

---

### 2. sqlx（Query Builder - SQL擴展）

**類型：** SQL Query Builder with struct mapping

**特點：**
- ✅ 接近原生 SQL，效能優異
- ✅ 靈活度高，完全控制 SQL
- ✅ 學習曲線低（熟悉 SQL 即可）
- ✅ 無 ORM 魔法，行為可預測
- ⚠️ 需手動管理關聯
- ⚠️ 無自動 Migration
- ⚠️ 重複代碼較多（CRUD boilerplate）

**適用場景：**
- 效能敏感的應用
- 複雜 SQL 查詢（分析、報表）
- 團隊精通 SQL
- 需要完全控制查詢邏輯

**範例：**
```go
// sqlx struct mapping
type User struct {
    ID        int       `db:"id"`
    Email     string    `db:"email"`
    Name      string    `db:"name"`
    CreatedAt time.Time `db:"created_at"`
}

// sqlx 查詢
var users []User
query := `SELECT * FROM users WHERE age > $1`
err := db.Select(&users, query, 18)
```

---

### 3. ent（Type-safe ORM - Graph-based）

**類型：** Entity Framework for Go (by Facebook)

**特點：**
- ✅ 完全 Type-safe，編譯時檢查
- ✅ Graph-based 查詢，關聯處理優雅
- ✅ 自動 Migration 生成
- ✅ 支援複雜驗證規則
- ✅ Code Generation 確保一致性
- ⚠️ 學習曲線較陡
- ⚠️ 社群相對較小
- ⚠️ Schema 定義較冗長

**適用場景：**
- 大型專案，需要強型別保證
- 複雜的資料關聯與驗證
- 團隊重視編譯時安全
- 需要自動 Migration（開發階段）

**範例：**
```go
// ent Schema 定義（需 code generation）
// schema/user.go
func (User) Fields() []ent.Field {
    return []ent.Field{
        field.String("email").Unique().NotEmpty(),
        field.String("name").MaxLen(100),
        field.Time("created_at").Default(time.Now),
    }
}

// ent 查詢（Type-safe）
users, err := client.User.
    Query().
    Where(user.AgeGT(18)).
    WithPosts().  // Preload posts
    All(ctx)
```

---

### 4. sqlc（SQL Generator - Type-safe SQL）

**類型：** SQL-first code generator

**特點：**
- ✅ 寫 SQL，生成 type-safe Go 代碼
- ✅ 編譯時檢查 SQL 語法
- ✅ 零執行時開銷（無反射）
- ✅ 極致效能（接近原生 SQL）
- ✅ 完全控制 SQL
- ⚠️ 需手動管理關聯
- ⚠️ 需額外的 Code Generation 步驟
- ⚠️ 無 ORM 便利功能

**適用場景：**
- 效能至上的應用（微秒級延遲）
- 團隊精通 SQL 且重視型別安全
- 不需要複雜 ORM 功能
- 希望完全掌控 SQL 查詢

**範例：**
```sql
-- queries/user.sql (sqlc 語法)
-- name: GetUser :one
SELECT * FROM users WHERE id = $1 LIMIT 1;

-- name: ListUsers :many
SELECT * FROM users WHERE age > $1;

-- name: CreateUser :one
INSERT INTO users (email, name) VALUES ($1, $2) RETURNING *;
```

```go
// sqlc 生成的 Go 代碼（自動生成，不手寫）
user, err := queries.GetUser(ctx, 123)
users, err := queries.ListUsers(ctx, 18)
newUser, err := queries.CreateUser(ctx, CreateUserParams{
    Email: "test@example.com",
    Name:  "Test User",
})
```

**完整範例：** 見 `examples/sqlc-example.md`

---

## 工具選擇對比表

| 特性 | GORM | sqlx | ent | sqlc |
|-----|------|------|-----|------|
| **學習曲線** | 中 | 低 | 高 | 中 |
| **效能** | 中 | 高 | 中 | 極高 |
| **型別安全** | ❌ Runtime | ❌ Runtime | ✅ Compile-time | ✅ Compile-time |
| **關聯管理** | ✅ 自動 | ❌ 手動 | ✅ Graph-based | ❌ 手動 |
| **SQL 控制** | 中 | 完全 | 中 | 完全 |
| **社群大小** | 最大 | 大 | 中 | 中 |
| **Code Generation** | ❌ 不需要 | ❌ 不需要 | ✅ 需要 | ✅ 需要 |
| **Migration** | 有（不建議生產用） | ❌ 無 | ✅ 自動生成 | ❌ 無 |
| **適合新手** | ✅ | ✅ | ❌ | ⚠️ |

---

## Migration 管理規則（CRITICAL）

⚠️ **Backend Developer 的 Migration 職責邊界**

### ❌ 絕對禁止（Production）

Backend Developer **絕對不可**在生產環境或正式開發流程中執行以下操作：

1. **GORM AutoMigrate**
   ```go
   // ❌ FORBIDDEN in production
   db.AutoMigrate(&User{}, &Post{})
   ```
   - 原因：無版本控制、無回滾機制、可能破壞資料

2. **ent Migration**
   ```bash
   # ❌ FORBIDDEN - Backend Developer 不執行
   ent migrate apply
   ```
   - 原因：Schema 變更應由 DBA Agent 專責管理

3. **任何手動 Migration 指令**
   ```bash
   # ❌ FORBIDDEN
   migrate up
   migrate down
   migrate force
   ```
   - 原因：Migration 執行由 DBA Agent 或 DevOps Agent 負責

4. **自行修改資料庫 Schema**
   - ❌ 直接執行 ALTER TABLE
   - ❌ 直接執行 CREATE TABLE
   - ❌ 使用 database client 手動修改

### ✅ Backend Developer 的職責

1. **使用 DBA 提供的 Schema**
   - 讀取 SCHEMA.sql
   - 根據 Schema 定義 Model（GORM/ent）
   - 根據 Schema 撰寫 SQL（sqlc/sqlx）

2. **請求 Schema 變更**（透過 DBA Agent）
   - 發現 Schema 需求變更時
   - 提交 Schema 變更請求給 DBA Agent
   - 等待 DBA Agent 產出新的 Migration

3. **測試環境的 Migration**（僅限測試）
   - ✅ 在 testcontainers 中使用 AutoMigrate（僅測試）
   ```go
   // ✅ OK in tests only
   func setupTestDB(t *testing.T) *gorm.DB {
       db := testcontainers.StartPostgres(t)
       db.AutoMigrate(&User{}, &Post{}) // OK for tests
       return db
   }
   ```

### Migration 流程（正確方式）

```
Backend Developer 需求 → DBA Agent
                          ↓
                    產出 Migration 腳本
                          ↓
                    Code Review
                          ↓
                    DevOps Agent 執行
                          ↓
                    Backend Developer 使用新 Schema
```

---

## 決策樹

```
開始選擇資料庫工具
    ↓
CLOUD_ARCHITECTURE.md 是否已指定？
    ├─ YES → 使用指定工具（無需選擇）
    └─ NO → 繼續評估（但應請求架構師確認）
        ↓
是否需要極致效能（微秒級延遲）？
    ├─ YES → sqlc
    └─ NO → 繼續
        ↓
團隊是否精通 SQL 且不需要 ORM？
    ├─ YES → sqlx
    └─ NO → 繼續
        ↓
是否需要強型別保證與 Graph-based 查詢？
    ├─ YES → ent
    └─ NO → 繼續
        ↓
是否需要快速開發與豐富的 ORM 功能？
    ├─ YES → GORM（推薦）
    └─ NO → sqlx（fallback）
```

---

## 實際專案建議

### 推薦組合

1. **一般業務系統**
   - 首選：GORM
   - 次選：ent（如果團隊重視型別安全）

2. **高效能 API**
   - 首選：sqlc
   - 次選：sqlx

3. **複雜分析系統**
   - 首選：sqlx（完全控制 SQL）
   - 次選：sqlc

4. **大型企業專案**
   - 首選：ent（編譯時安全）
   - 次選：GORM（快速開發）

### 混用策略（不建議但可接受）

如果專案規模大，可以在不同模組使用不同工具：

```
專案結構：
├── internal/
│   ├── user/          # GORM（CRUD為主）
│   ├── analytics/     # sqlc（複雜查詢）
│   └── report/        # sqlx（報表生成）
```

⚠️ 注意：混用會增加維護成本，僅在有明確理由時使用。

---

## 常見問題

### Q1: 為什麼不推薦 GORM AutoMigrate？

A: AutoMigrate 的問題：
- 無版本控制（無法追蹤 Schema 變更歷史）
- 無法回滾（出錯時難以恢復）
- 可能破壞資料（刪除欄位時直接 DROP COLUMN）
- 無團隊協作（多人開發時容易衝突）
- 無 Code Review（Schema 變更未經審核）

正確做法：使用專門的 Migration 工具（golang-migrate, Flyway, Liquibase），由 DBA Agent 管理。

### Q2: sqlc 和 sqlx 有什麼區別？

A: 核心區別：
- **sqlc**: 寫 SQL → 生成 Go 代碼（編譯時檢查）
- **sqlx**: 寫 SQL → 執行時映射 struct（執行時錯誤）

範例：
```go
// sqlc - 編譯時檢查
user, err := queries.GetUser(ctx, 123) // 型別安全
// 如果 GetUser 不存在 → 編譯錯誤

// sqlx - 執行時檢查
var user User
err := db.Get(&user, "SELECT * FROM users WHERE id = $1", 123)
// SQL 錯誤 → 執行時才發現
```

### Q3: 可以從 GORM 遷移到 sqlc 嗎？

A: 可以，但需要：
1. 使用 Strangler Fig Pattern（逐步替換）
2. 先寫 sqlc 查詢檔案（queries/*.sql）
3. 執行 sqlc generate
4. 逐個 API 端點遷移
5. 保持舊代碼運作直到完全遷移

完整遷移策略：見 `examples/legacy-refactoring-example.md`

### Q4: 測試時應該使用哪種工具？

A: 測試策略：
- **單元測試**: 使用 Mock（gomock）- 工具無關
- **整合測試**: 使用 testcontainers + 實際工具（GORM/sqlc/sqlx/ent）

```go
// 整合測試範例
func TestUserRepository(t *testing.T) {
    // 啟動真實 PostgreSQL（testcontainers）
    db := testcontainers.StartPostgres(t)

    // 使用實際的資料庫工具
    // GORM: db.AutoMigrate(&User{}) // OK for tests
    // sqlc: queries.CreateUser(ctx, params)
}
```

完整測試範例：見 `examples/testing-example.md`

---

## 總結

- **架構師已選型** → 直接使用指定工具
- **未選型** → 一般專案用 GORM，高效能用 sqlc
- **Migration** → 絕對不執行，由 DBA Agent 管理
- **測試** → testcontainers + 實際工具

Backend Developer 的核心職責是**使用**資料庫工具實作業務邏輯，而非管理資料庫 Schema。
