# ASP.NET Core Security Audit Guide

Use this guide when Phase 1 identifies ASP.NET Core (via `.csproj` files with
`Microsoft.AspNetCore`, `Program.cs` with `WebApplication.CreateBuilder`,
or `Startup.cs` with `ConfigureServices`).

## Detection

Confirmed when you see:
- `.csproj` files referencing `Microsoft.AspNetCore.App`
- `Program.cs` with `WebApplication.CreateBuilder()` or `Host.CreateDefaultBuilder()`
- `Startup.cs` with `ConfigureServices()` and `Configure()`
- `appsettings.json`, `appsettings.Development.json`
- `Controllers/` directory with `[ApiController]` or `ControllerBase`

## Critical Check Areas

### 1. Authentication & Authorization

```
Grep: \[Authorize\]
Grep: \[AllowAnonymous\]
Grep: AddAuthentication
Grep: AddAuthorization
Grep: RequireAuthorization
Grep: \[Roles\(
Grep: Policy
Grep: AddPolicy
Grep: IAuthorizationHandler
```

Check:
- Controllers/actions missing `[Authorize]` attribute
- `[AllowAnonymous]` on sensitive endpoints
- Missing `[Authorize(Roles = "Admin")]` on admin actions
- Authorization policies not enforced (`RequireAuthorization` missing in endpoint mapping)
- Custom `IAuthorizationHandler` with logic flaws
- Fallback policy not set → new endpoints default to anonymous

### 2. Input Validation & Model Binding

```
Grep: \[FromBody\]
Grep: \[FromQuery\]
Grep: \[FromRoute\]
Grep: \[FromForm\]
Grep: ModelState\.IsValid
Grep: \[Required\]
Grep: \[StringLength\]
Grep: \[Range\]
Grep: FluentValidation
Grep: \[ApiController\]
```

Check:
- Actions without `[ApiController]` → `ModelState.IsValid` not auto-checked
- Missing validation attributes on model properties
- `[FromBody]` accepting models without `[Required]` on critical fields
- Over-posting: Models accepting more fields than intended (use DTOs)
- Missing `[StringLength]` on string properties → unbounded input
- `[FromQuery]` parameters used in SQL without parameterization

### 3. Anti-Forgery (CSRF)

```
Grep: \[ValidateAntiForgeryToken\]
Grep: \[AutoValidateAntiforgeryToken\]
Grep: \[IgnoreAntiforgeryToken\]
Grep: AddAntiforgery
Grep: @Html\.AntiForgeryToken
```

Check:
- MVC forms without `[ValidateAntiForgeryToken]` on POST actions
- `[IgnoreAntiforgeryToken]` on state-changing endpoints
- Missing global `[AutoValidateAntiforgeryToken]` filter
- Note: API controllers using JWT/Bearer tokens typically don't need CSRF

### 4. Data Protection & Secrets

```
Grep: appsettings\.json
Grep: ConnectionStrings
Grep: IConfiguration
Grep: GetConnectionString
Grep: UserSecrets
Grep: IDataProtectionProvider
Grep: Configuration\[
```

Check:
- Connection strings with passwords in `appsettings.json` (committed to git)
- API keys or secrets in config files without User Secrets or Key Vault
- Missing `[JsonIgnore]` on sensitive model properties
- `Configuration["Secret"]` with hardcoded fallback values
- No data protection key rotation configured

### 5. Entity Framework & SQL

```
Grep: FromSqlRaw
Grep: FromSqlInterpolated
Grep: ExecuteSqlRaw
Grep: ExecuteSqlInterpolated
Grep: DbContext
Grep: SqlCommand
Grep: SqlConnection
```

Check:
- `FromSqlRaw()` with string concatenation → SQL injection
- `ExecuteSqlRaw()` with string interpolation → SQL injection
- `FromSqlInterpolated()` is SAFE (auto-parameterizes)
- Direct `SqlCommand` with string concatenation
- Missing parameterized queries in raw SQL

### 6. CORS Configuration

```
Grep: AddCors
Grep: UseCors
Grep: WithOrigins
Grep: AllowAnyOrigin
Grep: AllowCredentials
Grep: CorsPolicyBuilder
```

Check:
- `AllowAnyOrigin()` → overly permissive CORS
- `AllowAnyOrigin()` + `AllowCredentials()` → rejected by browsers but indicates misconfiguration
- Missing specific origin allowlist
- `AllowAnyMethod()` + `AllowAnyHeader()` when not needed

### 7. Error Handling & Information Disclosure

```
Grep: UseDeveloperExceptionPage
Grep: UseExceptionHandler
Grep: ASPNETCORE_ENVIRONMENT
Grep: IsDevelopment
Grep: app\.Environment
```

Check:
- `UseDeveloperExceptionPage()` without `IsDevelopment()` guard → stack traces in production
- Missing `UseExceptionHandler("/Error")` in production
- `ASPNETCORE_ENVIRONMENT` set to `Development` in production
- Custom error pages leaking internal details
- Missing global exception handling middleware

### 8. Security Headers & Middleware

```
Grep: UseHsts
Grep: UseHttpsRedirection
Grep: Content-Security-Policy
Grep: X-Content-Type-Options
Grep: X-Frame-Options
Grep: UseSecurityHeaders
```

Check:
- Missing `UseHsts()` and `UseHttpsRedirection()` in production
- No Content-Security-Policy header
- Missing `X-Content-Type-Options: nosniff`
- Missing `X-Frame-Options` or frame-ancestors CSP
- Middleware ordering issues (security middleware should be early in pipeline)

### 9. Identity & JWT

```
Grep: AddIdentity
Grep: AddDefaultIdentity
Grep: JwtBearer
Grep: TokenValidationParameters
Grep: ValidateIssuer
Grep: ValidateAudience
Grep: ValidateLifetime
Grep: IssuerSigningKey
```

Check:
- `ValidateIssuer = false` or `ValidateAudience = false` → token validation weakened
- `ValidateLifetime = false` → expired tokens accepted
- `IssuerSigningKey` hardcoded in source code
- Weak signing key (short symmetric key)
- Missing token refresh/rotation mechanism
- Password requirements too weak in Identity config

### 10. Logging & Sensitive Data

```
Grep: ILogger
Grep: _logger\.(Log|Info|Warn|Error|Debug)
Grep: Serilog
Grep: NLog
Grep: Console\.Write
```

Check:
- Logging request bodies containing passwords or tokens
- PII in log output without masking
- `Console.WriteLine` with sensitive data in production code
- Missing structured logging (sensitive data in message templates)

## Common False Positive Filters

- `[AllowAnonymous]` on login, register, health check endpoints → intentional
- `[ApiController]` auto-validates ModelState → manual check not needed
- `FromSqlInterpolated()` → auto-parameterized, safe
- `UseDeveloperExceptionPage()` inside `if (app.Environment.IsDevelopment())` → dev only
- Missing CSRF on `[ApiController]` using Bearer tokens → token-based auth handles CSRF
