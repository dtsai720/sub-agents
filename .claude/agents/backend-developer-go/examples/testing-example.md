# 測試完整範例

## 單元測試（Service 層）

### 使用 gomock 生成 Mock

```bash
# 安裝 mockgen
go install go.uber.org/mock/mockgen@latest

# 在 Repository 介面檔案加上 generate 註解
// go:generate mockgen -source=user_repository.go -destination=mocks/mock_user_repository.go -package=mocks

# 生成 Mock
go generate ./internal/repository/...
```

### Service 測試範例（使用 Table-Driven Tests）

```go
package service_test

import (
    "context"
    "errors"
    "log/slog"
    "os"
    "testing"

    "github.com/google/uuid"
    "github.com/stretchr/testify/assert"
    "go.uber.org/mock/gomock"

    "your-project/internal/apperrors"
    "your-project/internal/model"
    "your-project/internal/repository"
    "your-project/internal/repository/mocks"
    "your-project/internal/service"
)

func TestUserService_CreateUser(t *testing.T) {
    logger := slog.New(slog.NewTextHandler(os.Stdout, nil))

    tests := []struct {
        name    string
        req     service.CreateUserRequest
        setup   func(repo *mocks.MockUserRepository)
        wantErr bool
        errType error
    }{
        {
            name: "success - create new user",
            req: service.CreateUserRequest{
                Name:  "John Doe",
                Email: "john@example.com",
            },
            setup: func(repo *mocks.MockUserRepository) {
                // Email doesn't exist
                repo.EXPECT().
                    GetByEmail(gomock.Any(), "john@example.com").
                    Return(nil, repository.ErrUserNotFound)

                // Create succeeds
                repo.EXPECT().
                    Create(gomock.Any(), gomock.Any()).
                    DoAndReturn(func(ctx context.Context, user *model.User) error {
                        user.ID = uuid.New()
                        return nil
                    })
            },
            wantErr: false,
        },
        {
            name: "error - email already exists",
            req: service.CreateUserRequest{
                Name:  "John Doe",
                Email: "existing@example.com",
            },
            setup: func(repo *mocks.MockUserRepository) {
                // Email already exists
                repo.EXPECT().
                    GetByEmail(gomock.Any(), "existing@example.com").
                    Return(&model.User{Email: "existing@example.com"}, nil)
            },
            wantErr: true,
            errType: &apperrors.AppError{},
        },
        {
            name: "error - repository error on email check",
            req: service.CreateUserRequest{
                Name:  "John Doe",
                Email: "test@example.com",
            },
            setup: func(repo *mocks.MockUserRepository) {
                // Repository error
                repo.EXPECT().
                    GetByEmail(gomock.Any(), "test@example.com").
                    Return(nil, errors.New("database connection failed"))
            },
            wantErr: true,
        },
        {
            name: "error - repository error on create",
            req: service.CreateUserRequest{
                Name:  "John Doe",
                Email: "new@example.com",
            },
            setup: func(repo *mocks.MockUserRepository) {
                repo.EXPECT().
                    GetByEmail(gomock.Any(), "new@example.com").
                    Return(nil, repository.ErrUserNotFound)

                repo.EXPECT().
                    Create(gomock.Any(), gomock.Any()).
                    Return(errors.New("insert failed"))
            },
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Setup
            ctrl := gomock.NewController(t)
            defer ctrl.Finish()

            mockRepo := mocks.NewMockUserRepository(ctrl)
            tt.setup(mockRepo)

            svc := service.NewUserService(mockRepo, logger)

            // Execute
            user, err := svc.CreateUser(context.Background(), tt.req)

            // Assert
            if tt.wantErr {
                assert.Error(t, err)
                assert.Nil(t, user)

                if tt.errType != nil {
                    assert.IsType(t, tt.errType, err)
                }
            } else {
                assert.NoError(t, err)
                assert.NotNil(t, user)
                assert.Equal(t, tt.req.Email, user.Email)
                assert.Equal(t, tt.req.Name, user.Name)
                assert.NotEqual(t, uuid.Nil, user.ID)
            }
        })
    }
}

func TestUserService_GetUser(t *testing.T) {
    logger := slog.New(slog.NewTextHandler(os.Stdout, nil))

    userID := uuid.New()

    tests := []struct {
        name    string
        id      uuid.UUID
        setup   func(repo *mocks.MockUserRepository)
        wantErr bool
    }{
        {
            name: "success - user found",
            id:   userID,
            setup: func(repo *mocks.MockUserRepository) {
                repo.EXPECT().
                    GetByID(gomock.Any(), userID).
                    Return(&model.User{
                        ID:    userID,
                        Name:  "John Doe",
                        Email: "john@example.com",
                    }, nil)
            },
            wantErr: false,
        },
        {
            name: "error - user not found",
            id:   userID,
            setup: func(repo *mocks.MockUserRepository) {
                repo.EXPECT().
                    GetByID(gomock.Any(), userID).
                    Return(nil, repository.ErrUserNotFound)
            },
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            ctrl := gomock.NewController(t)
            defer ctrl.Finish()

            mockRepo := mocks.NewMockUserRepository(ctrl)
            tt.setup(mockRepo)

            svc := service.NewUserService(mockRepo, logger)

            user, err := svc.GetUser(context.Background(), tt.id)

            if tt.wantErr {
                assert.Error(t, err)
                assert.Nil(t, user)
            } else {
                assert.NoError(t, err)
                assert.NotNil(t, user)
                assert.Equal(t, tt.id, user.ID)
            }
        })
    }
}
```

---

## 整合測試（使用 testcontainers）

### Repository 整合測試

```go
package repository_test

import (
    "context"
    "log/slog"
    "os"
    "testing"

    "github.com/google/uuid"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/suite"
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "github.com/testcontainers/testcontainers-go/wait"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"

    "your-project/internal/model"
    "your-project/internal/repository"
)

type UserRepositoryTestSuite struct {
    suite.Suite
    db        *gorm.DB
    container *postgres.PostgresContainer
    repo      repository.UserRepository
}

// SetupSuite runs once before all tests
func (s *UserRepositoryTestSuite) SetupSuite() {
    ctx := context.Background()

    // Start PostgreSQL container
    pgContainer, err := postgres.RunContainer(ctx,
        testcontainers.WithImage("postgres:15-alpine"),
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("postgres"),
        postgres.WithPassword("postgres"),
        testcontainers.WithWaitStrategy(
            wait.ForLog("database system is ready to accept connections").
                WithOccurrence(2),
        ),
    )
    s.Require().NoError(err)
    s.container = pgContainer

    // Get connection string
    connStr, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
    s.Require().NoError(err)

    // Connect to database
    db, err := gorm.Open(postgresDriver.Open(connStr), &gorm.Config{})
    s.Require().NoError(err)
    s.db = db

    // Auto migrate (僅限測試環境使用 AutoMigrate，生產環境由 DBA 管理 Migration)
    err = db.AutoMigrate(&model.User{})
    s.Require().NoError(err)

    // Initialize repository
    logger := slog.New(slog.NewTextHandler(os.Stdout, nil))
    s.repo = repository.NewUserRepository(db, logger)
}

// TearDownSuite runs once after all tests
func (s *UserRepositoryTestSuite) TearDownSuite() {
    if s.container != nil {
        _ = s.container.Terminate(context.Background())
    }
}

// SetupTest runs before each test
func (s *UserRepositoryTestSuite) SetupTest() {
    // Clean up database before each test
    s.db.Exec("TRUNCATE TABLE users RESTART IDENTITY CASCADE")
}

func (s *UserRepositoryTestSuite) TestCreate() {
    ctx := context.Background()

    user := &model.User{
        Name:   "John Doe",
        Email:  "john@example.com",
        Status: "active",
    }

    err := s.repo.Create(ctx, user)
    s.NoError(err)
    s.NotEqual(uuid.Nil, user.ID)
    s.NotZero(user.CreatedAt)
}

func (s *UserRepositoryTestSuite) TestGetByID() {
    ctx := context.Background()

    // Create user
    user := &model.User{
        Name:   "Jane Doe",
        Email:  "jane@example.com",
        Status: "active",
    }
    err := s.repo.Create(ctx, user)
    s.Require().NoError(err)

    // Get user
    retrieved, err := s.repo.GetByID(ctx, user.ID)
    s.NoError(err)
    s.NotNil(retrieved)
    s.Equal(user.ID, retrieved.ID)
    s.Equal(user.Email, retrieved.Email)
}

func (s *UserRepositoryTestSuite) TestGetByID_NotFound() {
    ctx := context.Background()

    _, err := s.repo.GetByID(ctx, uuid.New())
    s.Error(err)
    s.ErrorIs(err, repository.ErrUserNotFound)
}

func (s *UserRepositoryTestSuite) TestList() {
    ctx := context.Background()

    // Create multiple users
    for i := 0; i < 15; i++ {
        user := &model.User{
            Name:   "User " + string(rune(i)),
            Email:  "user" + string(rune(i)) + "@example.com",
            Status: "active",
        }
        err := s.repo.Create(ctx, user)
        s.Require().NoError(err)
    }

    // Test pagination
    users, total, err := s.repo.List(ctx, 0, 10)
    s.NoError(err)
    s.Equal(int64(15), total)
    s.Len(users, 10)

    // Second page
    users, total, err = s.repo.List(ctx, 10, 10)
    s.NoError(err)
    s.Equal(int64(15), total)
    s.Len(users, 5)
}

func (s *UserRepositoryTestSuite) TestUpdate() {
    ctx := context.Background()

    // Create user
    user := &model.User{
        Name:   "Original Name",
        Email:  "original@example.com",
        Status: "active",
    }
    err := s.repo.Create(ctx, user)
    s.Require().NoError(err)

    // Update user
    user.Name = "Updated Name"
    err = s.repo.Update(ctx, user)
    s.NoError(err)

    // Verify update
    retrieved, err := s.repo.GetByID(ctx, user.ID)
    s.NoError(err)
    s.Equal("Updated Name", retrieved.Name)
}

func (s *UserRepositoryTestSuite) TestDelete() {
    ctx := context.Background()

    // Create user
    user := &model.User{
        Name:   "To Be Deleted",
        Email:  "delete@example.com",
        Status: "active",
    }
    err := s.repo.Create(ctx, user)
    s.Require().NoError(err)

    // Delete user
    err = s.repo.Delete(ctx, user.ID)
    s.NoError(err)

    // Verify deletion (soft delete)
    _, err = s.repo.GetByID(ctx, user.ID)
    s.Error(err)
    s.ErrorIs(err, repository.ErrUserNotFound)
}

// Run the test suite
func TestUserRepositoryTestSuite(t *testing.T) {
    suite.Run(t, new(UserRepositoryTestSuite))
}
```

---

## Handler 測試（使用 httptest）

```go
package handler_test

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/gin-gonic/gin"
    "github.com/google/uuid"
    "github.com/stretchr/testify/assert"
    "go.uber.org/mock/gomock"

    "your-project/internal/handler"
    "your-project/internal/model"
    "your-project/internal/service"
    "your-project/internal/service/mocks"
)

func TestUserHandler_CreateUser(t *testing.T) {
    gin.SetMode(gin.TestMode)

    tests := []struct {
        name       string
        body       any
        setup      func(svc *mocks.MockUserService)
        wantStatus int
    }{
        {
            name: "success",
            body: service.CreateUserRequest{
                Name:  "John Doe",
                Email: "john@example.com",
            },
            setup: func(svc *mocks.MockUserService) {
                svc.EXPECT().
                    CreateUser(gomock.Any(), gomock.Any()).
                    Return(&model.User{
                        ID:    uuid.New(),
                        Name:  "John Doe",
                        Email: "john@example.com",
                    }, nil)
            },
            wantStatus: http.StatusCreated,
        },
        {
            name: "invalid request - missing name",
            body: map[string]any{
                "email": "john@example.com",
            },
            setup:      func(svc *mocks.MockUserService) {},
            wantStatus: http.StatusBadRequest,
        },
        {
            name: "invalid request - invalid email",
            body: service.CreateUserRequest{
                Name:  "John Doe",
                Email: "invalid-email",
            },
            setup:      func(svc *mocks.MockUserService) {},
            wantStatus: http.StatusBadRequest,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Setup
            ctrl := gomock.NewController(t)
            defer ctrl.Finish()

            mockService := mocks.NewMockUserService(ctrl)
            tt.setup(mockService)

            h := handler.NewUserHandler(mockService)

            // Create request
            bodyBytes, _ := json.Marshal(tt.body)
            req := httptest.NewRequest(http.MethodPost, "/users", bytes.NewBuffer(bodyBytes))
            req.Header.Set("Content-Type", "application/json")

            // Create response recorder
            w := httptest.NewRecorder()

            // Setup router
            router := gin.New()
            router.POST("/users", h.CreateUser)

            // Execute
            router.ServeHTTP(w, req)

            // Assert
            assert.Equal(t, tt.wantStatus, w.Code)
        })
    }
}

func TestUserHandler_GetUser(t *testing.T) {
    gin.SetMode(gin.TestMode)

    userID := uuid.New()

    tests := []struct {
        name       string
        id         string
        setup      func(svc *mocks.MockUserService)
        wantStatus int
    }{
        {
            name: "success",
            id:   userID.String(),
            setup: func(svc *mocks.MockUserService) {
                svc.EXPECT().
                    GetUser(gomock.Any(), userID).
                    Return(&model.User{
                        ID:    userID,
                        Name:  "John Doe",
                        Email: "john@example.com",
                    }, nil)
            },
            wantStatus: http.StatusOK,
        },
        {
            name:       "invalid uuid",
            id:         "invalid-uuid",
            setup:      func(svc *mocks.MockUserService) {},
            wantStatus: http.StatusBadRequest,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            ctrl := gomock.NewController(t)
            defer ctrl.Finish()

            mockService := mocks.NewMockUserService(ctrl)
            tt.setup(mockService)

            h := handler.NewUserHandler(mockService)

            req := httptest.NewRequest(http.MethodGet, "/users/"+tt.id, nil)
            w := httptest.NewRecorder()

            router := gin.New()
            router.GET("/users/:id", h.GetUser)

            router.ServeHTTP(w, req)

            assert.Equal(t, tt.wantStatus, w.Code)
        })
    }
}
```

---

## 完整 E2E 測試範例

```go
package integration_test

import (
    "bytes"
    "context"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/gin-gonic/gin"
    "github.com/stretchr/testify/suite"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"

    "your-project/internal/handler"
    "your-project/internal/model"
    "your-project/internal/repository"
    "your-project/internal/service"
)

type E2ETestSuite struct {
    suite.Suite
    db        *gorm.DB
    container *postgres.PostgresContainer
    router    *gin.Engine
}

func (s *E2ETestSuite) SetupSuite() {
    ctx := context.Background()

    // Start PostgreSQL
    pgContainer, err := postgres.RunContainer(ctx, /* ... */)
    s.Require().NoError(err)
    s.container = pgContainer

    connStr, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
    s.Require().NoError(err)

    db, err := gorm.Open(postgresDriver.Open(connStr), &gorm.Config{})
    s.Require().NoError(err)
    s.db = db

    err = db.AutoMigrate(&model.User{})
    s.Require().NoError(err)

    // Setup application
    logger := slog.New(slog.NewTextHandler(os.Stdout, nil))
    userRepo := repository.NewUserRepository(db, logger)
    userService := service.NewUserService(userRepo, logger)
    userHandler := handler.NewUserHandler(userService)

    router := gin.New()
    v1 := router.Group("/api/v1")
    userHandler.RegisterRoutes(v1)

    s.router = router
}

func (s *E2ETestSuite) TearDownSuite() {
    if s.container != nil {
        _ = s.container.Terminate(context.Background())
    }
}

func (s *E2ETestSuite) SetupTest() {
    s.db.Exec("TRUNCATE TABLE users RESTART IDENTITY CASCADE")
}

func (s *E2ETestSuite) TestUserCRUD() {
    // 1. Create user
    createReq := map[string]any{
        "name":  "John Doe",
        "email": "john@example.com",
    }
    body, _ := json.Marshal(createReq)

    req := httptest.NewRequest(http.MethodPost, "/api/v1/users", bytes.NewBuffer(body))
    req.Header.Set("Content-Type", "application/json")
    w := httptest.NewRecorder()

    s.router.ServeHTTP(w, req)
    s.Equal(http.StatusCreated, w.Code)

    var createdUser model.User
    err := json.Unmarshal(w.Body.Bytes(), &createdUser)
    s.NoError(err)
    s.Equal("John Doe", createdUser.Name)

    // 2. Get user
    req = httptest.NewRequest(http.MethodGet, "/api/v1/users/"+createdUser.ID.String(), nil)
    w = httptest.NewRecorder()

    s.router.ServeHTTP(w, req)
    s.Equal(http.StatusOK, w.Code)

    // 3. Update user
    updateReq := map[string]any{
        "name": "Jane Doe",
    }
    body, _ = json.Marshal(updateReq)

    req = httptest.NewRequest(http.MethodPut, "/api/v1/users/"+createdUser.ID.String(), bytes.NewBuffer(body))
    req.Header.Set("Content-Type", "application/json")
    w = httptest.NewRecorder()

    s.router.ServeHTTP(w, req)
    s.Equal(http.StatusOK, w.Code)

    // 4. Delete user
    req = httptest.NewRequest(http.MethodDelete, "/api/v1/users/"+createdUser.ID.String(), nil)
    w = httptest.NewRecorder()

    s.router.ServeHTTP(w, req)
    s.Equal(http.StatusNoContent, w.Code)

    // 5. Verify deletion
    req = httptest.NewRequest(http.MethodGet, "/api/v1/users/"+createdUser.ID.String(), nil)
    w = httptest.NewRecorder()

    s.router.ServeHTTP(w, req)
    s.Equal(http.StatusNotFound, w.Code)
}

func TestE2ETestSuite(t *testing.T) {
    suite.Run(t, new(E2ETestSuite))
}
```

---

## Makefile 測試指令

```makefile
.PHONY: test test-unit test-integration test-coverage

# Run all tests
test:
	go test ./... -v

# Run unit tests only
test-unit:
	go test ./internal/service/... -v

# Run integration tests only
test-integration:
	go test ./tests/integration/... -v

# Run tests with coverage
test-coverage:
	go test ./... -coverprofile=coverage.out
	go tool cover -html=coverage.out -o coverage.html

# Generate mocks
generate-mocks:
	go generate ./...
```

---

## 最佳實踐

### 1. Table-Driven Tests
- 涵蓋正常、異常、邊界案例
- 每個測試案例獨立且明確

### 2. Mock 使用
- Service 測試使用 Mock Repository
- Handler 測試使用 Mock Service
- 使用 gomock 生成 Mock

### 3. 整合測試
- 使用 testcontainers 啟動真實資料庫
- 每個測試前清理資料庫
- 測試完整流程

### 4. Test Suite
- 使用 testify/suite 組織測試
- SetupSuite/TearDownSuite 管理資源
- SetupTest 清理測試資料

### 5. 測試覆蓋率
- 目標：> 80%
- 使用 `go test -cover` 檢查
- 產出 HTML 報告

### 6. 測試隔離
- 每個測試獨立運行
- 不依賴執行順序
- 清理測試資料
