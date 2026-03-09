# Go Web Framework Security Audit Guide

Use this guide when Phase 1 identifies Go web frameworks: Gin, Echo, Fiber, Chi,
or standard library `net/http`.

## Detection

Confirmed when you see:
- `go.mod` with `github.com/gin-gonic/gin`, `github.com/labstack/echo`, `github.com/gofiber/fiber`, `github.com/go-chi/chi`
- `net/http` import with `http.HandleFunc` or `http.ListenAndServe`
- `main.go` with router setup

## Architecture Awareness

Go web frameworks are lightweight. Security features like auth, CSRF, validation,
and rate limiting are almost always third-party middleware. Check that middleware
is properly applied and ordered.

## Critical Check Areas

### 1. Authentication & Middleware

```
Grep: func.*middleware|Middleware
Grep: Use\(.*auth|Use\(.*jwt
Grep: gin\.HandlerFunc
Grep: echo\.MiddlewareFunc
Grep: fiber\.Handler
Grep: c\.Next\(\)
Grep: jwt-go|golang-jwt
```

Check:
- Route groups without auth middleware
- Admin routes without role-based middleware
- Middleware calling `c.Next()` before auth validation completes
- JWT validation without checking expiry, issuer, or audience
- JWT secret hardcoded in source code
- Auth middleware only checking token format, not claims

### 2. Input Binding & Validation

**Gin:**
```
Grep: ShouldBind|ShouldBindJSON|ShouldBindQuery|ShouldBindUri
Grep: Bind\(|BindJSON\(|BindQuery\(
Grep: binding:"required
Grep: c\.Param\(|c\.Query\(|c\.PostForm\(
```

**Echo:**
```
Grep: c\.Bind\(
Grep: c\.QueryParam\(|c\.FormValue\(|c\.Param\(
```

**Fiber:**
```
Grep: c\.BodyParser\(|c\.Params\(|c\.Query\(
```

Check:
- `c.Param()`, `c.Query()`, `c.PostForm()` used directly in SQL or commands
- Binding without struct validation tags (`binding:"required"`)
- Missing struct validation (`validator` package)
- `ShouldBind` error ignored → invalid input processed
- No body size limit configured

### 3. SQL Injection

```
Grep: fmt\.Sprintf.*SELECT|fmt\.Sprintf.*INSERT|fmt\.Sprintf.*UPDATE|fmt\.Sprintf.*DELETE
Grep: db\.(Query|Exec|QueryRow)\(.*fmt\.Sprintf
Grep: db\.(Query|Exec|QueryRow)\(.*\+
Grep: Sprintf.*WHERE
Grep: Raw\(.*fmt|Raw\(.*\+
```

Check:
- `fmt.Sprintf` building SQL queries → SQL injection
- String concatenation in SQL queries
- Using `db.Query(query + userInput)` instead of `db.Query(query, userInput)`
- GORM `Raw()` with `fmt.Sprintf` → injection
- Safe pattern: `db.Query("SELECT * FROM users WHERE id = $1", id)` with placeholder

### 4. Command Injection

```
Grep: exec\.Command\(
Grep: exec\.CommandContext\(
Grep: os/exec
```

Check:
- `exec.Command()` with user input in command name or arguments
- Note: Go's `exec.Command(name, args...)` is array-form by default (safer than shell)
- But: if command name itself comes from user input → still dangerous
- Shell invocation: `exec.Command("sh", "-c", userInput)` → injection

### 5. Template Injection

```
Grep: template\.HTML\(
Grep: template\.JS\(
Grep: template\.URL\(
Grep: template\.New.*Parse\(
Grep: html/template
Grep: text/template
```

Check:
- `text/template` used instead of `html/template` for HTML → no auto-escaping
- `template.HTML()` type cast on user input → bypasses auto-escaping
- `template.JS()` or `template.URL()` with user input
- Template parsing from user-controlled strings

### 6. CORS Configuration

```
Grep: Access-Control-Allow-Origin
Grep: cors\.New|cors\.Default
Grep: AllowOrigins|AllowAllOrigins
Grep: AllowCredentials
```

Check:
- `AllowAllOrigins: true` or `Access-Control-Allow-Origin: *`
- `AllowCredentials: true` with wildcard origin
- Missing CORS middleware entirely (no cross-origin policy)
- Origin reflected from request without validation

### 7. Error Handling & Panics

```
Grep: panic\(
Grep: recover\(\)
Grep: log\.Fatal
Grep: c\.JSON\(.*err
Grep: http\.Error\(
```

Check:
- `panic()` in request handlers (crashes the goroutine)
- Missing recovery middleware (Gin has `gin.Recovery()`, Echo has `middleware.Recover()`)
- Error messages returned to client containing internal details
- `err.Error()` sent directly in API responses
- Missing structured error responses

### 8. File Operations

```
Grep: http\.ServeFile
Grep: c\.File\(|c\.SendFile\(
Grep: os\.Open\(.*c\.|os\.ReadFile\(.*c\.
Grep: filepath\.Join\(.*c\.
Grep: ioutil\.ReadFile
```

Check:
- `http.ServeFile()` with user-controlled path → path traversal
- `filepath.Join()` without `filepath.Clean()` and base-path validation
- File uploads without type/size validation
- Uploaded files stored with original user-provided names

### 9. Concurrency Safety

```
Grep: go func|goroutine
Grep: sync\.Mutex|sync\.RWMutex
Grep: sync\.Map
Grep: atomic\.
Grep: race
```

Check:
- Shared maps/slices accessed from multiple goroutines without mutex
- Request-scoped context used in background goroutines after handler returns
- Global state modified in handlers without synchronization
- Missing `sync.Mutex` on shared resources
- Database connections shared across goroutines improperly

### 10. TLS & Security Headers

```
Grep: ListenAndServeTLS
Grep: TLSConfig
Grep: InsecureSkipVerify
Grep: Secure\(|secure\.New
```

Check:
- `InsecureSkipVerify: true` in HTTP client config → TLS validation disabled
- Missing TLS for production deployment
- No security headers middleware (use `secure` package)
- Weak TLS configuration (old protocol versions, weak ciphers)

## Common False Positive Filters

- `exec.Command(fixedCommand, userArgs...)` with validated args → lower risk than shell
- `html/template` auto-escapes by default → XSS only if `template.HTML()` is used
- Gin's `ShouldBind` with proper struct tags → validation is present
- `gin.Recovery()` middleware registered → panics are caught
- GORM `Where("id = ?", id)` with placeholder → parameterized, safe
