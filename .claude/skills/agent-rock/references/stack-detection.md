# Stack Detection Heuristics

Use this file after identifying package manifests. The goal is to convert manifest clues into a
focused audit plan so you only load the framework references that matter.

## General Process

1. Read the primary manifests inside the target root.
2. Identify the dominant language and runtime.
3. Map direct dependencies and config files to likely frameworks.
4. Confirm the framework by reading one or two entry-point files before loading deeper guidance.
5. Load only the relevant framework reference files from this directory.

## JavaScript / TypeScript

**Manifest clues:**
- `express`, `koa`, `fastify`, `hapi`, `nestjs`, `next`, `nuxt`, `sails`
- ORM and API clues: `mongoose`, `prisma`, `sequelize`, `typeorm`, `apollo-server`

**Config and file clues:**
- `server.js`, `app.js`, `src/server.ts`, `src/app.ts`
- `next.config.js`, `next.config.mjs`, `app/`, `pages/`
- `nest-cli.json`, `main.ts`, `app.module.ts`

**When to load focused guidance:**
- Load [express-node.md](express-node.md) for Express-style routers, middleware chains, session auth, file upload flows, and common Node API patterns.
- Load [nextjs.md](nextjs.md) when you see `next` in dependencies, `next.config.js`, `app/` or `pages/` directory with file-based routing, Server Actions (`"use server"`), or `getServerSideProps`.
- For NestJS (`nest-cli.json`, `@nestjs/core`, `app.module.ts`), apply Express patterns but also check Guards, Interceptors, Pipes, and decorator-based auth.
- For Nuxt.js (`nuxt` in dependencies, `nuxt.config.ts`, `server/api/`), load [nextjs.md](nextjs.md) and adapt checks to Nuxt equivalents.

**Frontend SPA detection:**
- If you see `react`, `vue`, `@angular/core`, or `svelte` in dependencies, also load [frontend-security.md](frontend-security.md) for client-side specific checks.

## Python

**Manifest clues:**
- `django`, `djangorestframework`, `channels`
- `flask`, `flask-login`, `flask-wtf`
- `fastapi`, `starlette`, `uvicorn`, `pydantic`

**Config and file clues:**
- `manage.py`, `settings.py`, `urls.py`, `wsgi.py`, `asgi.py`
- `app.py`, `wsgi.py`, `blueprints/`
- `main.py`, `routers/`, `dependencies.py`

**When to load focused guidance:**
- Load [django.md](django.md) when you see Django settings, URLconf, ORM models, serializers, or DRF views.
- Load [fastapi.md](fastapi.md) when you see FastAPI app creation, Depends() injection, Pydantic models, or uvicorn startup.
- For Flask (`flask` in dependencies, `app.py`, `blueprints/`), apply general Python web patterns with focus on Jinja2 auto-escaping, Blueprint auth, CSRF via Flask-WTF, and session cookie security.

## Java / JVM

**Manifest clues:**
- `spring-boot`, `spring-security`, `spring-web`, `spring-data`
- `quarkus`, `micronaut`

**Config and file clues:**
- `application.yml`, `application.properties`
- `@RestController`, `@Controller`, `@Configuration`, `@EnableWebSecurity`
- `SecurityFilterChain`, `WebSecurityConfigurerAdapter`

**When to load focused guidance:**
- Load [spring.md](spring.md) when you see Spring MVC, Spring Security, or Spring Data patterns.

## Ruby

**Manifest clues:**
- `rails`, `devise`, `pundit`, `cancancan`, `sidekiq`

**Config and file clues:**
- `config/routes.rb`, `app/controllers/`, `app/models/`, `config/initializers/`
- `ApplicationController`, `ActiveRecord`, `strong_parameters`

**When to load focused guidance:**
- Load [rails.md](rails.md) when the app follows standard Rails routing, controller, and model conventions.

## PHP

**Manifest clues:**
- `laravel/framework`, `sanctum`, `passport`, `spatie/laravel-permission`
- `symfony/framework-bundle`

**Config and file clues:**
- `routes/web.php`, `routes/api.php`, `app/Http/Controllers/`, `app/Models/`
- `config/app.php`, `config/auth.php`, middleware classes

**When to load focused guidance:**
- Load [laravel.md](laravel.md) when the app uses Laravel routing, middleware, Eloquent, or request validation.

## C# / .NET

**Manifest clues:**
- `.csproj` files with `Microsoft.AspNetCore.App`
- `Microsoft.EntityFrameworkCore`, `Microsoft.AspNetCore.Identity`

**Config and file clues:**
- `Program.cs`, `Startup.cs`, `appsettings.json`
- `Controllers/`, `[ApiController]`, `[Authorize]`

**When to load focused guidance:**
- Load [aspnet.md](aspnet.md) when you see ASP.NET Core MVC, Web API, Identity, or Entity Framework patterns.

## Go Web Frameworks

**Manifest clues (go.mod):**
- `github.com/gin-gonic/gin`, `github.com/labstack/echo`
- `github.com/gofiber/fiber`, `github.com/go-chi/chi`
- `net/http` with router patterns

**Config and file clues:**
- `main.go` with `gin.Default()`, `echo.New()`, `fiber.New()`
- `http.HandleFunc`, `http.ListenAndServe`

**When to load focused guidance:**
- Load [go-web.md](go-web.md) when you see Gin, Echo, Fiber, Chi, or standard `net/http` router patterns.

## C / C++

**Build clues:**
- `CMakeLists.txt`, `Makefile`, `meson.build`, `compile_commands.json`
- source trees such as `src/`, `include/`, `lib/`

**Focus points:**
- Distinguish server/network code, CLI utilities, parsers, and privileged helpers.
- Prioritize memory safety issues on externally reachable parsing, network, file, or IPC paths.

## AI/ML Projects

**Manifest clues:**
- `tensorflow`, `torch`, `pytorch`, `keras`, `scikit-learn`, `transformers`
- `openai`, `anthropic`, `langchain`, `llama-index`, `cohere`
- `mlflow`, `wandb`, `ray`, `bentoml`, `seldon`

**File clues:**
- `*.pkl`, `*.pt`, `*.pth`, `*.h5`, `*.onnx`, `*.safetensors`
- `model_config.yaml`, `training/`, `inference/`, `notebooks/`
- Jupyter notebooks (`.ipynb`) with ML imports

**When to load focused guidance:**
- Load [ai-ml-security.md](ai-ml-security.md) when AI/ML libraries are detected.
- Focus on unsafe deserialization, prompt injection, API key exposure, and model supply chain.

## Infrastructure as Code

**File clues:**
- `Dockerfile`, `docker-compose.yml`, `.dockerignore`
- `*.tf`, `*.tfvars`, `terraform.tfstate`
- Kubernetes manifests (`k8s/`, `deploy/`, `helm/`)
- `serverless.yml`, `template.yaml` (SAM)
- `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`

**When to load focused guidance:**
- Load [cloud-native-security.md](cloud-native-security.md) when Docker, K8s, Terraform, or serverless configs are found.
- Load [supply-chain-advanced.md](supply-chain-advanced.md) when CI/CD pipelines or complex dependency configurations are detected.

## Ambiguous Repos

If multiple frameworks appear:
- Prioritize the code that serves external traffic or privileged operations.
- Load only the guides relevant to the reachable application surface.
- Treat tooling, examples, and old migrations as secondary unless they are still executed.
