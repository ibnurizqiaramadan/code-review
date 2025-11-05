# 🔵 Golang Backend Code Review Guidelines

Panduan khusus untuk code review Golang backend (Gin, Echo, Fiber, Chi)

---

## 📋 Golang-Specific Error Levels

| Level | Jenis Kesalahan | Contoh Golang | Denda |
|:------|:----------------|:--------------|:------|
| **L1** | Basic Logic | Typo variable, unused import, wrong type | Rp 200 |
| **L2** | Code Style | Non-idiomatic Go, inconsistent naming, no gofmt | Rp 400 |
| **L3** | Logic Error | Goroutine leak, race condition, panic not recovered | Rp 800 |
| **L3.5** | Error Handling | Error diabaikan, panic tanpa recover | Rp 1.000 |
| **L4** | Data Validation | Input tidak divalidasi, SQL injection | Rp 1.200 |
| **L5** | Performance | Goroutine tanpa limit, memory leak, inefficient loop | Rp 1.600 |
| **L6** | Security | Hardcoded credentials, no input sanitization | Rp 2.000 |
| **L7** | Architecture | Logic di handler, tight coupling, no interface | Rp 2.500 |
| **L9** | Testing/Docs | No unit test, API tanpa docs | Rp 600 |

---

## 🎯 Golang Best Practices

### **Case 1: Error Handling**
**❌ Salah:**
```go
// Mengabaikan error
data, _ := ioutil.ReadFile("config.json")

// Panic tanpa recover
func getUser(id string) User {
    user, err := db.FindUser(id)
    if err != nil {
        panic(err) // ❌ Jangan panic di production
    }
    return user
}
```

**✅ Benar:**
```go
// Selalu handle error
data, err := ioutil.ReadFile("config.json")
if err != nil {
    return fmt.Errorf("failed to read config: %w", err)
}

// Return error, jangan panic
func getUser(id string) (*User, error) {
    user, err := db.FindUser(id)
    if err != nil {
        return nil, fmt.Errorf("failed to find user %s: %w", id, err)
    }
    return user, nil
}

// Custom error types
type NotFoundError struct {
    Resource string
    ID       string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with ID %s not found", e.Resource, e.ID)
}

func getUserByID(id string) (*User, error) {
    user, err := db.FindUser(id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, &NotFoundError{Resource: "User", ID: id}
        }
        return nil, err
    }
    return user, nil
}
```

### **Case 2: Goroutine Management**
**❌ Salah:**
```go
// Goroutine leak - tidak ada kontrol
func processUsers(users []User) {
    for _, user := range users {
        go sendEmail(user.Email) // ❌ Unlimited goroutines
    }
}

// Race condition
var counter int
func increment() {
    for i := 0; i < 1000; i++ {
        go func() {
            counter++ // ❌ Race condition
        }()
    }
}
```

**✅ Benar:**
```go
// Gunakan WaitGroup dan limit goroutines
func processUsers(users []User) error {
    var wg sync.WaitGroup
    semaphore := make(chan struct{}, 10) // Max 10 concurrent goroutines
    errChan := make(chan error, len(users))

    for _, user := range users {
        wg.Add(1)
        go func(u User) {
            defer wg.Done()
            semaphore <- struct{}{} // Acquire
            defer func() { <-semaphore }() // Release

            if err := sendEmail(u.Email); err != nil {
                errChan <- fmt.Errorf("failed to send email to %s: %w", u.Email, err)
            }
        }(user)
    }

    wg.Wait()
    close(errChan)

    // Collect errors
    var errs []error
    for err := range errChan {
        errs = append(errs, err)
    }

    if len(errs) > 0 {
        return fmt.Errorf("failed to process %d users: %v", len(errs), errs)
    }

    return nil
}

// Gunakan sync.Mutex untuk race condition
type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

// Atau gunakan atomic
var counter int64
func increment() {
    atomic.AddInt64(&counter, 1)
}
```

### **Case 3: Context untuk Timeout & Cancellation**
**❌ Salah:**
```go
func fetchData() ([]byte, error) {
    resp, err := http.Get("https://api.example.com/data")
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    return ioutil.ReadAll(resp.Body)
}
```

**✅ Benar:**
```go
func fetchData(ctx context.Context) ([]byte, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", "https://api.example.com/data", nil)
    if err != nil {
        return nil, fmt.Errorf("failed to create request: %w", err)
    }

    client := &http.Client{
        Timeout: 10 * time.Second,
    }

    resp, err := client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("failed to fetch data: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("unexpected status code: %d", resp.StatusCode)
    }

    return ioutil.ReadAll(resp.Body)
}

// Usage dengan timeout
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()

data, err := fetchData(ctx)
if err != nil {
    log.Printf("Error: %v", err)
}
```

---

## 🏗️ Project Structure (Clean Architecture)

```
project/
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── domain/
│   │   ├── user.go
│   │   └── errors.go
│   ├── repository/
│   │   ├── user_repository.go
│   │   └── postgres/
│   │       └── user_repository_impl.go
│   ├── service/
│   │   └── user_service.go
│   ├── handler/
│   │   └── user_handler.go
│   └── middleware/
│       ├── auth.go
│       └── logger.go
├── pkg/
│   ├── database/
│   │   └── postgres.go
│   └── validator/
│       └── validator.go
├── config/
│   └── config.go
├── migrations/
└── tests/
```

---

## 🌐 Complete API Example (Gin Framework)

### **1. Domain Layer**
```go
// internal/domain/user.go
package domain

import (
    "time"
)

type User struct {
    ID        string    `json:"id" db:"id"`
    Email     string    `json:"email" db:"email"`
    Name      string    `json:"name" db:"name"`
    Password  string    `json:"-" db:"password"` // Never expose password
    Role      string    `json:"role" db:"role"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
    UpdatedAt time.Time `json:"updated_at" db:"updated_at"`
}

type CreateUserRequest struct {
    Email    string `json:"email" binding:"required,email"`
    Name     string `json:"name" binding:"required,min=2,max=100"`
    Password string `json:"password" binding:"required,min=8"`
    Role     string `json:"role" binding:"omitempty,oneof=user admin"`
}

type UpdateUserRequest struct {
    Email string `json:"email" binding:"omitempty,email"`
    Name  string `json:"name" binding:"omitempty,min=2,max=100"`
}

type LoginRequest struct {
    Email    string `json:"email" binding:"required,email"`
    Password string `json:"password" binding:"required"`
}

type LoginResponse struct {
    Token string `json:"token"`
    User  *User  `json:"user"`
}

// Custom errors
type AppError struct {
    Code       string `json:"code"`
    Message    string `json:"message"`
    StatusCode int    `json:"-"`
}

func (e *AppError) Error() string {
    return e.Message
}

func NewNotFoundError(resource string) *AppError {
    return &AppError{
        Code:       "NOT_FOUND",
        Message:    resource + " not found",
        StatusCode: 404,
    }
}

func NewValidationError(message string) *AppError {
    return &AppError{
        Code:       "VALIDATION_ERROR",
        Message:    message,
        StatusCode: 400,
    }
}
```

### **2. Repository Layer**
```go
// internal/repository/user_repository.go
package repository

import (
    "context"
    "your-project/internal/domain"
)

type UserRepository interface {
    Create(ctx context.Context, user *domain.User) error
    FindByID(ctx context.Context, id string) (*domain.User, error)
    FindByEmail(ctx context.Context, email string) (*domain.User, error)
    FindAll(ctx context.Context, limit, offset int) ([]*domain.User, int, error)
    Update(ctx context.Context, id string, user *domain.User) error
    Delete(ctx context.Context, id string) error
}

// internal/repository/postgres/user_repository_impl.go
package postgres

import (
    "context"
    "database/sql"
    "fmt"
    "your-project/internal/domain"
    "your-project/internal/repository"

    "github.com/jmoiron/sqlx"
)

type userRepository struct {
    db *sqlx.DB
}

func NewUserRepository(db *sqlx.DB) repository.UserRepository {
    return &userRepository{db: db}
}

func (r *userRepository) Create(ctx context.Context, user *domain.User) error {
    query := `
        INSERT INTO users (id, email, name, password, role, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6, $7)
    `
    _, err := r.db.ExecContext(ctx, query,
        user.ID, user.Email, user.Name, user.Password, user.Role,
        user.CreatedAt, user.UpdatedAt,
    )
    if err != nil {
        return fmt.Errorf("failed to create user: %w", err)
    }
    return nil
}

func (r *userRepository) FindByID(ctx context.Context, id string) (*domain.User, error) {
    var user domain.User
    query := `SELECT * FROM users WHERE id = $1`
    
    err := r.db.GetContext(ctx, &user, query, id)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, domain.NewNotFoundError("User")
        }
        return nil, fmt.Errorf("failed to find user: %w", err)
    }
    return &user, nil
}

func (r *userRepository) FindByEmail(ctx context.Context, email string) (*domain.User, error) {
    var user domain.User
    query := `SELECT * FROM users WHERE email = $1`
    
    err := r.db.GetContext(ctx, &user, query, email)
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, nil // Email not found is not an error
        }
        return nil, fmt.Errorf("failed to find user by email: %w", err)
    }
    return &user, nil
}

func (r *userRepository) FindAll(ctx context.Context, limit, offset int) ([]*domain.User, int, error) {
    var users []*domain.User
    query := `SELECT * FROM users ORDER BY created_at DESC LIMIT $1 OFFSET $2`
    
    err := r.db.SelectContext(ctx, &users, query, limit, offset)
    if err != nil {
        return nil, 0, fmt.Errorf("failed to find users: %w", err)
    }

    var total int
    countQuery := `SELECT COUNT(*) FROM users`
    err = r.db.GetContext(ctx, &total, countQuery)
    if err != nil {
        return nil, 0, fmt.Errorf("failed to count users: %w", err)
    }

    return users, total, nil
}
```

### **3. Service Layer**
```go
// internal/service/user_service.go
package service

import (
    "context"
    "fmt"
    "time"
    "your-project/internal/domain"
    "your-project/internal/repository"

    "github.com/golang-jwt/jwt/v5"
    "github.com/google/uuid"
    "golang.org/x/crypto/bcrypt"
)

type UserService interface {
    Register(ctx context.Context, req *domain.CreateUserRequest) (*domain.User, error)
    Login(ctx context.Context, req *domain.LoginRequest) (*domain.LoginResponse, error)
    GetAll(ctx context.Context, page, limit int) ([]*domain.User, int, error)
    GetByID(ctx context.Context, id string) (*domain.User, error)
    Update(ctx context.Context, id string, req *domain.UpdateUserRequest) (*domain.User, error)
    Delete(ctx context.Context, id string) error
}

type userService struct {
    repo      repository.UserRepository
    jwtSecret string
}

func NewUserService(repo repository.UserRepository, jwtSecret string) UserService {
    return &userService{
        repo:      repo,
        jwtSecret: jwtSecret,
    }
}

func (s *userService) Register(ctx context.Context, req *domain.CreateUserRequest) (*domain.User, error) {
    // Check if email exists
    existingUser, err := s.repo.FindByEmail(ctx, req.Email)
    if err != nil {
        return nil, err
    }
    if existingUser != nil {
        return nil, &domain.AppError{
            Code:       "EMAIL_EXISTS",
            Message:    "Email already registered",
            StatusCode: 409,
        }
    }

    // Hash password
    hashedPassword, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
    if err != nil {
        return nil, fmt.Errorf("failed to hash password: %w", err)
    }

    // Create user
    user := &domain.User{
        ID:        uuid.New().String(),
        Email:     req.Email,
        Name:      req.Name,
        Password:  string(hashedPassword),
        Role:      req.Role,
        CreatedAt: time.Now(),
        UpdatedAt: time.Now(),
    }

    if user.Role == "" {
        user.Role = "user"
    }

    if err := s.repo.Create(ctx, user); err != nil {
        return nil, err
    }

    return user, nil
}

func (s *userService) Login(ctx context.Context, req *domain.LoginRequest) (*domain.LoginResponse, error) {
    user, err := s.repo.FindByEmail(ctx, req.Email)
    if err != nil {
        return nil, err
    }
    if user == nil {
        return nil, &domain.AppError{
            Code:       "INVALID_CREDENTIALS",
            Message:    "Invalid email or password",
            StatusCode: 401,
        }
    }

    // Compare password
    err = bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(req.Password))
    if err != nil {
        return nil, &domain.AppError{
            Code:       "INVALID_CREDENTIALS",
            Message:    "Invalid email or password",
            StatusCode: 401,
        }
    }

    // Generate JWT token
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "user_id": user.ID,
        "role":    user.Role,
        "exp":     time.Now().Add(7 * 24 * time.Hour).Unix(),
    })

    tokenString, err := token.SignedString([]byte(s.jwtSecret))
    if err != nil {
        return nil, fmt.Errorf("failed to generate token: %w", err)
    }

    return &domain.LoginResponse{
        Token: tokenString,
        User:  user,
    }, nil
}

func (s *userService) GetAll(ctx context.Context, page, limit int) ([]*domain.User, int, error) {
    if page < 1 {
        page = 1
    }
    if limit < 1 || limit > 100 {
        limit = 10
    }

    offset := (page - 1) * limit
    return s.repo.FindAll(ctx, limit, offset)
}
```

### **4. Handler Layer (Gin)**
```go
// internal/handler/user_handler.go
package handler

import (
    "net/http"
    "strconv"
    "your-project/internal/domain"
    "your-project/internal/service"

    "github.com/gin-gonic/gin"
)

type UserHandler struct {
    service service.UserService
}

func NewUserHandler(service service.UserService) *UserHandler {
    return &UserHandler{service: service}
}

type Response struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   *ErrorData  `json:"error,omitempty"`
    Meta    *Meta       `json:"meta,omitempty"`
}

type ErrorData struct {
    Code    string `json:"code"`
    Message string `json:"message"`
}

type Meta struct {
    Page       int `json:"page"`
    Limit      int `json:"limit"`
    Total      int `json:"total"`
    TotalPages int `json:"total_pages"`
}

func (h *UserHandler) Register(c *gin.Context) {
    var req domain.CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, Response{
            Success: false,
            Error: &ErrorData{
                Code:    "VALIDATION_ERROR",
                Message: err.Error(),
            },
        })
        return
    }

    user, err := h.service.Register(c.Request.Context(), &req)
    if err != nil {
        handleError(c, err)
        return
    }

    c.JSON(http.StatusCreated, Response{
        Success: true,
        Data:    user,
    })
}

func (h *UserHandler) Login(c *gin.Context) {
    var req domain.LoginRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, Response{
            Success: false,
            Error: &ErrorData{
                Code:    "VALIDATION_ERROR",
                Message: err.Error(),
            },
        })
        return
    }

    result, err := h.service.Login(c.Request.Context(), &req)
    if err != nil {
        handleError(c, err)
        return
    }

    c.JSON(http.StatusOK, Response{
        Success: true,
        Data:    result,
    })
}

func (h *UserHandler) GetAll(c *gin.Context) {
    page, _ := strconv.Atoi(c.DefaultQuery("page", "1"))
    limit, _ := strconv.Atoi(c.DefaultQuery("limit", "10"))

    users, total, err := h.service.GetAll(c.Request.Context(), page, limit)
    if err != nil {
        handleError(c, err)
        return
    }

    totalPages := (total + limit - 1) / limit

    c.JSON(http.StatusOK, Response{
        Success: true,
        Data:    users,
        Meta: &Meta{
            Page:       page,
            Limit:      limit,
            Total:      total,
            TotalPages: totalPages,
        },
    })
}

func handleError(c *gin.Context, err error) {
    if appErr, ok := err.(*domain.AppError); ok {
        c.JSON(appErr.StatusCode, Response{
            Success: false,
            Error: &ErrorData{
                Code:    appErr.Code,
                Message: appErr.Message,
            },
        })
        return
    }

    c.JSON(http.StatusInternalServerError, Response{
        Success: false,
        Error: &ErrorData{
            Code:    "INTERNAL_ERROR",
            Message: "Something went wrong",
        },
    })
}
```

### **5. Middleware**
```go
// internal/middleware/auth.go
package middleware

import (
    "net/http"
    "strings"
    "your-project/internal/handler"

    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v5"
)

func AuthMiddleware(jwtSecret string) gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" {
            c.JSON(http.StatusUnauthorized, handler.Response{
                Success: false,
                Error: &handler.ErrorData{
                    Code:    "NO_TOKEN",
                    Message: "Authentication required",
                },
            })
            c.Abort()
            return
        }

        parts := strings.Split(authHeader, " ")
        if len(parts) != 2 || parts[0] != "Bearer" {
            c.JSON(http.StatusUnauthorized, handler.Response{
                Success: false,
                Error: &handler.ErrorData{
                    Code:    "INVALID_TOKEN_FORMAT",
                    Message: "Invalid token format",
                },
            })
            c.Abort()
            return
        }

        tokenString := parts[1]
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            return []byte(jwtSecret), nil
        })

        if err != nil || !token.Valid {
            c.JSON(http.StatusUnauthorized, handler.Response{
                Success: false,
                Error: &handler.ErrorData{
                    Code:    "INVALID_TOKEN",
                    Message: "Invalid or expired token",
                },
            })
            c.Abort()
            return
        }

        claims, ok := token.Claims.(jwt.MapClaims)
        if !ok {
            c.JSON(http.StatusUnauthorized, handler.Response{
                Success: false,
                Error: &handler.ErrorData{
                    Code:    "INVALID_CLAIMS",
                    Message: "Invalid token claims",
                },
            })
            c.Abort()
            return
        }

        c.Set("user_id", claims["user_id"])
        c.Set("role", claims["role"])
        c.Next()
    }
}

func RoleMiddleware(allowedRoles ...string) gin.HandlerFunc {
    return func(c *gin.Context) {
        role, exists := c.Get("role")
        if !exists {
            c.JSON(http.StatusForbidden, handler.Response{
                Success: false,
                Error: &handler.ErrorData{
                    Code:    "FORBIDDEN",
                    Message: "Access denied",
                },
            })
            c.Abort()
            return
        }

        userRole := role.(string)
        allowed := false
        for _, r := range allowedRoles {
            if r == userRole {
                allowed = true
                break
            }
        }

        if !allowed {
            c.JSON(http.StatusForbidden, handler.Response{
                Success: false,
                Error: &handler.ErrorData{
                    Code:    "FORBIDDEN",
                    Message: "Access denied",
                },
            })
            c.Abort()
            return
        }

        c.Next()
    }
}
```

---

## ✅ Golang Checklist

### **Code Quality**
- [ ] Semua error di-handle (no `_` untuk error)
- [ ] Gunakan `gofmt` dan `goimports`
- [ ] No unused imports/variables
- [ ] Idiomatic Go (follow Go conventions)
- [ ] Proper naming (camelCase, exported = PascalCase)

### **Concurrency**
- [ ] Goroutine management (WaitGroup, semaphore)
- [ ] No race conditions (use mutex/atomic)
- [ ] Context untuk timeout & cancellation
- [ ] Channel properly closed

### **Error Handling**
- [ ] Custom error types untuk business logic
- [ ] Error wrapping dengan `fmt.Errorf` dan `%w`
- [ ] No panic di production code
- [ ] Recover dari panic di goroutines

### **Performance**
- [ ] Efficient memory allocation
- [ ] Proper use of pointers
- [ ] Avoid goroutine leaks
- [ ] Connection pooling untuk database

### **Testing**
- [ ] Unit tests dengan table-driven tests
- [ ] Mock interfaces untuk testing
- [ ] Test coverage > 80%
- [ ] Benchmark tests untuk critical paths

---

Lihat [README.md](./README.md) untuk panduan umum backend.
