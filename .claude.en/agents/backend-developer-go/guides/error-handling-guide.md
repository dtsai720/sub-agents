# Go 錯誤處理完整指南

## 目錄

1. [Go 錯誤處理哲學](#go-錯誤處理哲學)
2. [自訂錯誤類型](#自訂錯誤類型)
3. [錯誤處理分層策略](#錯誤處理分層策略)
4. [統一錯誤回應格式](#統一錯誤回應格式)
5. [錯誤處理中介軟體](#錯誤處理中介軟體)
6. [錯誤日誌與追蹤](#錯誤日誌與追蹤)
7. [常見錯誤處理模式](#常見錯誤處理模式)

---

## Go 錯誤處理哲學

### 核心原則

1. **錯誤是值** - 使用 `error` 回傳值，不使用 `panic`
2. **明確處理** - 所有錯誤都要處理，不忽略
3. **上下文附加** - 錯誤應包含足夠的上下文資訊
4. **快速失敗** - Fail fast，盡早回傳錯誤
5. **錯誤包裝** - 使用 `fmt.Errorf` 或 `errors.Wrap` 保留原始錯誤

### 什麼時候使用 panic？

**只在真正的程式錯誤時使用：**
- ✅ 程式初始化失敗（無法繼續執行）
- ✅ 不可恢復的錯誤（如：記憶體損壞）
- ❌ 業務邏輯錯誤（使用 error）
- ❌ 可預期的錯誤（使用 error）
- ❌ 使用者輸入錯誤（使用 error）

```go
// ✅ Good: 初始化失敗時 panic
func init() {
    if err := loadConfig(); err != nil {
        panic("failed to load config: " + err.Error())
    }
}

// ❌ Bad: 業務邏輯使用 panic
func GetUser(id int) *User {
    user, err := db.FindUser(id)
    if err != nil {
        panic(err) // ❌ NEVER do this
    }
    return user
}

// ✅ Good: 業務邏輯回傳 error
func GetUser(id int) (*User, error) {
    user, err := db.FindUser(id)
    if err != nil {
        return nil, fmt.Errorf("get user: %w", err)
    }
    return user, nil
}
```

---

## 自訂錯誤類型

### 基本自訂錯誤

```go
// internal/errors/errors.go

package errors

import "fmt"

// AppError 應用程式錯誤基礎類型
type AppError struct {
    Code    string `json:"code"`              // 錯誤代碼（USER_NOT_FOUND）
    Message string `json:"message"`           // 使用者友善訊息
    Err     error  `json:"-"`                 // 原始錯誤（不序列化）
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %v", e.Message, e.Err)
    }
    return e.Message
}

// Unwrap 支援 errors.Is 和 errors.As
func (e *AppError) Unwrap() error {
    return e.Err
}

// 預定義錯誤代碼
const (
    ErrCodeNotFound          = "NOT_FOUND"
    ErrCodeValidation        = "VALIDATION_ERROR"
    ErrCodeUnauthorized      = "UNAUTHORIZED"
    ErrCodeForbidden         = "FORBIDDEN"
    ErrCodeConflict          = "CONFLICT"
    ErrCodeInternalError     = "INTERNAL_ERROR"
    ErrCodeBadRequest        = "BAD_REQUEST"
)

// 錯誤建構函數
func NotFound(message string, err error) *AppError {
    return &AppError{
        Code:    ErrCodeNotFound,
        Message: message,
        Err:     err,
    }
}

func ValidationError(message string, err error) *AppError {
    return &AppError{
        Code:    ErrCodeValidation,
        Message: message,
        Err:     err,
    }
}

func InternalError(message string, err error) *AppError {
    return &AppError{
        Code:    ErrCodeInternalError,
        Message: message,
        Err:     err,
    }
}

func Unauthorized(message string) *AppError {
    return &AppError{
        Code:    ErrCodeUnauthorized,
        Message: message,
        Err:     nil,
    }
}

func Forbidden(message string) *AppError {
    return &AppError{
        Code:    ErrCodeForbidden,
        Message: message,
        Err:     nil,
    }
}

func Conflict(message string, err error) *AppError {
    return &AppError{
        Code:    ErrCodeConflict,
        Message: message,
        Err:     err,
    }
}
```

### 帶 HTTP 狀態碼的錯誤

```go
// HTTPError 帶 HTTP 狀態碼的錯誤
type HTTPError struct {
    *AppError
    StatusCode int `json:"status"`
}

func (e *HTTPError) Error() string {
    return e.AppError.Error()
}

// 建構函數
func NewHTTPError(statusCode int, code, message string, err error) *HTTPError {
    return &HTTPError{
        AppError: &AppError{
            Code:    code,
            Message: message,
            Err:     err,
        },
        StatusCode: statusCode,
    }
}

// 便捷函數
func NotFoundHTTP(message string, err error) *HTTPError {
    return NewHTTPError(404, ErrCodeNotFound, message, err)
}

func BadRequestHTTP(message string, err error) *HTTPError {
    return NewHTTPError(400, ErrCodeBadRequest, message, err)
}

func UnauthorizedHTTP(message string) *HTTPError {
    return NewHTTPError(401, ErrCodeUnauthorized, message, nil)
}

func ForbiddenHTTP(message string) *HTTPError {
    return NewHTTPError(403, ErrCodeForbidden, message, nil)
}

func ConflictHTTP(message string, err error) *HTTPError {
    return NewHTTPError(409, ErrCodeConflict, message, err)
}

func InternalServerErrorHTTP(message string, err error) *HTTPError {
    return NewHTTPError(500, ErrCodeInternalError, message, err)
}
```

---

## 錯誤處理分層策略

### Repository 層錯誤處理

```go
// internal/repository/user_repository.go

func (r *userRepository) GetByID(ctx context.Context, id int) (*model.User, error) {
    var user model.User

    err := r.db.WithContext(ctx).First(&user, id).Error
    if err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            // 轉換為應用錯誤（不包含 GORM 實作細節）
            return nil, errors.NotFound(
                fmt.Sprintf("user with id %d not found", id),
                err,
            )
        }
        // 其他資料庫錯誤
        return nil, errors.InternalError(
            "failed to get user from database",
            err,
        )
    }

    return &user, nil
}

func (r *userRepository) Create(ctx context.Context, user *model.User) error {
    err := r.db.WithContext(ctx).Create(user).Error
    if err != nil {
        // 檢查唯一約束違反（如：重複 email）
        if strings.Contains(err.Error(), "duplicate key") {
            return errors.Conflict(
                "user with this email already exists",
                err,
            )
        }
        return errors.InternalError(
            "failed to create user",
            err,
        )
    }

    return nil
}
```

### Service 層錯誤處理

```go
// internal/service/user_service.go

func (s *userService) GetUser(ctx context.Context, id int) (*UserResponse, error) {
    // 驗證輸入
    if id <= 0 {
        return nil, errors.ValidationError(
            "invalid user id: must be positive",
            nil,
        )
    }

    // 呼叫 Repository
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        // Repository 已經轉換為應用錯誤，直接包裝並回傳
        return nil, fmt.Errorf("get user service: %w", err)
    }

    // 轉換為 Response DTO
    return &UserResponse{
        ID:    user.ID,
        Email: user.Email,
        Name:  user.Name,
    }, nil
}

func (s *userService) CreateUser(ctx context.Context, req CreateUserRequest) (*UserResponse, error) {
    // 驗證輸入
    if err := validateCreateUserRequest(req); err != nil {
        return nil, errors.ValidationError(
            "invalid user data",
            err,
        )
    }

    // 檢查業務規則（如：email 是否已存在）
    existingUser, err := s.repo.GetByEmail(ctx, req.Email)
    if err != nil && !errors.Is(err, errors.NotFound("", nil)) {
        // 非 NotFound 錯誤，是真實的資料庫錯誤
        return nil, fmt.Errorf("check existing user: %w", err)
    }
    if existingUser != nil {
        return nil, errors.Conflict(
            fmt.Sprintf("user with email %s already exists", req.Email),
            nil,
        )
    }

    // 建立 User
    user := &model.User{
        Email: req.Email,
        Name:  req.Name,
    }

    if err := s.repo.Create(ctx, user); err != nil {
        return nil, fmt.Errorf("create user service: %w", err)
    }

    return &UserResponse{
        ID:    user.ID,
        Email: user.Email,
        Name:  user.Name,
    }, nil
}
```

### Handler 層錯誤處理

```go
// internal/handler/user_handler.go

func (h *userHandler) GetUser(c *gin.Context) {
    // 解析參數
    idStr := c.Param("id")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        // 參數解析錯誤 → 400 Bad Request
        c.Error(errors.BadRequestHTTP(
            "invalid user id: must be a number",
            err,
        ))
        return
    }

    // 呼叫 Service
    user, err := h.service.GetUser(c.Request.Context(), id)
    if err != nil {
        // Service 已包裝錯誤，交給錯誤處理中介軟體
        c.Error(err)
        return
    }

    // 成功回應
    c.JSON(http.StatusOK, gin.H{
        "data": user,
    })
}

func (h *userHandler) CreateUser(c *gin.Context) {
    var req CreateUserRequest

    // 綁定與驗證請求
    if err := c.ShouldBindJSON(&req); err != nil {
        c.Error(errors.BadRequestHTTP(
            "invalid request body",
            err,
        ))
        return
    }

    // 呼叫 Service
    user, err := h.service.CreateUser(c.Request.Context(), req)
    if err != nil {
        c.Error(err)
        return
    }

    // 成功回應
    c.JSON(http.StatusCreated, gin.H{
        "data": user,
    })
}
```

---

## 統一錯誤回應格式

### RFC 7807 Problem Details

```go
// internal/errors/problem.go

// ProblemDetail RFC 7807 Problem Details for HTTP APIs
type ProblemDetail struct {
    Type     string `json:"type"`               // 錯誤類型 URI
    Title    string `json:"title"`              // 簡短標題
    Status   int    `json:"status"`             // HTTP 狀態碼
    Detail   string `json:"detail"`             // 詳細說明
    Instance string `json:"instance,omitempty"` // 請求路徑
    TraceID  string `json:"trace_id,omitempty"` // 追蹤 ID
}

// ToProblemDetail 將 AppError 轉換為 RFC 7807 格式
func ToProblemDetail(err error, instance string, traceID string) *ProblemDetail {
    var appErr *AppError
    var httpErr *HTTPError

    // 檢查是否為 HTTPError
    if errors.As(err, &httpErr) {
        return &ProblemDetail{
            Type:     "https://api.example.com/errors/" + httpErr.Code,
            Title:    getTitle(httpErr.Code),
            Status:   httpErr.StatusCode,
            Detail:   httpErr.Message,
            Instance: instance,
            TraceID:  traceID,
        }
    }

    // 檢查是否為 AppError
    if errors.As(err, &appErr) {
        return &ProblemDetail{
            Type:     "https://api.example.com/errors/" + appErr.Code,
            Title:    getTitle(appErr.Code),
            Status:   getStatusCode(appErr.Code),
            Detail:   appErr.Message,
            Instance: instance,
            TraceID:  traceID,
        }
    }

    // 未知錯誤
    return &ProblemDetail{
        Type:     "https://api.example.com/errors/INTERNAL_ERROR",
        Title:    "Internal Server Error",
        Status:   500,
        Detail:   "An unexpected error occurred",
        Instance: instance,
        TraceID:  traceID,
    }
}

func getTitle(code string) string {
    titles := map[string]string{
        ErrCodeNotFound:      "Not Found",
        ErrCodeValidation:    "Validation Error",
        ErrCodeUnauthorized:  "Unauthorized",
        ErrCodeForbidden:     "Forbidden",
        ErrCodeConflict:      "Conflict",
        ErrCodeInternalError: "Internal Server Error",
        ErrCodeBadRequest:    "Bad Request",
    }
    if title, ok := titles[code]; ok {
        return title
    }
    return "Unknown Error"
}

func getStatusCode(code string) int {
    statusCodes := map[string]int{
        ErrCodeNotFound:      404,
        ErrCodeValidation:    400,
        ErrCodeUnauthorized:  401,
        ErrCodeForbidden:     403,
        ErrCodeConflict:      409,
        ErrCodeInternalError: 500,
        ErrCodeBadRequest:    400,
    }
    if status, ok := statusCodes[code]; ok {
        return status
    }
    return 500
}
```

---

## 錯誤處理中介軟體

完整的框架特定實作請見：`guides/middleware-patterns.md`

### 核心原則（框架無關）

1. **統一錯誤回應格式** - 使用 RFC 7807 或自訂格式
2. **區分錯誤類型** - 業務錯誤（4xx）vs 系統錯誤（5xx）
3. **日誌記錄** - 系統錯誤記錄 stack trace
4. **避免洩漏敏感資訊** - 不回傳 SQL 錯誤、內部路徑
5. **支援追蹤** - 附加 Trace ID

### Gin 框架實作概要

```go
// internal/middleware/error_handler.go (Gin)

func ErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next() // 執行後續 handlers

        // 檢查是否有錯誤
        if len(c.Errors) > 0 {
            err := c.Errors.Last().Err

            // 提取 Trace ID
            traceID := c.GetString("trace_id")

            // 轉換為 Problem Detail
            problem := errors.ToProblemDetail(err, c.Request.URL.Path, traceID)

            // 記錄錯誤（僅 5xx）
            if problem.Status >= 500 {
                log.Error().
                    Err(err).
                    Str("trace_id", traceID).
                    Str("path", c.Request.URL.Path).
                    Msg("Internal server error")
            }

            // 回應
            c.JSON(problem.Status, problem)
        }
    }
}
```

詳細實作請見：`guides/middleware-patterns.md`

---

## 錯誤日誌與追蹤

### 結構化日誌

```go
import "log/slog"

func (s *userService) GetUser(ctx context.Context, id int) (*UserResponse, error) {
    user, err := s.repo.GetByID(ctx, id)
    if err != nil {
        // 記錄錯誤（包含上下文）
        s.logger.Error("failed to get user",
            slog.Int("user_id", id),
            slog.String("error", err.Error()),
            slog.String("trace_id", getTraceID(ctx)),
        )
        return nil, fmt.Errorf("get user service: %w", err)
    }

    // 記錄成功（可選）
    s.logger.Info("user retrieved successfully",
        slog.Int("user_id", id),
        slog.String("trace_id", getTraceID(ctx)),
    )

    return toUserResponse(user), nil
}
```

### Trace ID 傳遞

```go
// 中介軟體設定 Trace ID
func TraceMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        traceID := c.GetHeader("X-Trace-ID")
        if traceID == "" {
            traceID = generateTraceID()
        }

        c.Set("trace_id", traceID)
        c.Header("X-Trace-ID", traceID)

        // 將 Trace ID 注入 Context
        ctx := context.WithValue(c.Request.Context(), "trace_id", traceID)
        c.Request = c.Request.WithContext(ctx)

        c.Next()
    }
}

// 從 Context 提取 Trace ID
func getTraceID(ctx context.Context) string {
    if traceID, ok := ctx.Value("trace_id").(string); ok {
        return traceID
    }
    return ""
}
```

---

## 常見錯誤處理模式

### 1. 錯誤檢查與包裝

```go
// ✅ Good
result, err := doSomething()
if err != nil {
    return nil, fmt.Errorf("do something: %w", err)
}

// ❌ Bad: 忽略錯誤
result, _ := doSomething()

// ❌ Bad: 不包裝錯誤（失去上下文）
result, err := doSomething()
if err != nil {
    return nil, err
}
```

### 2. 錯誤類型判斷

```go
// 使用 errors.Is 判斷錯誤類型
err := repo.GetByID(ctx, id)
if errors.Is(err, gorm.ErrRecordNotFound) {
    return nil, errors.NotFound("user not found", err)
}

// 使用 errors.As 提取錯誤
var appErr *errors.AppError
if errors.As(err, &appErr) {
    log.Error("app error", "code", appErr.Code)
}
```

### 3. 多錯誤處理

```go
// 收集多個錯誤
var errs []error

if err := validateEmail(req.Email); err != nil {
    errs = append(errs, err)
}

if err := validatePassword(req.Password); err != nil {
    errs = append(errs, err)
}

if len(errs) > 0 {
    return nil, errors.ValidationError(
        "validation failed",
        fmt.Errorf("multiple errors: %v", errs),
    )
}
```

### 4. Defer 中的錯誤處理

```go
func processFile(path string) (err error) {
    f, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("open file: %w", err)
    }

    defer func() {
        if closeErr := f.Close(); closeErr != nil {
            // 如果已有錯誤，記錄 close 錯誤但不覆蓋
            if err != nil {
                log.Error("failed to close file", "error", closeErr)
            } else {
                // 沒有其他錯誤，回傳 close 錯誤
                err = fmt.Errorf("close file: %w", closeErr)
            }
        }
    }()

    // Process file...
    return nil
}
```

---

## 總結

### 錯誤處理最佳實踐

1. ✅ **所有錯誤都要處理** - 不忽略任何 error
2. ✅ **使用自訂錯誤類型** - AppError, HTTPError
3. ✅ **錯誤分層轉換** - Repository → Service → Handler
4. ✅ **統一錯誤格式** - RFC 7807 Problem Details
5. ✅ **使用錯誤中介軟體** - 統一處理錯誤回應
6. ✅ **記錄錯誤日誌** - 包含 Trace ID 與上下文
7. ✅ **避免洩漏敏感資訊** - 不回傳 SQL 錯誤、內部路徑
8. ✅ **使用 errors.Is / errors.As** - 判斷錯誤類型

### 避免的錯誤模式

1. ❌ 使用 panic 處理業務錯誤
2. ❌ 忽略錯誤回傳值（`_`）
3. ❌ 不包裝錯誤（失去上下文）
4. ❌ 回傳 nil error 與 nil result
5. ❌ 在日誌中記錄敏感資訊
6. ❌ 使用字串比對判斷錯誤類型

### 相關資源

- **Middleware 實作**：`guides/middleware-patterns.md`
- **Handler 範例**：`examples/handler-example.md`
- **Service 範例**：`examples/service-example.md`
- **Testing 範例**：`examples/testing-example.md`
