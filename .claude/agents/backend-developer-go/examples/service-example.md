# Service 層完整範例

## Service 介面定義

```go
package service

import (
    "context"

    "github.com/google/uuid"
    "your-project/internal/model"
)

type UserService interface {
    CreateUser(ctx context.Context, req CreateUserRequest) (*model.User, error)
    GetUser(ctx context.Context, id uuid.UUID) (*model.User, error)
    ListUsers(ctx context.Context, offset, limit int) ([]*model.User, int64, error)
    UpdateUser(ctx context.Context, id uuid.UUID, req UpdateUserRequest) (*model.User, error)
    DeleteUser(ctx context.Context, id uuid.UUID) error
}
```

---

## Request/Response DTO

```go
package service

// CreateUserRequest represents the request to create a user
type CreateUserRequest struct {
    Name  string `json:"name" binding:"required,min=1,max=100"`
    Email string `json:"email" binding:"required,email"`
}

// UpdateUserRequest represents the request to update a user
type UpdateUserRequest struct {
    Name   *string `json:"name,omitempty" binding:"omitempty,min=1,max=100"`
    Email  *string `json:"email,omitempty" binding:"omitempty,email"`
    Status *string `json:"status,omitempty" binding:"omitempty,oneof=active inactive"`
}

// UserResponse represents a user response (if need to hide fields)
type UserResponse struct {
    ID        uuid.UUID `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    Status    string    `json:"status"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}
```

---

## Service 實作

```go
package service

import (
    "context"
    "errors"
    "log/slog"

    "github.com/google/uuid"
    "your-project/internal/apperrors"
    "your-project/internal/model"
    "your-project/internal/repository"
)

type userService struct {
    repo   repository.UserRepository
    logger *slog.Logger
}

func NewUserService(repo repository.UserRepository, logger *slog.Logger) UserService {
    return &userService{
        repo:   repo,
        logger: logger,
    }
}

// CreateUser creates a new user
func (s *userService) CreateUser(ctx context.Context, req CreateUserRequest) (*model.User, error) {
    // Business logic: Check email uniqueness
    existing, err := s.repo.GetByEmail(ctx, req.Email)
    if err != nil && !errors.Is(err, repository.ErrUserNotFound) {
        s.logger.Error("failed to check email", "error", err, "email", req.Email)
        return nil, apperrors.NewInternalError("failed to create user")
    }

    if existing != nil {
        return nil, apperrors.NewConflictError("user", "email already exists")
    }

    // Create user model
    user := &model.User{
        Name:   req.Name,
        Email:  req.Email,
        Status: "active",
    }

    // Save to repository
    if err := s.repo.Create(ctx, user); err != nil {
        s.logger.Error("failed to create user", "error", err)
        return nil, apperrors.NewInternalError("failed to create user")
    }

    s.logger.Info("user created", "user_id", user.ID, "email", user.Email)
    return user, nil
}

// GetUser retrieves a user by ID
func (s *userService) GetUser(ctx context.Context, id uuid.UUID) (*model.User, error) {
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        if errors.Is(err, repository.ErrUserNotFound) {
            return nil, apperrors.NewNotFoundError("user", "user not found")
        }
        s.logger.Error("failed to get user", "error", err, "id", id)
        return nil, apperrors.NewInternalError("failed to get user")
    }

    return user, nil
}

// ListUsers retrieves a paginated list of users
func (s *userService) ListUsers(ctx context.Context, offset, limit int) ([]*model.User, int64, error) {
    // Validate pagination parameters
    if offset < 0 {
        offset = 0
    }
    if limit < 1 || limit > 100 {
        limit = 10
    }

    users, total, err := s.repo.List(ctx, offset, limit)
    if err != nil {
        s.logger.Error("failed to list users", "error", err)
        return nil, 0, apperrors.NewInternalError("failed to list users")
    }

    return users, total, nil
}

// UpdateUser updates a user
func (s *userService) UpdateUser(ctx context.Context, id uuid.UUID, req UpdateUserRequest) (*model.User, error) {
    // Get existing user
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        if errors.Is(err, repository.ErrUserNotFound) {
            return nil, apperrors.NewNotFoundError("user", "user not found")
        }
        s.logger.Error("failed to get user", "error", err, "id", id)
        return nil, apperrors.NewInternalError("failed to update user")
    }

    // Business logic: If email is being changed, check uniqueness
    if req.Email != nil && *req.Email != user.Email {
        existing, err := s.repo.GetByEmail(ctx, *req.Email)
        if err != nil && !errors.Is(err, repository.ErrUserNotFound) {
            s.logger.Error("failed to check email", "error", err, "email", *req.Email)
            return nil, apperrors.NewInternalError("failed to update user")
        }
        if existing != nil {
            return nil, apperrors.NewConflictError("user", "email already exists")
        }
        user.Email = *req.Email
    }

    // Update fields if provided
    if req.Name != nil {
        user.Name = *req.Name
    }
    if req.Status != nil {
        user.Status = *req.Status
    }

    // Save to repository
    if err := s.repo.Update(ctx, user); err != nil {
        s.logger.Error("failed to update user", "error", err, "id", id)
        return nil, apperrors.NewInternalError("failed to update user")
    }

    s.logger.Info("user updated", "user_id", user.ID)
    return user, nil
}

// DeleteUser deletes a user
func (s *userService) DeleteUser(ctx context.Context, id uuid.UUID) error {
    // Business logic: Check if user exists
    _, err := s.repo.GetByID(ctx, id)
    if err != nil {
        if errors.Is(err, repository.ErrUserNotFound) {
            return apperrors.NewNotFoundError("user", "user not found")
        }
        s.logger.Error("failed to get user", "error", err, "id", id)
        return apperrors.NewInternalError("failed to delete user")
    }

    // Delete from repository
    if err := s.repo.Delete(ctx, id); err != nil {
        s.logger.Error("failed to delete user", "error", err, "id", id)
        return apperrors.NewInternalError("failed to delete user")
    }

    s.logger.Info("user deleted", "user_id", id)
    return nil
}
```

---

## 進階範例：多 Repository 協作

### Order Service（協調多個 Repository）

```go
package service

import (
    "context"
    "errors"
    "log/slog"

    "github.com/google/uuid"
    "your-project/internal/apperrors"
    "your-project/internal/model"
    "your-project/internal/repository"
)

type OrderService interface {
    CreateOrder(ctx context.Context, req CreateOrderRequest) (*model.Order, error)
    GetOrder(ctx context.Context, id uuid.UUID) (*model.Order, error)
    ListUserOrders(ctx context.Context, userID uuid.UUID, offset, limit int) ([]*model.Order, int64, error)
}

type orderService struct {
    orderRepo   repository.OrderRepository
    userRepo    repository.UserRepository
    productRepo repository.ProductRepository
    logger      *slog.Logger
}

func NewOrderService(
    orderRepo repository.OrderRepository,
    userRepo repository.UserRepository,
    productRepo repository.ProductRepository,
    logger *slog.Logger,
) OrderService {
    return &orderService{
        orderRepo:   orderRepo,
        userRepo:    userRepo,
        productRepo: productRepo,
        logger:      logger,
    }
}

type CreateOrderRequest struct {
    UserID uuid.UUID        `json:"user_id" binding:"required"`
    Items  []OrderItemInput `json:"items" binding:"required,min=1,dive"`
}

type OrderItemInput struct {
    ProductID uuid.UUID `json:"product_id" binding:"required"`
    Quantity  int       `json:"quantity" binding:"required,min=1"`
}

func (s *orderService) CreateOrder(ctx context.Context, req CreateOrderRequest) (*model.Order, error) {
    // 1. Validate user exists
    user, err := s.userRepo.GetByID(ctx, req.UserID)
    if err != nil {
        if errors.Is(err, repository.ErrUserNotFound) {
            return nil, apperrors.NewNotFoundError("user", "user not found")
        }
        return nil, apperrors.NewInternalError("failed to validate user")
    }

    // 2. Validate all products exist and calculate total
    var total float64
    var orderItems []model.OrderItem

    for _, item := range req.Items {
        product, err := s.productRepo.GetByID(ctx, item.ProductID)
        if err != nil {
            if errors.Is(err, repository.ErrProductNotFound) {
                return nil, apperrors.NewBadRequestError("product " + item.ProductID.String() + " not found")
            }
            return nil, apperrors.NewInternalError("failed to validate product")
        }

        itemTotal := product.Price * float64(item.Quantity)
        total += itemTotal

        orderItems = append(orderItems, model.OrderItem{
            ProductID: item.ProductID,
            Quantity:  item.Quantity,
            Price:     product.Price,
        })
    }

    // 3. Create order
    order := &model.Order{
        UserID:     req.UserID,
        Amount:     total,
        Status:     "pending",
        OrderItems: orderItems,
    }

    if err := s.orderRepo.Create(ctx, order); err != nil {
        s.logger.Error("failed to create order", "error", err)
        return nil, apperrors.NewInternalError("failed to create order")
    }

    s.logger.Info("order created",
        "order_id", order.ID,
        "user_id", user.ID,
        "amount", total,
    )

    return order, nil
}
```

---

## 交易處理範例

### 使用 GORM Transaction

```go
package service

import (
    "context"
    "gorm.io/gorm"
)

type PaymentService interface {
    ProcessPayment(ctx context.Context, req ProcessPaymentRequest) error
}

type paymentService struct {
    db          *gorm.DB
    orderRepo   repository.OrderRepository
    paymentRepo repository.PaymentRepository
    logger      *slog.Logger
}

type ProcessPaymentRequest struct {
    OrderID uuid.UUID `json:"order_id" binding:"required"`
    Amount  float64   `json:"amount" binding:"required,gt=0"`
    Method  string    `json:"method" binding:"required,oneof=credit_card paypal"`
}

func (s *paymentService) ProcessPayment(ctx context.Context, req ProcessPaymentRequest) error {
    // Start transaction
    return s.db.Transaction(func(tx *gorm.DB) error {
        // 1. Get order
        order, err := s.orderRepo.GetByIDWithLock(ctx, tx, req.OrderID)
        if err != nil {
            return apperrors.NewNotFoundError("order", "order not found")
        }

        // 2. Validate order status
        if order.Status != "pending" {
            return apperrors.NewBadRequestError("order is not pending")
        }

        // 3. Validate amount
        if req.Amount != order.Amount {
            return apperrors.NewBadRequestError("payment amount mismatch")
        }

        // 4. Create payment record
        payment := &model.Payment{
            OrderID: req.OrderID,
            Amount:  req.Amount,
            Method:  req.Method,
            Status:  "completed",
        }

        if err := s.paymentRepo.CreateWithTx(ctx, tx, payment); err != nil {
            return apperrors.NewInternalError("failed to create payment")
        }

        // 5. Update order status
        order.Status = "paid"
        if err := s.orderRepo.UpdateWithTx(ctx, tx, order); err != nil {
            return apperrors.NewInternalError("failed to update order")
        }

        s.logger.Info("payment processed",
            "order_id", req.OrderID,
            "payment_id", payment.ID,
            "amount", req.Amount,
        )

        return nil
    })
}
```

---

## 業務邏輯驗證範例

### 自訂驗證器

```go
package service

import (
    "errors"
    "regexp"
    "strings"
)

// ValidatePassword validates password strength
func ValidatePassword(password string) error {
    if len(password) < 8 {
        return errors.New("password must be at least 8 characters")
    }

    hasUpper := regexp.MustCompile(`[A-Z]`).MatchString(password)
    hasLower := regexp.MustCompile(`[a-z]`).MatchString(password)
    hasNumber := regexp.MustCompile(`[0-9]`).MatchString(password)

    if !hasUpper || !hasLower || !hasNumber {
        return errors.New("password must contain uppercase, lowercase, and number")
    }

    return nil
}

// ValidateUsername validates username format
func ValidateUsername(username string) error {
    if len(username) < 3 || len(username) > 20 {
        return errors.New("username must be 3-20 characters")
    }

    // Only alphanumeric and underscore
    matched := regexp.MustCompile(`^[a-zA-Z0-9_]+$`).MatchString(username)
    if !matched {
        return errors.New("username can only contain letters, numbers, and underscore")
    }

    // Cannot start with number
    if regexp.MustCompile(`^[0-9]`).MatchString(username) {
        return errors.New("username cannot start with a number")
    }

    return nil
}

// SanitizeInput removes dangerous characters
func SanitizeInput(input string) string {
    input = strings.TrimSpace(input)
    input = strings.ReplaceAll(input, "<", "")
    input = strings.ReplaceAll(input, ">", "")
    return input
}
```

### 在 Service 中使用驗證器

```go
func (s *userService) CreateUser(ctx context.Context, req CreateUserRequest) (*model.User, error) {
    // Validate username
    if err := ValidateUsername(req.Username); err != nil {
        return nil, apperrors.NewBadRequestError(err.Error())
    }

    // Validate password
    if err := ValidatePassword(req.Password); err != nil {
        return nil, apperrors.NewBadRequestError(err.Error())
    }

    // Sanitize input
    req.Name = SanitizeInput(req.Name)

    // ... rest of the logic
}
```

---

## 最佳實踐

### 1. 錯誤處理
- 轉換 Repository 錯誤為 AppError
- 區分業務錯誤（400、404、409）與系統錯誤（500）
- 使用 `errors.Is()` 檢查錯誤類型

### 2. 依賴注入
- Service 透過 Constructor 接收 Repository
- 使用介面而非具體實作
- 方便測試（可 Mock Repository）

### 3. 業務邏輯集中
- 驗證規則放在 Service 層
- 不在 Handler 或 Repository 層處理業務邏輯
- Service 是唯一的業務邏輯入口

### 4. DTO 使用
- 使用 Request/Response DTO 與 Model 分離
- Request DTO 包含驗證 tags
- 避免直接暴露 Model 給外部

### 5. 日誌記錄
- 記錄關鍵業務操作（創建、更新、刪除）
- 記錄錯誤與關鍵參數
- 使用結構化日誌

### 6. 交易處理
- 多個 Repository 操作需要原子性時使用 Transaction
- Transaction 內的錯誤要 rollback
- 避免在 Transaction 內執行長時間操作

### 7. 參數驗證
- 分頁參數設定預設值與上下限
- 驗證 UUID 格式
- 驗證業務規則（如：email 唯一性）
