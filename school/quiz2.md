# 19 Security Sins - Complete Exercise Guide

## SIN 1: BUFFER OVERFLOWS

### Live Example Exercise
**Platform:** https://onlinegdb.com/xj3sOXGQ-

**Task:** Break authentication without knowing the correct password (Line 7)

### Step-by-Step Solution:

**Step 1: Understand the Code Structure**
- Examine the program to identify where the password is stored
- Look for a buffer variable and how input is being read
- Typically, you'll see something like: `char buffer[size]` and `gets()` or `strcpy()` being used

**Step 2: Identify the Vulnerability**
- Find the buffer that stores the password
- Look for the return address location in memory relative to this buffer
- Calculate the offset (number of bytes) between the buffer start and the return address

**Step 3: Calculate the Payload**
- Determine buffer size (e.g., 8 bytes)
- Add extra bytes to reach the return address (e.g., 16 bytes total to account for other local variables)
- The exploit string would be: `[junk data to fill buffer][junk to overwrite EBP][new return address]`

**Step 4: Craft the Attack Input**
- Input a string longer than the buffer size
- Example: If buffer is 8 bytes, input 20+ characters like: `AAAABBBBCCCCDDDDEEEE`
- This overwrites the stack and eventually the return address

**Step 5: Execute**
- Enter the malicious input when prompted for password
- The program should jump to your controlled memory location
- In this case, you're redirecting execution to bypass the authentication check or execute shell code

### WHY This Works:

**Memory Layout Understanding:**
```
Stack grows downward (lower addresses)
[Buffer - 8 bytes]
[EBP - 4 bytes]
[Return Address - 4 bytes]
[Other local variables]
```

When you write beyond the buffer's boundary, you overwrite adjacent memory. The return address tells the CPU where to go next. By overwriting it with an address you control, you redirect program execution.

**Key Concept:** The program doesn't check if your input exceeds the buffer size. `gets()` and `strcpy()` are "unsafe" because they don't perform bounds checking—they just copy data until they hit a null terminator, regardless of buffer size.

---

## SIN 2: COMMAND INJECTION

### C/C++ Example Exercise
**Platform:** https://onlinegdb.com/LUrXh_5_I

**Task:** Create a file in any folder with content "you're hacked"

### Step-by-Step Solution:

**Step 1: Analyze the Vulnerable Code**
- Look for `system()` function calls
- Identify where user input is being passed to `system()`
- Typical pattern: `system("command " + user_input)`

**Step 2: Identify Command Separators**
- Recognize shell metacharacters: `;` (command separator), `|` (pipe), `&` (background), `&&` (logical AND), `||` (logical OR)
- The semicolon is most useful: it allows you to chain multiple commands

**Step 3: Craft the Injection Payload**
- Suppose the vulnerable code is: `system(input)`
- Your malicious input: `main.cpp; echo 'you\'re hacked' > hacked.txt; cat hacked.txt`
- The actual command executed becomes:
  ```
  main.cpp; echo 'you're hacked' > hacked.txt;
  ```

**Step 4: Execute the Attack**
- Enter your payload when the program prompts for input
- The semicolon terminates the first command
- Your injected command then executes with the program's privileges

**Step 5: Verify Success**
- Check if the file was created: `cat hacked.txt` or wherever you specified
- Read the file contents to confirm the data was written

### WHY This Works:

**Shell Interpretation:**
The shell interprets special characters as operators, not literal text. When you provide `command; another_command`, the shell sees this as two separate commands to execute sequentially.

**Privilege Escalation:**
Your injected command runs with the same privileges as the vulnerable program. If the program runs as root or with elevated privileges, your injected commands do too.

**No Input Validation:**
The program passes user input directly to the shell without sanitizing it, removing special characters, or escaping them properly.

### Python Example Exercise
**Platform:** https://onlinegdb.com/NroK_JEYzQ

**Task:** Print secret's value by calling dispatcher only

### Step-by-Step Solution:

**Step 1: Understand Python's exec()**
- `exec()` dynamically executes Python code from a string
- User input goes directly into `exec()` without validation

**Step 2: Identify the Target Variable**
- The secret is stored in a variable (e.g., `secret = "FLAG{...}"`)
- You need to call the dispatcher, which limits what you can do

**Step 3: Craft a Python Expression**
- You can import modules and access globals within the exec'd code
- Payload: `dispatcher(__import__('os').environ.get('SECRET'))` or similar, OR
- If the secret is in the global scope: access it through globals()
- Payload: `print(globals()['secret'])`

**Step 4: Inject through the Dispatcher Call**
- Input: `'; print(globals()['secret']); '`
- Or construct a call that uses Python's introspection capabilities

**Step 5: Observe Output**
- The secret value gets printed
- You've successfully bypassed the intended control flow

### WHY This Works:

**Dynamic Code Execution:**
`exec()` treats the string as Python source code. You're not restricted to the dispatcher function—you can use any Python construct including imports, function calls, and variable access.

**Access to Globals:**
In Python, `globals()` returns a dictionary of all global variables. By accessing this dictionary, you can retrieve any variable defined at the module level, including secrets.

---

## SIN 3: SQL INJECTION

### Live Example & Practice
**Platform:** http://192.168.0.183:45710/sqli (Password: 14305710)

**Task 1: Login with exact username "admin"**

### Step-by-Step Solution:

**Step 1: Analyze the Vulnerable Query**
- The application likely builds SQL with string concatenation:
  ```sql
  SELECT * FROM users WHERE username = '" + username_input + "' AND password = '" + password_input + "'"
  ```

**Step 2: Craft Boolean Logic Injection**
- Username input: `admin' --`
- The `--` comments out the rest of the query
- Actual SQL executed:
  ```sql
  SELECT * FROM users WHERE username = 'admin' --' AND password = '...'
  ```
- The password check is commented out, so it's ignored

**Step 3: Submit the Payload**
- Enter username: `admin' --`
- Enter any password (it won't matter because it's commented out)
- Click login

**Step 4: Verify Success**
- You should be logged in as admin without knowing the password
- The application displays admin's content

### WHY This Works:

**SQL Comment Syntax:**
In SQL, `--` starts a line comment. Everything after it is ignored. By placing `--` after the username, you eliminate the password check entirely.

**No Input Sanitization:**
The application concatenates user input directly into the SQL query without escaping special characters or validating the input format.

### Task 2: Login with empty password

### Step-by-Step Solution:

**Step 1: Craft an OR Condition**
- Username input: `' OR '1'='1`
- Password input: anything (e.g., `anything`)
- The actual SQL becomes:
  ```sql
  SELECT * FROM users WHERE username = '' OR '1'='1' AND password = 'anything'
  ```

**Step 2: Understand the Logic**
- `username = ''` is false (unless there's an empty username)
- `'1'='1'` is always true
- Due to operator precedence: `false OR (true AND password_check)` still evaluates differently
- A better payload: Username: `' OR 1=1 --` achieves: `WHERE username = '' OR 1=1 --`

**Step 3: Alternative - Direct True Condition**
- Username: `admin' OR 'a'='a`
- Password: (any input)
- SQL becomes: `WHERE username = 'admin' OR 'a'='a' AND password = '...'`
- The `OR 'a'='a'` (always true) bypasses the original authentication

**Step 4: Submit and Verify**
- The application logs you in without requiring a valid username or password

### WHY This Works:

**Boolean-Based SQL Injection:**
By injecting `OR '1'='1'`, you add a condition that's always true. SQL evaluates `condition1 OR true` as true, regardless of condition1's value, satisfying the WHERE clause for all rows.

### Best Practices to Prevent (For Your Understanding):

**Use Prepared Statements:**
```sql
query = "SELECT * FROM users WHERE username = ? AND password = ?"
cursor.execute(query, (username, password))
```
The `?` placeholder ensures user input is treated as data, not code. The SQL structure is compiled before any data is inserted.

---

## SIN 4: CROSS-SITE SCRIPTING (XSS)

### Stored XSS Live Example
**Platform:** http://192.168.0.183:45710/xss (Password: 14305710)

**Task:** Inject an XSS script to pop up an alert with your name on /xss_show

### Step-by-Step Solution:

**Step 1: Locate the Comment Input Field**
- Find the form where users can submit comments
- This is typically a text area or input field

**Step 2: Craft Your XSS Payload**
- Input: `<script>alert('YourName')</script>`
- Replace 'YourName' with your actual name
- Example: `<script>alert('John')</script>`

**Step 3: Submit the Form**
- Click the submit/post button
- The payload is sent to the server and stored in the database

**Step 4: Navigate to /xss_show**
- Go to the page that displays saved comments
- When the comment is rendered, the browser parses the HTML
- The `<script>` tag is executed
- An alert box pops up with your name

### WHY This Works:

**No Input Sanitization:**
The application accepts user input containing HTML/JavaScript tags without filtering or escaping them.

**Storage and Retrieval:**
The malicious script is stored in the database. Every time the /xss_show page loads, it retrieves and displays this comment, executing the script each time.

**Browser Parsing:**
The browser treats the retrieved comment as HTML, not plain text. It parses the `<script>` tag and executes the JavaScript code within it.

### Reflected XSS Live Example
**Platform:** http://192.168.0.183:45710/xss_r (Password: 14305710)

**Task:** Create a URL that pops up an alert with your name when a user clicks it

### Step-by-Step Solution:

**Step 1: Understand Reflected XSS**
- The application takes a URL parameter (e.g., `username`) and displays it in the welcome message
- Example normal usage: `/xss_r?username=John` displays "Welcome, John!"

**Step 2: Craft the Malicious URL**
- Base URL: `http://192.168.0.183:45710/xss_r`
- Payload parameter: `username=<script>alert('YourName')</script>`
- Full URL: `http://192.168.0.183:45710/xss_r?username=<script>alert('YourName')</script>`
- URL-encoded version: `http://192.168.0.183:45710/xss_r?username=%3Cscript%3Ealert%28%27YourName%27%29%3C%2Fscript%3E`

**Step 3: Share or Visit the Link**
- Send this URL to someone or visit it yourself
- They see a normal-looking URL from the trusted domain
- The page loads with your injected script

**Step 4: Script Executes**
- The parameter value is rendered in the welcome message without sanitization
- The browser parses the `<script>` tag and executes it
- Alert pops up immediately

### WHY This Works:

**Reflected in Response:**
The malicious code isn't stored—it's reflected back in the immediate response. The parameter value is directly embedded in the HTML response.

**URL Obfuscation:**
Users trust URLs from the legitimate domain. They don't notice the malicious parameter (especially if URL-encoded or shortened).

**No Output Encoding:**
The application doesn't HTML-encode the parameter when displaying it. `<`, `>`, and other special characters should be converted to HTML entities (`&lt;`, `&gt;`, etc.).

### Harm from XSS - Real Example:

**British Airways Data Breach (2018):**
- XSS vulnerability allowed attackers to inject code into the BA website
- Users entering credit card details on the compromised site had their data intercepted
- 380,000 customers affected
- £183 million fine
- Attackers stole session cookies, allowing them to impersonate users and access accounts

---

## SIN 5: FORMAT STRING PROBLEMS

### Live Example
**Platform:** https://onlinegdb.com/-YIE5R5wg

**Task 1: Read and understand the code**

### Step-by-Step Analysis:

**Step 1: Identify the Vulnerable Function**
- Look for `printf()`, `scanf()`, or similar format functions
- Typically: `printf(user_input)` instead of `printf("%s", user_input)`

**Step 2: Understand Format String Specifiers**
- `%x` - read a 4-byte value from the stack as hex
- `%s` - read a pointer from the stack and print the string it points to
- `%n` - write a value to a memory address (pointed to on the stack)
- `%p` - print pointer value (like %x but often used for clarity)

**Step 3: Test with Normal Input**
- Input: `hello`
- Output: `hello` (printed as is)
- Input: `%x %x %x`
- Output: prints several hex values from the stack (this reveals memory contents)

### Task 2: 'Steal'/print the secret value

### Step-by-Step Solution:

**Step 1: Understand the Target**
- A secret value is stored somewhere in memory (typically on the stack or in a global variable)
- You need to read it without having direct access

**Step 2: Reconnaissance - Find the Offset**
- Use `%x` to read stack values one by one
- Input: `%x` - prints first stack value
- Input: `%x %x` - prints second stack value
- Continue until you can identify which offset contains the secret

**Background Knowledge - Using Positional Parameters:**
- Format: `%n$x` where n is the position on the stack
- Example: `%1$x` reads the first parameter, `%5$x` reads the fifth

**Step 3: Craft the Exploit**
- If the secret is at offset 12, use: `%12$x`
- If the secret is a pointer to a string, use: `%12$s` (reads the string the pointer points to)

**Step 4: Input and Execute**
- Enter your format string payload
- The program prints the secret value

**Example Walkthrough:**
- Suppose the secret address is pushed as the 10th parameter on the stack
- Input: `%10$p` or `%10$x` - reveals the memory address
- If that address contains another pointer: Input: `%10$s` - dereferences and prints the string

### WHY This Works:

**Format Function Behavior:**
When `printf(user_input)` is called, the format function reads from the stack based on the format specifiers you provide. Each `%x` consumes and prints one stack value.

**Stack Layout:**
```
[Local variables on stack]
[Function arguments]
[Return address]
```

When you provide `%10$x`, the function treats the 10th parameter as a hex value to print. If you placed data there, it gets leaked.

**Memory Reading:**
`%s` is particularly powerful because it dereferences a pointer. If you read a memory address using `%x` and that address points to a string, using `%s` prints the entire string.

**No Bounds Checking:**
The function doesn't verify that you're only accessing valid parameters. You can read arbitrarily deep into the stack by increasing the offset number.

---

## SIN 6: INTEGER OVERFLOW

### Live Example - Solidity Smart Contract
**Platform:** Remix IDE (https://remix.ethereum.org/)

**Contract Code:**
```solidity
pragma solidity 0.7.0;
contract ChangeBalance {
    uint8 public balance;
    function decrease() public {
        balance--;
    }
    function increase() public {
        balance++;
    }
}
```

**Task:** Demonstrate integer overflow/underflow

### Step-by-Step Solution:

**Step 1: Understand uint8 Constraints**
- `uint8` can store values from 0 to 255
- When you exceed 255, it wraps back to 0
- When you go below 0, it wraps to 255

**Step 2: Test Overflow**
- Set balance to 255 (the maximum)
- Call `increase()` once
- Result: balance wraps to 0 instead of 256

**Step 3: Test Underflow**
- Set balance to 0 (the minimum)
- Call `decrease()` once
- Result: balance wraps to 255 instead of -1

**Step 4: Exploit Scenario**
- Imagine this represents account balance in a smart contract
- If you underflow the balance, you could go from 0 to 255 units
- Or if decreasing, an underflow gives you massive value (in an attacker's perspective)

### WHY This Works:

**Binary Representation:**
`uint8` uses 8 bits. Maximum value is `11111111` in binary = 255 in decimal.

When you add 1 to 255:
```
  255 = 11111111
+   1 = 00000001
------
 256 = 100000000 (9 bits needed, but only 8 bits available)
       The overflow bit is discarded, leaving 00000000 = 0
```

When you subtract 1 from 0:
```
    0 = 00000000
-   1 = 00000001
------
   -1 requires a sign bit, but uint8 is unsigned
       So it wraps around to 11111111 = 255
```

### Real-World Impact:

**Pizza Protocol Attack (December 2021):**
- $5 million stolen due to overflow vulnerability
- Attackers manipulated token amounts through overflow

---

## SIN 7: TRUSTING NETWORK NAME RESOLUTION (DNS)

### Concept Exercise:

**Task:** Understand DNS vulnerabilities and why trusting it is dangerous

### Step-by-Step Understanding:

**Step 1: Understand DNS Hierarchy**
- User's computer queries local DNS server
- Local DNS server queries TLD (Top Level Domain) server
- TLD server directs to authoritative nameserver
- Authoritative nameserver returns the IP address

**Step 2: Identify the Vulnerability**
- DNS uses UDP (connectionless, no encryption)
- No authentication of responses
- Attacker can intercept queries or responses
- Attacker can return false IP addresses (DNS spoofing)

**Step 3: Attack Scenario - DNS Spoofing**
- You want to visit `amazon.com`
- Attacker intercepts your DNS query
- Instead of the real Amazon IP, attacker returns their malicious server IP
- Your browser connects to the attacker's server thinking it's Amazon
- You enter credentials on the fake Amazon site—attacker now has your login

**Step 4: Man-in-the-Middle Prevention**
- DNS over HTTPS (DoH) encrypts the DNS query
- DNS over TLS (DoT) is another encryption method
- DNSSEC adds cryptographic signatures to verify authenticity

### WHY This Works:

**Lack of Encryption:**
DNS queries and responses are sent in plaintext. Any device on the network path can see and modify them.

**No Authentication:**
The client has no way to verify that the response truly came from the authoritative server. An attacker can craft a convincing response.

**First-Response-Wins:**
DNS doesn't verify the source of responses. The first response received (even if malicious) is accepted.

### Protection Strategies:

- Use DNS over HTTPS (DoH) to encrypt queries
- Implement DNSSEC to verify responses cryptographically
- Use trustworthy DNS providers
- Implement certificate pinning to detect spoofed certificates

---

## SIN 8: FAILING TO PROTECT NETWORK TRAFFIC

### Understanding Network Attacks:

**Step 1: Identify Insecure Protocols**
- HTTP (no encryption), SMTP, IMAP, POP (all plaintext)
- Telnet, FTP (transmit credentials in plaintext)
- These use no encryption for confidentiality or authentication

**Step 2: Recognize Attack Types**

**Eavesdropping:**
- Attacker passively listens to network traffic
- In plaintext, they see all data including passwords and sensitive information
- Example: Intercepting HTTP login credentials

**Replay Attack:**
- Attacker captures an authentication message
- Replays it later to gain unauthorized access
- Example: Capturing a session cookie and reusing it

**Spoofing:**
- Attacker impersonates a legitimate server or client
- Example: DNS spoofing sends you to the attacker's server

**Tampering:**
- Attacker modifies data in transit
- Example: Changing the amount in a bank transfer message

**Hijacking:**
- Attacker cuts out one party and takes their place
- Example: Taking over an established TCP connection

**Step 3: Protection Strategy**
- Use SSL/TLS for all sensitive communications
- Implement strong authentication before exchanging data
- Use cryptographic message authentication (HMAC) for integrity
- Assume attackers have network access; design accordingly

### WHY Standard Practice:

**The Safest Assumption:**
Treat the network as untrusted. Encrypt everything sensitive. Use TLS/SSL as a standard for all web traffic.

---

## SIN 9: FAILING TO STORE AND PROTECT DATA

### Understanding Data Protection:

**Step 1: Identify Two Main Issues**

**Issue 1: Weak Access Control**
- Files with permissions like "Everyone: Full Control"
- Database accessible without authentication
- No ACLs (Access Control Lists) on sensitive resources

**Example of Weak Permissions:**
```
File: customer_data.db
Permissions: Everyone - Read/Write
```
Anyone on the system can access customer data.

**Issue 2: Hardcoded Secrets**
- API keys, passwords, or encryption keys in source code
- Example:
  ```python
  password = "MyPassword123"
  api_key = "sk_live_abc123xyz"
  ```

**Step 2: Hardcoding Secret Exploitation**
- Attacker gains access to source code (through GitHub, decompilation, etc.)
- Extracts credentials directly
- Uses credentials to access protected resources with no audit trail

**Step 3: Weak File Permissions Exploitation**
- Attacker gains filesystem access (local privilege escalation, shared server, etc.)
- Directly copies or modifies protected files
- No logging or detection of unauthorized access

**Step 4: Best Practices for Protection**

**For Files:**
- Apply restrictive ACLs (e.g., Owner: Read/Write only)
- Encrypt sensitive files at rest
- Use operating system encryption (like BitLocker, FileVault)
- Scan filesystem before and after deployment

**For Secrets:**
- Store in environment variables
- Use secrets management systems (HashiCorp Vault, AWS Secrets Manager)
- Never commit secrets to version control
- Rotate credentials regularly
- Use operating system keyrings for storage

### WHY This Works:

**Defense in Depth:**
Multiple layers of protection ensure that even if one layer fails, others remain:
1. Strong file permissions prevent unauthorized filesystem access
2. Encryption prevents reading files even if accessed
3. Secrets management prevents hardcoding
4. Audit logging detects unauthorized access

---

## SIN 10: WEAK RANDOM NUMBERS

### Understanding the Vulnerability:

**Step 1: Purpose of Random Numbers**
- Cryptographic keys generation
- Session ID generation
- CSRF token generation
- Nonce values for challenge-response

**Step 2: Weak Random Generation**
- Using `rand()` or `Math.random()` (designed for games, not security)
- Using predictable seeds (current time, process ID)
- Using low entropy sources

**Example of Weak Generation:**
```python
import random
random.seed(int(time.time()))  # Seed is predictable (current time)
session_id = random.randint(0, 999999)  # Only 1 million possibilities
```

**Step 3: Exploitation - Private Key Example**
- A private key should be 256 bits (2^256 possibilities)
- If generated with weak randomness seeded by timestamp (32 bits), attacker only needs to try ~2^32 variations
- For 32-bit seed, this is 4.3 billion attempts (brute-forceable in seconds on modern hardware)

**Real Attack:**
- Wintermute hack ($160M stolen)
- Weak random number generation allowed attackers to predict session IDs and transaction IDs
- Attackers could forge transactions without proper authorization

**Step 4: Protection Strategy**
- Use cryptographic random number generators (CRNGs)
- In Python: `secrets` module
- In JavaScript: `crypto.getRandomValues()`
- In C/C++: system entropy source (e.g., `/dev/urandom` on Linux)
- Seed with at least 128 bits of entropy

**Proper Generation:**
```python
import secrets
session_id = secrets.token_hex(16)  # 128 bits of entropy
```

### WHY This Works:

**Entropy is Key:**
- Weak randomness has low entropy (few possible values)
- Cryptographic randomness has high entropy (essentially all possible values equally likely)
- Attacker must try all possibilities to break weak randomness
- With weak randomness, attacker can drastically reduce search space

---

## SIN 11: IMPROPER FILE ACCESS

### Understanding File Access Vulnerabilities:

**Step 1: Recognize Three Main Issues**

**Issue 1: Time-of-Check-Time-of-Use (TOCTOU)**
```
1. Program checks if file is readable: if (access("file.txt", R_OK))
2. Time gap - another process modifies the file
3. Program opens and reads file: open("file.txt")
```
Between check and use, an attacker could replace the file with a symbolic link.

**Issue 2: Filename Manipulation**
- Attacker controls the filename
- Input: `../../etc/passwd` accesses parent directories
- Input: `/etc/shadow` accesses absolute paths
- Program doesn't validate the filename path

**Issue 3: Symbolic Link Following**
- Program opens a "file" that's actually a symbolic link
- Points to a sensitive system file
- Program operates on the wrong file

**Step 2: TOCTOU Exploitation**
```
Vulnerable code:
if (access("/tmp/file.txt", R_OK) == 0) {
    // Time gap here!
    int fd = open("/tmp/file.txt", O_WRONLY);
    write(fd, data, size);
}

Attack:
1. Attacker creates symbolic link: /tmp/file.txt -> /etc/important_config
2. Program checks access (succeeds)
3. Attacker quickly removes link and creates another: /tmp/file.txt -> /root/.ssh/authorized_keys
4. Program opens and writes to attacker's link
5. Attacker's key is added to root's authorized_keys
```

**Step 3: Protection Strategies**

**For TOCTOU:**
- Combine check and use atomically
- Use `open()` with flags instead of separate `access()` and `open()`
- Use `stat()` after opening to verify file properties

**For Path Traversal:**
- Validate filenames strictly (whitelist allowed characters)
- Reject paths with `..`, `/`, or other special characters
- Use a chroot jail or sandboxed directory

**For Symbolic Links:**
- Use `lstat()` instead of `stat()` (doesn't follow symlinks)
- Check `S_ISLNK(stat.st_mode)` to detect symbolic links
- Avoid following symlinks in sensitive operations

### WHY This Works:

**Race Conditions:**
File operations aren't atomic. Time gaps between checks and uses allow attackers to intervene.

**Insufficient Validation:**
If the program doesn't validate user-supplied filenames, attackers can specify arbitrary paths.

---

## SIN 12: IMPROPER USE OF SSL/TLS

### Understanding SSL/TLS Certificate Validation:

**Step 1: What SSL/TLS Should Do**
- Encrypt data in transit (confidentiality)
- Verify data hasn't been modified (integrity)
- Authenticate the server's identity (authentication)

**Step 2: Common Implementation Mistakes**

**Mistake 1: Not Validating Certificate Chain**
- Server presents a certificate
- Certificate should chain to a trusted root CA
- If not validated, attacker's self-signed certificate is accepted

**Mistake 2: Ignoring Expiration**
- Certificate has validity period (e.g., 1-2 years)
- Expired certificates should be rejected
- If ignored, old compromised certificates are accepted

**Mistake 3: Not Checking Hostname**
- Certificate for `amazon.com` shouldn't be accepted for `evil.com`
- Hostname verification prevents man-in-the-middle attacks
- `openssl s_client` might not verify hostname but browsers do

**Mistake 4: Not Checking Revocation**
- If a certificate's private key is compromised, it's revoked
- Certificate Revocation Lists (CRLs) list revoked certs
- If revocation status isn't checked, revoked certs are accepted

**Mistake 5: Ignoring Key Usage**
- Certificate marked for "Server Authentication" shouldn't be used for client authentication
- Misuse can create security holes

**Step 3: Proper Validation Procedure**

```
Before sending sensitive data over SSL/TLS:
1. Obtain server certificate
2. Verify certificate chain to a trusted root CA
3. Check certificate is within validity period (not expired)
4. Verify hostname matches certificate CN or SAN
5. Check certificate is not revoked (via CRL or OCSP)
6. Verify certificate key usage field allows the intended use
7. Only then: proceed with secure communication
```

**Step 4: Real-World Impact**
- If any step is skipped, attacker can intercept via man-in-the-middle
- Attacker uses their own certificate
- Client accepts it if validation is incomplete
- Attacker sees all data

### WHY This Works:

**Certificate Authority System:**
Only trusted CAs can issue certificates for real domains. By validating the chain to a trusted root, you verify the certificate issuer is legitimate. Skipping this allows any certificate to be accepted.

**Hostname Binding:**
The hostname in the certificate ensures the certificate is for the server you're connecting to, not another server.

## SIN 13: USE OF WEAK PASSWORD SYSTEMS

### Understanding Password Vulnerabilities:

**Step 1: Identify Common Weaknesses**

**Password Compromise:**
- Weak passwords (dictionary words, common patterns)
- Reused passwords across services
- No protection against brute-force attacks

**Default Passwords:**
- Application ships with default credentials
- Users forget to change them
- Attackers know default credentials

**Replay Attacks:**
- Attacker captures authentication traffic
- Replays it later to gain access
- Solution: Use cryptographic challenges, not predictable patterns

**Brute-Force Attacks:**
- Attacker tries many password combinations
- Weak hashing makes this fast
- Solution: Use computationally expensive hashing (PBKDF2, bcrypt, Argon2)

**Improper Storage:**
- Passwords stored in plaintext
- Passwords stored with weak hashing (MD5, SHA1)
- Passwords stored with no salt

**User Feedback Issues:**
- "Username not found" vs. "Password incorrect" - reveals which users exist
- Allows attackers to enumerate valid usernames

**Forgotten Password Handling:**
- Returning password instead of resetting it exposes it
- Weak security questions are guessable
- SMS-based recovery is vulnerable to SIM swapping

**Step 2: Exploitation Examples**

**Dictionary Attack:**
- Attacker captures password hash (e.g., from database breach)
- Tries common passwords: `password123`, `admin`, `letmein`, `123456`
- Uses precomputed hash tables (rainbow tables)
- With weak hashing, this takes seconds to minutes

**Rainbow Table Attack:**
- Attacker purchases or generates a rainbow table
- Contains millions of password hashes precomputed
- Simply looks up stolen hash in table
- Gets password instantly (if no salt used)

**Brute-Force Without Protection:**
- If password stored as MD5 hash without salt
- Attacker can try millions of passwords per second on modern GPU
- 8-character password with weak hashing: hours to crack

**Step 3: Best Practices for Password Security**

**For Developers:**

```python
# WRONG - Never do this
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()  # Weak!

# RIGHT - Use proper password hashing
import bcrypt
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))

# Or use PBKDF2
from hashlib import pbkdf2_hmac
password_hash = pbkdf2_hmac('sha256', password.encode(), salt, 100000)
```

**Implementation Checklist:**
- Use bcrypt, Argon2, or PBKDF2 (never MD5, SHA1, or unsalted hashes)
- Automatically salt each password (good libraries do this)
- Make hashing computationally expensive (high iteration count)
- Don't return exact error messages ("user not found" vs "wrong password")
- Use single generic message: "Invalid username or password"
- Force password changes on first login
- Prevent weak passwords programmatically
- Implement account lockout after failed attempts
- Log authentication failures
- Use multifactor authentication (MFA)

**Step 4: Why This Works**

**Salting:**
```
Without salt:
password="admin" → MD5 → 21232f297a57a5a743894a0e4a801fc3

With salt:
password="admin" + salt="xyz123" → PBKDF2 → completely different hash
Even if attacker gets this hash, they can't use precomputed rainbow tables
```

**Computationally Expensive Hashing:**
- bcrypt with rounds=12 takes ~0.3 seconds per hash attempt
- Reduces effectiveness of brute-force attacks dramatically
- Attacker trying 1 million passwords would take 3+ days

**Multifactor Authentication:**
- Even if password is compromised, second factor prevents access
- Types: SMS/Email codes, authenticator apps, hardware keys, biometrics

## SIN 14: UNAUTHENTICATED KEY EXCHANGE

### Understanding Key Exchange Vulnerabilities:

**Step 1: The Basic Problem**

Before establishing encrypted communication, two parties must exchange keys. But how do they verify each other's identity?

**Vulnerable Scenario (Textbook MITM):**
```
Alice wants to communicate securely with Bob.

CORRECT FLOW:
Alice: "Hi Bob, here's my public key (KA+)"
Bob: "Hi Alice, here's my public key (KB+)"
[They verify each other's identity somehow]
[They now communicate encrypted]

VULNERABLE FLOW:
Alice: "Hi, I'm Alice, here's my public key"
Trudy (attacker) intercepts and says: "I'm Alice, here's my public key (KT+)"
Bob receives Trudy's public key thinking it's from Alice
Trudy receives Bob's public key
Result: Trudy can decrypt/encrypt messages between them
```

**Step 2: The Man-in-the-Middle Attack**

**Setup:**
- Trudy positions herself between Alice and Bob
- She can intercept all communications

**Attack Sequence:**
1. Alice sends to Bob: "Here's my public key KA+"
2. Trudy intercepts and stores KA+
3. Trudy sends to Bob: "Here's my public key KT+" (claiming to be Alice)
4. Bob receives and thinks KT+ belongs to Alice
5. Same process in reverse: Bob → Trudy (intercepts KB+) → Alice (sends KT+)
6. Result: 
   - Alice encrypts with KT+ thinking it's Bob's
   - Trudy decrypts with her private key KT-
   - Trudy re-encrypts with Bob's public key KB+
   - Bob receives message thinking it's from Alice
   - Complete visibility for Trudy

**Step 3: Real-World Examples**

**SSL/TLS Handshake Without Verification:**
- Client receives server certificate
- If client doesn't verify the certificate chain
- Attacker serves their own certificate
- Handshake succeeds but with attacker as "server"

**Diffie-Hellman Without Authentication:**
- Two parties agree on shared secret using ECDH
- If neither authenticates the other
- Attacker can substitute their own values
- Attacker gets shared secret and can decrypt

**Step 4: Protection Strategies**

**Option 1: Public Key Infrastructure (PKI)**
- Third-party Certificate Authority (CA) signs certificates
- CA verifies the entity's identity before signing
- Client verifies certificate signature using CA's public key
- Ensures certificate came from legitimate CA

**Option 2: Out-of-Band Verification**
- Exchange fingerprints through secure channel (phone, in-person)
- Verify fingerprint matches before trusting key
- Example: Signal protocol shows identity key fingerprints to users

**Option 3: Pre-Shared Knowledge**
- Both parties already know each other's public keys
- No need to exchange them insecurely
- Less practical for general internet usage

**Option 4: Use Established Protocols**
- TLS/SSL handles authentication for you
- Developed and vetted by security experts
- Use rather than implementing custom solutions

**Step 5: Why This Works**

**The Signature Chain:**
```
Root CA → Intermediate CA → Server Certificate
Each certificate signed by the one above it
Client trusts Root CA (pre-installed)
Can verify entire chain cryptographically
Ensures only legitimate servers can have valid certificates
```

**Challenge-Response Authentication:**
```
Alice sends random challenge to Bob
Bob signs challenge with his private key
Alice verifies signature with Bob's public key
Proves Bob has the corresponding private key
Bob does same for Alice
```

### WHY Skipping This is Dangerous:

- No verification that you're talking to who you think you are
- Attacker can intercept and impersonate either party
- Encryption provides confidentiality but not authentication
- Can result in complete compromise of "secure" connection

---

## SIN 15: SIGNAL RACE CONDITIONS

### Understanding Race Conditions:

**Step 1: What is a Race Condition?**

A race condition occurs when two or more processes/threads access shared resources concurrently, and the final outcome depends on the timing/ordering of execution (which is unpredictable).

**Simple Example - Two Threads Incrementing:**
```
Shared variable: count = 0

Thread 1:                    Thread 2:
1. Load count (0)           1. Load count (0)
2. Add 1 (1)                2. Add 1 (1)
3. Store count (1)          3. Store count (1)

Expected: count = 2
Actual: count = 1 (both threads read before either writes)
```

**Step 2: Reentrancy Attacks in Smart Contracts**

**Vulnerable Smart Contract Example:**
```solidity
mapping(address => uint) public balances;

function withdraw(uint amount) public {
    require(balances[msg.sender] >= amount);
    
    // Vulnerable: External call before state update
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success);
    
    balances[msg.sender] -= amount;  // Updated AFTER call
}
```

**Attack Sequence:**
1. Attacker calls `withdraw(1 ether)` with balance of 1 ether
2. Contract sends 1 ether to attacker's contract
3. Attacker's contract's fallback function triggered
4. Fallback calls `withdraw(1 ether)` again (recursive call)
5. Check passes: balance still shows 1 ether (hasn't been decremented yet)
6. Another 1 ether sent
7. Attacker can repeat until contract drained

**Step 3: Famous Reentrancy Attacks**

**The DAO Hack (2016):**
- $50 million in Ethereum stolen
- Classic reentrancy vulnerability
- Led to Ethereum hard fork to recover funds

**Uniswap/Lendf.Me Hack (April 2020):**
- $25 million stolen through reentrancy

**Multiple 2021 Hacks:**
- BurgerSwap: $7.2 million
- SURGEBNB: $4 million
- CREAM FINANCE: $18.8 million
- Siren Protocol: $3.5 million

**Step 4: Prevention Strategies**

**Strategy 1: Checks-Effects-Interactions (CEI) Pattern**
```solidity
// SECURE: Update state BEFORE external call
function withdraw(uint amount) public {
    require(balances[msg.sender] >= amount);           // CHECKS
    
    balances[msg.sender] -= amount;                     // EFFECTS (update state first)
    
    (bool success, ) = msg.sender.call{value: amount}("");  // INTERACTIONS (last)
    require(success);
}
```

**Strategy 2: Reentrancy Guard (Mutex)**
```solidity
bool private locked;

modifier nonReentrant() {
    require(!locked, "No reentrancy");
    locked = true;
    _;
    locked = false;
}

function withdraw(uint amount) public nonReentrant {
    // ...rest of code...
}
```

**Strategy 3: Pull Over Push**
```solidity
// Instead of sending funds directly
// Let users withdraw themselves
// You maintain balance accounting
// Users call withdraw() when ready
```

**Step 5: Why This Works**

**State Update First:**
If you decrement balance before making external calls, the second recursive call checks the already-decremented balance. It fails the require check and can't proceed.

**Reentrancy Guard:**
Lock prevents function from being called recursively. Once locked=true, any recursive call is blocked until lock=false.

**General Principle:**
Make sure resources are in consistent state before relinquishing control to untrusted code. Untrusted code could exploit timing to abuse the inconsistent state.

---

## SIN 16: MAGIC URLS AND HIDDEN FORM FIELDS

### Understanding These Vulnerabilities:

**Step 1: Magic URLs**

Magic URLs are URLs that contain authentication tokens, session IDs, or sensitive data visible in the URL.

**Example:**
```
http://www.example.com/user?id=TXkkZWNyZStwQSQkdzByRA==
```

**Decoding the Base64:**
```
TXkkZWNyZStwQSQkdzByRA== → My$ecre+pA$$w0rD
```

**Vulnerabilities:**
- URL visible in browser history
- URL stored in server logs
- URL in HTTP Referer headers sent to other sites
- URL bookmarked or shared
- Credentials in plaintext if HTTP (not HTTPS)

**Step 2: Hidden Form Fields**

Hidden form fields store sensitive data hoping users won't see/modify it.

**Example HTML:**
```html
<form method="POST" action="order.php">
    <input type="text" name="item" value="Widget">
    <input type="hidden" name="price" value="9.99">
    <input type="hidden" name="discount" value="0">
    <input type="submit">
</form>
```

**Exploitation:**
```
User right-clicks → View Source
User sees: <input type="hidden" name="price" value="9.99">
User modifies in DevTools or intercepts request
Changes price to 0.99 before submitting
Server accepts modified value
User pays fraction of real price
```

**Step 3: Why These Fail**

**No Client-Side Security:**
Anything in HTML/URL is on the client, which attacker controls. Never trust client-side data.

**Interception:**
Tools like Burp Suite, Fiddler, or browser DevTools let anyone:
- View request before sending
- Modify any value
- Replay modified requests

**Step 4: Protection Strategies**

**Best Practice: Server-Side Validation**
```python
# WRONG - Trusting hidden field value
price_from_form = request.form.get('price')
total = int(price_from_form)  # User can change this!

# RIGHT - Calculate server-side
item_id = request.form.get('item_id')
product = database.get_product(item_id)  # Get real price from DB
price = product.price  # Don't trust client's price
total = price * quantity
```

**For Sensitive Data:**

**Option 1: Use SSL/TLS + Server-Side Session**
```python
# Store sensitive data on server
session_id = request.cookies.get('session_id')
session = get_session(session_id)  # Retrieved from server-side storage
user_id = session['user_id']
item_id = session['item_id']
price = get_product_price(item_id)  # From database
```

**Option 2: Cryptographic Signing**
```python
# If you MUST send data to client, sign it
import hmac
import hashlib

data = {'item_id': 5, 'price': 9.99}
secret_key = "server_secret"
signature = hmac.new(
    secret_key.encode(),
    json.dumps(data).encode(),
    hashlib.sha256
).hexdigest()

# Client receives: data + signature
# On return, verify signature matches
# If user modifies data, signature won't match
```

**Option 3: Don't Use Hidden Fields for Prices**
- Use session IDs or order IDs instead
- Store actual prices server-side
- Client sees description, not price
- Better UX: "Your order" with calculated total

**Step 5: Why This Works**

**Server Authority:**
Server controls database and business logic. Client can't forge authoritative information.

**Cryptographic Signing:**
Even if attacker modifies data, signature fails. Any tampering invalidates it.

---

## SIN 17: FAILING TO HANDLE ERRORS

### Understanding Error Handling Vulnerabilities:

**Step 1: Types of Error Handling Failures**

**Type 1: Yielding Too Much Information**
```
Attacker tries SQL injection: admin' --
Server error: "Syntax error in SQL query near admin' --"
Reveals: Application uses SQL, query structure exposed
Better: "An error occurred. Please try again."
```

**Type 2: Ignoring Error Codes**
```c
FILE *fp = fopen("file.txt", "r");
// No error check - if file doesn't exist, fp is NULL
// fread(fp, ...) crashes or reads garbage
```

**Type 3: Misinterpreting Errors**
```c
int result = pthread_create(&thread, NULL, func, arg);
// Return value: 0 = success, non-zero = error
if (result) {
    printf("Success!\n");  // WRONG! 0 means success
}
```

**Type 4: Using Useless Error Values**
```c
char* read_file(const char* path) {
    FILE *fp = fopen(path, "r");
    if (fp == NULL) {
        return NULL;  // Caller doesn't know if memory was allocated or file not found
    }
    // ...
}
```

**Type 5: Handling Wrong Exceptions**
```python
try:
    result = int(user_input)
except ValueError:
    print("Invalid input")
    result = 0  # But what if out of memory? Different error type
# If MemoryError occurs, it's not caught, program crashes
```

**Step 2: Denial of Service Through Error Handling**

**Scenario:**
```python
def process_request(data):
    try:
        result = parse_complex_data(data)
    except:
        pass  # Ignore all errors
    
    # What if process_complex_data() fails partway?
    # Result might be in inconsistent state
    # Program continues with invalid state
    # Leads to crash or weird behavior
```

**Attack:**
1. Send malformed request
2. Parser enters inconsistent state
3. Subsequent requests fail
4. Service becomes unavailable (DoS)

**Step 3: Information Leakage Through Errors**

**Stack Trace Exposure:**
```
User gets 404 error page, but sees:
File: /var/www/html/private/config.php line 42
Module: database
Function: connect_db
This reveals: Directory structure, internal functions, code organization
```

**Environmental Information:**
```
"Cannot connect to database at db.internal.company.com:5432"
Reveals: Internal network structure, database server details
```

**Step 4: Best Practices for Error Handling**

**Do:**
- Check return values of all security-related functions
- Log errors on server with full details
- Return generic messages to users
- Recover gracefully from errors
- Use specific exception types
- Validate error conditions thoroughly

**Don't:**
- Ignore error codes
- Display stack traces to users
- Leak system information in error messages
- Return NULL for different error types
- Assume operations succeeded if error check is missing

**Step 5: Proper Error Handling Example**

```python
import logging

logger = logging.getLogger(__name__)

def authenticate_user(username, password):
    try:
        # Input validation
        if not username or not password:
            logger.warning(f"Authentication attempt with missing credentials from {request.remote_addr}")
            return False, "Invalid credentials"  # Generic message
        
        # Database query
        user = User.query.filter_by(username=username).first()
        if user is None:
            logger.warning(f"Authentication attempt for non-existent user: {username}")
            return False, "Invalid credentials"  # Don't reveal user doesn't exist
        
        # Password verification
        if not verify_password(password, user.password_hash):
            logger.warning(f"Failed authentication for user: {username}")
            return False, "Invalid credentials"
        
        # Success
        logger.info(f"Successful authentication for user: {username}")
        return True, user
        
    except DatabaseError as e:
        logger.error(f"Database error during authentication: {str(e)}", exc_info=True)
        return False, "System error occurred"  # Generic message
    except Exception as e:
        logger.error(f"Unexpected error during authentication: {str(e)}", exc_info=True)
        return False, "System error occurred"
```

### WHY This Works:

**Separation of Concerns:**
- Detailed errors logged on secure server
- Generic errors sent to client
- Attacker doesn't get useful information
- Security team can debug issues from logs

**Robust Error Handling:**
- Checking every critical function prevents cascading failures
- Graceful recovery keeps system available
- Prevents attacker from triggering crashes

---

## SIN 18: POOR USABILITY OF SECURITY

### Understanding Security-Usability Trade-offs:

**Step 1: The Core Conflict**

Users want security but don't want extra hassle. Developers often choose the path of least resistance.

**Usability Problems:**
- Too little information (users don't understand)
- Too much information (users ignore it)
- Too many prompts (users click through)
- Inaccurate warnings (cry-wolf effect)
- Generic error codes (users confused)

**Real-World Example:**
```
Browser shows: "This site's security certificate has an error"
User thinks: "This is annoying, let me click proceed anyway"
User has no idea: The site could be compromised
Result: User visits attacker's site thinking it's legitimate
```

**Step 2: Common Usability Failures**

**Failure 1: Overwhelming Information**
```
Technical message shown to regular user:
"SSL certificate validation failed: hostname mismatch between 
certificate CN and request hostname. Certificate CN: *.example.com, 
Request hostname: sub.example.com. OpenSSL error: X509_V_ERR_HOSTNAME_MISMATCH"
```
User response: "That sounds technical. I'll click 'Proceed anyway.'"

**Better:** "This site's security certificate may be invalid. Are you sure you want to continue?"

**Failure 2: Ignored Warnings**
```
Users see browser HTTPS warnings so often (even for harmless reasons)
They become desensitized
When real security warning appears, they click through
(This is "cry-wolf" effect)
```

**Failure 3: Complex Password Requirements**
```
User forced to create: "Kx8#mP2$vQ4@lN!dE"
User writes it on sticky note: password on monitor
Better: Encourage passphrase "MyDogAte3Shoes" (harder to guess, easier to remember)
```

**Step 3: Security vs. Convenience Trade-off Examples**

**Two-Factor Authentication:**
- **Security benefit:** Even if password compromised, account safe
- **Usability cost:** Users must carry phone, wait for code, entry time

**Solution:** Remember trusted device option, allow multiple authentication methods

**Certificate Pinning:**
- **Security benefit:** Prevents man-in-the-middle even with compromised CA
- **Usability cost:** If key rotated, users get locked out

**Solution:** Implement backup pins, have key rotation process

**Strong Passwords:**
- **Security benefit:** Harder to brute-force
- **Usability cost:** Hard to remember, tedious to type

**Solution:** Use password managers, allow longer passphrases

**Step 4: Designing Secure and Usable Systems**

**Best Practices:**

1. **Understand Your Users**
   - Are they tech-savvy or non-technical?
   - What's their security knowledge level?
   - What are their constraints (time, devices)?

2. **Progressive Disclosure**
   - Show basic information first
   - Allow users to access advanced details if needed
   - Don't dump everything at once

3. **Make Security Actionable**
   ```
   BAD: "Security error"
   GOOD: "Your password was found in a data breach. 
          Please create a new password. Click here for instructions."
   ```

4. **Test with Real Users**
   - Don't assume you understand user behavior
   - Observe how non-technical people interact
   - Fix confusion points

5. **Secure by Default**
   - Don't make users choose security
   - Use secure settings automatically
   - Provide options for power users

6. **Clear and Honest Communication**
   ```
   When asking for sensitive access:
   "This app needs access to your photos to apply filters.
   We will never store or share your photos. You can revoke
   access anytime in Settings."
   ```

### WHY This Matters:

Security systems that are too complex get bypassed by users. The most secure system is one users actually use and understand.

---

## SIN 19: INFORMATION LEAKAGE

### Understanding Information Disclosure:

**Step 1: Types of Information Leakage**

**Intentional Leakage:**
- User controls it: choosing to share data
- Often regretted later (Facebook-Cambridge Analytica scandal)

**Accidental Leakage:**
- Developer didn't realize data was exposed
- Sensitive data in error messages, logs, filenames
- Data left in memory or backups

**Step 2: Side Channel Attacks**

**Timing Channels:**
```
// VULNERABLE
if (password == user_input) {
    login_success()
}

This uses string comparison which stops at first difference.
Timing measurements:
- "aaaaaa" vs correct password starting with "a" takes longer to fail
- Attacker can try single character at a time
- Success if timing changes significantly
```

**Storage Channels:**
```
Filename: "PlanToBuyExampleCorp.doc"
Even if encrypted, filename reveals intent
Attacker gains valuable intelligence

Encrypted file size: 50 MB
Attacker infers something large is stored there
```

**Metadata Leakage:**
```
Email metadata:
- Recipient list
- Timestamps
- Subject line (often unencrypted even in encrypted email)

File metadata:
- Author name
- Last modified date
- Software used
```

**Step 3: Real-World Information Leakage Examples**

**Facebook-Cambridge Analytica (2018):**
- Psychological profile data collected
- 87 million users affected
- Data used for political manipulation
- Users didn't know data was shared
- Regulatory fines: Billions in total

**GitHub Secret Leakage:**
```
Developer commits code with:
AWS_SECRET_KEY = "AKIAIOSFODNN7EXAMPLE"
GitHub public repository
Attacker scans public repos for credentials
Uses AWS key to launch attacks or incur charges
```

**Jenkins Log Leakage:**
```
Build logs publicly accessible
Contain API keys, database passwords
Environment variables exposed
Attacker gains infrastructure access
```

**Step 4: Common Information Leakage Points**

**Error Messages:**
```
WRONG: "Database connection failed: db.internal.example.com:5432"
RIGHT: "Connection error. Our team has been notified."
```

**Log Files:**
```
WRONG: password=user123 (credentials in logs)
RIGHT: password=*** (mask sensitive data)
```

**HTTP Headers:**
```
Server: Apache/2.4.41 (Ubuntu)  (reveals software version)
X-Powered-By: PHP/7.4.3          (reveals technology stack)
```

**Filenames:**
```
database_backup_2024.sql (reveals backup exists and date)
user_passwords_2024.csv (reveals sensitive file)
```

**HTML Comments:**
```html
<!-- DEBUG: User ID = 12345, Auth token = abc123xyz -->
(Credentials in comments visible in page source)
```

**Step 5: Protection Strategies**

**For Error Messages:**
```python
# WRONG
try:
    data = parse_json(user_input)
except json.JSONDecodeError as e:
    return {"error": str(e)}  # Exposes parsing details

# RIGHT
try:
    data = parse_json(user_input)
except json.JSONDecodeError:
    logger.error("JSON parsing error", exc_info=True)
    return {"error": "Invalid input format"}
```

**For Filenames:**
```python
# WRONG
sensitive_file = f"{username}_private_key_{year}.pem"

# RIGHT - Use generic identifier
filename_id = secrets.token_hex(16)
store_file_mapping(filename_id, actual_file)
# Serve: GET /file/{filename_id}
```

**For Metadata:**
```python
# Strip metadata from uploaded files
from PIL import Image

img = Image.open(uploaded_file)
data = list(img.getdata())
img_without_metadata = Image.new(img.mode, img.size)
img_without_metadata.putdata(data)
img_without_metadata.save(secure_location)
```

**For Timing Attacks:**
```python
# WRONG - Fast fail on first difference
if password == user_input:
    return True

# RIGHT - Constant time comparison
import hmac
if hmac.compare_digest(password, user_input):
    return True
# hmac.compare_digest takes same time regardless of where difference is
```

**For Access Control:**
```python
# Use proper ACLs on all files
import os
os.chmod('sensitive_file.txt', 0o600)  # Owner read/write only

# Or encrypt sensitive data
from cryptography.fernet import Fernet
cipher = Fernet(key)
encrypted_data = cipher.encrypt(sensitive_data)
```

### WHY This Works:

**Defense in Depth:**
Multiple layers prevent information leakage:
1. Encrypt sensitive data at rest and in transit
2. Use generic error messages
3. Strip metadata
4. Use proper access controls
5. Remove sensitive comments
6. Use constant-time comparisons

**Least Privilege:**
Only expose information that's necessary. If data isn't needed, don't store or transmit it.

---

## EXTRA SINS

### Insecure Direct Object References (IDOR)

**What It Is:**
Application allows users to access objects by guessing or incrementing IDs without proper authorization checks.

**Example:**
```
User A can access: /api/user/12/profile
User A changes URL: /api/user/13/profile
Gets User B's private profile without permission
```

**Real Example from Seattle University:**
```
http://dlweb.megamation.com/seattleuniversity/DLWEB_seattleu.php/?WO_NO=2022005512
Attacker increments WO_NO number
Accesses other users' work orders
```

**Prevention:**
- Check user permissions before returning object
- Don't expose internal IDs in URLs (use UUIDs with authorization)
- Log access attempts

### Cross-Site Request Forgery (CSRF)

**What It Is:**
Attacker tricks authenticated user into performing unintended actions on a website.

**Attack Sequence:**
```
1. User logs into: www.bank.example.com
2. User's browser still has session cookie
3. User visits attacker's site: evil.example.com
4. Evil site contains: <img src="https://bank.example.com/transfer?to=attacker&amount=1000">
5. Browser automatically includes session cookie in request
6. Bank transfers $1000 to attacker without user knowledge
```

**Prevention:**
- Use CSRF tokens (unpredictable, per-session values)
- Check Origin/Referer headers
- Use SameSite cookie attribute
- Require user confirmation for sensitive operations

### Unvalidated Redirects and Forwards

**What It Is:**
Application redirects users to untrusted URL based on user input.

**Attack:**
```java
// VULNERABLE CODE
response.sendRedirect(request.getParameter("url"));

// ATTACK
http://example.com/redirect?url=http://malicious.com/phishing
// User sees trusted domain, clicks link
// Gets redirected to attacker's phishing site
```

**Prevention:**
- Whitelist allowed URLs
- Validate URL belongs to your domain
- Use relative URLs instead of accepting URLs as parameters

### XML External Entity (XXE) Attack

**What It Is:**
Attacker injects malicious XML that references external entities, causing information disclosure or DoS.

**Example:**
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<foo>&xxe;</foo>
```

**Prevention:**
- Disable XML external entity processing
- Use safe XML parsers
- Validate XML schema

---

## INSECURE DESERIALIZATION

### What It Is:

Deserialization is converting data (from network, file, etc.) back into a usable object. If the data is untrusted and the deserializer is insecure, attackers can execute arbitrary code.

**Key Difference:**
- **Serialization:** Object → Data format (JSON, pickle, binary)
- **Deserialization:** Data format → Object

### Step 1: Understanding the Vulnerability

**Python Pickle Example (VERY DANGEROUS):**
```python
# VULNERABLE CODE
import pickle
user_data = pickle.loads(untrusted_input)
```

**Why Pickle is Dangerous:**
Pickle can serialize/deserialize Python objects including function calls. An attacker can craft a pickle that executes code when deserialized.

**Malicious Pickle Example:**
```python
import pickle
import os

# Attacker creates this payload
class Exploit:
    def __reduce__(self):
        return (os.system, ('rm -rf /',))

payload = pickle.dumps(Exploit())
# When victim deserializes this payload, os.system('rm -rf /') executes!
```

### Step 2: Other Vulnerable Serialization Formats

**Java Serialization:**
- Same problem as Python pickle
- Can construct gadget chains
- Remote Code Execution possible

**YAML (Insecure):**
```yaml
# VULNERABLE YAML
!!python/object/apply:os.system
args: ['cat /etc/passwd']
```

### Step 3: Safe vs. Unsafe Deserialization

**UNSAFE - Never do this:**
```python
import pickle
data = pickle.loads(user_provided_data)  # Arbitrary code execution!

import yaml
data = yaml.load(user_provided_data)     # Can execute code!
```

**SAFE - Do this instead:**
```python
import json
data = json.loads(user_provided_data)  # JSON only supports basic types

import yaml
data = yaml.safe_load(user_provided_data)  # Limited to safe types
```

### Step 4: Why JSON is Safer

**JSON Supported Types:**
- Strings, numbers, booleans, null
- Arrays and objects
- NO function calls, NO code execution possible

**Attack Surface:**
```python
# Even with malicious JSON, worst case is unexpected data types
malicious_json = '{"exec": "rm -rf /", "code": "system()"}'
data = json.loads(malicious_json)
# Result: Just a dictionary with string values
# No code executed
```

### Step 5: Protection Strategies

**Strategy 1: Use Safe Formats**
```python
# GOOD - JSON has no code execution capability
import json
data = json.loads(untrusted_input)

# GOOD - YAML with safe_load
import yaml
data = yaml.safe_load(untrusted_input)

# BAD - Pickle can execute code
import pickle
data = pickle.loads(untrusted_input)  # DON'T USE!
```

**Strategy 2: Input Validation**
```python
import json

# Even safer - validate schema
import jsonschema

schema = {
    "type": "object",
    "properties": {
        "username": {"type": "string"},
        "age": {"type": "integer", "minimum": 0, "maximum": 150}
    },
    "required": ["username"],
    "additionalProperties": False  # Reject unknown fields
}

data = json.loads(untrusted_input)
jsonschema.validate(data, schema)
```

**Strategy 3: Run Deserialization in Sandbox**
```python
# If you absolutely must deserialize untrusted data
import subprocess

# Run in isolated process with restricted permissions
result = subprocess.run(
    ['unpacker', '--safe'],
    input=untrusted_data,
    capture_output=True,
    timeout=5
)
```

### WHY This Works:

**Type System Limitations:**
JSON inherently can't represent function calls or code. No code execution path exists, no matter how malicious the input.

**Explicit Validation:**
By validating against a schema, you reject any unexpected structure before processing.

**Sandboxing:**
Even if deserialization fails, damage is isolated to sandbox with no access to real system.

---

## SERVER-SIDE REQUEST FORGERY (SSRF)

### What It Is:

Attacker makes the server fetch a URL of attacker's choosing, bypassing network restrictions. The server accesses internal resources the attacker cannot directly reach.

### Step 1: Understanding the Attack

**Vulnerable Code:**
```python
@app.route('/download')
def download():
    url = request.args.get('url')  # User provides URL
    response = requests.get(url)   # Server fetches it
    return response.content
```

**Attack Scenario:**
```
Normal use: /download?url=https://example.com/document.pdf
Attacker: /download?url=http://internal.example.com/admin
Result: Server (trusted by firewall) fetches internal admin page
Attacker sees: Admin interface (confidential)
```

**Why It Works:**
- Server is inside corporate network/firewall
- Attacker cannot access internal.example.com directly
- But server can (it's trusted internally)
- Attacker uses server as proxy to access restricted resources

### Step 2: Attack Variations

**Accessing Internal Metadata Services:**
```
AWS EC2 metadata: http://169.254.169.254/latest/meta-data/
Attacker fetches: /download?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
Result: Gets AWS access keys from metadata service
```

**Accessing Internal Services:**
```
Redis server: /download?url=http://localhost:6379/
Database: /download?url=http://db.internal.company.com:5432/
Admin panel: /download?url=http://admin.internal.company.com/
```

**Port Scanning:**
```
Attacker tries: /download?url=http://192.168.1.1:22
If response is fast: Port is open
If timeout: Port is closed
Attacker maps internal network
```

### Step 3: Protection Strategies

**Strategy 1: Whitelist Allowed Domains**
```python
ALLOWED_DOMAINS = [
    'example.com',
    'cdn.example.com',
    'api.example.com'
]

@app.route('/download')
def download():
    url = request.args.get('url')
    
    # Parse and validate URL
    from urllib.parse import urlparse
    parsed = urlparse(url)
    
    if parsed.hostname not in ALLOWED_DOMAINS:
        return "Invalid domain", 400
    
    response = requests.get(url)
    return response.content
```

**Strategy 2: Block Internal IP Ranges**
```python
import ipaddress

BLOCKED_RANGES = [
    ipaddress.ip_network('10.0.0.0/8'),      # Private network
    ipaddress.ip_network('172.16.0.0/12'),   # Private network
    ipaddress.ip_network('192.168.0.0/16'),  # Private network
    ipaddress.ip_network('127.0.0.0/8'),     # Localhost
    ipaddress.ip_network('169.254.0.0/16'),  # Link-local
]

@app.route('/download')
def download():
    url = request.args.get('url')
    parsed = urlparse(url)
    
    try:
        ip = ipaddress.ip_address(parsed.hostname)
        for blocked_range in BLOCKED_RANGES:
            if ip in blocked_range:
                return "Invalid target", 400
    except ValueError:
        # Not an IP, do DNS check instead
        pass
    
    response = requests.get(url, timeout=5)
    return response.content
```

**Strategy 3: Use Allowlist Only**
```python
# Most restrictive - only allow specific pre-approved URLs
ALLOWED_URLS = [
    'https://cdn.example.com/file1.pdf',
    'https://cdn.example.com/file2.pdf',
    'https://api.example.com/v1/data'
]

@app.route('/download')
def download():
    url = request.args.get('url')
    
    if url not in ALLOWED_URLS:
        return "URL not allowed", 400
    
    response = requests.get(url)
    return response.content
```

**Strategy 4: DNS Rebinding Protection**
```python
# Check DNS result is not internal IP
import socket

def is_safe_to_fetch(url):
    parsed = urlparse(url)
    
    try:
        # Resolve hostname to IP
        ip = socket.gethostbyname(parsed.hostname)
        
        # Check if internal IP
        ip_obj = ipaddress.ip_address(ip)
        for blocked_range in BLOCKED_RANGES:
            if ip_obj in blocked_range:
                return False
        return True
    except:
        return False

@app.route('/download')
def download():
    url = request.args.get('url')
    
    if not is_safe_to_fetch(url):
        return "Invalid target", 400
    
    response = requests.get(url)
    return response.content
```

### WHY This Works:

**Whitelist Enforcement:**
By explicitly allowing only known-good URLs/domains, you prevent access to unknown internal resources.

**IP Range Blocking:**
Internal networks use reserved IP ranges. Blocking these prevents access to internal services.

**DNS Verification:**
Checking that resolved IP is not internal prevents DNS rebinding attacks where hostname resolves to different IP later.

---

## KEY TAKEAWAYS FOR EXAM

### The "7 Categories" Framework

**Category 1: Input Validation (Sins 1-6)**
- Buffer Overflows
- Command Injection
- SQL Injection
- Cross-Site Scripting
- Format Strings
- Integer Overflow

→ **Lesson:** Always validate and sanitize user input

**Category 2: API Abuse (Sin 7)**
- Trusting network information

→ **Lesson:** Don't trust DNS or network data

**Category 3: Security Features (Sins 8-14)**
- Failing to protect network traffic
- Failing to store/protect data
- Weak random numbers
- Improper file access
- Improper SSL/TLS use
- Weak passwords
- Unauthenticated key exchange

→ **Lesson:** Implement proper cryptography and authentication

**Category 4: Time and State (Sins 15-16)**
- Race conditions
- Magic URLs/hidden fields

→ **Lesson:** Make operations atomic, don't trust client

**Category 5: Error Handling (Sin 17)**
- Failure to handle errors

→ **Lesson:** Check errors, log securely, return generic messages

**Category 6: Code Quality (Sin 18)**
- Poor usability

→ **Lesson:** Security should be usable

**Category 7: Encapsulation (Sin 19)**
- Information leakage

→ **Lesson:** Only expose necessary information