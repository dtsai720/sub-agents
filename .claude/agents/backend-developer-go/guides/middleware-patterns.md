# Go HTTP 中介軟體模式與實作

## 目錄

1. [中介軟體基礎概念](#中介軟體基礎概念)
2. [錯誤處理中介軟體](#錯誤處理中介軟體)
3. [日誌中介軟體](#日誌中介軟體)
4. [認證中介軟體](#認證中介軟體)
5. [CORS 中介軟體](#cors-中介軟體)
6. [Rate Limiting 中介軟體](#rate-limiting-中介軟體)
7. [中介軟體順序與最佳實踐](#中介軟體順序與最佳實踐)

---

## 中介軟體基礎概念

### 什麼是中介軟體？

中介軟體（Middleware）是 HTTP 請求處理鏈中的一個環節，可以：
- 在請求到達 Handler 前處理請求（Pre-processing）
- 在 Handler 執行後處理回應（Post-processing）
- 修改請求或回應
- 提前終止請求處理

### 中介軟體模式（框架無關）

```
Request → Middleware 1 → Middleware 2 → Handler → Middleware 2 → Middleware 1 → Response
             ↓                ↓                       ↑                ↑
          Logging          Auth                   Error          Recovery
```

### 框架特定實作方式

不同框架的中介軟體實作方式不同：

**Gin:**
```go
func MyMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // Before request
        c.Next() // 執行下一個 handler
        // After request
    }
}
```

**Echo:**
```go
func MyMiddleware(next echo.HandlerFunc) echo.HandlerFunc {
    return func(c echo.Context) error {
        // Before request
        err := next(c) // 執行下一個 handler
        // After request
        return err
    }
}
```

**Fiber:**
```go
func MyMiddleware(c *fiber.Ctx) error {
    // Before request
    err := c.Next() // 執行下一個 handler
    // After request
    return err
}
```

---

## 錯誤處理中介軟體

### 核心需求（框架無關）

1. **統一錯誤回應格式** - RFC 7807 Problem Details
2. **區分錯誤類型** - 業務錯誤（4xx）vs 系統錯誤（5xx）
3. **記錄系統錯誤** - 包含 stack trace
4. **避免洩漏敏感資訊** - 不回傳 SQL 錯誤、內部路徑
5. **支援自訂錯誤類型** - AppError, HTTPError

### Gin 實作

```go
// internal/middleware/error_handler.go

package middleware

import (
    "net/http"
    "github.com/gin-gonic/gin"
    "myapp/internal/errors"
    "log/slog"
)

// ErrorHandler Gin 錯誤處理中介軟體
func ErrorHandler(logger *slog.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next() // 執行後續 handlers

        // 檢查是否有錯誤
        if len(c.Errors) == 0 {
            return
        }

        // 取最後一個錯誤
        err := c.Errors.Last().Err

        // 提取 Trace ID
        traceID, _ := c.Get("trace_id")
        traceIDStr := ""
        if traceID != nil {
            traceIDStr = traceID.(string)
        }

        // 轉換為 Problem Detail
        problem := errors.ToProblemDetail(err, c.Request.URL.Path, traceIDStr)

        // 記錄錯誤（僅 5xx）
        if problem.Status >= 500 {
            logger.Error("internal server error",
                slog.String("error", err.Error()),
                slog.String("trace_id", traceIDStr),
                slog.String("path", c.Request.URL.Path),
                slog.String("method", c.Request.Method),
                slog.Int("status", problem.Status),
            )
        } else {
            // 4xx 錯誤僅記錄 info（非錯誤）
            logger.Info("client error",
                slog.String("error", err.Error()),
                slog.String("trace_id", traceIDStr),
                slog.String("path", c.Request.URL.Path),
                slog.Int("status", problem.Status),
            )
        }

        // 避免重複回應
        if c.Writer.Written() {
            return
        }

        // 回應
        c.JSON(problem.Status, problem)
    }
}

// Recovery Panic 恢復中介軟體
func Recovery(logger *slog.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        defer func() {
            if r := recover(); r != nil {
                traceID, _ := c.Get("trace_id")
                traceIDStr := ""
                if traceID != nil {
                    traceIDStr = traceID.(string)
                }

                logger.Error("panic recovered",
                    slog.Any("panic", r),
                    slog.String("trace_id", traceIDStr),
                    slog.String("path", c.Request.URL.Path),
                )

                problem := &errors.ProblemDetail{
                    Type:     "https://api.example.com/errors/INTERNAL_ERROR",
                    Title:    "Internal Server Error",
                    Status:   500,
                    Detail:   "An unexpected error occurred",
                    Instance: c.Request.URL.Path,
                    TraceID:  traceIDStr,
                }

                c.JSON(http.StatusInternalServerError, problem)
                c.Abort()
            }
        }()

        c.Next()
    }
}
```

### Echo 實作

```go
// internal/middleware/error_handler.go (Echo)

package middleware

import (
    "net/http"
    "github.com/labstack/echo/v4"
    "myapp/internal/errors"
    "log/slog"
)

// ErrorHandler Echo 錯誤處理中介軟體
func ErrorHandler(logger *slog.Logger) echo.MiddlewareFunc {
    return func(next echo.HandlerFunc) echo.HandlerFunc {
        return func(c echo.Context) error {
            err := next(c)
            if err == nil {
                return nil
            }

            // 提取 Trace ID
            traceID := c.Get("trace_id")
            traceIDStr := ""
            if traceID != nil {
                traceIDStr = traceID.(string)
            }

            // 轉換為 Problem Detail
            problem := errors.ToProblemDetail(err, c.Request().URL.Path, traceIDStr)

            // 記錄錯誤
            if problem.Status >= 500 {
                logger.Error("internal server error",
                    slog.String("error", err.Error()),
                    slog.String("trace_id", traceIDStr),
                    slog.String("path", c.Request().URL.Path),
                    slog.Int("status", problem.Status),
                )
            } else {
                logger.Info("client error",
                    slog.String("error", err.Error()),
                    slog.String("trace_id", traceIDStr),
                    slog.String("path", c.Request().URL.Path),
                    slog.Int("status", problem.Status),
                )
            }

            // 回應
            return c.JSON(problem.Status, problem)
        }
    }
}

// Recovery Echo Panic 恢復中介軟體
func Recovery(logger *slog.Logger) echo.MiddlewareFunc {
    return func(next echo.HandlerFunc) echo.HandlerFunc {
        return func(c echo.Context) (err error) {
            defer func() {
                if r := recover(); r != nil {
                    traceID := c.Get("trace_id")
                    traceIDStr := ""
                    if traceID != nil {
                        traceIDStr = traceID.(string)
                    }

                    logger.Error("panic recovered",
                        slog.Any("panic", r),
                        slog.String("trace_id", traceIDStr),
                        slog.String("path", c.Request().URL.Path),
                    )

                    problem := &errors.ProblemDetail{
                        Type:     "https://api.example.com/errors/INTERNAL_ERROR",
                        Title:    "Internal Server Error",
                        Status:   500,
                        Detail:   "An unexpected error occurred",
                        Instance: c.Request().URL.Path,
                        TraceID:  traceIDStr,
                    }

                    err = c.JSON(http.StatusInternalServerError, problem)
                }
            }()

            return next(c)
        }
    }
}
```

### Fiber 實作

```go
// internal/middleware/error_handler.go (Fiber)

package middleware

import (
    "github.com/gofiber/fiber/v2"
    "myapp/internal/errors"
    "log/slog"
)

// ErrorHandler Fiber 全域錯誤處理器
func ErrorHandler(logger *slog.Logger) fiber.ErrorHandler {
    return func(c *fiber.Ctx, err error) error {
        // 提取 Trace ID
        traceID := c.Locals("trace_id")
        traceIDStr := ""
        if traceID != nil {
            traceIDStr = traceID.(string)
        }

        // 轉換為 Problem Detail
        problem := errors.ToProblemDetail(err, c.Path(), traceIDStr)

        // 記錄錯誤
        if problem.Status >= 500 {
            logger.Error("internal server error",
                slog.String("error", err.Error()),
                slog.String("trace_id", traceIDStr),
                slog.String("path", c.Path()),
                slog.Int("status", problem.Status),
            )
        } else {
            logger.Info("client error",
                slog.String("error", err.Error()),
                slog.String("trace_id", traceIDStr),
                slog.String("path", c.Path()),
                slog.Int("status", problem.Status),
            )
        }

        // 回應
        return c.Status(problem.Status).JSON(problem)
    }
}

// Recovery Fiber Panic 恢復中介軟體
func Recovery(logger *slog.Logger) fiber.Handler {
    return func(c *fiber.Ctx) error {
        defer func() {
            if r := recover(); r != nil {
                traceID := c.Locals("trace_id")
                traceIDStr := ""
                if traceID != nil {
                    traceIDStr = traceID.(string)
                }

                logger.Error("panic recovered",
                    slog.Any("panic", r),
                    slog.String("trace_id", traceIDStr),
                    slog.String("path", c.Path()),
                )

                problem := &errors.ProblemDetail{
                    Type:     "https://api.example.com/errors/INTERNAL_ERROR",
                    Title:    "Internal Server Error",
                    Status:   500,
                    Detail:   "An unexpected error occurred",
                    Instance: c.Path(),
                    TraceID:  traceIDStr,
                }

                c.Status(500).JSON(problem)
            }
        }()

        return c.Next()
    }
}
```

---

## 日誌中介軟體

### Gin 實作

```go
// internal/middleware/logger.go (Gin)

func Logger(logger *slog.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        path := c.Request.URL.Path
        method := c.Request.Method

        // 執行請求
        c.Next()

        // 計算延遲
        latency := time.Since(start)
        statusCode := c.Writer.Status()

        // 提取 Trace ID
        traceID, _ := c.Get("trace_id")
        traceIDStr := ""
        if traceID != nil {
            traceIDStr = traceID.(string)
        }

        // 記錄請求
        logger.Info("http request",
            slog.String("method", method),
            slog.String("path", path),
            slog.Int("status", statusCode),
            slog.Duration("latency", latency),
            slog.String("client_ip", c.ClientIP()),
            slog.String("trace_id", traceIDStr),
            slog.String("user_agent", c.Request.UserAgent()),
        )
    }
}

// TraceMiddleware Trace ID 中介軟體
func TraceMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        traceID := c.GetHeader("X-Trace-ID")
        if traceID == "" {
            traceID = generateTraceID()
        }

        c.Set("trace_id", traceID)
        c.Header("X-Trace-ID", traceID)

        c.Next()
    }
}

func generateTraceID() string {
    return fmt.Sprintf("%d-%s", time.Now().UnixNano(), uuid.New().String()[:8])
}
```

---

## 認證中介軟體

### JWT 認證範例（Gin）

```go
// internal/middleware/auth.go (Gin)

func JWTAuth(jwtSecret string) gin.HandlerFunc {
    return func(c *gin.Context) {
        // 提取 Authorization header
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.Error(errors.UnauthorizedHTTP("missing authorization header"))
            c.Abort()
            return
        }

        // 檢查 Bearer 格式
        parts := strings.SplitN(authHeader, " ", 2)
        if len(parts) != 2 || parts[0] != "Bearer" {
            c.Error(errors.UnauthorizedHTTP("invalid authorization header format"))
            c.Abort()
            return
        }

        tokenString := parts[1]

        // 解析 JWT
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            // 驗證簽名演算法
            if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
            }
            return []byte(jwtSecret), nil
        })

        if err != nil || !token.Valid {
            c.Error(errors.UnauthorizedHTTP("invalid or expired token"))
            c.Abort()
            return
        }

        // 提取 claims
        claims, ok := token.Claims.(jwt.MapClaims)
        if !ok {
            c.Error(errors.UnauthorizedHTTP("invalid token claims"))
            c.Abort()
            return
        }

        // 儲存使用者資訊
        userID, ok := claims["user_id"].(float64)
        if !ok {
            c.Error(errors.UnauthorizedHTTP("invalid user_id in token"))
            c.Abort()
            return
        }

        c.Set("user_id", int(userID))
        c.Set("user_email", claims["email"])

        c.Next()
    }
}

// Optional JWT（允許匿名存取，但會解析 JWT）
func OptionalJWTAuth(jwtSecret string) gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            // 沒有 token，繼續執行（匿名存取）
            c.Next()
            return
        }

        // 有 token，嘗試解析
        parts := strings.SplitN(authHeader, " ", 2)
        if len(parts) == 2 && parts[0] == "Bearer" {
            tokenString := parts[1]
            token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
                return []byte(jwtSecret), nil
            })

            if err == nil && token.Valid {
                claims, ok := token.Claims.(jwt.MapClaims)
                if ok {
                    if userID, ok := claims["user_id"].(float64); ok {
                        c.Set("user_id", int(userID))
                        c.Set("user_email", claims["email"])
                    }
                }
            }
        }

        c.Next()
    }
}
```

---

## CORS 中介軟體

### Gin 實作

```go
// internal/middleware/cors.go (Gin)

func CORS() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("Access-Control-Allow-Origin", "*") // 生產環境應指定具體域名
        c.Header("Access-Control-Allow-Methods", "GET, POST, PUT, PATCH, DELETE, OPTIONS")
        c.Header("Access-Control-Allow-Headers", "Origin, Content-Type, Authorization, X-Trace-ID")
        c.Header("Access-Control-Expose-Headers", "X-Trace-ID")
        c.Header("Access-Control-Max-Age", "86400") // 24 hours

        // 處理 OPTIONS 預檢請求
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(204)
            return
        }

        c.Next()
    }
}

// 生產環境 CORS（指定域名）
func ProductionCORS(allowedOrigins []string) gin.HandlerFunc {
    return func(c *gin.Context) {
        origin := c.GetHeader("Origin")

        // 檢查 Origin 是否在允許清單
        allowed := false
        for _, allowedOrigin := range allowedOrigins {
            if origin == allowedOrigin {
                allowed = true
                break
            }
        }

        if allowed {
            c.Header("Access-Control-Allow-Origin", origin)
            c.Header("Access-Control-Allow-Credentials", "true")
            c.Header("Access-Control-Allow-Methods", "GET, POST, PUT, PATCH, DELETE, OPTIONS")
            c.Header("Access-Control-Allow-Headers", "Origin, Content-Type, Authorization, X-Trace-ID")
            c.Header("Access-Control-Expose-Headers", "X-Trace-ID")
            c.Header("Access-Control-Max-Age", "86400")
        }

        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(204)
            return
        }

        c.Next()
    }
}
```

---

## Rate Limiting 中介軟體

### Gin 實作（基於 IP）

```go
// internal/middleware/rate_limit.go (Gin)

import (
    "sync"
    "time"
    "golang.org/x/time/rate"
)

type IPRateLimiter struct {
    ips map[string]*rate.Limiter
    mu  sync.RWMutex
    r   rate.Limit // 每秒允許的請求數
    b   int        // Burst size
}

func NewIPRateLimiter(r rate.Limit, b int) *IPRateLimiter {
    return &IPRateLimiter{
        ips: make(map[string]*rate.Limiter),
        r:   r,
        b:   b,
    }
}

func (i *IPRateLimiter) GetLimiter(ip string) *rate.Limiter {
    i.mu.Lock()
    defer i.mu.Unlock()

    limiter, exists := i.ips[ip]
    if !exists {
        limiter = rate.NewLimiter(i.r, i.b)
        i.ips[ip] = limiter
    }

    return limiter
}

func RateLimit(rateLimiter *IPRateLimiter) gin.HandlerFunc {
    return func(c *gin.Context) {
        ip := c.ClientIP()
        limiter := rateLimiter.GetLimiter(ip)

        if !limiter.Allow() {
            c.Error(errors.NewHTTPError(
                429,
                "RATE_LIMIT_EXCEEDED",
                "Too many requests, please try again later",
                nil,
            ))
            c.Abort()
            return
        }

        c.Next()
    }
}

// 使用範例（在 main.go）
// rateLimiter := middleware.NewIPRateLimiter(10, 20) // 每秒 10 個請求，burst 20
// router.Use(middleware.RateLimit(rateLimiter))
```

---

## 中介軟體順序與最佳實踐

### 推薦順序（由外到內）

```go
// main.go (Gin 範例)

func setupRouter(logger *slog.Logger, config *config.Config) *gin.Engine {
    router := gin.New() // 不使用預設中介軟體

    // 1. Recovery（最外層，捕獲 panic）
    router.Use(middleware.Recovery(logger))

    // 2. CORS（處理預檢請求）
    router.Use(middleware.CORS())

    // 3. Trace ID（生成追蹤 ID）
    router.Use(middleware.TraceMiddleware())

    // 4. Logger（記錄請求）
    router.Use(middleware.Logger(logger))

    // 5. Rate Limiting（限流）
    rateLimiter := middleware.NewIPRateLimiter(100, 200)
    router.Use(middleware.RateLimit(rateLimiter))

    // 6. Error Handler（最內層，處理錯誤回應）
    router.Use(middleware.ErrorHandler(logger))

    // Public routes（無需認證）
    router.POST("/auth/login", authHandler.Login)
    router.POST("/auth/register", authHandler.Register)

    // Protected routes（需要認證）
    authorized := router.Group("/api/v1")
    authorized.Use(middleware.JWTAuth(config.JWTSecret))
    {
        authorized.GET("/users/:id", userHandler.GetUser)
        authorized.PUT("/users/:id", userHandler.UpdateUser)
        // ...
    }

    return router
}
```

### 中介軟體順序原則

1. **Recovery** - 最外層，捕獲所有 panic
2. **CORS** - 處理預檢請求（OPTIONS），提前回應
3. **Trace ID** - 生成追蹤 ID，後續中介軟體可使用
4. **Logger** - 記錄請求（包含 Trace ID）
5. **Rate Limiting** - 限流（提前拒絕過多請求）
6. **Error Handler** - 統一錯誤處理（最內層）
7. **Auth** - 認證（僅保護特定路由）

### 常見錯誤

❌ **錯誤順序範例：**
```go
// ❌ Bad: Error Handler 在 Logger 之前
router.Use(middleware.ErrorHandler(logger))
router.Use(middleware.Logger(logger))
// 問題：錯誤回應不會被記錄

// ❌ Bad: Recovery 不在最外層
router.Use(middleware.Logger(logger))
router.Use(middleware.Recovery(logger))
// 問題：Logger 中的 panic 不會被捕獲

// ❌ Bad: Trace ID 在 Logger 之後
router.Use(middleware.Logger(logger))
router.Use(middleware.TraceMiddleware())
// 問題：日誌中沒有 Trace ID
```

---

## 總結

### 中介軟體開發最佳實踐

1. ✅ **遵循框架慣例** - 使用框架推薦的中介軟體模式
2. ✅ **正確排序** - Recovery → CORS → Trace → Logger → Rate Limit → Error → Auth
3. ✅ **避免重複邏輯** - 在中介軟體中統一處理
4. ✅ **使用 Context 傳遞資料** - Trace ID、User ID 等
5. ✅ **記錄關鍵事件** - 請求、錯誤、認證失敗
6. ✅ **區分環境** - 開發/生產使用不同中介軟體配置
7. ✅ **效能考量** - 避免在中介軟體中執行重複的資料庫查詢

### 相關資源

- **錯誤處理**：`guides/error-handling-guide.md`
- **Handler 範例**：`examples/handler-example.md`
- **完整應用範例**：`examples/gorm-example.md`, `examples/handler-example.md`
