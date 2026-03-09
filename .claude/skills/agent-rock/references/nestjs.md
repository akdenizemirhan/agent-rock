# NestJS Security Audit Guide

Use this guide when Phase 1 identifies NestJS (via `@nestjs/core` in package.json,
`nest-cli.json`, or `app.module.ts`).

## Detection

Confirmed when you see:
- `@nestjs/core`, `@nestjs/common` in package.json
- `nest-cli.json` in project root
- `app.module.ts` with `@Module()` decorator
- `main.ts` with `NestFactory.create()`

## Architecture Awareness

NestJS uses decorators extensively for auth, validation, and routing. Security controls
are typically applied via Guards, Interceptors, Pipes, and Middleware — check that these
are properly registered globally or per-controller/route.

## Critical Check Areas

### 1. Guards (Authentication & Authorization)

```
Grep: @UseGuards
Grep: CanActivate
Grep: AuthGuard
Grep: RolesGuard
Grep: JwtAuthGuard
Grep: APP_GUARD
Grep: @Public
Grep: @SetMetadata
```

Check:
- Controllers/routes missing `@UseGuards()` for auth
- Global guard registered via `APP_GUARD` → verify it covers all routes
- `@Public()` or `@SetMetadata('isPublic', true)` on sensitive endpoints
- Custom guards that return `true` without proper validation
- Missing role-based guards on admin endpoints
- Guard bypass via decorator ordering issues

### 2. Pipes (Input Validation)

```
Grep: ValidationPipe
Grep: ParseIntPipe
Grep: ParseUUIDPipe
Grep: @UsePipes
Grep: class-validator
Grep: class-transformer
Grep: @IsString|@IsEmail|@IsNumber|@MinLength|@MaxLength
Grep: whitelist.*true|forbidNonWhitelisted
```

Check:
- `ValidationPipe` not registered globally → routes accept unvalidated input
- Missing `whitelist: true` → extra properties pass through (mass assignment)
- Missing `forbidNonWhitelisted: true` → unknown fields silently accepted
- DTOs without class-validator decorators (no actual validation)
- `@Body()` without DTO type → raw object accepted
- `transform: true` with unsafe type coercion
- Missing `ParseIntPipe`/`ParseUUIDPipe` on path parameters

### 3. Interceptors & Middleware

```
Grep: @UseInterceptors
Grep: NestInterceptor
Grep: NestMiddleware
Grep: APP_INTERCEPTOR
Grep: ClassSerializerInterceptor
Grep: @Exclude
Grep: @Expose
```

Check:
- `ClassSerializerInterceptor` not used → response may include sensitive fields
- Entities without `@Exclude()` on sensitive fields (password, internal IDs)
- Custom interceptors that modify response without sanitization
- Middleware ordering issues (logging before auth → logs unauthenticated requests)
- Missing error interceptor → stack traces in responses

### 4. TypeORM / Database Security

```
Grep: @InjectRepository
Grep: Repository<
Grep: getRepository
Grep: createQueryBuilder
Grep: \.query\(
Grep: raw\(
```

Check:
- `createQueryBuilder` with string concatenation → SQL injection
- `.query()` with template literals → raw SQL injection
- Repository methods with user input in `where` conditions without parameterization
- Missing transaction isolation on multi-step operations
- Entities with sensitive fields not excluded from serialization

### 5. Configuration & Secrets

```
Grep: ConfigService
Grep: @nestjs/config
Grep: process\.env
Grep: .env
Grep: JWT_SECRET|SECRET_KEY|DATABASE_URL
```

Check:
- Secrets hardcoded instead of using `ConfigService`
- `.env` files committed to git
- `ConfigService.get()` without validation (accepts undefined)
- Missing `ConfigModule.forRoot({ validationSchema })` → no env validation
- JWT secrets that are weak or default values

### 6. CORS & Security Headers

```
Grep: enableCors
Grep: app\.use\(helmet
Grep: origin.*true|origin.*\*
Grep: @nestjs/throttler
Grep: ThrottlerGuard
```

Check:
- `enableCors({ origin: true })` → reflects any origin (credential leak)
- `enableCors({ origin: '*' })` → open CORS
- Missing `helmet` middleware (no security headers)
- Missing `@nestjs/throttler` → no rate limiting
- ThrottlerGuard not applied globally

### 7. WebSocket (Gateway) Security

```
Grep: @WebSocketGateway
Grep: @SubscribeMessage
Grep: @ConnectedSocket
Grep: handleConnection
Grep: handleDisconnect
```

Check:
- WebSocket gateways without auth in `handleConnection`
- `@SubscribeMessage` handlers without input validation
- Missing namespace/room authorization
- Broadcasting sensitive data to all connected clients

### 8. File Upload

```
Grep: @UseInterceptors.*FileInterceptor
Grep: FileInterceptor
Grep: FilesInterceptor
Grep: MulterModule
Grep: @UploadedFile
```

Check:
- Missing file type validation (MIME type + extension)
- Missing file size limits in Multer config
- Upload destination inside public/static directory
- Original filename used without sanitization

### 9. Exception Filters

```
Grep: @Catch
Grep: ExceptionFilter
Grep: HttpException
Grep: APP_FILTER
Grep: AllExceptionsFilter
```

Check:
- No global exception filter → framework default may leak details
- Custom filters returning internal error details
- Catching all exceptions without proper logging
- Different error formats for 401 vs 403 vs 404 (information leakage)

## Common False Positive Filters

- `@Public()` on truly public endpoints (login, register, health check) → not a finding
- Missing `@UseGuards()` on controller with global `APP_GUARD` → guard is applied
- `whitelist: true` on global `ValidationPipe` → DTO properties are filtered globally
- `@Exclude()` on entity fields with `ClassSerializerInterceptor` → fields are hidden
