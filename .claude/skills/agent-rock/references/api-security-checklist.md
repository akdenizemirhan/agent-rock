# API Security Checklist

Treat every grep hit as a lead, not a finding. Missing framework helpers, middleware names,
or validation libraries are not enough on their own. Confirm the behavior on a concrete route,
handler, schema, or error path before reporting anything.

When a framework is confirmed, load the matching focused guide:
- [express-node.md](express-node.md)
- [django.md](django.md)
- [spring.md](spring.md)
- [rails.md](rails.md)
- [laravel.md](laravel.md)

## 1. Authentication & Authorization on Endpoints

**What to check:**
- Endpoints/routes without authentication middleware
- Missing role-based access control on sensitive operations
- Broken object-level authorization (BOLA/IDOR) — user A accessing user B's resources
- Broken function-level authorization — regular user accessing admin functions

**How to audit:**

1. Find all route/endpoint definitions:
```
# Express.js
"app\.(get|post|put|delete|patch|all)\(|router\.(get|post|put|delete|patch|all)\("

# Django
"path\(|re_path\(|url\(|urlpatterns"

# Flask
"@app\.route|@.*\.route|@blueprint\.route"

# FastAPI
"@app\.(get|post|put|delete|patch)|@router\.(get|post|put|delete|patch)"

# Spring
"@(Get|Post|Put|Delete|Patch|Request)Mapping"

# Go (net/http, gin, echo)
"\.GET\(|\.POST\(|\.PUT\(|\.DELETE\(|HandleFunc\(|Handle\("

# Rails
"resources\s|resource\s|get\s|post\s|put\s|patch\s|delete\s.*=>"

# Laravel
"Route::(get|post|put|delete|patch|any)\("

# .NET
"\[Http(Get|Post|Put|Delete|Patch)\]|\[AllowAnonymous\]"
```

2. For each endpoint, verify:
   - Is authentication middleware/decorator applied?
   - Is authorization (role/permission) check present?
   - Does it validate that the requesting user owns the accessed resource?

**Red flags:**
- CRUD operations on resources using only the resource ID from the request
- Admin endpoints without admin role verification
- `[AllowAnonymous]` or `@PermitAll` on sensitive endpoints
- Missing `@login_required`, `@authenticated`, `isAuthenticated` checks

---

## 2. Rate Limiting

**What to check:**
- Missing rate limiting on authentication endpoints (login, register, forgot-password)
- Missing rate limiting on API endpoints in general
- Missing rate limiting on resource-intensive operations (file upload, search, export)

**Grep patterns:**
```
# Rate limiting libraries/middleware
"express-rate-limit|rate.limit|ratelimit|throttle|RateLimiter|@Throttle"
"slowapi|flask-limiter|django-ratelimit|rack-attack"

# If none are found, inspect login, signup, password reset, upload, search, and export routes manually.
# Only report a finding when a concrete high-risk route lacks an effective limiting control.
```

---

## 3. Mass Assignment / Over-Posting

**What to check:**
- Request body directly used to create/update database records without field filtering
- Missing allowlist of updatable fields

**Framework-specific patterns:**

```
# Express/Node.js — spreading req.body into DB operations
"\.create\(req\.body|\.update\(req\.body|Object\.assign\(.*req\.body|\.save\(req\.body"
"\.findByIdAndUpdate\(.*req\.body|\.findOneAndUpdate\(.*req\.body"
"spread.*req\.body|\.create\(\{.*\.\.\.req"

# Django — missing fields/exclude in forms/serializers
"fields\s*=\s*'__all__'|fields\s*=\s*\"__all__\""
"exclude\s*=\s*\[\]|exclude\s*=\s*\(\)"

# Rails — check for strong parameters usage
"params\.permit!|params\[:.*\](?!\.permit)|params\.to_unsafe_hash"

# Laravel — $fillable vs $guarded
"\\$guarded\s*=\s*\[\]|\\$guarded\s*=\s*array\(\)"

# Spring — binding all fields
"@ModelAttribute|WebDataBinder.*setAllowedFields"
```

---

## 4. Input Validation & Request Schema

**What to check:**
- Missing request body validation/schema enforcement
- No input length limits (allows DoS via large payloads)
- Missing type checking on inputs
- File upload without type/size restrictions

**Grep patterns:**
```
# File upload without restrictions
"multer\(|upload\.|formidable|busboy|multipart"
(then check for file type validation and size limits)

# Missing body size limits
"bodyParser|express\.json\(|express\.urlencoded\("
(check for limit option: { limit: '10mb' })

# Request validation libraries
# Their absence is only a lead. Confirm an endpoint accepts unvalidated or unbounded input before reporting.
"joi|yup|zod|class-validator|cerberus|marshmallow|pydantic|FlaskForm|WTForms"
```

---

## 5. Data Exposure in API Responses

**What to check:**
- Returning full database objects/models instead of DTOs/serialized responses
- Including internal fields (password hashes, internal IDs, timestamps, flags)
- Stack traces in error responses
- Verbose error messages revealing database schema or internal structure

Only report a confirmed finding when the response path demonstrates exposure of real sensitive attributes,
or when framework serialization behavior in this repo clearly proves those attributes are included.
If the code only shows a broad model or object response without demonstrated sensitive fields, record a
`hardening_observation` instead of a confirmed vulnerability.

**Grep patterns:**
```
# Returning raw DB objects
"res\.json\(user|res\.json\(.*findOne|res\.send\(.*find\(|response\.json\(.*objects\.get"
"return JsonResponse\(.*__dict__|return Response\(.*serializer\.data"

# Error details exposed
"res\.(json|send)\(.*err|res\.(json|send)\(.*error\.(message|stack)"
"return.*500.*error\.(message|stack|trace)"
"traceback\.format_exc|traceback\.print_exc"
```

---

## 6. Pagination & Resource Limits

**What to check:**
- Missing pagination on list endpoints (can return entire database)
- No maximum page size limit
- Missing timeout on database queries
- Unbounded aggregation or export operations

**Grep patterns:**
```
# List endpoints without pagination
"\.find\(\)|\.findAll\(\)|\.all\(\)|\.list\(\)|\.select\(\)"
(check if limit/offset or cursor pagination is applied)

# Missing max page size
"pageSize|page_size|per_page|limit"
(check if there is a maximum value enforced)
```

---

## 7. GraphQL-Specific Issues

**What to check:**
- Introspection enabled in production
- No query depth or complexity limits
- Missing field-level authorization
- Batching attacks possible (multiple operations in one request)
- Nested fragment/alias abuse for DoS
- Subscription authentication
- Directive abuse

**Grep patterns:**
```
# Introspection
"introspection\s*:\s*true|enableIntrospection|__schema"

# Depth/complexity limit hooks
"depthLimit|queryComplexity|costAnalysis|maxDepth|complexity"
(if absent, inspect the active GraphQL server setup and only report after confirming there is no equivalent guard)

# GraphQL setup
"ApolloServer|graphqlHTTP|GraphQLModule|Strawberry|graphene|Ariadne|gqlgen"
(if found, check all items below)

# Batch queries
"batching|batch.*query|allowBatchedHttpRequests"

# Subscriptions
"subscription|PubSub|withFilter|subscribe\s*:"

# Persisted queries
"persistedQueries|automaticPersistedQueries|APQ"
```

### 7.1 Query Complexity & Cost Analysis

Check whether the GraphQL server enforces:
- **Max depth limit**: Prevents deeply nested queries (e.g., `user { friends { friends { friends... } } }`)
- **Complexity/cost limit**: Assigns cost to fields and rejects queries exceeding a budget
- **Max aliases**: Prevents alias-based DoS (`a1: user(id:1), a2: user(id:2), ... a1000: user(id:1000)`)
- **Timeout**: Query execution timeout to prevent long-running resolvers

If none of these are configured, report as Medium severity (DoS via resource exhaustion).

### 7.2 Batch Query Abuse

```
Grep: allowBatchedHttpRequests|batching|batch
```

Check:
- Server accepts arrays of operations in a single HTTP request
- No limit on batch size → send thousands of operations in one request
- Batch queries bypass per-request rate limiting
- If batching is enabled, verify batch size is capped

### 7.3 Nested Fragment DoS

Check:
- No fragment depth/spread limits → circular or deeply nested fragments cause exponential resolution
- Missing validation of fragment complexity before execution
- Fragments that reference other fragments without depth limit

### 7.4 Field-Level Authorization

```
Grep: @auth|@hasRole|@isAuthenticated|directive.*auth
Grep: fieldResolver|resolve.*context\.user
```

Check:
- Sensitive fields (email, phone, SSN, internal IDs) accessible without auth
- Authorization checked at query level but not at field/resolver level
- Different users see same fields regardless of role
- Mutations without authorization directive

### 7.5 Subscription Security

```
Grep: subscription|PubSub|subscribe\(|onSubscription
```

Check:
- Subscription endpoints without authentication
- Missing authorization on subscription events (user A receives user B's updates)
- No connection limit per user (resource exhaustion)
- Sensitive data broadcast to all subscribers

### 7.6 Persisted Queries vs Dynamic Queries

```
Grep: persistedQueries|automaticPersistedQueries|whitelistQueries
```

Check:
- Production server accepting arbitrary dynamic queries (attack surface)
- If persisted queries are used, verify fallback to dynamic queries is disabled
- If APQ (Automatic Persisted Queries) is enabled, the first request still sends the full query

### 7.7 Information Disclosure

Check:
- Introspection enabled in production → exposes full schema
- Detailed error messages from resolvers (field names, types, internal errors)
- Suggestion feature enabled (`didYouMean`) → schema enumeration
- Debug mode enabled in GraphQL server config

---

## 8. SSRF (Server-Side Request Forgery)

**What to check:**
- User-supplied URLs fetched by the server without validation
- URL parameters used in internal HTTP requests
- Webhook URLs that can target internal services
- Image/file download from user-provided URLs

**Grep patterns:**
```
# Server-side HTTP requests with user input
"fetch\(.*req\.|axios\(.*req\.|requests\.get\(.*request\.|http\.Get\(.*r\."
"urllib\.request\.urlopen|urlopen\(.*request|HttpClient.*request\."
"RestTemplate.*request\.|WebClient.*request\."

# Webhook URL handling
"webhook.*url|callback.*url|notification.*url|redirect.*url"

# URL validation (good sign — but verify it blocks internal IPs)
"isValidUrl|validateUrl|url\.parse|new URL\("
```

---

## 9. HTTP Method Security

**What to check:**
- State-changing operations allowed via GET requests
- HTTP method override enabled without restrictions
- Missing CSRF protection on state-changing endpoints

**Grep patterns:**
```
# State changes on GET
"app\.get\(.*delete|app\.get\(.*update|app\.get\(.*create|app\.get\(.*modify"
"@app\.route.*methods.*GET.*def.*delete|@app\.route.*methods.*GET.*def.*update"

# Method override
"methodOverride|_method|X-HTTP-Method-Override"

# CSRF protection
"csrf|csrfToken|_token|antiforgery|AntiForgeryToken"
(if absent, confirm a cookie or session based state-changing flow is exposed without equivalent protection)
```

---

## 10. Proper Error Handling in APIs

**What to check:**
- Different error responses for "not found" vs "not authorized" (information leakage)
- Detailed error messages in production responses
- Missing global error handler

**Grep patterns:**
```
# Inconsistent auth errors (can reveal if resources exist)
"404.*not found|403.*forbidden|401.*unauthorized"
(check if 404 is returned for both "not found" and "not authorized to see" — should be consistent)

# Global error handler
"app\.use\(.*err.*req.*res|@app\.errorhandler|exception_handler|@ExceptionHandler"
(if absent, trace an application error path and confirm errors are handled inconsistently or leak internals before reporting)
```
