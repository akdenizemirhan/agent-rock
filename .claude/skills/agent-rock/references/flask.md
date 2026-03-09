# Flask Security Audit Guide

Use this guide when Phase 1 identifies Flask (via `flask` in requirements.txt/pyproject.toml,
`from flask import Flask`, or `app.py` with Flask patterns).

## Detection

Confirmed when you see:
- `flask` in requirements.txt, Pipfile, or pyproject.toml
- `from flask import Flask` or `Flask(__name__)`
- `@app.route` decorators
- `app.py`, `wsgi.py`, or `blueprints/` directory

## Architecture Awareness

Flask is a micro-framework — it provides minimal security out of the box. Unlike Django,
Flask does NOT include CSRF protection, user auth, or ORM by default. Security depends
almost entirely on extensions and developer discipline.

## Critical Check Areas

### 1. Authentication & Session

```
Grep: flask_login|flask-login|LoginManager
Grep: @login_required
Grep: current_user
Grep: session\[
Grep: flask_jwt|flask-jwt|jwt_required
Grep: flask_httpauth
Grep: SECRET_KEY
```

Check:
- Routes handling sensitive data missing `@login_required`
- `SECRET_KEY` hardcoded or weak (Flask signs session cookies with it)
- `SECRET_KEY` set to default or placeholder value
- Session data trusted without validation (Flask sessions are client-side by default)
- Missing session timeout / expiration
- `current_user` checked but `is_authenticated` not verified
- JWT tokens without expiry validation

### 2. CSRF Protection

```
Grep: CSRFProtect|csrf_protect
Grep: flask_wtf|flask-wtf|FlaskForm
Grep: csrf\.init_app
Grep: csrf_token
Grep: WTF_CSRF
```

Check:
- **No CSRF protection at all** → state-changing forms/AJAX vulnerable
- CSRFProtect not initialized → `csrf.init_app(app)` missing
- API endpoints accepting cookies without CSRF tokens
- `WTF_CSRF_ENABLED = False` in config
- AJAX requests not including CSRF token in headers
- Forms not using `{{ form.hidden_tag() }}` or `{{ csrf_token() }}`

### 3. Jinja2 Template Security

```
Grep: render_template_string
Grep: Template\(
Grep: Markup\(
Grep: \|safe
Grep: {% autoescape false %}
Grep: from_string
```

Check:
- `render_template_string()` with user input → **server-side template injection**
- `Template(user_input)` or `Environment().from_string(user_input)` → SSTI
- `Markup(user_input)` → bypasses auto-escaping
- `|safe` filter on user-controlled content → XSS
- `{% autoescape false %}` blocks → no escaping
- Note: `render_template()` with `.html` files auto-escapes by default (safe)

### 4. SQL & Database

```
Grep: db\.engine\.(execute|raw)
Grep: text\(.*f"|text\(.*format
Grep: cursor\.execute\(.*f"|cursor\.execute\(.*%
Grep: SQLAlchemy
Grep: flask_sqlalchemy
```

Check:
- Raw SQL with f-strings or `.format()` → SQL injection
- `text()` with string interpolation
- `engine.execute()` with user-controlled SQL
- SQLAlchemy ORM is safe by default, but `text()` and `raw()` bypass it
- Missing parameterized queries in raw SQL

### 5. File Handling

```
Grep: send_file
Grep: send_from_directory
Grep: safe_join
Grep: secure_filename
Grep: request\.files
Grep: upload_folder
```

Check:
- `send_file()` with user-controlled path → path traversal
- Missing `secure_filename()` on uploaded file names
- Upload directory inside static/public folder
- Missing file type/size validation on uploads
- `send_from_directory()` with user input in filename parameter

### 6. Configuration Security

```
Grep: app\.config
Grep: DEBUG\s*=\s*True
Grep: TESTING\s*=\s*True
Grep: ENV\s*=\s*['"]development
Grep: app\.run\(.*debug
```

Check:
- `DEBUG = True` in production → exposes Werkzeug debugger (RCE risk)
- `app.run(debug=True)` → interactive debugger on errors
- `TESTING = True` in production → relaxed security
- Missing `SESSION_COOKIE_SECURE`, `SESSION_COOKIE_HTTPONLY`, `SESSION_COOKIE_SAMESITE`
- `PERMANENT_SESSION_LIFETIME` not set (default is 31 days)
- Config loaded from file without environment separation

### 7. Blueprint Security

```
Grep: Blueprint\(
Grep: register_blueprint
Grep: @.*_bp\.route
Grep: before_request
Grep: before_app_request
```

Check:
- Blueprints without `@bp.before_request` auth checks
- Admin blueprints without consistent auth enforcement
- URL prefix conflicts between blueprints
- Missing error handlers on blueprints (falling through to app-level)

### 8. Request Data Handling

```
Grep: request\.args
Grep: request\.form
Grep: request\.json
Grep: request\.data
Grep: request\.values
Grep: request\.get_json
```

Check:
- `request.args.get()` used directly in SQL, file paths, or redirects
- `request.get_json(force=True)` → accepts non-JSON content-type
- `request.values` merges GET and POST → parameter confusion
- Missing input validation on request data
- Large request body without size limit (`MAX_CONTENT_LENGTH` not set)

### 9. CORS Configuration

```
Grep: flask_cors|flask-cors
Grep: CORS\(
Grep: cross_origin
Grep: Access-Control
```

Check:
- `CORS(app)` without origin restriction → open CORS
- `supports_credentials=True` with `origins="*"` → credential leak
- `@cross_origin()` without origin parameter
- Missing CORS entirely on API endpoints used by frontends

### 10. Error Handling

```
Grep: @app\.errorhandler
Grep: abort\(
Grep: traceback
Grep: app\.logger
```

Check:
- Missing custom error handlers (404, 500) → default pages leak info
- `traceback.format_exc()` in error responses
- Different error codes for auth vs not-found (info leakage)
- Sensitive data in `app.logger` output

## Common False Positive Filters

- `render_template('file.html', data=user_input)` → Jinja2 auto-escapes, safe
- `DEBUG = True` guarded by `if os.environ.get('FLASK_ENV') == 'development'` → dev only
- `|safe` used on internally-generated HTML, not user input → verify source
- Flask-Login `@login_required` on all sensitive routes → auth is present
