# 遺留代碼重構範例（無 DI 到有 DI）

## 問題場景

現有代碼無法測試，因為：
1. 使用全域變數 `db *gorm.DB`
2. 函數內部直接使用 `db.Create()`
3. 無依賴注入，無法 Mock

## 原始代碼（Legacy Code）

```go
package user

import "gorm.io/gorm"

// 全域變數（無法在測試中替換）
var db *gorm.DB

func InitDB(database *gorm.DB) {
    db = database
}

// 無法測試的函數（依賴全域 db）
func CreateUser(name, email string) error {
    user := &User{Name: name, Email: email}
    return db.Create(user).Error
}

func GetUser(id uint) (*User, error) {
    var user User
    if err := db.First(&user, id).Error; err != nil {
        return nil, err
    }
    return &user, nil
}

type User struct {
    ID    uint   `gorm:"primaryKey"`
    Name  string
    Email string
}
```

---

## 重構策略：5 個階段

### Stage 1: 建立測試接縫（Create Seams）

**步驟 1.1: 提取介面**

```go
// 新增：定義 DB 介面（不修改 gorm.DB）
type DB interface {
    Create(value any) *gorm.DB
    First(dest any, conds ...any) *gorm.DB
}

// gorm.DB 自動實作此介面（Go 隱式介面）
```

**步驟 1.2: 參數化依賴**

```go
// 新增：接受參數的內部函數
func createUserWithDB(database DB, name, email string) error {
    user := &User{Name: name, Email: email}
    return database.Create(user).Error
}

// 保持原函數不變（向後相容）
func CreateUser(name, email string) error {
    return createUserWithDB(db, name, email)
}

// 新增：接受參數的內部函數
func getUserWithDB(database DB, id uint) (*User, error) {
    var user User
    if err := database.First(&user, id).Error; err != nil {
        return nil, err
    }
    return &user, nil
}

// 保持原函數不變（向後相容）
func GetUser(id uint) (*User, error) {
    return getUserWithDB(db, id)
}
```

**✅ Stage 1 完成檢查：**
- 原有函數行為不變（向後相容）
- 代碼可編譯
- 新增了可測試的內部函數

---

### Stage 2: 撰寫保護性測試（Safety Net Tests）

**步驟 2.1: 整合測試（使用 testcontainers）**

```go
package user_test

import (
    "context"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "your-project/user"
)

func setupTestDB(t *testing.T) *gorm.DB {
    ctx := context.Background()
    pgContainer, _ := postgres.RunContainer(ctx,
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("postgres"),
        postgres.WithPassword("postgres"),
    )
    connStr, _ := pgContainer.ConnectionString(ctx, "sslmode=disable")
    db, _ := gorm.Open(postgres.Open(connStr), &gorm.Config{})
    db.AutoMigrate(&user.User{})
    return db
}

// 整合測試：測試現有行為（作為安全網）
func TestCreateUser_Integration(t *testing.T) {
    db := setupTestDB(t)
    user.InitDB(db) // 使用全域 db

    // 測試現有函數
    err := user.CreateUser("John", "john@example.com")
    assert.NoError(t, err)

    // 驗證結果
    retrieved, err := user.GetUser(1)
    assert.NoError(t, err)
    assert.Equal(t, "John", retrieved.Name)
}
```

**步驟 2.2: 測試新的內部函數（可測試版本）**

```go
// 建立 Mock DB
type MockDB struct {
    CreateFunc func(value any) *gorm.DB
    FirstFunc  func(dest any, conds ...any) *gorm.DB
}

func (m *MockDB) Create(value any) *gorm.DB {
    return m.CreateFunc(value)
}

func (m *MockDB) First(dest any, conds ...any) *gorm.DB {
    return m.FirstFunc(dest, conds...)
}

// 單元測試：測試內部函數（使用 Mock）
func TestCreateUserWithDB(t *testing.T) {
    mockDB := &MockDB{
        CreateFunc: func(value any) *gorm.DB {
            // Mock 成功
            return &gorm.DB{}
        },
    }

    // 現在可以測試了！
    err := user.createUserWithDB(mockDB, "John", "john@example.com")
    assert.NoError(t, err)
}
```

**✅ Stage 2 完成檢查：**
- 整合測試通過（保護現有行為）
- 內部函數可以單元測試

---

### Stage 3: 漸進式引入 DI（Incremental DI Introduction）

**步驟 3.1: 建立 Repository 介面與實作**

```go
package repository

import (
    "context"
    "gorm.io/gorm"
    "your-project/user"
)

// 新介面（使用 DI）
type UserRepository interface {
    Create(ctx context.Context, user *user.User) error
    GetByID(ctx context.Context, id uint) (*user.User, error)
}

// 新實作（使用 DI）
type userRepository struct {
    db *gorm.DB
}

func NewUserRepository(db *gorm.DB) UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(ctx context.Context, u *user.User) error {
    return r.db.WithContext(ctx).Create(u).Error
}

func (r *userRepository) GetByID(ctx context.Context, id uint) (*user.User, error) {
    var u user.User
    if err := r.db.WithContext(ctx).First(&u, id).Error; err != nil {
        return nil, err
    }
    return &u, nil
}
```

**步驟 3.2: Adapter Pattern 橋接舊代碼**

```go
// Adapter：將舊代碼包裝為新介面
type LegacyUserRepositoryAdapter struct{}

func NewLegacyUserRepositoryAdapter() UserRepository {
    return &LegacyUserRepositoryAdapter{}
}

func (a *LegacyUserRepositoryAdapter) Create(ctx context.Context, u *user.User) error {
    // 調用舊代碼
    return user.CreateUser(u.Name, u.Email)
}

func (a *LegacyUserRepositoryAdapter) GetByID(ctx context.Context, id uint) (*user.User, error) {
    // 調用舊代碼
    return user.GetUser(id)
}
```

**步驟 3.3: 建立 Service 層（使用新介面）**

```go
package service

import (
    "context"
    "your-project/repository"
)

type UserService interface {
    CreateUser(ctx context.Context, name, email string) error
    GetUser(ctx context.Context, id uint) (*user.User, error)
}

type userService struct {
    repo repository.UserRepository // 使用介面，可以是舊或新實作
}

func NewUserService(repo repository.UserRepository) UserService {
    return &userService{repo: repo}
}

func (s *userService) CreateUser(ctx context.Context, name, email string) error {
    u := &user.User{Name: name, Email: email}
    return s.repo.Create(ctx, u)
}

func (s *userService) GetUser(ctx context.Context, id uint) (*user.User, error) {
    return s.repo.GetByID(ctx, id)
}
```

**✅ Stage 3 完成檢查：**
- 新介面已建立
- Adapter 可將舊代碼橋接到新介面
- Service 層可測試（可注入 Mock Repository）

---

### Stage 4: 逐步替換（Incremental Replacement）

**步驟 4.1: Strangler Fig 模式（一次遷移一個端點）**

```go
package handler

import (
    "github.com/gin-gonic/gin"
    "your-project/service"
    "your-project/repository"
    "your-project/user"
)

type UserHandler struct {
    service service.UserService
}

func NewUserHandler(svc service.UserService) *UserHandler {
    return &UserHandler{service: svc}
}

// 新端點：使用新架構
func (h *UserHandler) CreateUserV2(c *gin.Context) {
    var req struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }
    c.BindJSON(&req)

    // 使用新 Service（有 DI）
    err := h.service.CreateUser(c.Request.Context(), req.Name, req.Email)
    if err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }
    c.JSON(200, gin.H{"message": "ok"})
}

// 舊端點：保持不變（使用舊代碼）
func CreateUserV1(c *gin.Context) {
    var req struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }
    c.BindJSON(&req)

    // 使用舊代碼（全域 db）
    err := user.CreateUser(req.Name, req.Email)
    if err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }
    c.JSON(200, gin.H{"message": "ok"})
}
```

**步驟 4.2: main.go 中並行註冊**

```go
package main

import (
    "github.com/gin-gonic/gin"
    "your-project/handler"
    "your-project/service"
    "your-project/repository"
    "your-project/user"
)

func main() {
    db := initDB()

    // 初始化舊代碼（全域 db）
    user.InitDB(db)

    // 初始化新架構（DI）
    userRepo := repository.NewUserRepository(db)
    userService := service.NewUserService(userRepo)
    userHandler := handler.NewUserHandler(userService)

    router := gin.Default()

    // 舊端點（使用全域 db）
    router.POST("/api/v1/users", handler.CreateUserV1)

    // 新端點（使用 DI）
    router.POST("/api/v2/users", userHandler.CreateUserV2)

    router.Run(":8080")
}
```

**步驟 4.3: 並行執行驗證（Parallel Run）**

```go
// Feature Flag 控制流量分配
func (h *UserHandler) CreateUser(c *gin.Context) {
    var req struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }
    c.BindJSON(&req)

    // 同時執行新舊代碼
    var errOld, errNew error
    var userOld, userNew *user.User

    // 執行舊代碼
    errOld = user.CreateUser(req.Name, req.Email)

    // 執行新代碼
    errNew = h.service.CreateUser(c.Request.Context(), req.Name, req.Email)

    // 比對結果
    if errOld != errNew {
        logger.Warn("parallel run mismatch", "old_err", errOld, "new_err", errNew)
    }

    // 使用舊代碼的結果（保守）
    if errOld != nil {
        c.JSON(500, gin.H{"error": errOld.Error()})
        return
    }
    c.JSON(200, gin.H{"message": "ok"})
}
```

**✅ Stage 4 完成檢查：**
- 新舊代碼並行運行
- 逐步切換流量到新端點
- 驗證新代碼行為與舊代碼一致

---

### Stage 5: 清理與驗證（Cleanup & Validation）

**步驟 5.1: 移除舊代碼**

```go
// 刪除 user/legacy.go 中的舊函數
// - CreateUser(name, email string)
// - GetUser(id uint)
// - 全域變數 db

// 刪除 Adapter
// - LegacyUserRepositoryAdapter

// 更新 main.go：僅使用新架構
func main() {
    db := initDB()

    // 僅初始化新架構（DI）
    userRepo := repository.NewUserRepository(db)
    userService := service.NewUserService(userRepo)
    userHandler := handler.NewUserHandler(userService)

    router := gin.Default()

    // 僅使用新端點
    router.POST("/api/v1/users", userHandler.CreateUser)

    router.Run(":8080")
}
```

**步驟 5.2: 補充單元測試**

```go
package service_test

import (
    "context"
    "testing"
    "github.com/stretchr/testify/assert"
    "go.uber.org/mock/gomock"
    "your-project/repository/mocks"
    "your-project/service"
    "your-project/user"
)

func TestUserService_CreateUser(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockRepo := mocks.NewMockUserRepository(ctrl)
    mockRepo.EXPECT().
        Create(gomock.Any(), gomock.Any()).
        Return(nil)

    svc := service.NewUserService(mockRepo)

    // 現在可以輕鬆測試！
    err := svc.CreateUser(context.Background(), "John", "john@example.com")
    assert.NoError(t, err)
}
```

**✅ Stage 5 完成檢查：**
- 舊代碼已移除
- 測試覆蓋率 > 80%
- 所有端點使用新架構（DI）
- 代碼可維護、可測試

---

## 重構時間軸

| Stage | 描述 | 預估時間 | 風險 |
|-------|------|---------|------|
| Stage 1 | 建立測試接縫 | 30 分鐘 | 低（不改行為） |
| Stage 2 | 撰寫保護性測試 | 45 分鐘 | 低（僅加測試） |
| Stage 3 | 引入 DI | 60 分鐘 | 中（新舊並存） |
| Stage 4 | 逐步替換 | 90 分鐘 | 中（並行運行） |
| Stage 5 | 清理 | 30 分鐘 | 低（已驗證） |
| **總計** | | **4-5 小時** | |

---

## 關鍵成功因素

1. **小步前進**：每個 Stage < 50 行修改
2. **頻繁 Commit**：每個步驟都 commit
3. **持續測試**：每次修改後執行測試
4. **向後相容**：舊代碼與新代碼並存
5. **漸進替換**：一次遷移一個端點

---

## 常見陷阱

❌ **錯誤做法**：
- 直接刪除舊代碼重寫（Big Bang）
- 同時重構多個模組
- 沒有保護性測試就重構

✅ **正確做法**：
- 建立測試接縫 → 加測試 → 重構
- 一次只重構一個模組
- 新舊代碼並行運行驗證
