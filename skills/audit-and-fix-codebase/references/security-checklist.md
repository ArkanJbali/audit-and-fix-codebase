# Security Audit Checklist

Detailed guidance for Pass 1 section 1 of the audit. Read this file when you
need to deepen a security finding or decide severity. The SKILL.md gives the
categories; this file gives the concrete things to look for.

## Contents

- Secrets and credentials
- Injection vulnerabilities
- Authentication and authorization
- Sensitive data exposure
- Configuration weaknesses
- Severity rubric

---

## Secrets and credentials

### What to scan for

- API keys, tokens, passwords, connection strings, private keys, JWT secrets,
  or signing keys in source code or committed config files.
- `.env` files committed to version control.
- Hardcoded credentials in test files that mirror production secrets.
- Access tokens embedded in URLs, example code, or documentation.
- Secrets in `.gitignore`-d files that WERE committed before being added to
  `.gitignore` (check git history with `git log -p`).

### How to check

```bash
# Quick grep for common patterns
rg -i '(api[_-]?key|secret|password|token|auth|credential)\s*[:=]\s*['"'"'"]' --include '*.{py,js,ts,env,json,yaml,yml,config,ini,cs,go,rb,php,java,rs}'

# Check git history for committed secrets
git log --all --diff-filter=A --pretty=format:'%h %s' -- '*.env'
```

### Severity

- **Critical**: Production credentials, private keys, or database connection
  strings in committed code or visible in git history. Immediate rotation needed.
- **High**: Test/staging credentials using real endpoints, or any credential
  accessible to all developers.
- **Medium**: Example credentials in documentation, or internal API keys with
  limited scope.
- **Low**: Placeholder credentials clearly marked as such (e.g., `CHANGE_ME`).

### What to do

1. Move the secret to environment variables or a secret store.
2. Create or update `.env.example` with placeholder values.
3. Remind the user: an ever-committed secret must be **rotated** — removing
   it from code is not enough; revoke the old key/token.

---

## Injection vulnerabilities

### SQL injection

Look for string concatenation or interpolation in query building:

```python
# Bad
query = f"SELECT * FROM users WHERE id = {user_id}"
cursor.execute(query)

# Good
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
```

Check: raw SQL in string interpolation, `cursor.execute(f"...")`, ORM `.raw()`
calls, `db.query("..." + param)`, `wpdb->prepare()` skipped, SQL in string-
based query builders.

### Cross-site scripting (XSS)

Check for unescaped output:

- `dangerouslySetInnerHTML` (React), `innerHTML` / `outerHTML` (vanilla JS),
  `v-html` (Vue), `<?= $var ?>` without escaping (PHP).
- User content rendered via `markdown` or `render` functions without sanitization.
- Reflected input in error messages or form values.
- `document.write()`, `eval()` of strings containing user input.

### Command injection

Check for shell command construction with user input:

```python
# Bad
os.system(f"grep {user_input} /var/log/app.log")

# Good
subprocess.run(["grep", user_input, "/var/log/app.log"], check=True)
```

### Path traversal

Check file operations that use user-supplied paths:

```python
# Bad
open(f"/uploads/{filename}", "rb")

# Good
safe_path = os.path.join(UPLOAD_DIR, os.path.basename(filename))
```

### Server-side request forgery (SSRF)

Check user-supplied URLs in server-side requests:

- `requests.get(user_url)`, `fetch(user_url)`, `httpx.get(user_url)`.
- URL fetching without allow-list or scheme validation.
- Webhook handlers, image proxy, link preview features.

### Insecure deserialization

Check for:
- `pickle.loads(user_data)`, `yaml.load(user_data)` (without `Loader`).
- `JSON.parse()` on user input (safe; JSON doesn't execute code).
- `unserialize()` in PHP on user-controlled data.
- `Marshal.Load()` in Go / `BinaryFormatter` in .NET.

### Severity

- **Critical**: SQL/command injection reachable from external input, or
  deserialization of untrusted data with code-execution risk.
- **High**: XSS in persisted user content, SSRF without allow-list, path
  traversal in authenticated endpoints.
- **Medium**: XSS in reflected/transient contexts, path traversal limited to
  read operations.

---

## Authentication and authorization

### Missing or weak auth

- Routes, endpoints, or GraphQL resolvers without auth checks.
- Auth check applied to individual routes but missing on a bulk/export/admin
  endpoint.
- Admin panels, debug endpoints, or staging routes exposed in production.
- Default credentials unchanged (`admin/admin`, `root/root`).

### Insecure Direct Object References (IDOR)

Check object-level access control:

```python
# Bad — no ownership check
def get_order(order_id):
    return db.orders.find_one(order_id)

# Good — ownership verified
def get_order(order_id, user_id):
    return db.orders.find_one({"_id": order_id, "user_id": user_id})
```

### Mass assignment / over-posting

Check endpoints that accept a full model object from the client:

```python
# Bad — client sets any field
User.update(request.json)

# Good — only allowed fields
User.update(request.json, only=['name', 'email'])
```

### Rate limiting

- Login, password-reset, OTP, and registration endpoints lack rate limiting.
- API endpoints lack per-user or per-IP throttling.

### Severity

- **Critical**: Unauthenticated access to sensitive data, IDOR allowing
  cross-user data access, mass assignment setting `is_admin` or `role`.
- **High**: Missing rate limiting on auth-related endpoints, IDOR with
  enumeration risk.
- **Medium**: Rate limiting absent on non-critical endpoints, auth on some
  routes but inconsistently applied.

---

## Sensitive data exposure

### In logs

- Passwords, tokens, or PII logged at `info` or `debug` level.
- Full request/response bodies logged in production.
- Stack traces exposing internal paths, SQL queries, or config.

### In client-side storage

- JWTs or session tokens in `localStorage` (vulnerable to XSS; prefer
  `httpOnly` cookies).
- PII, credit card numbers, or API keys in `localStorage` / `sessionStorage`.
- Secrets embedded in single-page app bundles.

### In URLs

- Tokens or session IDs in URL query parameters (leaked via referrer headers
  and server logs).
- Password reset tokens in URLs without expiration.

### In error responses

- Stack traces returned to the client in production.
- SQL queries or internal path names in error messages.
- Verbose error messages revealing whether an email/user exists (enumeration).

### Severity

- **Critical**: Plaintext passwords or tokens in logs, PII exposed to
  unauthorized users.
- **High**: Session tokens in `localStorage`, verbose errors in production,
  PII in client bundles.
- **Medium**: Internal paths in error messages, debug logging in production.
- **Low**: Inconsistently applied log sanitization.

---

## Configuration weaknesses

### CORS

- `Access-Control-Allow-Origin: *` on authenticated endpoints.
- Reflective CORS headers that echo the request `Origin` without validation.
- Permissive `Access-Control-Allow-Credentials: true` without origin check.

### Debug mode in production

- `DEBUG=True`, `APP_DEBUG=true`, `NODE_ENV=development` in production config.
- Development-only middleware, routes, or packages loaded in production.
- Interactive debuggers (e.g., Werkzeug debugger, Laravel debugbar) enabled.

### Password hashing

- Plaintext password storage (critical).
- MD5, SHA1, or unsalted SHA256 for passwords.
- Missing bcrypt/argon2/scrypt/PBKDF2.

### Rate limiting

- No rate limiting on login, registration, or password reset.
- API endpoints without any throttling.

### Severity

- **Critical**: Plaintext passwords, debug mode exposing interactive
  shells/admin panels.
- **High**: Permissive CORS on auth endpoints, weak password hashing.
- **Medium**: Debug mode on staging, CORS too permissive for public data.

---

## Severity rubric

| Severity | Definition |
|----------|------------|
| **Critical** | Direct data breach, remote code execution, or authentication bypass. Must fix before deploy. |
| **High** | Significant security weakness with moderate exploitation barrier. Fix before next deploy. |
| **Medium** | Defense-in-depth gap or minor exposure. Fix within the iteration. |
| **Low** | Best-practice gap with minimal real-world risk. Fix when convenient. |

---

## Sources

See [sources.md](sources.md) for OWASP, CWE, and other security references.
