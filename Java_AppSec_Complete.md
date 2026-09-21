# Java Application Security — Complete Guide
### From Base Level to Architect Level

---

## TL;DR

Security is not a feature you add at the end — it is a property of every line of code. This document covers every layer: how attacks work, what vulnerable code looks like, what secure code looks like, and why the fix works. Organized from ground-level basics to advanced architectural controls.

**Reading order for an interview:** Input Validation → Auth → Crypto → Dependency Scanning → Secrets → Logging → Advanced attacks → Finding vulns.

---

## PART 1 — INJECTION ATTACKS (The #1 Attack Category)

---

### 1.1 SQL Injection

#### What it is

SQL injection happens when user-supplied input is concatenated directly into a SQL string. The attacker can break out of the intended query and run arbitrary SQL — read all data, bypass authentication, delete tables, or even get OS command execution on some databases.

#### Vulnerable code

```java
// DANGEROUS — never do this
public User login(String username, String password) {
    String sql = "SELECT * FROM users WHERE username='" + username
               + "' AND password='" + password + "'";
    Statement stmt = conn.createStatement();
    ResultSet rs = stmt.executeQuery(sql);
    return rs.next() ? mapUser(rs) : null;
}

// Attack input:
// username = admin'--
// password = anything
// Resulting query: SELECT * FROM users WHERE username='admin'--' AND password='anything'
// The -- comments out the password check → attacker logs in as admin with no password
```

#### Secure code

```java
// SAFE — PreparedStatement separates SQL structure from data
public User login(String username, String password) {
    String sql = "SELECT * FROM users WHERE username = ? AND password_hash = ?";
    PreparedStatement pstmt = conn.prepareStatement(sql);
    pstmt.setString(1, username);
    pstmt.setString(2, hashPassword(password));  // compare hash, not plaintext
    ResultSet rs = pstmt.executeQuery();
    return rs.next() ? mapUser(rs) : null;
}
```

**Why it works:** The `?` placeholders are compiled into the query plan first. When you call `setString()`, the value is bound as a data parameter — it can never change the query structure. Even if the user types `admin'--`, the database treats the entire string as the username value, not as SQL syntax.

#### JPA/Hibernate pitfalls — JPQL injection is real

```java
// VULNERABLE — string concatenation in JPQL
String jpql = "FROM User WHERE username = '" + username + "'";
Query query = em.createQuery(jpql);

// SAFE — named parameters
String jpql = "FROM User WHERE username = :username";
Query query = em.createQuery(jpql).setParameter("username", username);

// SAFE — Criteria API (type-safe, immune to injection)
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<User> cq = cb.createQuery(User.class);
Root<User> root = cq.from(User.class);
cq.where(cb.equal(root.get("username"), username));
```

---

### 1.2 Command Injection

#### What it is

Your app calls `Runtime.exec()` or `ProcessBuilder` with user-supplied data. The attacker injects shell metacharacters (`; | & $()`) to run additional OS commands.

#### Vulnerable code

```java
// DANGEROUS
public String pingHost(String hostname) throws Exception {
    String cmd = "ping -c 1 " + hostname;
    Process p = Runtime.getRuntime().exec(cmd);
    // Attack: hostname = "google.com; rm -rf /tmp"
    // Runs: ping -c 1 google.com; rm -rf /tmp
}
```

#### Secure code

```java
// SAFE — use array form, OS shell is never invoked
public String pingHost(String hostname) throws Exception {
    // Validate first — whitelist only valid hostnames
    if (!hostname.matches("^[a-zA-Z0-9.-]{1,253}$")) {
        throw new IllegalArgumentException("Invalid hostname");
    }
    ProcessBuilder pb = new ProcessBuilder("ping", "-c", "1", hostname);
    // Each argument is a separate array element — no shell parsing
    pb.redirectErrorStream(true);
    Process p = pb.start();
    return new String(p.getInputStream().readAllBytes());
}
```

**Why it works:** When you pass a `String[]` (or list) to `ProcessBuilder`, the OS `exec()` syscall is called directly — no shell is spawned to interpret metacharacters. The semicolons and pipes are passed as literal characters to `ping`.

---

### 1.3 XML External Entity (XXE) Injection

#### What it is

XML parsers by default support "external entities" — references to files or URLs (`file:///etc/passwd`, `http://internal-service`). An attacker uploads a crafted XML document to read internal files or trigger SSRF.

#### Attack payload

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<user><name>&xxe;</name></user>
<!-- The parser replaces &xxe; with the contents of /etc/passwd -->
```

#### Vulnerable code

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(inputStream);  // Processes external entities by default
```

#### Secure code

```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();

// Disable every form of external entity processing
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);
dbf.setXIncludeAware(false);
dbf.setExpandEntityReferences(false);

DocumentBuilder db = dbf.newDocumentBuilder();
Document doc = db.parse(inputStream);  // Now safe
```

**Rule:** Whenever you parse untrusted XML, disable DOCTYPE declarations entirely. The easiest way is setting `disallow-doctype-decl` to true — it rejects the entire document if it contains a DOCTYPE.

---

### 1.4 Spring Expression Language (SpEL) Injection

#### What it is

Spring's `@Value` and `SpelExpressionParser` evaluate expressions at runtime. If user input reaches an `ExpressionParser`, they can execute arbitrary Java code.

#### Vulnerable code

```java
// DANGEROUS — user controls the expression
@GetMapping("/calculate")
public String calculate(@RequestParam String expr) {
    ExpressionParser parser = new SpelExpressionParser();
    return parser.parseExpression(expr).getValue(String.class);
    // Attack: expr = "T(java.lang.Runtime).getRuntime().exec('calc')"
    // Executes OS command
}
```

#### Secure code

Never pass user input directly to `SpelExpressionParser`. If you need dynamic expressions, use a safe math evaluator library (e.g., `exp4j`) or validate against a strict allowlist.

```java
// SAFE — validate against allowlist of known operations
private static final Pattern SAFE_EXPR = Pattern.compile("^[0-9+\\-*/().\\s]+$");

public String calculate(@RequestParam String expr) {
    if (!SAFE_EXPR.matcher(expr).matches()) {
        throw new IllegalArgumentException("Invalid expression");
    }
    // Now safe to evaluate as math-only expression
}
```

---

## PART 2 — AUTHENTICATION & SESSION MANAGEMENT

---

### 2.1 Password Hashing — The Right Way

#### Why MD5 and SHA-1 are broken for passwords

MD5 and SHA-1 are **fast** hash functions — designed for data integrity, not password storage. A modern GPU can compute 10 billion MD5 hashes per second. An attacker with a database of stolen `MD5(password)` values can crack most passwords in minutes using rainbow tables or brute force.

**Passwords need slow hashes** — hashing algorithms deliberately designed to be expensive, so brute-forcing is impractical.

#### The three acceptable algorithms

**BCrypt** — industry standard, adaptive cost factor, built-in salt.
```java
// Dependency: spring-security-crypto or org.mindrot:jbcrypt

import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(12); // cost factor 12

// When user registers:
String hash = encoder.encode(rawPassword);
userRepository.save(user.withPasswordHash(hash));

// When user logs in:
if (!encoder.matches(rawPassword, storedHash)) {
    throw new BadCredentialsException("Invalid credentials");
}
// matches() is timing-safe (constant-time comparison) — prevents timing attacks
```

**Argon2** — winner of Password Hashing Competition (2015), better than BCrypt for modern hardware. Configurable memory + parallelism + iterations.
```java
// Spring Security 5.8+
PasswordEncoder encoder = new Argon2PasswordEncoder(
    16,    // salt length bytes
    32,    // hash length bytes
    1,     // parallelism
    65536, // memory (64 MB) — makes GPU attacks expensive
    3      // iterations
);
String hash = encoder.encode(rawPassword);
```

**PBKDF2** — NIST-recommended, FIPS-compliant. Use when compliance requires it.
```java
// Java standard library
SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
KeySpec spec = new PBEKeySpec(
    password.toCharArray(),
    salt,         // 16+ bytes of SecureRandom salt
    310_000,      // NIST recommends 310,000 iterations for SHA-256 (2023)
    256           // output length in bits
);
byte[] hash = factory.generateSecret(spec).getEncoded();
```

#### Never do this

```java
// ALL OF THESE ARE WRONG:
MessageDigest.getInstance("MD5").digest(password.getBytes())   // fast, no salt
MessageDigest.getInstance("SHA-1").digest(password.getBytes()) // fast, no salt
MessageDigest.getInstance("SHA-256").digest(password.getBytes()) // fast, no salt
password.hashCode()  // obvious disaster
```

---

### 2.2 JWT Security — Common Mistakes and Fixes

#### Attack 1 — Algorithm confusion (alg=none)

Some JWT libraries accept `"alg": "none"` — meaning no signature required. An attacker modifies the payload and sets `alg: none` to forge any token.

```java
// VULNERABLE — library accepts any algorithm the token specifies
Jwt jwt = Jwts.parser().setSigningKey(secret).parseClaimsJws(token);
// If attacker sends token with alg:none, library may skip verification

// SAFE — explicitly specify the algorithm, never trust the header
Jwt jwt = Jwts.parserBuilder()
    .setSigningKey(Keys.hmacShaKeyFor(secret))
    .requireAlgorithm("HS256")  // reject tokens with any other alg
    .build()
    .parseClaimsJws(token);
```

#### Attack 2 — Algorithm confusion (RS256 → HS256)

If your server uses RS256 (asymmetric — private key signs, public key verifies), an attacker can:
1. Grab your public key (it's public — JWKS endpoint)
2. Change the token header to `"alg": "HS256"`
3. Sign the forged token using **your public key as the HMAC secret**
4. A vulnerable library verifies the HMAC using the public key → accepts the forged token

```java
// SAFE — hardcode the expected algorithm, never read it from the token header
JwtParser parser = Jwts.parserBuilder()
    .setSigningKey(publicKey)            // RSA public key
    .requireAlgorithm("RS256")           // hard-coded — attacker cannot override
    .build();
```

#### Attack 3 — Weak HS256 secret

HS256 tokens signed with a short/guessable secret can be cracked offline with tools like `hashcat`. Rule: HS256 secret must be at least 256 bits (32 bytes) of cryptographic randomness.

```java
// WRONG — short, guessable secret
String secret = "myapp";
// WRONG — company name
String secret = "SAP-Labs-2026";

// RIGHT — 256-bit random secret, stored in Vault/env var, never in code
byte[] secretBytes = new byte[32];
new SecureRandom().nextBytes(secretBytes);
SecretKey key = Keys.hmacShaKeyFor(secretBytes);

// Better: use RS256 (asymmetric) for production — no shared secret to leak
KeyPairGenerator gen = KeyPairGenerator.getInstance("RSA");
gen.initialize(2048);
KeyPair pair = gen.generateKeyPair();
// Sign with pair.getPrivate(), verify with pair.getPublic()
```

#### Always validate these JWT claims

```java
Jwts.parserBuilder()
    .setSigningKey(publicKey)
    .requireIssuer("https://auth.mycompany.com")   // reject tokens from other issuers
    .requireAudience("my-api")                      // reject tokens meant for other services
    .requireAlgorithm("RS256")
    .build()
    .parseClaimsJws(token);
// exp (expiry) is checked automatically by JJWT
```

---

### 2.3 Session Management

#### Session fixation

Before authenticating the user, always **rotate the session ID**. If you don't, an attacker who plants a known session ID can wait for the victim to log in and then use that ID to impersonate them.

```java
// Spring Security handles this automatically with:
http.sessionManagement()
    .sessionFixation().changeSessionId()  // NEW session ID on login (default in Spring Security 5+)
    .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
```

#### Secure cookie flags

```java
// In Spring Boot (application.properties):
// server.servlet.session.cookie.secure=true      → only sent over HTTPS
// server.servlet.session.cookie.http-only=true   → not accessible via JavaScript (blocks XSS theft)
// server.servlet.session.cookie.same-site=Strict → not sent in cross-site requests (CSRF protection)

// Programmatically:
@Bean
public CookieSerializer cookieSerializer() {
    DefaultCookieSerializer serializer = new DefaultCookieSerializer();
    serializer.setCookieName("SESSIONID");
    serializer.setUseHttpOnlyCookie(true);
    serializer.setUseSecureCookie(true);
    serializer.setSameSite("Strict");
    return serializer;
}
```

---

## PART 3 — AUTHORIZATION & ACCESS CONTROL

---

### 3.1 Broken Object Level Authorization (BOLA / IDOR)

#### What it is

The most common API vulnerability (OWASP API #1). Your endpoint takes an ID from the request and fetches that resource — without checking that the current user *owns* that resource.

#### Vulnerable code

```java
// DANGEROUS — any logged-in user can read any invoice by guessing the ID
@GetMapping("/invoices/{id}")
public Invoice getInvoice(@PathVariable Long id) {
    return invoiceRepository.findById(id)  // no ownership check!
        .orElseThrow(() -> new NotFoundException());
}
// Attack: loop through /invoices/1, /invoices/2, /invoices/3...
// Reads every invoice in the database
```

#### Secure code

```java
// SAFE — always scope to the authenticated user
@GetMapping("/invoices/{id}")
public Invoice getInvoice(@PathVariable Long id,
                          @AuthenticationPrincipal UserDetails currentUser) {
    return invoiceRepository.findByIdAndOwnerId(id, currentUser.getId())
        .orElseThrow(() -> new AccessDeniedException("Invoice not found"));
    // Note: throw 403, not 404 — don't confirm the ID exists for unauthorized users
    // Actually: 404 is better to avoid enumeration — pick one policy and be consistent
}

// Repository method ensures the SQL always includes owner_id:
// SELECT * FROM invoices WHERE id = ? AND owner_id = ?
```

**Multi-tenant addition:** In SAP-style multi-tenant systems, the check is:
```java
invoiceRepository.findByIdAndTenantId(id, TenantContext.getCurrentTenantId())
```

Both the user ownership AND the tenant ownership must be verified.

---

### 3.2 Spring Method-Level Security

```java
// Enable in your config:
@EnableMethodSecurity(prePostEnabled = true)

// On service methods:
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long userId) { ... }

@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
public UserProfile getProfile(Long userId) { ... }
// User can see their own profile; ADMIN can see anyone's

@PostAuthorize("returnObject.ownerId == authentication.principal.id")
public Document getDocument(Long docId) { ... }
// Check AFTER fetch — if returned doc doesn't belong to requester, throw 403
```

---

### 3.3 Privilege Escalation — Mass Assignment

#### What it is

When you map request body directly to a model object, an attacker can include extra fields like `isAdmin=true` that they shouldn't be allowed to set.

#### Vulnerable code

```java
// DANGEROUS — binds ALL request fields to the entity
@PutMapping("/users/{id}")
public User updateUser(@PathVariable Long id, @RequestBody User user) {
    user.setId(id);
    return userRepository.save(user);  // Attacker can include "admin": true in body
}
```

#### Secure code

```java
// SAFE — use a DTO that only contains fields users are allowed to update
@PutMapping("/users/{id}")
public User updateUser(@PathVariable Long id,
                       @RequestBody @Valid UserUpdateRequest request) {
    User existing = userRepository.findById(id).orElseThrow();
    existing.setDisplayName(request.getDisplayName()); // only safe fields
    existing.setEmail(request.getEmail());
    // No way to set isAdmin through this endpoint
    return userRepository.save(existing);
}

// UserUpdateRequest only has the fields users can change:
public class UserUpdateRequest {
    @NotBlank String displayName;
    @Email String email;
    // No isAdmin, no role, no tenantId
}
```

---

## PART 4 — CRYPTOGRAPHY

---

### 4.1 AES — Use GCM, Never ECB

#### Why ECB (Electronic Codebook) is broken

ECB encrypts each 16-byte block independently. Identical plaintext blocks produce identical ciphertext blocks. If you encrypt an image with ECB, the outlines of the image are still visible in the ciphertext — the pattern leaks. ECB provides no semantic security.

```java
// WRONG — ECB mode
Cipher.getInstance("AES/ECB/PKCS5Padding")

// WRONG — CBC without authentication (vulnerable to padding oracle attacks)
Cipher.getInstance("AES/CBC/PKCS5Padding")

// RIGHT — GCM mode: authenticated encryption
// Provides both confidentiality AND integrity (detects tampering)
public byte[] encrypt(byte[] plaintext, SecretKey key) throws Exception {
    byte[] iv = new byte[12]; // GCM standard IV size: 12 bytes
    new SecureRandom().nextBytes(iv);

    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    GCMParameterSpec spec = new GCMParameterSpec(128, iv); // 128-bit auth tag
    cipher.init(Cipher.ENCRYPT_MODE, key, spec);

    byte[] ciphertext = cipher.doFinal(plaintext);

    // Prepend IV to ciphertext — IV is not secret, needed for decryption
    ByteBuffer buf = ByteBuffer.allocate(iv.length + ciphertext.length);
    buf.put(iv);
    buf.put(ciphertext);
    return buf.array();
}

public byte[] decrypt(byte[] ivAndCiphertext, SecretKey key) throws Exception {
    ByteBuffer buf = ByteBuffer.wrap(ivAndCiphertext);
    byte[] iv = new byte[12];
    buf.get(iv);
    byte[] ciphertext = new byte[buf.remaining()];
    buf.get(ciphertext);

    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    cipher.init(Cipher.DECRYPT_MODE, key, new GCMParameterSpec(128, iv));
    return cipher.doFinal(ciphertext); // Throws if ciphertext was tampered
}
```

**Why GCM:** AES-GCM is authenticated encryption — it detects if the ciphertext was modified. If someone tampers with encrypted data, `doFinal()` throws `AEADBadTagException`. CBC does not detect tampering, making it vulnerable to padding oracle attacks.

---

### 4.2 SecureRandom vs Math.random

```java
// WRONG — predictable, not cryptographically random
double token = Math.random();
int otp = (int)(Math.random() * 1000000);
String sessionId = String.valueOf(System.currentTimeMillis()); // trivially predictable

// RIGHT — cryptographically secure random number generator (CSPRNG)
SecureRandom sr = new SecureRandom();

// Generate a random token
byte[] tokenBytes = new byte[32]; // 256 bits
sr.nextBytes(tokenBytes);
String token = Base64.getUrlEncoder().withoutPadding().encodeToString(tokenBytes);

// Generate a 6-digit OTP
int otp = sr.nextInt(1_000_000); // 0–999999

// Generate a UUID (also fine, uses SecureRandom internally)
String sessionId = UUID.randomUUID().toString();
```

**Why it matters:** `Math.random()` uses a linear congruential generator seeded with the current time. An attacker who observes one output can predict all future outputs. `SecureRandom` uses the OS entropy source (`/dev/urandom` on Linux) — cryptographically unpredictable.

---

### 4.3 TLS Configuration — Disable Weak Protocols and Ciphers

```java
// When creating HTTPS connections programmatically:
SSLContext ctx = SSLContext.getInstance("TLS");
ctx.init(keyManagers, trustManagers, new SecureRandom());

SSLParameters params = new SSLParameters();
// Only allow TLS 1.2 and 1.3 — disable SSLv3, TLS 1.0, TLS 1.1
params.setProtocols(new String[]{"TLSv1.2", "TLSv1.3"});
// Only allow strong cipher suites
params.setCipherSuites(new String[]{
    "TLS_AES_256_GCM_SHA384",           // TLS 1.3
    "TLS_CHACHA20_POLY1305_SHA256",     // TLS 1.3
    "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384", // TLS 1.2
    "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"  // TLS 1.2
});
```

In `application.properties` for Spring Boot:
```properties
server.ssl.enabled=true
server.ssl.protocol=TLS
server.ssl.enabled-protocols=TLSv1.2,TLSv1.3
# Disable weak ciphers — explicitly exclude RC4, DES, 3DES, NULL, EXPORT
server.ssl.ciphers=TLS_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

---

## PART 5 — DEPENDENCY & SUPPLY CHAIN SECURITY

---

### 5.1 Why Transitive Dependencies Are Dangerous

Your `pom.xml` has 20 direct dependencies. Each of those has their own dependencies. A typical Spring Boot app has 200+ jars on the classpath. One CVE in any of them — including ones you've never heard of — is your vulnerability.

**Log4Shell (CVE-2021-44228)** — A critical RCE in Log4j 2.x, included transitively in thousands of Java applications. Most teams didn't even know they were using Log4j — it was pulled in by a third-party library. This is why transitive dependency scanning is non-negotiable.

### 5.2 OWASP Dependency-Check (Maven)

```xml
<!-- pom.xml — add to plugins section -->
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>9.0.9</version>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS> <!-- Fail CI if any CVE >= 7.0 (HIGH) -->
        <format>HTML</format>
        <outputDirectory>${project.build.directory}/dependency-check-report</outputDirectory>
    </configuration>
    <executions>
        <execution>
            <goals><goal>check</goal></goals>
        </execution>
    </executions>
</plugin>
```

Run: `mvn dependency-check:check`

This downloads the NVD (National Vulnerability Database) and cross-references every jar in your project. Any jar with a known CVE score above your threshold fails the build.

### 5.3 GitHub Dependabot / Snyk

For automated monitoring — these tools watch your `pom.xml` or `build.gradle` and open PRs automatically when a dependency has a new CVE:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "maven"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

### 5.4 Audit Commands

```bash
# Maven — list all dependency versions including transitive
mvn dependency:tree

# Find a specific library in the tree (e.g., find all log4j versions)
mvn dependency:tree | grep log4j

# Gradle
./gradlew dependencies
./gradlew dependencyInsight --dependency log4j-core

# Check for known vulnerabilities via Snyk CLI
snyk test --all-projects
```

---

## PART 6 — SECRETS MANAGEMENT

---

### 6.1 What NEVER Goes in Code or Git

```java
// CATASTROPHICALLY WRONG — all of these have been leaked in real breaches

// Hardcoded DB password
DataSource ds = DriverManager.getConnection(
    "jdbc:postgresql://prod-db.company.com:5432/appdb",
    "appuser",
    "S3cur3P@ssw0rd123"   // ← anyone with repo access has prod DB access
);

// Hardcoded API key
String apiKey = "sk-live-abc123xyz789";

// Hardcoded JWT secret
String jwtSecret = "mySecretKey";

// Hardcoded encryption key
byte[] aesKey = "1234567890123456".getBytes();
```

Git remembers everything — even if you delete the file and commit again, the secret is in git history forever. Tools like `truffleHog` and `git-secrets` scan git history specifically for this.

### 6.2 The Right Way — Environment Variables and Vault

**Environment variables (minimum baseline):**
```java
// Read from environment — never hardcode
String dbPassword = System.getenv("DB_PASSWORD");
String jwtSecret   = System.getenv("JWT_SECRET");
String apiKey      = System.getenv("EXTERNAL_API_KEY");

// In Spring Boot — application.properties:
spring.datasource.password=${DB_PASSWORD}
// Then set DB_PASSWORD in your container/CF/K8s environment
```

**K8s Secrets (better):**
```yaml
# Create secret
kubectl create secret generic app-secrets \
  --from-literal=db-password=actualpassword \
  --from-literal=jwt-secret=actualSecret256bits

# Mount in pod
envFrom:
  - secretRef:
      name: app-secrets
```

**HashiCorp Vault (production standard):**
```java
// Spring Cloud Vault — auto-fetches secrets at startup
// application.properties:
spring.cloud.vault.token=s.xxxxxx
spring.cloud.vault.scheme=https
spring.cloud.vault.host=vault.company.com
spring.cloud.vault.kv.enabled=true
spring.cloud.vault.kv.backend=secret
spring.cloud.vault.kv.application-name=my-service

// Vault auto-injects DB_PASSWORD, JWT_SECRET into Spring's Environment
// Your code reads: @Value("${db.password}") — no hardcoding anywhere
```

### 6.3 Detecting Leaked Secrets

```bash
# Pre-commit hook — scan staged files before every commit
pip install detect-secrets
detect-secrets scan > .secrets.baseline
# Add to pre-commit hooks — blocks commits with secrets

# Scan existing git history
trufflehog git https://github.com/yourorg/yourrepo
# Reports any high-entropy strings (potential secrets) in history

# GitHub native secret scanning — enable in repo settings
# Automatically detects patterns for 200+ secret types (AWS keys, GitHub tokens, etc.)
```

---

## PART 7 — LOGGING SECURITY

---

### 7.1 What NEVER Gets Logged

```java
// WRONG — logging sensitive data
log.info("User {} logged in with password {}", username, password);
log.debug("JWT token: {}", jwtToken);         // token can be replayed from logs
log.info("Processing card number {}", cardNumber);
log.error("Auth failed for user {}", email);  // leaks valid usernames via log enumeration

// WRONG — logging PII that may violate GDPR
log.info("Created user: name={}, email={}, dob={}", name, email, dateOfBirth);
```

```java
// RIGHT — log what happened, not the sensitive values
log.info("User login attempt: userId={}, result=SUCCESS", userId);
log.info("Payment processed: transactionId={}, amount={}", txnId, amount);
log.error("Auth failed: userId={}", userId); // no password, no full email
// Log the ID, not the sensitive value itself
```

### 7.2 Log Injection

If you log user-supplied input directly, an attacker can inject fake log lines or CRLF characters to forge log entries and confuse your log analysis.

```java
// VULNERABLE
String userInput = request.getParameter("username");
log.info("Login attempt for user: " + userInput);
// Attack: username = "admin\nINFO: Payment successful for admin"
// Creates a fake log line making it look like admin paid successfully

// SAFE — sanitize before logging
String safeInput = userInput.replaceAll("[\r\n\t]", "_");
log.info("Login attempt for user: {}", safeInput);
// Or use structured logging — field values can't break log structure
log.info("Login attempt", kv("username", safeInput));
```

### 7.3 Structured Logging with MDC (Mapped Diagnostic Context)

```java
// In a servlet filter or Spring interceptor — inject context on every request
@Component
public class LoggingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        String correlationId = UUID.randomUUID().toString();
        String tenantId = extractTenantId(req);

        MDC.put("correlationId", correlationId);
        MDC.put("tenantId", tenantId);
        MDC.put("requestPath", ((HttpServletRequest)req).getRequestURI());

        try {
            chain.doFilter(req, res);
        } finally {
            MDC.clear(); // ALWAYS clear — thread pool reuses threads
        }
    }
}

// logback.xml — include MDC fields in every log line automatically
<pattern>%d{ISO8601} [%thread] [correlationId=%X{correlationId}] [tenant=%X{tenantId}] %-5level %logger - %msg%n</pattern>
```

Now every log line automatically includes `correlationId` and `tenantId` without you passing them as parameters everywhere. In Kibana/ELK you can filter `correlationId:abc-123` to see the entire request trace across all services.

---

## PART 8 — ADVANCED VULNERABILITIES

---

### 8.1 Java Deserialization — Remote Code Execution

#### What it is

Java's `ObjectInputStream.readObject()` can execute arbitrary code during deserialization if the classpath contains certain "gadget" libraries (Apache Commons Collections, Spring Framework, etc.). This is how Jenkins, WebLogic, and many enterprise Java apps were owned in real breaches.

#### Vulnerable code

```java
// DANGEROUS — deserializing untrusted data
ObjectInputStream ois = new ObjectInputStream(untrustedInputStream);
Object obj = ois.readObject(); // Could execute OS commands if gadgets are on classpath
```

#### Secure approaches

```java
// Option 1: Use a safe serialization format instead — JSON or Protobuf
// Never use Java native serialization for data crossing trust boundaries

// Option 2: If you must use ObjectInputStream, use a ValidatingObjectInputStream
// (Apache Commons IO) to allowlist deserializable classes
ObjectInputStream ois = new ValidatingObjectInputStream(inputStream);
ois.accept(MyExpectedClass.class, AnotherSafeClass.class);
// Throws InvalidClassException if any other class is deserialized

// Option 3: JVM-level protection via -Djdk.serialFilter
// Set in JVM flags:
// -Djdk.serialFilter=com.myapp.*;!* 
// Only allows classes in com.myapp package, blocks everything else

// Option 4: Override resolveClass to implement your own filter
ObjectInputStream ois = new ObjectInputStream(inputStream) {
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {
        if (!ALLOWED_CLASSES.contains(desc.getName())) {
            throw new InvalidClassException("Unauthorized deserialization: " + desc.getName());
        }
        return super.resolveClass(desc);
    }
};
```

**Rule:** Never deserialize data from untrusted sources using Java native serialization. Use JSON (Jackson, Gson), MessagePack, or Protobuf instead.

---

### 8.2 SSRF — Server-Side Request Forgery

#### What it is

Your app fetches a URL based on user input. An attacker supplies an internal URL — `http://169.254.169.254/latest/meta-data/` (AWS metadata endpoint), `http://internal-service.company.com`, or `file:///etc/passwd` — to read internal resources your server can reach but the attacker can't.

#### Vulnerable code

```java
// DANGEROUS — fetching user-supplied URL without validation
@GetMapping("/fetch")
public String fetchUrl(@RequestParam String url) throws Exception {
    URL u = new URL(url);
    return new String(u.openStream().readAllBytes());
    // Attack: url = "http://169.254.169.254/latest/meta-data/iam/security-credentials/"
    // Returns AWS IAM credentials — full account takeover
}
```

#### Secure code

```java
// SAFE — validate the URL against an allowlist before fetching
private static final Set<String> ALLOWED_DOMAINS = Set.of(
    "api.github.com", "api.stripe.com"
);

public String fetchUrl(String urlStr) throws Exception {
    URL url = new URL(urlStr);

    // 1. Only allow https
    if (!"https".equals(url.getProtocol())) {
        throw new IllegalArgumentException("Only HTTPS allowed");
    }

    // 2. Allowlist the hostname — block internal/private addresses
    String host = url.getHost();
    if (!ALLOWED_DOMAINS.contains(host)) {
        throw new IllegalArgumentException("Host not allowed: " + host);
    }

    // 3. Resolve the hostname to IP and check it's not private/loopback
    InetAddress addr = InetAddress.getByName(host);
    if (addr.isLoopbackAddress() || addr.isSiteLocalAddress() || addr.isLinkLocalAddress()) {
        throw new IllegalArgumentException("Internal addresses not allowed");
    }

    // Now safe to fetch
    return new String(url.openStream().readAllBytes());
}
```

---

### 8.3 Path Traversal in File Upload

#### What it is

When handling uploaded files, if you use the user-supplied filename directly, an attacker uploads a file named `../../../etc/cron.d/evil` which writes to a system path outside your intended upload directory.

#### Vulnerable code

```java
// DANGEROUS — using original filename from upload
@PostMapping("/upload")
public String upload(@RequestParam MultipartFile file) throws Exception {
    String filename = file.getOriginalFilename(); // User-controlled!
    Path target = Paths.get("/uploads/" + filename);
    Files.write(target, file.getBytes());
    // Attack: filename = "../../etc/cron.d/evil"
    // Writes to /etc/cron.d/evil — scheduled command execution
    return "Uploaded: " + filename;
}
```

#### Secure code

```java
@PostMapping("/upload")
public String upload(@RequestParam MultipartFile file) throws Exception {
    // 1. Never use original filename — generate your own safe name
    String safeFilename = UUID.randomUUID().toString()
                        + getExtension(file.getOriginalFilename()); // .jpg, .pdf only

    // 2. Validate MIME type (not just extension — attackers rename files)
    String detectedType = Files.probeContentType(tempPath); // java.nio
    if (!ALLOWED_TYPES.contains(detectedType)) {
        throw new IllegalArgumentException("File type not allowed: " + detectedType);
    }

    // 3. Resolve within the upload directory and verify it stays inside
    Path uploadDir = Paths.get("/uploads").toAbsolutePath().normalize();
    Path targetPath = uploadDir.resolve(safeFilename).normalize();

    if (!targetPath.startsWith(uploadDir)) {
        throw new SecurityException("Path traversal detected");
    }

    // 4. Limit file size
    if (file.getSize() > 10 * 1024 * 1024) { // 10 MB max
        throw new IllegalArgumentException("File too large");
    }

    Files.write(targetPath, file.getBytes());
    return "Uploaded successfully";
}
```

---

## PART 9 — HTTP SECURITY HEADERS (Spring Security)

---

### 9.1 Configuring All Security Headers

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.headers(headers -> headers
            // HSTS: browser only connects via HTTPS for 1 year
            .httpStrictTransportSecurity(hsts -> hsts
                .includeSubDomains(true)
                .maxAgeInSeconds(31536000)
            )
            // Clickjacking protection: this page cannot be embedded in an iframe
            .frameOptions(frame -> frame.deny())
            // MIME sniffing: browser must use declared Content-Type
            .contentTypeOptions(Customizer.withDefaults())
            // XSS filter (legacy browsers)
            .xssProtection(xss -> xss.headerValue(XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK))
            // Content Security Policy: only load scripts from your own domain
            .contentSecurityPolicy(csp -> csp
                .policyDirectives("default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:")
            )
        );
        return http.build();
    }
}
```

### 9.2 CORS — The Most Misconfigured Header

#### Dangerous CORS configs

```java
// WRONG 1 — allows any origin to make credentialed requests
http.cors(cors -> cors.configurationSource(request -> {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("*"));
    config.setAllowCredentials(true); // IMPOSSIBLE combination — browsers block this
    return config;                    // But custom HTTP clients ignore CORS entirely
}));

// WRONG 2 — reflecting the Origin header back (effectively allows any origin)
config.setAllowedOriginPatterns(List.of("*"));
config.setAllowCredentials(true);
```

#### Secure CORS config

```java
// RIGHT — explicit allowlist of trusted origins
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();

    config.setAllowedOrigins(List.of(
        "https://app.company.com",
        "https://admin.company.com"
    ));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
    config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
    config.setAllowCredentials(true);
    config.setMaxAge(3600L);

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/api/**", config);
    return source;
}
```

**Rule:** Never use `allowedOrigins("*")` with `allowCredentials(true)`. If you need to accept requests from multiple origins, maintain an explicit allowlist and validate the `Origin` header against it at request time.

---

## PART 10 — HOW TO FIND VULNERABILITIES

---

### 10.1 SAST — Static Application Security Testing (Find Issues in Code)

**SpotBugs + FindSecBugs** — free, runs on your code without executing it:
```xml
<!-- pom.xml -->
<plugin>
    <groupId>com.github.spotbugs</groupId>
    <artifactId>spotbugs-maven-plugin</artifactId>
    <version>4.8.3.0</version>
    <dependencies>
        <dependency>
            <groupId>com.h3xstream.findsecbugs</groupId>
            <artifactId>findsecbugs-plugin</artifactId>
            <version>1.13.0</version>
        </dependency>
    </dependencies>
</plugin>
```

Run: `mvn spotbugs:check` — finds SQL injection, command injection, hardcoded secrets, weak crypto, XXE, path traversal, and more.

**SonarQube** — full platform with dashboard. Detects security vulnerabilities, code smells, and coverage. Run as a Docker container locally:
```bash
docker run -d -p 9000:9000 sonarqube:community
mvn sonar:sonar -Dsonar.host.url=http://localhost:9000
```

**Semgrep** — fast, rule-based scanner. Excellent for custom patterns specific to your codebase:
```bash
# Scan for common Java security issues
semgrep --config=p/java-security-audit .

# Custom rule: find any place where user input goes into Runtime.exec()
# Write rules in YAML that describe code patterns you want to flag
```

### 10.2 DAST — Dynamic Application Security Testing (Find Issues at Runtime)

**OWASP ZAP** — free web application scanner. Point it at your running app:
```bash
# Run ZAP in Docker against your local app
docker run -t owasp/zap2docker-stable zap-baseline.py \
    -t http://localhost:8080 \
    -r zap-report.html

# Active scan (more thorough — actually tries attacks):
docker run -t owasp/zap2docker-stable zap-full-scan.py \
    -t http://localhost:8080
```

ZAP automatically tests for: XSS, SQL injection, CSRF, insecure headers, open redirects, path traversal, and more. Great for catching issues SAST misses (logic bugs, runtime behavior).

**Burp Suite** — professional intercepting proxy. Used for manual pentesting:
- Configure your browser to proxy through Burp
- All requests are intercepted and logged
- Burp Scanner actively probes each endpoint
- Intruder tool: automate parameter fuzzing
- Repeater: modify and replay individual requests manually

### 10.3 Dependency Vulnerability Scanning

```bash
# OWASP Dependency-Check (standalone)
dependency-check.sh --project "MyApp" --scan ./target --format HTML

# Snyk (free tier available)
snyk test                    # scan for vulnerabilities
snyk monitor                 # continuous monitoring
snyk fix                     # auto-upgrade vulnerable deps

# GitHub Actions — automatic on every PR
# Add to .github/workflows/security.yml:
```

```yaml
name: Security Scan
on: [push, pull_request]
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Snyk
        uses: snyk/actions/maven@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'MyApp'
          path: '.'
          format: 'HTML'
          args: --failOnCVSS 7
```

### 10.4 Manual Code Review — What to Look For

When reviewing Java code for security, systematically check these patterns:

```
INJECTION:
□ Any String concatenation into SQL queries?
□ Any Runtime.exec() or ProcessBuilder with user input?
□ Any XML parsing without disabling external entities?
□ Any SpelExpressionParser with user-controlled expressions?

AUTHENTICATION:
□ Password hashed with BCrypt/Argon2? (not MD5/SHA1/SHA256)
□ JWT algorithm hardcoded? (not read from token header)
□ JWT claims validated (exp, iss, aud)?
□ Session ID rotated on login?

AUTHORIZATION:
□ Every data-fetch scoped to current user/tenant?
□ Every write endpoint checks ownership before modification?
□ Method-level security annotations present?
□ DTOs used for input (no mass assignment)?

CRYPTOGRAPHY:
□ AES in GCM mode? (not ECB, not CBC without auth)
□ SecureRandom used? (not Math.random())
□ Secrets read from environment? (not hardcoded)
□ TLS 1.2+ enforced? (no SSLv3, TLS 1.0/1.1)

LOGGING:
□ No passwords, tokens, PII in log statements?
□ User input sanitized before logging?
□ MDC cleared in finally block?

DEPENDENCIES:
□ OWASP dependency-check in CI pipeline?
□ No critical/high CVEs unresolved?
□ Old/unmaintained libraries replaced?
```

---

## PART 11 — SECURITY TESTING IN CI/CD PIPELINE

```yaml
# Full security pipeline — add to your GitHub Actions / Jenkins
stages:
  1. SAST (SpotBugs + Semgrep)         ← Every commit, fast (< 2 min)
  2. Dependency-Check (OWASP / Snyk)   ← Every commit, blocks on HIGH CVEs
  3. Unit tests with security cases     ← Every commit
  4. DAST (OWASP ZAP baseline scan)    ← Every PR, against deployed preview env
  5. Penetration test                   ← Quarterly, by security team or Bug Bounty

# The goal: catch 80% of issues at step 1–2 (cheap, automated)
# Step 4–5 catches what static analysis misses (logic bugs, runtime behavior)
```

---

## PART 12 — SECURITY QUICK REFERENCE

### Never Do This

| Bad Practice | Why | Fix |
|---|---|---|
| `String sql = "... WHERE id='" + input + "'"` | SQL injection | `PreparedStatement` with `?` |
| `Runtime.exec("ping " + hostname)` | Command injection | `ProcessBuilder` with array args + whitelist |
| `new DocumentBuilder().parse(xml)` (default) | XXE | Disable external entities |
| `MD5(password)` or `SHA256(password)` | Crackable in seconds | BCrypt/Argon2 |
| `Math.random()` for tokens/OTPs | Predictable | `SecureRandom` |
| `AES/ECB/PKCS5Padding` | Pattern leakage | `AES/GCM/NoPadding` |
| Hardcoded secrets in code/git | Leaked in repo | Env vars / Vault |
| `log.info("token: " + jwt)` | Credentials in logs | Log IDs, not secrets |
| `ObjectInputStream.readObject()` on untrusted data | RCE via gadget chains | Use JSON/Protobuf |
| `ALLOWED_ORIGINS = "*"` with credentials | Any site can impersonate user | Explicit origin allowlist |
| No ownership check on `GET /resource/{id}` | BOLA/IDOR | Scope query to current user |
| `@RequestBody Entity entity` (direct binding) | Mass assignment | Use input DTOs |
| Fetching user-supplied URLs without validation | SSRF | Protocol + host allowlist + IP check |
| Using original filename for file upload | Path traversal | Generate UUID filename |

### Security Libraries — The Standard Stack

```xml
<!-- Spring Security — auth, authz, CSRF, headers, sessions -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- BCrypt password encoder (included in spring-security-crypto) -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
</dependency>

<!-- JJWT — JWT creation and validation -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>

<!-- OWASP Java Encoder — encode output to prevent XSS -->
<dependency>
    <groupId>org.owasp.encoder</groupId>
    <artifactId>encoder</artifactId>
    <version>1.2.3</version>
</dependency>

<!-- Apache Commons Validator — input validation (email, URL, IP) -->
<dependency>
    <groupId>commons-validator</groupId>
    <artifactId>commons-validator</artifactId>
    <version>1.8.0</version>
</dependency>
```

---

## PART 13 — INTERVIEW QUESTIONS ON JAVA SECURITY

**Q: What is SQL injection and how do you prevent it in Java?**
SQL injection = user input changes the structure of a SQL query. Fix: always use `PreparedStatement` with `?` placeholders or JPA named parameters. Never concatenate user input into SQL strings.

**Q: What is the difference between authentication and authorization?**
Authentication = proving who you are (login, JWT validation). Authorization = proving you're allowed to do what you're trying to do (RBAC, ownership check, scope validation). A valid JWT proves you're authenticated — it doesn't automatically authorize every action.

**Q: Why is MD5 not acceptable for password storage?**
MD5 is a fast hash — billions of hashes per second on a GPU. Attackers use precomputed rainbow tables or brute force. Password hashes need to be slow (BCrypt, Argon2) and salted (unique random value per password prevents rainbow tables).

**Q: What is BOLA/IDOR and how do you fix it?**
Broken Object Level Authorization — your endpoint doesn't verify the requested resource belongs to the requester. An attacker changes `/invoices/123` to `/invoices/124` to read someone else's invoice. Fix: always append `AND owner_id = :currentUserId` (or `AND tenant_id = :currentTenantId`) to every data query.

**Q: What is deserialization vulnerability in Java?**
Java's `ObjectInputStream.readObject()` can execute code during deserialization if "gadget" libraries (Apache Commons Collections, Spring) are on the classpath. Attackers send crafted serialized payloads to achieve RCE. Fix: use JSON/Protobuf for data exchange, never deserialize untrusted Java objects, or use `ValidatingObjectInputStream` with a strict class allowlist.

**Q: How do you prevent SSRF?**
Server-Side Request Forgery — app fetches a user-controlled URL, allowing access to internal services. Fix: (1) validate protocol (HTTPS only), (2) allowlist permitted hostnames, (3) resolve hostname to IP and reject private/loopback ranges (`10.x`, `172.16.x`, `192.168.x`, `127.x`, `169.254.x`).

**Q: What is the alg confusion attack on JWT?**
If you accept any algorithm from the JWT header, an attacker changes `"alg":"RS256"` to `"alg":"HS256"` and signs the forged token with your public key as the HMAC secret. Your library verifies it and accepts the token. Fix: hardcode the expected algorithm in your JWT parser — never read it from the token.

**Q: How do you manage secrets in a Spring Boot application?**
Development: `.env` file (not committed to git). Production: environment variables injected by K8s Secrets or Cloud Foundry `VCAP_SERVICES`. Best practice: HashiCorp Vault with Spring Cloud Vault — secrets are fetched at startup, rotated without redeploy, and never stored in files.
