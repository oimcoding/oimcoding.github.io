# Essay Topic 3: ESP Verification Process

## Reference: L7_NetworkSecurity.pdf (pages 19-23)

---

## Overview of IPsec and ESP

**IPsec (Internet Protocol Security):**
- Provides security at the network layer (Layer 3)
- Protects all traffic between hosts/networks
- Two main protocols: AH (Authentication Header) and ESP

**ESP (Encapsulating Security Payload):**
- Provides confidentiality (encryption)
- Provides integrity (authentication)
- Provides anti-replay protection
- Can work in transport mode or tunnel mode

---

## ESP Datagram Structure

```
+------------------+
| New IP Header    | (in tunnel mode)
+------------------+
| ESP Header       | ← SPI (Security Parameter Index)
|   - SPI          |   Seq# (Sequence Number)
|   - Seq Number   |
+------------------+
|                  |
| Original IP      | ← ENCRYPTED
| Datagram         |
| (payload)        |
|                  |
+------------------+
| ESP Trailer      | ← ENCRYPTED
|   - Padding      |   (includes padding, pad length, next header)
|   - Pad Length   |
|   - Next Header  |
+------------------+
| ESP Auth Data    | ← Authentication/Integrity Check
|   (ICV)          |   (computed over ESP header through trailer)
+------------------+
```

---

## Step-by-Step ESP Verification Process

### Step 1: Extract Security Association Information

**Action:** Client receives ESP packet and extracts SPI from ESP header

**Purpose:**
- SPI (Security Parameter Index) identifies the security association (SA)
- SA contains shared keys, algorithms, and parameters
- Client looks up SA in its Security Association Database (SAD)
- SA includes:
  - Encryption algorithm and key (e.g., AES key)
  - Authentication algorithm and key (e.g., HMAC-SHA256 key)
  - Sequence number counter
  - Anti-replay window settings

**Why Important:** Cannot decrypt or verify without the correct SA

---

### Step 2: Check Sequence Number (Anti-Replay)

**Action:** Extract Sequence Number from ESP header and verify it

**Verification Process:**
- ESP header contains 32-bit sequence number
- Client maintains a sliding receive window (e.g., 64 packets)
- Check if sequence number falls within acceptable window
- Check if this sequence number was already received (replay detection)

**Anti-Replay Window:**
- Tracks recently received sequence numbers
- Packets with seq# too old are rejected
- Packets with duplicate seq# are rejected (replay attack)
- Window slides forward as new packets arrive

**Result:**
- **Accept:** Sequence number is new and within window
- **Reject:** Sequence number is duplicate or outside window (potential replay attack)

**Why Important:** Prevents attacker from capturing and replaying old packets

---

### Step 3: Verify Authentication/Integrity (ICV)

**Action:** Verify the ESP Authentication Data (ICV - Integrity Check Value)

**Verification Process:**

1. **Extract ICV from packet**
   - ICV is at the end of ESP packet (ESP Auth field)
   - Length depends on authentication algorithm (e.g., 12-16 bytes)

2. **Compute expected ICV**
   - Use authentication key from SA
   - Compute HMAC over:
     - ESP Header (including SPI and Seq#)
     - Encrypted payload
     - ESP Trailer
   - Do NOT include the ICV itself in computation
   - Algorithm specified in SA (e.g., HMAC-SHA256, HMAC-SHA1)

3. **Compare ICVs**
   - Compare received ICV with computed ICV
   - Must match exactly (constant-time comparison to prevent timing attacks)

**Result:**
- **Match:** Packet is authentic and unmodified
- **No Match:** Packet has been tampered with or corrupted (REJECT)

**Why Important:**
- Ensures packet came from legitimate sender (has shared key)
- Ensures packet was not modified in transit
- Provides integrity and authentication

---

### Step 4: Decrypt the Payload

**Action:** After authentication passes, decrypt the encrypted portion

**Decryption Process:**

1. **Extract encrypted data**
   - Encrypted portion: Original IP datagram + ESP Trailer
   - Does NOT include ESP Header or ESP Auth Data

2. **Get decryption key and algorithm**
   - Retrieved from SA (identified by SPI)
   - Algorithm examples: AES-CBC, AES-GCM, 3DES

3. **Decrypt the payload**
   - Use encryption key from SA
   - Apply decryption algorithm
   - Some modes (like GCM) provide both encryption and authentication

4. **Extract original datagram**
   - Decrypt reveals: Original IP packet + ESP Trailer
   - ESP Trailer contains padding, pad length, and next header field

**Why Decrypt AFTER Authentication:**
- Prevents decryption oracle attacks
- Don't waste CPU on decrypting invalid packets
- Authentication is faster than decryption

---

### Step 5: Remove Padding and Extract Original Packet

**Action:** Parse ESP Trailer and extract original IP datagram

**Process:**

1. **Read ESP Trailer fields** (at end of decrypted data)
   - Pad Length: indicates how many padding bytes
   - Next Header: indicates protocol of decrypted packet (e.g., IP, TCP)

2. **Remove padding**
   - Padding may be added for block cipher alignment
   - Use Pad Length to determine how many bytes to remove

3. **Extract original datagram**
   - Remove ESP overhead (header, trailer, auth data)
   - Retrieve original IP packet or transport-layer segment
   - Use Next Header field to identify protocol

4. **Process decrypted packet**
   - Pass to appropriate protocol handler (IP, TCP, UDP, etc.)
   - Continue normal packet processing

---

### Step 6: Update Anti-Replay Window

**Action:** After successful verification, update the receive window

**Process:**
- Mark this sequence number as received
- Advance the sliding window if needed
- Update highest sequence number received
- This prevents future replays of this packet

---

## Complete Verification Flow Diagram

```
ESP Packet Received
        ↓
[1] Extract SPI → Look up SA (keys, algorithms)
        ↓
[2] Check Sequence Number → Anti-replay window check
        ↓
    Valid seq#?
        ↓ YES
[3] Verify ICV (ESP Auth Data)
    - Compute HMAC over ESP header + encrypted payload + trailer
    - Compare with received ICV
        ↓
    ICV Match?
        ↓ YES
[4] Decrypt Payload
    - Use encryption key from SA
    - Decrypt original datagram + ESP trailer
        ↓
[5] Remove Padding
    - Parse ESP trailer
    - Extract original IP packet
        ↓
[6] Update Anti-Replay Window
    - Mark sequence number as received
        ↓
Process Original Packet
```

---

## Security Properties Enforced

1. **Confidentiality (Encryption)**
   - Original IP datagram is encrypted
   - Eavesdropper cannot read payload
   - Only entities with shared key can decrypt

2. **Integrity (Authentication)**
   - ICV ensures packet not modified
   - Computed using HMAC with shared key
   - Any modification detected in Step 3

3. **Authentication**
   - Only sender with shared key can create valid ICV
   - Verifies packet came from legitimate peer

4. **Anti-Replay**
   - Sequence numbers prevent replay attacks
   - Sliding window tracks received packets
   - Duplicate/old packets rejected in Step 2

---

## Common ESP Modes

**Transport Mode:**
- Encrypts only the payload (transport layer and above)
- Original IP header remains visible
- Used for end-to-end protection between hosts

**Tunnel Mode:**
- Encrypts entire original IP datagram
- New IP header added (with gateway addresses)
- Used for VPN gateway-to-gateway tunnels

---

## Key Points for Verification

**Order Matters:**
1. SPI lookup FIRST (need SA to do anything)
2. Sequence number check BEFORE authentication (fast check)
3. Authentication BEFORE decryption (security best practice)
4. Decryption AFTER authentication passes
5. Remove padding AFTER decryption
6. Update window AFTER all checks pass

**Failure Handling:**
- If any step fails → DROP packet silently
- Do not send error message (prevents reconnaissance)
- Log the failure for security monitoring

---

## Essay Answer Structure

**For "In ESP, how does client verify (step by step)":**

**Step 1: Look up Security Association**
- Extract SPI from ESP header
- Retrieve SA from SAD (contains keys and algorithms)

**Step 2: Verify Sequence Number**
- Extract sequence number from ESP header
- Check against anti-replay sliding window
- Reject if duplicate or outside window

**Step 3: Verify Authentication (ICV)**
- Compute HMAC over ESP header + encrypted payload + trailer
- Use authentication key from SA
- Compare computed ICV with received ICV in ESP Auth field
- Reject if mismatch (tampering detected)

**Step 4: Decrypt Payload**
- Use encryption key from SA
- Decrypt original datagram + ESP trailer
- Apply algorithm specified in SA (e.g., AES-CBC)

**Step 5: Remove Padding and Extract Original Packet**
- Parse ESP trailer for pad length and next header
- Remove padding bytes
- Extract original IP datagram

**Step 6: Update Anti-Replay Window**
- Mark sequence number as received
- Advance sliding window

**Key Security Properties:**
- Confidentiality through encryption
- Integrity through ICV/HMAC
- Authentication through shared keys
- Anti-replay through sequence numbers

**Critical Point:** Authentication (ICV verification) happens BEFORE decryption to prevent attacks and improve performance.
