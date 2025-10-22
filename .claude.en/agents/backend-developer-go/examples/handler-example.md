# Handler & Middleware 完整範例

## Handler 層範例

### Handler 結構定義

```go
package handler

import (
    "net/http"

    "github.com/gin-gonic/gin"
    "github.com/google/uuid"
    "your-project/internal/service"
)

type UserHandler struct {
    service service.UserService
}

func NewUserHandler(service service.UserService) *UserHandler {
    return &UserHandler{service: service}
}
```

### 路由註冊

```go
func (h *UserHandler) RegisterRoutes(router *gin.RouterGroup) {
    users := router.Group("/users")
    {
        users.POST("", h.CreateUser)
        users.GET("/:id", h.GetUser)
        users.GET("", h.ListUsers)
        users.PUT("/:id", h.UpdateUser)
        users.DELETE("/:id", h.Delete)
    }
}
```

### HTTP Handlers 實作

```go
// CreateUser handles POST /users
func (h *UserHandler) CreateUser(c *gin.Context) {
    var req service.CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user, err := h.service.CreateUser(c.Request.Context(), req)
    if err != nil {
        // Error handling middleware will handle this
        _ = c.Error(err)
        return
    }

    c.JSON(http.StatusCreated, user)
}

// GetUser handles GET /users/:id
func (h *UserHandler) GetUser(c *gin.Context) {
    id, err := uuid.Parse(c.Param("id"))
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid user id"})
        return
    }

    user, err := h.service.GetUser(c.Request.Context(), id)
    if err != nil {
        _ = c.Error(err)
        return
    }

    c.JSON(http.StatusOK, user)
}

// ListUsers handles GET /users?page=1&limit=10
func (h *UserHandler) ListUsers(c *gin.Context) {
    // Parse query parameters with defaults
    page := c.DefaultQuery("page", "1")
    limit := c.DefaultQuery("limit", "10")

    pageInt := 1
    limitInt := 10

    fmt.Sscanf(page, "%d", &pageInt)
    fmt.Sscanf(limit, "%d", &limitInt)

    // Validate pagination
    if pageInt < 1 {
        pageInt = 1
    }
    if limitInt < 1 || limitInt > 100 {
        limitInt = 10
    }

    offset := (pageInt - 1) * limitInt

    users, total, err := h.service.ListUsers(c.Request.Context(), offset, limitInt)
    if err != nil {
        _ = c.Error(err)
        return
    }

    c.JSON(http.StatusOK, gin.H{
        "data": users,
        "pagination": gin.H{
            "total":   total,
            "page":    pageInt,
            "limit":   limitInt,
            "pages":   (total + int64(limitInt) - 1) / int64(limitInt),
        },
    })
}

// UpdateUser handles PUT /users/:id
func (h *UserHandler) UpdateUser(c *gin.Context) {
    id, err := uuid.Parse(c.Param("id"))
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid user id"})
        return
    }

    var req service.UpdateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    user, err := h.service.UpdateUser(c.Request.Context(), id, req)
    if err != nil {
        _ = c.Error(err)
        return
    }

    c.JSON(http.StatusOK, user)
}

// DeleteUser handles DELETE /users/:id
func (h *UserHandler) DeleteUser(c *gin.Context) {
    id, err := uuid.Parse(c.Param("id"))
    if err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": "invalid user id"})
        return
    }

    if err := h.service.DeleteUser(c.Request.Context(), id); err != nil {
        _ = c.Error(err)
        return
    }

    c.Status(http.StatusNoContent)
}
```

---

## Middleware 範例

### 錯誤處理 Middleware（RFC 7807 格式）

```go
package middleware

import (
    "errors"
    "log/slog"
    "net/http"

    "github.com/gin-gonic/gin"
    "your-project/internal/apperrors"
)

// ErrorHandler is a middleware that handles errors
func ErrorHandler(logger *slog.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next() // Process request

        // Check if there are any errors
        if len(c.Errors) == 0 {
            return
        }

        err := c.Errors.Last().Err

        // Handle different error types
        var appErr *apperrors.AppError
        if errors.As(err, &appErr) {
            c.JSON(appErr.StatusCode, gin.H{
                "type":     appErr.Type,
                "title":    appErr.Title,
                "status":   appErr.StatusCode,
                "detail":   appErr.Detail,
                "instance": c.Request.URL.Path,
            })
            return
        }

        // Log unexpected errors
        logger.Error("unexpected error",
            "error", err,
            "path", c.Request.URL.Path,
            "method", c.Request.Method,
        )

        // Generic error response
        c.JSON(http.StatusInternalServerError, gin.H{
            "type":     "internal_server_error",
            "title":    "Internal Server Error",
            "status":   http.StatusInternalServerError,
            "detail":   "An unexpected error occurred",
            "instance": c.Request.URL.Path,
        })
    }
}
```

### 日誌 Middleware

```go
package middleware

import (
    "log/slog"
    "time"

    "github.com/gin-gonic/gin"
    "github.com/google/uuid"
)

// Logger is a middleware that logs HTTP requests
func Logger(logger *slog.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        // Generate request ID
        requestID := uuid.New().String()
        c.Set("request_id", requestID)

        start := time.Now()
        path := c.Request.URL.Path
        query := c.Request.URL.RawQuery

        // Process request
        c.Next()

        // Calculate latency
        latency := time.Since(start)

        // Log request details
        logger.Info("http request",
            "request_id", requestID,
            "method", c.Request.Method,
            "path", path,
            "query", query,
            "status", c.Writer.Status(),
            "latency_ms", latency.Milliseconds(),
            "client_ip", c.ClientIP(),
            "user_agent", c.Request.UserAgent(),
        )
    }
}
```

### CORS Middleware

```go
package middleware

import (
    "github.com/gin-gonic/gin"
)

// CORS is a middleware that enables CORS
func CORS() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
        c.Writer.Header().Set("Access-Control-Allow-Credentials", "true")
        c.Writer.Header().Set("Access-Control-Allow-Headers", "Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization, accept, origin, Cache-Control, X-Requested-With")
        c.Writer.Header().Set("Access-Control-Allow-Methods", "POST, OPTIONS, GET, PUT, DELETE, PATCH")

        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(204)
            return
        }

        c.Next()
    }
}
```

### 認證 Middleware（JWT 範例）

```go
package middleware

import (
    "net/http"
    "strings"

    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v5"
)

// AuthMiddleware validates JWT tokens
func AuthMiddleware(jwtSecret string) gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "authorization header required"})
            c.Abort()
            return
        }

        // Extract token from "Bearer <token>"
        tokenString := strings.TrimPrefix(authHeader, "Bearer ")
        if tokenString == authHeader {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "invalid authorization format"})
            c.Abort()
            return
        }

        // Parse and validate token
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (any, error) {
            return []byte(jwtSecret), nil
        })

        if err != nil || !token.Valid {
            c.JSON(http.StatusUnauthorized, gin.H{"error": "invalid token"})
            c.Abort()
            return
        }

        // Extract claims
        if claims, ok := token.Claims.(jwt.MapClaims); ok {
            c.Set("user_id", claims["user_id"])
            c.Set("email", claims["email"])
        }

        c.Next()
    }
}
```

### Rate Limiting Middleware

```go
package middleware

import (
    "net/http"
    "sync"
    "time"

    "github.com/gin-gonic/gin"
)

type rateLimiter struct {
    requests map[string][]time.Time
    mu       sync.Mutex
    limit    int
    window   time.Duration
}

func NewRateLimiter(limit int, window time.Duration) *rateLimiter {
    return &rateLimiter{
        requests: make(map[string][]time.Time),
        limit:    limit,
        window:   window,
    }
}

func (rl *rateLimiter) Middleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        clientIP := c.ClientIP()

        rl.mu.Lock()
        defer rl.mu.Unlock()

        now := time.Now()
        requests := rl.requests[clientIP]

        // Remove old requests outside the window
        var valid []time.Time
        for _, t := range requests {
            if now.Sub(t) < rl.window {
                valid = append(valid, t)
            }
        }

        // Check if limit exceeded
        if len(valid) >= rl.limit {
            c.JSON(http.StatusTooManyRequests, gin.H{
                "error": "rate limit exceeded",
            })
            c.Abort()
            return
        }

        // Add current request
        valid = append(valid, now)
        rl.requests[clientIP] = valid

        c.Next()
    }
}
```

---

## 完整 main.go 整合範例

```go
package main

import (
    "context"
    "log/slog"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gin-gonic/gin"
    "your-project/internal/config"
    "your-project/internal/handler"
    "your-project/internal/middleware"
    "your-project/internal/repository"
    "your-project/internal/service"
)

func main() {
    // Initialize logger
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))

    // Initialize database
    db, err := config.InitDB()
    if err != nil {
        logger.Error("failed to initialize database", "error", err)
        os.Exit(1)
    }

    // Initialize repositories
    userRepo := repository.NewUserRepository(db, logger)

    // Initialize services
    userService := service.NewUserService(userRepo, logger)

    // Initialize handlers
    userHandler := handler.NewUserHandler(userService)

    // Setup Gin router
    router := gin.New()

    // Global middleware
    router.Use(middleware.Logger(logger))
    router.Use(middleware.ErrorHandler(logger))
    router.Use(middleware.CORS())
    router.Use(gin.Recovery())

    // Health check
    router.GET("/health", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"status": "ok"})
    })

    // API routes
    v1 := router.Group("/api/v1")
    {
        userHandler.RegisterRoutes(v1)
    }

    // Setup HTTP server
    srv := &http.Server{
        Addr:    ":8080",
        Handler: router,
    }

    // Start server in goroutine
    go func() {
        logger.Info("starting server", "port", 8080)
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            logger.Error("server error", "error", err)
            os.Exit(1)
        }
    }()

    // Graceful Shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    logger.Info("shutting down server...")

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        logger.Error("server forced to shutdown", "error", err)
    }

    logger.Info("server exited")
}
```

---

## 自訂錯誤類型（RFC 7807）

```go
package apperrors

import "net/http"

type AppError struct {
    Type       string `json:"type"`
    Title      string `json:"title"`
    StatusCode int    `json:"status"`
    Detail     string `json:"detail"`
    Err        error  `json:"-"`
}

func (e *AppError) Error() string {
    return e.Detail
}

func (e *AppError) Unwrap() error {
    return e.Err
}

// Common errors
func NewNotFoundError(resource, detail string) *AppError {
    return &AppError{
        Type:       resource + "_not_found",
        Title:      resource + " Not Found",
        StatusCode: http.StatusNotFound,
        Detail:     detail,
    }
}

func NewBadRequestError(detail string) *AppError {
    return &AppError{
        Type:       "bad_request",
        Title:      "Bad Request",
        StatusCode: http.StatusBadRequest,
        Detail:     detail,
    }
}

func NewConflictError(resource, detail string) *AppError {
    return &AppError{
        Type:       resource + "_conflict",
        Title:      resource + " Conflict",
        StatusCode: http.StatusConflict,
        Detail:     detail,
    }
}

func NewInternalError(detail string) *AppError {
    return &AppError{
        Type:       "internal_server_error",
        Title:      "Internal Server Error",
        StatusCode: http.StatusInternalServerError,
        Detail:     detail,
    }
}
```

---

## 最佳實踐

### 1. 統一錯誤處理
- 使用中介軟體統一處理錯誤
- Handler 中使用 `c.Error(err)` 收集錯誤
- 不在 Handler 中直接回傳 JSON 錯誤

### 2. 依賴注入
- Handler 透過 Constructor 接收 Service
- 避免在 Handler 中直接使用全域變數

### 3. 路由組織
- 使用 `RegisterRoutes` 方法註冊路由
- 按資源分組（users、orders、products）

### 4. 請求驗證
- 使用 `ShouldBindJSON` 綁定與驗證
- 使用 `binding` tags 定義驗證規則

### 5. 分頁處理
- 使用 query parameters（page、limit）
- 回傳 pagination 元資料（total、pages）

### 6. 日誌記錄
- 使用結構化日誌（slog）
- 記錄 request_id 追蹤請求
