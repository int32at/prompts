# Security Code Review Reference

## OWASP Top 10 (2021)

### A01: Broken Access Control
**What to check:**
- Missing authorization checks on endpoints/functions
- Insecure direct object references (IDOR)
- Force browsing to authenticated pages
- Missing function-level access control
- API lacking access controls
- Elevation of privilege attacks

**Examples:**
```
# Bad: No authorization check
GET /api/users/123/salary

# Bad: Direct object reference without authorization
DELETE /api/order/{id}  # Can delete any order

# Good: Check ownership
if order.userId != currentUser.id:
    raise Forbidden
```

### A02: Cryptographic Failures
**What to check:**
- Sensitive data transmitted in clear text (HTTP not HTTPS)
- Old or weak cryptographic algorithms (MD5, SHA1, DES)
- Default or weak crypto keys
- Missing encryption for sensitive data at rest
- Weak random number generation for security
- Missing or improper certificate validation

**Red flags:**
- `MD5`, `SHA1` for passwords
- `math.random()`, `Random()` for security tokens
- Hardcoded encryption keys
- HTTP for sensitive data

### A03: Injection
**What to check:**
- SQL injection via string concatenation
- Command injection in system calls
- LDAP injection
- XPath injection
- NoSQL injection
- OS command injection

**Examples:**
```python
# Bad: SQL injection
query = f"SELECT * FROM users WHERE name = '{user_input}'"

# Good: Parameterized query
query = "SELECT * FROM users WHERE name = ?"
cursor.execute(query, (user_input,))
```

```javascript
// Bad: Command injection
exec(`ping ${userInput}`)

// Good: Sanitize and validate
const sanitized = userInput.replace(/[^a-zA-Z0-9.-]/g, '')
exec(`ping ${sanitized}`)
```

### A04: Insecure Design
**What to check:**
- Missing security requirements in design
- Lack of threat modeling
- No security controls for business logic
- Missing rate limiting on sensitive operations
- No security by design practices

**Examples:**
- Password reset without rate limiting
- Account enumeration via different error messages
- Missing CAPTCHA on sensitive forms
- No email verification for account changes

### A05: Security Misconfiguration
**What to check:**
- Default credentials still in use
- Unnecessary features enabled
- Default accounts enabled
- Error messages revealing stack traces
- Missing security headers
- Outdated software/libraries
- Directory listing enabled
- Verbose error messages in production

**Red flags:**
- `DEBUG=True` in production
- Exposing `.git`, `.env` files
- Default admin passwords
- Missing HTTPS
- Missing security headers (CSP, X-Frame-Options, etc.)

### A06: Vulnerable and Outdated Components
**What to check:**
- Outdated libraries with known vulnerabilities
- Unmaintained dependencies
- Not tracking component versions
- Components from untrusted sources

**Tools to recommend:**
- npm audit
- pip-audit
- OWASP Dependency-Check
- Snyk
- GitHub Dependabot

### A07: Identification and Authentication Failures
**What to check:**
- Weak password requirements
- Credential stuffing not prevented
- Missing multi-factor authentication
- Exposing session IDs in URLs
- Session IDs not invalidated on logout
- Weak session management
- Missing account lockout
- Predictable session tokens

**Examples:**
```python
# Bad: Weak password check
if len(password) >= 6:
    create_user()

# Good: Strong password requirements
if len(password) >= 12 and has_uppercase and has_number and has_special:
    create_user()
```

### A08: Software and Data Integrity Failures
**What to check:**
- Insecure deserialization
- Applications using plugins/libraries from untrusted sources
- Insecure CI/CD pipeline
- Auto-update without integrity verification
- Missing digital signatures

**Examples:**
```python
# Bad: Pickle deserialization
import pickle
data = pickle.loads(user_input)  # Dangerous!

# Good: Use JSON for untrusted data
import json
data = json.loads(user_input)
```

### A09: Security Logging and Monitoring Failures
**What to check:**
- Login/logout/authentication failures not logged
- Warnings and errors not logged
- Logs not monitored
- Logs stored locally only
- Missing audit trail for high-value transactions

**What should be logged:**
- Authentication attempts (success/failure)
- Authorization failures
- Input validation failures
- Application errors
- Security-relevant configuration changes

### A10: Server-Side Request Forgery (SSRF)
**What to check:**
- URL input without validation
- Fetching remote resources without sanitization
- Missing allowlist for remote resources
- Network segmentation not enforced

**Examples:**
```python
# Bad: SSRF vulnerability
url = request.params['url']
response = requests.get(url)  # Can access internal services

# Good: Validate against allowlist
ALLOWED_DOMAINS = ['api.example.com']
parsed = urlparse(url)
if parsed.netloc not in ALLOWED_DOMAINS:
    raise ValueError("Domain not allowed")
```

## Common Security Patterns

### Input Validation
```python
# Always validate input
def process_user_id(user_id):
    if not isinstance(user_id, int):
        raise ValueError("Invalid user ID")
    if user_id <= 0:
        raise ValueError("User ID must be positive")
    return get_user(user_id)
```

### Output Encoding
```javascript
// Prevent XSS
function displayUserContent(content) {
    // Bad: innerHTML with user content
    element.innerHTML = content;
    
    // Good: textContent or sanitize
    element.textContent = content;
    // Or use DOMPurify.sanitize(content)
}
```

### Secure Password Storage
```python
# Good: Use bcrypt or Argon2
import bcrypt

hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())

# Verify
if bcrypt.checkpw(password.encode(), hashed):
    login_user()
```

### Secure Session Management
```python
# Good: Secure session configuration
app.config.update(
    SESSION_COOKIE_SECURE=True,      # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,    # No JavaScript access
    SESSION_COOKIE_SAMESITE='Lax',   # CSRF protection
    PERMANENT_SESSION_LIFETIME=1800  # 30 min timeout
)
```

### CSRF Protection
```python
# Ensure CSRF tokens are used
# Flask-WTF, Django CSRF middleware, etc.

@app.route('/transfer', methods=['POST'])
@csrf_protect
def transfer():
    # Protected endpoint
    pass
```

### Rate Limiting
```python
from flask_limiter import Limiter

limiter = Limiter(app, key_func=get_remote_address)

@app.route('/api/login', methods=['POST'])
@limiter.limit("5 per minute")
def login():
    # Rate limited endpoint
    pass
```

## Secrets Management

### What to check:
- Hardcoded credentials in code
- API keys in version control
- Connection strings in config files
- Passwords in environment variables (better but not ideal)

### Red flags:
```python
# Bad: Hardcoded credentials
API_KEY = "sk_live_abc123..."
DATABASE_URL = "postgresql://user:password@localhost/db"

# Bad: In code
password = "MySecretPassword123"

# Better: Environment variables (but not committed)
API_KEY = os.environ.get('API_KEY')

# Best: Secret management service
API_KEY = secret_manager.get_secret('api-key')
```

## File Upload Security

### What to check:
- No file type validation
- No file size limits
- Uploaded files served from same domain
- Executable files allowed
- No virus scanning
- Path traversal in filenames

### Best practices:
```python
# Good: Validate file uploads
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

def secure_filename(filename):
    # Remove path traversal attempts
    filename = os.path.basename(filename)
    # Remove special characters
    filename = re.sub(r'[^a-zA-Z0-9._-]', '', filename)
    return filename
```

## API Security

### Best Practices:
- Use HTTPS for all endpoints
- Implement proper authentication (OAuth 2.0, JWT)
- Validate all inputs
- Use rate limiting
- Implement proper CORS policy
- Version your API
- Don't expose sensitive data in URLs
- Use proper HTTP status codes

### JWT Security:
```javascript
// Good: Secure JWT configuration
const token = jwt.sign(
    { userId: user.id },
    process.env.JWT_SECRET,
    {
        expiresIn: '1h',
        algorithm: 'HS256'
    }
);

// Verify with proper error handling
try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
} catch (err) {
    // Handle invalid token
}
```

## Security Headers

### Essential headers:
```
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
Referrer-Policy: no-referrer-when-downgrade
```

## Regular Expression DoS (ReDoS)

### What to check:
```javascript
// Bad: Vulnerable to ReDoS
const regex = /^(a+)+$/;
regex.test(userInput); // Can hang with "aaaaaaaaaaaaa...b"

// Good: Use non-backtracking patterns
const regex = /^a+$/;
```
