# 19 Sins - Prevention Essays for Each Exercise

---

## SIN 1: BUFFER OVERFLOWS
**Exercise:** Break authentication by overflowing buffer without knowing password

### How to Prevent

Use safe string handling functions that enforce bounds checking. Replace `gets()` and `strcpy()` with `fgets()` and `strncpy()` that accept buffer size parameters. Specifically, declare buffer with known size and use `strncpy(buffer, input, sizeof(buffer) - 1)` to prevent writing beyond boundaries. Compiler-based defenses like stack canaries (enabled with `-fstack-protector`) detect when stack memory is corrupted. Address Space Layout Randomization (ASLR) randomizes memory addresses, making predetermined return address injection impossible. Finally, use higher-level languages like C++ strings or Python that handle bounds checking automatically.

---

## SIN 2: COMMAND INJECTION (C/C++)
**Exercise:** Create file with content "you're hacked" using shell metacharacters

### How to Prevent

Never pass user input to `system()`. Instead, use `execve()` with an argument array, bypassing shell interpretation. If the program must run a shell command, validate input strictly: whitelist only alphanumeric characters, dots, and forward slashes. Remove or reject special characters like `;`, `|`, `&`, `$`, and backticks. Alternatively, escape shell metacharacters by wrapping input in single quotes. The safest approach is restructuring the program to avoid shell entirely—for example, use library functions directly instead of shell commands.

---

## SIN 2: COMMAND INJECTION (Python)
**Exercise:** Print secret value by calling dispatcher only

### How to Prevent

Never use `exec()` or `eval()` on user input. These functions treat strings as executable code. If dynamic code execution is necessary, use restricted environments that limit available functions and variables. Alternatively, use safe alternatives like `ast.literal_eval()` for parsing structured data. Implement strict input validation and type checking before any execution. For this specific exercise, store secrets outside the exec'd scope or use a controlled dispatcher with no access to globals(). Use `globals()` manipulation restrictions or run code in a sandboxed subprocess.

---

## SIN 3: SQL INJECTION (Task 1)
**Exercise:** Login as admin without knowing password using comment syntax

### How to Prevent

Use parameterized queries (prepared statements) exclusively. Instead of concatenating user input into SQL strings, use placeholders: `SELECT * FROM users WHERE username = ? AND password = ?` with parameters passed separately. The database driver ensures placeholders are treated as data, never as code. Stored procedures provide another layer: they pre-compile SQL structure before data insertion. Input validation helps but isn't sufficient alone—always trust parameterized queries as the primary defense. Never rely on escaping special characters.

---

## SIN 3: SQL INJECTION (Task 2)
**Exercise:** Login with empty password using boolean logic injection

### How to Prevent

Same as Task 1: use parameterized queries. With prepared statements, `' OR '1'='1` becomes literal text, not SQL logic. The query compares the exact string `' OR '1'='1` against database values, which won't match any username. Boolean injection succeeds only when input is concatenated into SQL strings without placeholders. Implement input validation as secondary defense: reject empty passwords, validate username format, require minimum password length. Return generic error messages rather than revealing whether username or password caused failure.

---

## SIN 4: CROSS-SITE SCRIPTING (Stored)
**Exercise:** Inject script to pop alert when viewing comments

### How to Prevent

Sanitize user input before storing in database. Strip or escape HTML/JavaScript tags: convert `<` to `&lt;`, `>` to `&gt;`, quotes to `&quot;`. Use libraries designed for this like OWASP's ESAPI or DOMPurify. When retrieving comments for display, encode output again—assume stored data might be malicious. Set HTTP headers like `Content-Security-Policy: script-src 'self'` to restrict script execution to trusted sources only. Use templating engines that auto-escape by default. Validate comment length and reject scripts server-side before storage.

---

## SIN 4: CROSS-SITE SCRIPTING (Reflected)
**Exercise:** Create URL with injected script that pops alert

### How to Prevent

HTML-encode all user input from URL parameters before rendering in page. Convert `<script>` tags to `&lt;script&gt;` so browser renders as text, not code. Use `encodeURIComponent()` or equivalent when constructing URLs. Implement Content Security Policy headers forbidding inline scripts. Validate URL parameters against expected format—for a username parameter, only allow alphanumeric characters. Never use `document.innerHTML` with user data; use `textContent` instead. Use templating engines that auto-escape output by default.

---

## SIN 5: FORMAT STRING
**Exercise:** Steal secret value from memory using format specifiers

### How to Prevent

Never pass user input as the format string to `printf()` or similar functions. Use fixed format strings: `printf("%s", user_input)` instead of `printf(user_input)`. If dynamic format strings are unavoidable, validate they contain only safe specifiers (reject `%x`, `%s`, `%n`). Store sensitive data in separate, protected memory regions inaccessible through stack reading. Implement strict input validation on format strings. Compiler warnings flag dangerous patterns—enable all warnings and treat as errors. Use memory protection mechanisms that prevent reading arbitrary stack values.

---

## SIN 6: INTEGER OVERFLOW
**Exercise:** Demonstrate uint8 underflow/overflow in Solidity

### How to Prevent

Check for overflow/underflow before arithmetic operations. In modern Solidity (0.8+), overflow checking is automatic and reverts on violation. For legacy versions, use SafeMath library: `balance = balance.sub(amount)` reverts if result would be negative. Validate all calculations before assignment: ensure subtraction won't go below zero, addition won't exceed max value. Use sufficient bit widths (`uint256` instead of `uint8` when possible). Implement explicit checks: `require(balance >= amount, "Insufficient balance")`. Test edge cases: zero values, maximum values, boundary conditions.

---

## SIN 7: TRUSTING DNS
**Exercise:** Understand DNS spoofing vulnerability

### How to Prevent

Use DNS over HTTPS (DoH) or DNS over TLS (DoT) to encrypt queries, preventing interception. Implement DNSSEC validation: cryptographically verify DNS responses using digital signatures. For critical services, use IP addresses directly instead of domain names. Validate SSL/TLS certificates completely—don't trust hostnames without certificate verification. Implement certificate pinning: hard-code expected certificate fingerprints for critical domains. Monitor DNS response timing and patterns for anomalies. Use secure DNS providers that implement DNSSEC. Never rely solely on DNS for security-critical decisions.

---

## SIN 8: FAILING TO PROTECT NETWORK TRAFFIC
**Exercise:** Conceptual understanding of network attacks

### How to Prevent

Encrypt all network traffic using TLS 1.2 or higher. Use HTTPS exclusively for web applications—never HTTP for sensitive data. Implement strong authentication before transmitting sensitive data. Use VPNs for remote access. Implement message authentication codes (HMAC) for integrity verification. Assume attackers have network access and control intermediate devices. Disable insecure protocols (HTTP, Telnet, FTP). Use certificate pinning for mobile apps. Implement replay attack prevention with timestamps and nonces. Test with packet sniffers to verify no sensitive data transmits unencrypted.

---

## SIN 9: FAILING TO STORE AND PROTECT DATA
**Exercise:** Conceptual understanding of data storage vulnerabilities

### How to Prevent

Apply restrictive file permissions: owner read/write only (600). Never use "Everyone: Full Control" or "World: Write". Encrypt sensitive files at rest using OS-level encryption (BitLocker, FileVault). Store secrets in environment variables or dedicated secrets management systems (HashiCorp Vault, AWS Secrets Manager). Never hardcode credentials in source code. Use ACLs to restrict file access. Encrypt database backups. Audit file permissions before and after deployment. Implement logging to detect unauthorized access. Rotate credentials regularly. Use code scanning tools to prevent secrets in version control.

---

## SIN 10: WEAK RANDOM NUMBERS
**Exercise:** Conceptual understanding of cryptographic randomness

### How to Prevent

Use system cryptographic random number generators: `secrets` module in Python, `crypto.getRandomValues()` in JavaScript, `/dev/urandom` in C/Linux. Never use `rand()` or `Math.random()` for cryptographic purposes. Seed generators with high entropy (128+ bits minimum). For session IDs and tokens, use `secrets.token_hex(32)` or equivalent. Validate CRNG availability—fail securely if unavailable. For key generation, use established libraries like OpenSSL. Never seed with predictable values like timestamps or PIDs. Generate keys with sufficient bit length (256+ for symmetric keys). Test randomness with statistical tests.

---

## SIN 11: IMPROPER FILE ACCESS
**Exercise:** Conceptual understanding of TOCTOU and path traversal

### How to Prevent

Combine file checks and usage atomically using `open()` with specific flags rather than separate `access()` and `open()` calls. Validate filenames strictly: whitelist allowed characters, reject `..`, `/`, and special characters. Use `lstat()` instead of `stat()` to detect symbolic links. Detect symlinks with `S_ISLNK(stat.st_mode)` and refuse to follow them for sensitive operations. Store temporary files in user-specific directories, never world-writable locations. Canonicalize paths to resolve symlinks: `realpath()` or equivalent. Implement access control checks after opening files. Use chroot jails or containers to sandbox file operations.

---

## SIN 12: IMPROPER SSL/TLS USE
**Exercise:** Conceptual understanding of certificate validation

### How to Prevent

Verify complete certificate chain to a trusted root CA before any secure communication. Check certificate expiration—reject expired certificates. Validate hostname matches certificate CN or SAN extension. Verify certificate key usage field permits the intended operation. Check revocation status via CRL or OCSP. Implement this validation in code: don't rely on SSL library defaults. Use HTTPS consistently—partial SSL (some pages encrypted, others not) creates vulnerabilities. Implement certificate pinning for mobile apps. Download fresh CRLs regularly. Use established libraries like OpenSSL correctly. Test validation with invalid/expired certificates to confirm rejection.

---

## SIN 13: WEAK PASSWORD SYSTEMS
**Exercise:** Conceptual understanding of password vulnerabilities

### How to Prevent

Use strong password hashing: bcrypt, Argon2, or PBKDF2—never MD5 or unsalted SHA1. Hash rounds/work factor should be high (bcrypt rounds ≥ 12). Implement mandatory password changes on first login. Enforce password complexity programmatically: minimum 12 characters, mix of cases/numbers/symbols. Lock accounts after 5 failed attempts. Return generic error messages ("Invalid username or password") rather than revealing whether username exists. Log authentication failures. Don't return forgotten passwords; send reset tokens instead. Use multifactor authentication. Validate passwords against known breach databases (HaveIBeenPwned API). Encrypt sensitive database data separately from password hashes.

---

## SIN 14: UNAUTHENTICATED KEY EXCHANGE
**Exercise:** Conceptual understanding of man-in-the-middle attacks

### How to Prevent

Use established protocols (TLS/SSL) for key exchange instead of custom implementations. Verify certificate chains—don't accept certificates without CA validation. Exchange fingerprints through secure out-of-band channels (phone call, in-person meeting). Implement mutual authentication: both parties verify each other's identity. Use forward secrecy (ephemeral keys) so compromise of one session doesn't reveal others. Use ECDH with authentication rather than bare key exchange. If custom protocol is necessary, consult cryptographers. Never skip authentication steps. Implement challenge-response protocols. Detect man-in-the-middle attempts by monitoring for multiple connections with different keys.

---

## SIN 15: SIGNAL RACE CONDITIONS (Reentrancy)
**Exercise:** Conceptual understanding of race conditions in smart contracts

### How to Prevent

Update state before external calls: implement the Checks-Effects-Interactions (CEI) pattern. Decrement balance before sending funds; if sending fails, balance already decreased prevents re-entry. Use reentrancy guards (mutex locks): set flag before external call, check flag prevents recursive calls. Implement pull-over-push pattern: let users withdraw themselves rather than sending automatically. Use established libraries like OpenZeppelin's ReentrancyGuard. For multithreaded code, use locks (mutexes) to ensure state updates are atomic. Avoid calling untrusted code. Write code without side effects or dependencies on execution order. Thoroughly test concurrent execution scenarios.

---

## SIN 16: MAGIC URLS AND HIDDEN FORMS
**Exercise:** Conceptual understanding of client-side security illusion

### How to Prevent

Never embed sensitive data in URLs or hidden form fields. Store sensitive information server-side in session storage. Pass only session IDs or opaque tokens to clients. Validate all server-side calculations: retrieve prices from database, not from client forms. For data that must be sent to client, sign it cryptographically—tampering invalidates signature. Use HTTPS exclusively (doesn't solve this alone; client still controls data). Implement strict input validation on server: reject unexpected values. For prices/amounts, fetch from authoritative source. Assume all client data is attacker-controlled. Never trust hidden fields or URL parameters for authorization or amounts.

---

## SIN 17: FAILING TO HANDLE ERRORS
**Exercise:** Conceptual understanding of error handling vulnerabilities

### How to Prevent

Check return values of all critical functions. Log detailed errors server-side; return generic messages to clients. Never expose stack traces, file paths, or internal structure to users. Fail securely: if error handling fails, deny access rather than granting it. Use specific exception types; catch what you expect, not generic exceptions. Implement comprehensive logging with timestamps and context. Test error paths: what happens when operations fail? Ensure graceful degradation, not crash or weird behavior. Use assertions for invariants. Validate error codes carefully—don't assume 0 always means success. Monitor error logs for attack patterns. Return single generic message for all authentication failures.

---

## SIN 18: POOR USABILITY
**Exercise:** Conceptual understanding of security-usability trade-offs

### How to Prevent

Understand your user base: design security for their capability level. Provide clear, actionable security messages without technical jargon. Use progressive disclosure: basic info first, advanced details on request. Make security the default—don't make users choose insecurity. Test security features with real users; observe where they struggle. Make security warnings contextual and honest. Implement remember-device options to reduce multifactor authentication friction. Use password managers—don't require users to memorize complex passwords. Provide simple, step-by-step recovery processes. Avoid overwhelming users with too many dialogs. Make dangerous operations require explicit confirmation. Balance security and usability: overly strict policies are bypassed.

---

## SIN 19: INFORMATION LEAKAGE
**Exercise:** Conceptual understanding of side channels

### How to Prevent

Use constant-time comparison for sensitive values: `hmac.compare_digest()` instead of `==`. Time all comparisons identically whether they match or not. Strip metadata from files: remove author info, timestamps, creation software. Mask file names: use UUIDs instead of descriptive names. Encrypt sensitive filenames. Avoid filenames like "PlanToBuyCompany.doc". Don't log sensitive data; mask credit cards, tokens, passwords. Use generic error messages that don't reveal whether user/password caused failure. Encrypt files at rest so even file names stay protected. Minimize timing variations in code paths. Monitor system for information leaks. Use code analysis tools to detect sensitive data exposure.

---

## EXTRA: INSECURE DIRECT OBJECT REFERENCES (IDOR)
**Exercise:** Conceptual understanding of authorization bypass

### How to Prevent

Check user permissions before returning any object. Don't expose internal database IDs in URLs; use UUIDs or encrypted references. Verify user owns the requested resource: `if (resource.owner_id != current_user.id) reject()`. Use indirect references: store mapping between public reference and internal ID, look up each time. Implement authorization on every endpoint, not just login. Log access attempts to detect enumeration attacks. Randomize IDs to prevent guessing. Use opaque tokens instead of sequential numbers. Test thoroughly: try accessing other users' resources. Implement role-based access control (RBAC) consistently.

---

## EXTRA: CROSS-SITE REQUEST FORGERY (CSRF)
**Exercise:** Conceptual understanding of forced unintended actions

### How to Prevent

Implement CSRF tokens: generate unique, unpredictable token per session. Include token in forms; validate on submission. Tokens must be tied to user session, not global. Use SameSite cookie attribute (`SameSite=Strict`). Check Referer header: reject requests from different origins. For state-changing operations (POST, PUT, DELETE), require tokens and Referer checks. Implement proper authentication. Use short token expiration. Store tokens securely. Test: attempt CSRF from different domain. Don't rely on Referer alone—use tokens as primary defense.

---

## EXTRA: SERVER-SIDE REQUEST FORGERY (SSRF)
**Exercise:** Conceptual understanding of server fetching attacker-controlled URLs

### How to Prevent

Whitelist allowed domains explicitly—use allowlist, not blocklist. Validate URLs before fetching. Block internal IP ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 127.0.0.0/8). Resolve hostnames and verify resulting IP is not internal. Use DNS rebinding protection: check DNS result again before connecting. Implement strict URL parsing. Reject URLs with unusual ports. Disable URL following (redirects). Run fetch operations with minimal privileges. Implement timeout to prevent resource exhaustion. Log all external requests. Use security groups/firewall to prevent internal access. Never let users specify arbitrary URLs for fetching.

---

## EXTRA: INSECURE DESERIALIZATION
**Exercise:** Conceptual understanding of code execution via untrusted data

### How to Prevent

Never deserialize untrusted data. Use safe formats: JSON inherently supports only basic types. If deserialization is necessary, use `json.loads()` instead of `pickle.loads()` or `yaml.load()`. Use `yaml.safe_load()` for YAML. Avoid Java serialization for untrusted data. Implement strict input validation before deserialization: check type and schema. Use libraries with restricted deserialization (allowlists for classes). Run deserialization in sandboxed subprocess with restricted permissions. Implement length/size limits. Never update serialization libraries—upgrade them. Test with malicious payloads. Use code scanning tools to detect dangerous deserialization patterns.

