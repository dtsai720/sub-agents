# GORM Model & Repository 完整範例

## Model 層範例

### 基本 Model 定義

```go
package model

import (
    "time"
    "github.com/google/uuid"
    "gorm.io/gorm"
)

type User struct {
    ID        uuid.UUID      `gorm:"type:uuid;primary_key;default:gen_random_uuid()" json:"id"`
    Name      string         `gorm:"type:varchar(100);not null" json:"name" binding:"required"`
    Email     string         `gorm:"type:varchar(255);unique;not null" json:"email" binding:"required,email"`
    Status    string         `gorm:"type:varchar(20);not null;default:'active'" json:"status"`
    CreatedAt time.Time      `gorm:"not null;default:CURRENT_TIMESTAMP" json:"created_at"`
    UpdatedAt time.Time      `gorm:"not null;default:CURRENT_TIMESTAMP" json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`

    // Associations
    Orders []Order `gorm:"foreignKey:UserID" json:"orders,omitempty"`
}

func (User) TableName() string {
    return "users"
}

// BeforeCreate hook
func (u *User) BeforeCreate(tx *gorm.DB) error {
    if u.ID == uuid.Nil {
        u.ID = uuid.New()
    }
    return nil
}
```

### 關聯關係範例

```go
package model

// One-to-Many: User has many Orders
type Order struct {
    ID        uuid.UUID      `gorm:"type:uuid;primary_key" json:"id"`
    UserID    uuid.UUID      `gorm:"type:uuid;not null;index" json:"user_id"`
    Amount    float64        `gorm:"type:decimal(10,2);not null" json:"amount"`
    Status    string         `gorm:"type:varchar(20);not null" json:"status"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`

    // Belongs To
    User User `gorm:"foreignKey:UserID" json:"user,omitempty"`

    // Has Many
    OrderItems []OrderItem `gorm:"foreignKey:OrderID" json:"order_items,omitempty"`
}

func (Order) TableName() string {
    return "orders"
}

// Many-to-Many: Order has many Products through OrderItem
type OrderItem struct {
    ID        uuid.UUID `gorm:"type:uuid;primary_key" json:"id"`
    OrderID   uuid.UUID `gorm:"type:uuid;not null;index" json:"order_id"`
    ProductID uuid.UUID `gorm:"type:uuid;not null;index" json:"product_id"`
    Quantity  int       `gorm:"not null" json:"quantity"`
    Price     float64   `gorm:"type:decimal(10,2);not null" json:"price"`

    Order   Order   `gorm:"foreignKey:OrderID"`
    Product Product `gorm:"foreignKey:ProductID"`
}

type Product struct {
    ID          uuid.UUID `gorm:"type:uuid;primary_key" json:"id"`
    Name        string    `gorm:"type:varchar(200);not null" json:"name"`
    Price       float64   `gorm:"type:decimal(10,2);not null" json:"price"`
    Description string    `gorm:"type:text" json:"description"`
}
```

---

## Repository 層範例

### Repository 介面定義

```go
package repository

import (
    "context"
    "github.com/google/uuid"
    "your-project/internal/model"
)

type UserRepository interface {
    Create(ctx context.Context, user *model.User) error
    GetByID(ctx context.Context, id uuid.UUID) (*model.User, error)
    GetByEmail(ctx context.Context, email string) (*model.User, error)
    List(ctx context.Context, offset, limit int) ([]*model.User, int64, error)
    Update(ctx context.Context, user *model.User) error
    Delete(ctx context.Context, id uuid.UUID) error
}
```

### Repository 實作

```go
package repository

import (
    "context"
    "errors"
    "log/slog"

    "github.com/google/uuid"
    "gorm.io/gorm"
    "your-project/internal/model"
)

// Custom Errors
var (
    ErrUserNotFound      = errors.New("user not found")
    ErrUserEmailExists   = errors.New("user email already exists")
    ErrDatabaseError     = errors.New("database error")
)

type userRepository struct {
    db     *gorm.DB
    logger *slog.Logger
}

func NewUserRepository(db *gorm.DB, logger *slog.Logger) UserRepository {
    return &userRepository{db: db, logger: logger}
}

// Create creates a new user
func (r *userRepository) Create(ctx context.Context, user *model.User) error {
    if err := r.db.WithContext(ctx).Create(user).Error; err != nil {
        r.logger.Error("failed to create user", "error", err)
        return err
    }
    return nil
}

// GetByID retrieves a user by ID
func (r *userRepository) GetByID(ctx context.Context, id uuid.UUID) (*model.User, error) {
    var user model.User
    if err := r.db.WithContext(ctx).First(&user, "id = ?", id).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, ErrUserNotFound
        }
        r.logger.Error("failed to get user", "error", err, "id", id)
        return nil, ErrDatabaseError
    }
    return &user, nil
}

// GetByEmail retrieves a user by email
func (r *userRepository) GetByEmail(ctx context.Context, email string) (*model.User, error) {
    var user model.User
    if err := r.db.WithContext(ctx).Where("email = ?", email).First(&user).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, ErrUserNotFound
        }
        r.logger.Error("failed to get user by email", "error", err, "email", email)
        return nil, ErrDatabaseError
    }
    return &user, nil
}

// List retrieves a paginated list of users
func (r *userRepository) List(ctx context.Context, offset, limit int) ([]*model.User, int64, error) {
    var users []*model.User
    var total int64

    // Count total records
    if err := r.db.WithContext(ctx).Model(&model.User{}).Count(&total).Error; err != nil {
        r.logger.Error("failed to count users", "error", err)
        return nil, 0, ErrDatabaseError
    }

    // Fetch paginated results
    if err := r.db.WithContext(ctx).
        Offset(offset).
        Limit(limit).
        Find(&users).Error; err != nil {
        r.logger.Error("failed to list users", "error", err)
        return nil, 0, ErrDatabaseError
    }

    return users, total, nil
}

// Update updates a user
func (r *userRepository) Update(ctx context.Context, user *model.User) error {
    result := r.db.WithContext(ctx).Save(user)
    if result.Error != nil {
        r.logger.Error("failed to update user", "error", result.Error, "id", user.ID)
        return ErrDatabaseError
    }
    if result.RowsAffected == 0 {
        return ErrUserNotFound
    }
    return nil
}

// Delete soft-deletes a user
func (r *userRepository) Delete(ctx context.Context, id uuid.UUID) error {
    result := r.db.WithContext(ctx).Delete(&model.User{}, "id = ?", id)
    if result.Error != nil {
        r.logger.Error("failed to delete user", "error", result.Error, "id", id)
        return ErrDatabaseError
    }
    if result.RowsAffected == 0 {
        return ErrUserNotFound
    }
    return nil
}
```

### 進階查詢範例（Preload、Join）

```go
// GetWithOrders retrieves a user with their orders (Preload)
func (r *userRepository) GetWithOrders(ctx context.Context, id uuid.UUID) (*model.User, error) {
    var user model.User
    if err := r.db.WithContext(ctx).
        Preload("Orders").
        First(&user, "id = ?", id).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, ErrUserNotFound
        }
        return nil, ErrDatabaseError
    }
    return &user, nil
}

// ListActiveUsers retrieves users with status = 'active'
func (r *userRepository) ListActiveUsers(ctx context.Context, offset, limit int) ([]*model.User, int64, error) {
    var users []*model.User
    var total int64

    query := r.db.WithContext(ctx).Where("status = ?", "active")

    if err := query.Model(&model.User{}).Count(&total).Error; err != nil {
        return nil, 0, ErrDatabaseError
    }

    if err := query.Offset(offset).Limit(limit).Find(&users).Error; err != nil {
        return nil, 0, ErrDatabaseError
    }

    return users, total, nil
}

// ListUsersWithOrderCount retrieves users with their order count (Join + Group By)
func (r *userRepository) ListUsersWithOrderCount(ctx context.Context) ([]map[string]any, error) {
    var results []map[string]any

    err := r.db.WithContext(ctx).
        Table("users").
        Select("users.id, users.name, users.email, COUNT(orders.id) as order_count").
        Joins("LEFT JOIN orders ON orders.user_id = users.id").
        Group("users.id, users.name, users.email").
        Scan(&results).Error

    if err != nil {
        r.logger.Error("failed to list users with order count", "error", err)
        return nil, ErrDatabaseError
    }

    return results, nil
}
```

---

## GORM Tags 參考

### 常用 Tags

```go
type Example struct {
    // Primary Key
    ID uint `gorm:"primary_key"`

    // UUID Primary Key
    ID uuid.UUID `gorm:"type:uuid;primary_key;default:gen_random_uuid()"`

    // Auto Increment
    ID uint `gorm:"autoIncrement"`

    // Column Name
    UserName string `gorm:"column:user_name"`

    // Data Type
    Age int `gorm:"type:int"`
    Amount float64 `gorm:"type:decimal(10,2)"`

    // Not Null
    Name string `gorm:"not null"`

    // Unique
    Email string `gorm:"unique"`

    // Index
    Email string `gorm:"index"`
    Email string `gorm:"uniqueIndex"`

    // Default Value
    Status string `gorm:"default:'active'"`

    // Foreign Key
    UserID uint `gorm:"foreignKey:UserID"`

    // Embedded Struct
    Address Address `gorm:"embedded;embeddedPrefix:address_"`

    // Ignore Field
    IgnoredField string `gorm:"-"`

    // Soft Delete
    DeletedAt gorm.DeletedAt `gorm:"index"`

    // Timestamps
    CreatedAt time.Time `gorm:"autoCreateTime"`
    UpdatedAt time.Time `gorm:"autoUpdateTime"`
}
```

---

## 最佳實踐

### 1. 依賴注入

```go
// Constructor Injection
func NewUserRepository(db *gorm.DB, logger *slog.Logger) UserRepository {
    return &userRepository{db: db, logger: logger}
}
```

### 2. Context 使用

```go
// 所有資料庫操作都使用 WithContext
r.db.WithContext(ctx).First(&user, "id = ?", id)
```

### 3. 錯誤處理

```go
// 區分 Not Found 與 Database Error
if errors.Is(err, gorm.ErrRecordNotFound) {
    return nil, ErrUserNotFound
}
return nil, ErrDatabaseError
```

### 4. 日誌記錄

```go
// 記錄錯誤與關鍵參數
r.logger.Error("failed to get user", "error", err, "id", id)
```

### 5. 避免 N+1 查詢

```go
// 使用 Preload 避免 N+1
r.db.Preload("Orders").Find(&users)
```

---

## 資料庫連線初始化

```go
package config

import (
    "fmt"
    "log/slog"
    "os"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "gorm.io/gorm/logger"
)

func InitDB() (*gorm.DB, error) {
    dsn := os.Getenv("DATABASE_URL")
    if dsn == "" {
        return nil, fmt.Errorf("DATABASE_URL is not set")
    }

    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info),
    })
    if err != nil {
        return nil, fmt.Errorf("failed to connect database: %w", err)
    }

    // Connection Pool Settings
    sqlDB, err := db.DB()
    if err != nil {
        return nil, err
    }

    sqlDB.SetMaxIdleConns(10)
    sqlDB.SetMaxOpenConns(100)

    slog.Info("database connected successfully")
    return db, nil
}
```
