# Key Concepts for Multiple Choice Questions

## Quick reference for all important concepts from course materials

---

## Network Basics (L6)

### Internet Architecture (L6 p.3-5)

**Components:**
- **Hosts/End Systems:** Devices running network applications at Internet's edge
- **Packet Switches:** Routers and switches that forward packets
- **Communication Links:** Fiber, copper, radio, satellite (transmission rate = bandwidth)
- **Networks:** Collection of devices, routers, links managed by organization
- **Protocols:** Rules defining format, order of messages and actions (e.g., HTTP, TCP, IP, WiFi)

**Internet Standards:**
- RFC (Request for Comments): Internet standard documents
- IETF (Internet Engineering Task Force): Standards organization

### Protocol Layers (L6 p.8)

**Five-Layer Internet Protocol Stack:**
1. **Application Layer:** Supporting network applications
   - Protocols: HTTP, IMAP, SMTP, DNS
   - Message

2. **Transport Layer:** Process-to-process data transfer
   - Protocols: TCP, UDP
   - Segment

3. **Network Layer:** Routing datagrams from source to destination
   - Protocols: IP, routing protocols
   - Datagram

4. **Link Layer:** Data transfer between neighboring network elements
   - Protocols: Ethernet, 802.11 (WiFi), PPP
   - Frame

5. **Physical Layer:** Bits "on the wire"

**Encapsulation:** (L6 p.9-13)
- Application data (M) → Transport adds header (H_t | M) → Network adds header (H_n | H_t | M) → Link adds header (H_l | H_n | H_t | M)
- Each layer adds its own header to encapsulate upper layer's data
- At destination, headers stripped off layer by layer

---

### Application Layer: HTTP (L6 p.14-22)

**HTTP Basics:** (L6 p.16)
- Hypertext Transfer Protocol
- Web's application-layer protocol
- Client/server model: browser requests, server responds
- Uses TCP on port 80

**HTTP Request Message:** (L6 p.17, 19)
- ASCII human-readable format
- Request line: method (GET, POST, HEAD) + URL + version
- Header lines: Host, User-Agent, Accept, Accept-Language, Accept-Encoding, Connection
- Blank line (CRLF) indicates end of headers
- Optional entity body (for POST)

**HTTP Response Message:** (L6 p.18, 20)
- Status line: protocol + status code + status phrase
- Header lines: Date, Server, Last-Modified, Content-Length, Content-Type
- Blank line
- Entity body (requested data)

**HTTP Response Splitting Attack:** (L6 p.21-22)
- Attacker passes malicious data with CRLF characters (\r\n or %0d %0a)
- If application includes user input in HTTP response header, attacker can inject additional headers
- Example: User input "name\r\nContent-Security-Policy: default-src 'none'\r\n" creates fake header
- Can lead to XSS, cache poisoning, credential theft

---

### Transport Layer (L6 p.36-48)

**Sockets:** (L6 p.32-33)
- Door between application process and transport layer
- Process sends/receives messages through socket
- **Socket = (Source IP, Source Port, Destination IP, Destination Port, Protocol)**
- Each side has a socket

**Addressing Processes:** (L6 p.31)
- **IP address:** 32-bit identifier for host device
- **Port number:** Identifies process on host
  - HTTP server: port 80
  - Mail server: port 25
- Full identifier: IP address + port number

**TCP Overview:** (L6 p.37-42)
- **Point-to-point:** One sender, one receiver
- **Reliable, in-order byte stream:** No message boundaries
- **Full duplex:** Bidirectional data flow
- **Connection-oriented:** Handshaking initializes state before data exchange
- **Flow controlled:** Sender won't overwhelm receiver
- **Pipelining:** TCP congestion/flow control set window size
- **MSS:** Maximum Segment Size

**TCP Segment Structure:** (L6 p.38)
- Source/destination port (16 bits each)
- Sequence number (32 bits): Counts bytes into bytestream
- Acknowledgment number (32 bits): Next expected byte; ACK bit indicates this is ACK
- Header length, flags (SYN, FIN, RST, PSH, URG, ACK, C, E)
- Receive window: Flow control, bytes receiver willing to accept
- Checksum, urgent data pointer
- Options (variable length)
- Application data (variable length)

**TCP Three-Way Handshake:** (L6 p.39-42)
1. **Client sends SYN:**
   - SYN bit = 1
   - Client's initial sequence number (ISN)
   - Client state: SYN_SENT

2. **Server sends SYN-ACK:**
   - SYN bit = 1, ACK bit = 1
   - Server's ISN
   - ACK number = client's ISN + 1
   - Server state: SYN_RCVD

3. **Client sends ACK:**
   - ACK bit = 1
   - ACK number = server's ISN + 1
   - May contain client-to-server data
   - Both states: ESTABLISHED

**SYN Flood Attack:** (L6 p.43)
- Attacker sends succession of SYN requests with spoofed source IPs
- Server allocates resources for half-open connections
- Server's connection table fills up
- System becomes unresponsive to legitimate traffic
- Defense: SYN cookies

**UDP (User Datagram Protocol):** (L6 p.44-47)
- **Connectionless:** No handshaking
- **Unreliable:** Segments may be lost or delivered out-of-order
- **No frills:** Bare bones transport protocol
- **Best effort service**

**Why UDP?**
- No connection establishment (no RTT delay)
- Simple: no connection state
- Small header size (8 bytes vs 20+ for TCP)
- No congestion control (can blast away as fast as desired)

**UDP Segment Header:** (L6 p.47)
- Source port, destination port (16 bits each)
- Length: bytes of UDP segment including header
- Checksum
- Application data (payload)

**UDP Use Cases:**
- Streaming multimedia (loss tolerant, rate sensitive)
- DNS
- SNMP
- HTTP/3

---

### Network Layer (L6 p.50-57)

**Network Layer Functions:** (L6 p.50)
- **Forwarding:** Move packets from router's input to output
- **Routing:** Determine path from source to destination
  - Implemented by routing protocols (OSPF, BGP) or SDN controller

**IP Datagram Format:** (L6 p.51)
- Version (4 bits): IP protocol version
- Header length (4 bits): In 32-bit words
- Type of service (8 bits): Diffserv (0:5), ECN (6:7)
- Total length (16 bits): Total datagram length in bytes (max 64K, typically ≤1500)
- 16-bit identifier, flags, fragment offset: Fragmentation/reassembly
- TTL (8 bits): Time to live, decremented at each router
- Upper layer protocol (8 bits): E.g., TCP or UDP
- Header checksum (16 bits)
- Source IP address (32 bits)
- Destination IP address (32 bits)
- Options (variable, if any)
- Payload data (typically TCP or UDP segment)

**DHCP (Dynamic Host Configuration Protocol):** (L6 p.52)
- **Goal:** Host dynamically obtains IP address from network server when joining network
- Can renew lease on address
- Allows reuse of addresses (only hold while connected)
- Supports mobile users

**DHCP Process:**
1. Host broadcasts DHCP discover message [optional]
2. DHCP server responds with DHCP offer message [optional]
3. Host requests IP address: DHCP request message
4. DHCP server sends address: DHCP ack message

**DNS (Domain Name System):** (L6 p.53-55)
- **Services:**
  1. Hostname-to-IP-address translation
  2. Host aliasing (canonical vs alias names)
  3. Mail server aliasing
  4. Load distribution (multiple IPs for one name)

**DNS Message Format:** (L6 p.54)
- Identification (16 bits): Query ID, reply uses same ID
- Flags: Query/reply, recursion desired, recursion available, reply authoritative
- Number of questions, answer RRs, authority RRs, additional RRs
- Questions section (variable # of questions)
- Answers section (variable # of RRs)
- Authority section (variable # of RRs)
- Additional info section (variable # of RRs)

**DNS Iterative Query:** (L6 p.55)
- Client asks local DNS server
- Local DNS asks root DNS server → returns TLD server
- Local DNS asks TLD server → returns authoritative server
- Local DNS asks authoritative server → returns IP address
- Contacted server replies with name of server to contact ("I don't know, but ask this server")

---

### Link Layer (L6 p.58-70)

**Ethernet Frame Structure:** (L6 p.58)
- Preamble: Synchronization
- Destination MAC address (6 bytes)
- Source MAC address (6 bytes)
- Type (2 bytes): Upper layer protocol (e.g., IP)
- Data/Payload: Encapsulated datagram (46-1500 bytes)
- CRC (4 bytes): Cyclic redundancy check for error detection

**MAC Addresses:** (L6 p.59)
- 48-bit MAC address burned into NIC ROM
- Example: 1A-2F-BB-76-09-AD (hexadecimal notation)
- **Function:** Get frame from one interface to another physically-connected interface (same subnet)
- Different from IP address (network layer)

**IP vs MAC Addresses:**
- **IP address (32-bit):** Network-layer address, used for Layer 3 forwarding
- **MAC address (48-bit):** Link-layer address, used locally on same subnet

**ARP (Address Resolution Protocol):** (L6 p.60-63)
- **Question:** How to determine interface's MAC address, knowing its IP address?
- **ARP Table:** Each IP node has table with IP/MAC address mappings
  - Format: <IP address; MAC address; TTL>
  - TTL (Time To Live): Typically 20 minutes

**ARP Protocol Process:**
1. **A wants to send to B, but doesn't know B's MAC:**
   - A broadcasts ARP query with B's IP address
   - Destination MAC = FF-FF-FF-FF-FF-FF (broadcast)
   - All nodes on LAN receive ARP query

2. **B receives ARP query:**
   - B replies to A with ARP response
   - Sends B's MAC address to A (unicast, not broadcast)

3. **A receives B's reply:**
   - A caches B's IP-to-MAC mapping in ARP table
   - Entry expires after TTL

**ARP Attacks:** (L6 p.64)
- **ARP Spoofing/Poisoning:**
  - Hacker sends fake ARP packets
  - Links attacker's MAC with legitimate victim's IP
  - (Attacker's MAC, Victim's IP)
  - Enables man-in-the-middle attacks

- **ARP Flooding:**
  - Attacker sends large number of ARP request packets
  - Overwhelms network switches
  - Can fill ARP tables, cause DoS

**Example: Sending Packet from A to B via Router R:** (L6 p.65-70)
- **A creates IP datagram:** Source = A's IP, Destination = B's IP
- **A creates link-layer frame:** Destination MAC = R's MAC (next hop)
- **Frame sent to R:** R receives, extracts IP datagram
- **R determines outgoing interface:** Looks up B in forwarding table
- **R creates new frame:** Source MAC = R's outgoing interface MAC, Destination MAC = B's MAC
- **Frame sent to B:** B receives, extracts IP datagram, passes to upper layers

**Key Point:**
- IP addresses (source/destination) remain unchanged end-to-end
- MAC addresses change at each hop (link-by-link)

---

### Key Network Concepts

**Protocol Definition:** (L6 p.6)
- Define format and order of messages sent/received among network entities
- Define actions taken on message transmission/receipt

**Four Types of Harm on Network Journey:** (L6 p.74)
1. **Interception:** Unauthorized viewing (eavesdropping)
2. **Modification:** Unauthorized change (tampering)
3. **Fabrication:** Unauthorized creation (spoofing)
4. **Interruption:** Preventing authorized access (DoS)

---

## Network Security Basics (L7)

### Denial of Service (DoS) Attacks

**DoS Overview** (L7 p.3)
- Attack on availability of service
- Categories: Network bandwidth, System resources, Application resources

**Flooding Attacks** (L7 p.4)
- Send massive volume of traffic to overwhelm target
- Target runs out of bandwidth or processing capacity
- Examples: UDP flood, ICMP flood, HTTP flood, TCP SYN flood

**HTTP-Based Attacks** (L7 p.5)
- HTTP flood: Bombards web servers with HTTP requests
- Slowloris: Monopolizes connections with incomplete HTTP requests
- Spidering: Bots recursively follow all links on website

**Source Address Spoofing** (L7 p.5)
- Use forged source addresses via raw socket interface
- Makes attacking systems harder to identify

**SYN Flooding** (L7 p.6)
- Exploits TCP three-way handshake
- Attacker sends many SYN packets with spoofed source IPs
- Server allocates resources for half-open connections
- Server's connection table fills up, legitimate connections refused
- Attack on system resources, specifically network handling code
- Defense: SYN cookies

**Reflection Attacks** (L7 p.7)
- Attacker spoofs victim's IP address as source
- Sends requests to intermediary servers
- Intermediary servers send responses to victim (who didn't request them)
- Victim overwhelmed by responses from many servers
- Goal: Generate enough volume to flood link without alerting intermediary

**Amplification Attacks** (L7 p.8)
- Variant of reflection attacks
- Small request generates multiple large responses
- Amplification factor: response size / request size
- Examples:
  - Smurf attack: ICMP to broadcast address
  - DNS amplification (factor of 50x+), NTP amplification
  - Memcached (50,000x amplification)
- Combines spoofing + reflection + amplification

**DDoS (Distributed DoS)** (L7 p.9-10)
- DoS attack from multiple sources simultaneously
- Uses zombies/botnets (compromised computers)
- Much harder to defend against
- Cannot simply block one IP address
- Famous attacks: Google 2.54 Tbps (2017), AWS 2.3 Tbps (2020), GitHub 1.3 Tbps (2018)

---

## Transport Layer Security (TLS) (L7 p.11-18)

**TLS Purpose** (L7 p.12)
- Confidentiality: encryption prevents eavesdropping
- Integrity: detect tampering with data
- Authentication: verify identity of server (and optionally client)

**TLS Handshake Steps** (L7 p.13-14)
1. Client Hello: cipher suites, random nonce (R_C)
2. Server Hello: selected cipher, random nonce (R_S), certificate
3. Client verifies certificate
4. Key exchange: client sends encrypted Pre-Master Secret (PMS)
5. Both derive session keys from PMS, R_C, R_S
6. Encrypted communication begins

**Cipher Suite** (L7 p.16)
- Specifies algorithms for TLS connection
- TLS 1.3 has only 5 cipher suites (vs 37 in TLS 1.2)
- Examples: TLS_AES_256_GCM_SHA384, TLS_CHACHA20_POLY1305_SHA256
- Format includes: encryption algorithm, MAC algorithm

**Certificate Authority (CA)** (L7 p.14)
- Trusted third party that signs certificates
- CA's public key is pre-installed in browsers/OS
- Certificate chain: server cert → intermediate CA → root CA
- Client verifies certificate signature using CA's public key

**Session Keys** (L7 p.14)
- Symmetric keys derived from PMS using Key Derivation Function (KDF)
- Four keys: Kc (client encrypt), Mc (client MAC), Ks (server encrypt), Ms (server MAC)
- Used for encrypting actual data (faster than public key crypto)
- Separate keys for client-to-server and server-to-client

**TLS Records** (L7 p.15)
- Data broken into series of records (not stream encryption)
- Each record has: length, data, MAC
- Encrypted using symmetric key (Kc or Ks)
- MAC includes TLS sequence number to prevent reordering/replay
- Allows receiver to act on each record as it arrives

**TLS Connection Close** (L7 p.16)
- Record types: type 0 for data, type 1 for close
- Prevents truncation attack (attacker forging TCP close)
- MAC computed over: length, type, data, sequence number

**TLS 1.3 Handshake** (L7 p.17)
- Reduced to 1 RTT (vs 2 RTT in older versions)
- Client guesses key agreement protocol in first message
- Client can send data immediately after server hello
- Improves performance for HTTPS connections

**TLS vs SSL**
- SSL (Secure Sockets Layer) is deprecated (2015)
- TLS (Transport Layer Security) is modern version
- TLS 1.3 (2018) is current standard

---

## IPsec and ESP (L7 p.19-23)

**IPsec** (L7 p.20)
- Security at network layer (Layer 3)
- Protects all traffic between hosts/networks (including BGP, DNS)
- Two protocols: AH (Authentication Header) and ESP (Encapsulating Security Payload)

**ESP (Encapsulating Security Payload)** (L7 p.20)
- Provides encryption (confidentiality)
- Provides authentication (integrity)
- Provides anti-replay protection
- More widely used than AH

**Security Association (SA)** (L7 p.21)
- One-way relationship between sender and receiver (directional)
- Contains: encryption key, authentication key, algorithms, SPI, sequence counter
- SAD (Security Association Database) stores all SAs
- Identified by: SPI + destination IP + protocol
- Before sending data, SA must be established

**SPI (Security Parameter Index)** (L7 p.21)
- 32-bit identifier in ESP header
- Used to look up the correct SA in SAD
- Different SPI for each direction (bidirectional = 2 SAs)
- Tells receiving entity which SA to use

**Sequence Number** (L7 p.21-22)
- 32-bit counter in ESP header
- Sender initializes to 0, increments for each packet
- Prevents replay attacks
- Receiver uses sliding window to check for duplicates
- Packets outside window or duplicates are rejected

**ICV (Integrity Check Value)** (L7 p.21)
- Authentication data at end of ESP packet (ESP Auth field)
- HMAC/MAC computed over ESP header + encrypted payload + ESP trailer
- Verifies authenticity and integrity
- Checked BEFORE decryption (prevents decryption oracle attacks)

**ESP Structure** (L7 p.21)
- ESP Header: SPI, Sequence Number
- Encrypted: Original IP datagram + ESP Trailer
- ESP Trailer: Padding, Pad Length, Next Header
- ESP Auth: ICV/MAC

**Transport Mode vs Tunnel Mode** (L7 p.20)
- Transport: only payload encrypted, original IP header visible
- Tunnel: entire datagram encrypted, new IP header added
- Transport: host-to-host connections
- Tunnel: gateway-to-gateway VPNs

**IKE (Internet Key Exchange)** (L7 p.23)
- Automates SA establishment (vs manual keying)
- Practical for VPNs with hundreds of endpoints
- Negotiates encryption algorithms, keys, and SA parameters

---

## Onion Routing / TOR (L7 p.24-27)

**Why Onion Routing?** (L7 p.25)
- TLS hides payload but headers (IP addresses) are exposed
- IPsec hides headers but VPN provider can monitor traffic
- Onion Routing provides anonymity (hides who we are)

**Onion Routing Concept** (L7 p.25-26)
- Encrypts data multiple times (layered like an onion)
- Routes through several network nodes (e.g., X → Y → Z)
- No single node sees complete origin, destination, AND contents
- Difficult to trace data back to original source

**Onion Routing Process** (L7 p.26)
1. Sender picks forward hosts (e.g., X, Y, Z)
2. Encrypts message with destination's public key
3. Appends header Z→B encrypted with Z's public key
4. Appends header Y→Z encrypted with Y's public key
5. Appends header X→Y encrypted with X's public key
6. Each relay decrypts one layer, forwards to next
7. Like peeling an onion layer by layer

**TOR (The Onion Router)** (L7 p.27)
- Most well-known onion routing network
- Over 5,000 volunteer hosts around the world
- Uses 3-hop circuits: Entry → Middle → Exit node
- Provides anonymity for web browsing

**TOR Properties**
- Entry node knows client IP but not destination
- Middle node knows neither source nor destination
- Exit node knows destination but not original source
- Provides anonymity, not encryption of content

**TOR Limitations**
- Exit node can see unencrypted traffic (use HTTPS with TOR)
- Slower than direct connection (multiple hops)
- Exit nodes can be malicious (traffic analysis)
- Not resistant to global passive adversary

---

## Firewalls (L7 p.28-31)

**Firewall Purpose** (L7 p.28)
- Isolates organization's internal network from Internet
- Controls traffic: allows some packets, blocks others
- Enforces security policy
- Prevents DoS attacks (e.g., SYN flooding)
- Allows only authorized access

**Three Types of Firewalls** (L7 p.28)
- Stateless packet filters
- Stateful packet filters
- Application gateways

**Stateless Packet Filter** (L7 p.29)
- Examines each packet independently (no state)
- Filters based on: source/dest IP, source/dest port, protocol, TCP flags
- Simple rules: allow/deny
- Fast but limited
- Cannot track connection state
- Example vulnerabilities: accepts ACK packets without SYN

**Stateless Filter Examples** (L7 p.30)
- Block all UDP except DNS: drop all incoming UDP except port 53
- Block incoming TCP connections: drop all incoming TCP SYN packets
- Prevent traceroute: drop all outgoing ICMP TTL expired
- Prevent Smurf DoS: drop ICMP to broadcast addresses

**Stateful Packet Filter** (L7 p.30-31)
- Tracks state of every TCP connection
- Maintains connection state table
- Tracks SYN (setup), FIN (teardown), sequence numbers
- Determines if packets "make sense" for the connection
- Blocks unsolicited packets (no matching state)
- Timeouts for inactive connections
- More secure than stateless

**Application Gateway / Proxy** (L7 p.31)
- Filters at application layer (not just IP/TCP/UDP)
- Terminates connections: client → gateway → server
- Deep packet inspection of application data
- Can authenticate users at gateway
- More thorough inspection but slower
- Example: telnet gateway, HTTP proxy

---

## Intrusion Detection Systems (IDS) (L7 p.32)

**IDS vs Packet Filtering**
- Packet filtering: operates only on TCP/IP headers, no correlation
- IDS: deep packet inspection + correlation analysis

**IDS Capabilities**
- **Deep Packet Inspection**: examines packet contents (not just headers)
  - Checks character strings against database of known attacks/viruses
- **Correlation Analysis**: examines correlation among multiple packets
  - Detects: port scanning, network mapping, DoS attacks

**Signature-Based IDS** (L7 p.32)
- Performs pattern matching against known attacks
- Database of attack signatures
- Performs well against DoS attacks
- Fast and accurate for known attacks
- Cannot detect new/unknown attacks (zero-days)

**Heuristic / Anomaly-Based IDS** (L7 p.32)
- Builds model of acceptable behavior
- Flags exceptions to the model
- Uses Inference Engine (learning tool)
- Can be State-based or Model-based
- Can detect unknown attacks
- Higher false positive rate

---

## Privacy and Tracking (L8)

### Privacy Concepts

**PII (Personally Identifiable Information)** (L8 p.2)
- Information that can identify an individual
- Examples: name, SSN, email, IP address, phone number
- Subject to privacy regulations

**GDPR (General Data Protection Regulation)** (L8 p.36-38)
- EU privacy law (2018)
- Requires consent for data collection
- Right to access, delete, port data
- Heavy fines for violations
- Cookie consent banners required

---

### Tracking Methods

**HTTP Cookies** (L8 p.3-4)
- Server sends Set-Cookie header
- Browser stores key-value pairs
- Browser includes cookie in subsequent requests
- First-party: from website's domain
- Third-party: from embedded content (ads, trackers)

**Third-Party Tracking** (L8 p.3-4)
- Tracker embedded in multiple websites (as iframe, image, script)
- Tracker sets cookie on first visit
- Same cookie sent when visiting other sites with same tracker
- Tracker correlates user activity across sites

**Cookie Syncing** (L8 p.6)
- Multiple trackers share/synchronize user IDs
- Tracker A and Tracker B exchange mappings
- Enables cross-tracker user profiling
- User activity combined from multiple tracking networks

**Canvas Fingerprinting** (L8 p.5, 7)
- JavaScript draws on HTML5 canvas element
- `canvas.toDataURL()` extracts image data
- Rendering varies by device (GPU, fonts, OS)
- Hash of image creates unique identifier
- Invisible to user, persists across cookie clearing

**Canvas Font Fingerprinting** (L8 p.7)
- Measures how fonts render on canvas
- Different systems render fonts differently
- Tests multiple fonts for unique signature
- More stable than general canvas fingerprinting

**Browser Fingerprinting** (L8 p.5, 8)
- Collects device characteristics without cookies
- Attributes: User-Agent, screen resolution, timezone, plugins, fonts
- Combination creates unique fingerprint
- Survives cookie clearing, incognito mode

**WebRTC IP Leaking** (L8 p.7)
- WebRTC API reveals local/public IP addresses
- Works even through VPN/proxy
- JavaScript can access without user permission

**AudioContext Fingerprinting** (L8 p.7)
- Uses Web Audio API to generate audio signal
- Audio processing varies by hardware
- Creates fingerprint from audio output
- Works without speakers

**Battery API** (L8 p.8)
- Reveals battery level and charging status
- Can be used as short-term identifier
- Combined with other attributes for tracking

---

### Privacy Protections

**Incognito/Private Mode**
- Doesn't save cookies/history after session
- Does NOT hide IP address from websites
- Does NOT prevent fingerprinting
- ISP and network admin can still see traffic

**VPN (Virtual Private Network)**
- Encrypts traffic to VPN server
- Hides IP address from websites (shows VPN IP)
- Prevents ISP from seeing content (but VPN provider can)
- Can be defeated by WebRTC IP leaking

**Ad Blockers**
- Block third-party tracking scripts
- Prevent ads and trackers from loading
- Reduces fingerprinting surface
- May not catch all tracking methods

**Do Not Track (DNT)**
- HTTP header requesting not to be tracked
- Purely voluntary for websites
- Widely ignored
- Actually increases fingerprinting surface (one more bit of info)

---

### L8 Privacy Research Papers - Key Findings

**Purpose of this section:** Understand the main contribution and findings of each research paper

---

**1. Third-Party Web Tracking Studies**

**What it's about:**
- Measurement studies of how widespread third-party tracking is on the web
- Identifying which trackers are most prevalent
- Understanding tracking ecosystems and relationships between trackers

**Key findings/What's new:**
- Documented prevalence of third-party trackers across top websites
- Identified dominant tracking companies (Google, Facebook, etc.)
- Showed trackers embedded as scripts, iframes, images, beacons
- Revealed tracker cooperation through cookie syncing
- Measured tracking even on sensitive sites (health, financial)

---

**2. Cookie Synchronization/ID Synchronization Research**

**What it's about:**
- How multiple tracking companies share user identifiers
- Mechanisms trackers use to link user IDs across different tracking services
- Building unified profiles from multiple tracker databases

**Key findings/What's new:**
- First large-scale measurement of cookie syncing practices
- Showed trackers exchange user mappings through redirect chains
- Demonstrated how syncing creates super-profiles combining data from multiple sources
- Revealed hidden data sharing between supposedly separate trackers
- Identified synchronization as major privacy threat beyond individual trackers

---

**3. Browser Fingerprinting Research**

**What it's about:**
- Measuring uniqueness of browser/device configurations
- Testing how many bits of entropy different browser attributes provide
- Evaluating fingerprinting as alternative to cookies

**Key findings/What's new:**
- Demonstrated browsers are highly fingerprintable even without cookies
- Showed combination of attributes (fonts, plugins, canvas, screen size) creates unique fingerprints
- Measured stability of fingerprints over time
- Found fingerprints survive cookie clearing and private browsing
- Quantified re-identification rates using fingerprints

---

**4. Canvas Fingerprinting Studies**

**What it's about:**
- Specific technique using HTML5 Canvas API for fingerprinting
- How rendering differences create unique device signatures
- Prevalence of canvas fingerprinting in the wild

**Key findings/What's new:**
- First to identify and measure canvas fingerprinting on real websites
- Showed canvas rendering varies by GPU, drivers, fonts, OS
- Found canvas fingerprinting scripts on top 100K websites
- Demonstrated technique is invisible to users (no permission required)
- More stable than other fingerprinting methods
- Revealed commercial fingerprinting libraries (e.g., AddThis, Ligatus)

---

**5. Canvas Font Fingerprinting**

**What it's about:**
- Enhanced fingerprinting using font rendering on canvas
- Measuring installed fonts through rendering variations
- Improved stability compared to general canvas fingerprinting

**Key findings/What's new:**
- Font rendering provides more stable fingerprint than general canvas
- Different OS/browsers render same fonts differently
- Can detect installed fonts without explicit font enumeration
- Harder to defend against than font enumeration APIs
- Provides high uniqueness with minimal attributes tested

---

**6. WebRTC IP Leaking Studies**

**What it's about:**
- How WebRTC API reveals local and public IP addresses
- Bypassing VPN/proxy protections through browser APIs
- Privacy implications of real-time communication features

**Key findings/What's new:**
- Discovered WebRTC leaks real IP even when using VPN/proxy
- Works through JavaScript without user permission
- Affects all major browsers (Chrome, Firefox, etc.)
- Can reveal internal network topology
- Demonstrated tracking/identification despite anonymity tools

---

**7. AudioContext Fingerprinting**

**What it's about:**
- Using Web Audio API to generate hardware-dependent fingerprints
- How audio processing varies by sound cards and audio stacks
- Adding audio characteristics to fingerprinting arsenal

**Key findings/What's new:**
- First to demonstrate audio processing creates unique fingerprints
- Works even on devices without speakers
- Audio fingerprints depend on hardware (sound card, DSP)
- Adds additional entropy to fingerprinting
- Combined with other methods increases uniqueness

---

**8. Battery Status API Tracking**

**What it's about:**
- Exploiting Battery Status API for user tracking
- Using battery level as short-term identifier
- Privacy implications of hardware status APIs

**Key findings/What's new:**
- Battery level can serve as temporary tracking identifier
- Revealing battery level + charging time + discharge time provides unique signature
- Can be used for short-term cross-site tracking
- Demonstrated privacy risks of seemingly innocuous APIs
- Led to API restrictions in browsers

---

**9. GDPR and Cookie Consent Research**

**What it's about:**
- Impact of GDPR on web tracking practices
- Effectiveness of cookie consent banners
- Compliance with privacy regulations

**Key findings/What's new:**
- Measured changes in tracking before/after GDPR enforcement
- Found many consent banners are non-compliant (dark patterns)
- Showed tracking often occurs before consent given
- Documented pre-checked boxes and confusing interfaces
- Revealed gap between regulation and actual compliance
- Some trackers moved from cookies to fingerprinting to avoid regulation

---

**10. Tracking on Mobile Platforms**

**What it's about:**
- How tracking differs on mobile apps vs web
- Mobile-specific tracking identifiers (IDFA, Android ID)
- Cross-app tracking ecosystems

**Key findings/What's new:**
- Mobile tracking more pervasive than web
- Apps share data with dozens of third parties
- Persistent identifiers harder to reset than cookies
- Less transparency about tracking in mobile apps
- Found tracking in sensitive app categories (health, children)

---

**11. Privacy Policies and User Understanding**

**What it's about:**
- Gap between privacy policies and actual practices
- User comprehension of privacy notices
- Effectiveness of privacy disclosures

**Key findings/What's new:**
- Privacy policies written at college reading level (too complex)
- Users don't read policies (too long, time-consuming)
- Actual data practices often exceed what policies describe
- Vague language allows broad interpretation
- Disconnect between user expectations and reality

---

**12. Tracking Detection and Blocking**

**What it's about:**
- Methods to detect tracking scripts
- Effectiveness of ad blockers and anti-tracking tools
- Evasion techniques used by trackers

**Key findings/What's new:**
- Filter lists (EasyList, EasyPrivacy) block most known trackers
- Trackers evolve to evade blockers (domain rotation, CNAME cloaking)
- First-party tracking harder to block than third-party
- Tracking prevention has arms race dynamic
- Fingerprinting harder to block than cookie-based tracking

---

**13. Cross-Device Tracking Studies**

**What it's about:**
- Linking users across multiple devices (phone, tablet, laptop)
- Deterministic and probabilistic matching techniques
- Building unified user profiles across devices

**Key findings/What's new:**
- Companies use login-based linking (deterministic)
- Also use behavioral patterns for probabilistic matching
- Cross-device tracking enables more complete surveillance
- Often opaque to users (hidden in privacy policies)
- Allows tracking even when cookies cleared on one device

---

**Common Themes Across L8 Papers:**
- **Increasing sophistication:** Tracking evolves to evade protections
- **Lack of transparency:** Users unaware of extent of tracking
- **Regulatory challenges:** Laws struggle to keep up with technology
- **Arms race:** Continuous cycle of tracking innovations and countermeasures
- **Ecosystem complexity:** Multiple parties cooperate in tracking
- **Beyond cookies:** Many tracking methods don't rely on cookies
- **Privacy-utility tradeoff:** Tracking enables services but threatens privacy

---

## Quick Recognition for Multiple Choice

**If question mentions:**
- **"Spoofed source IP" + "many servers respond"** → Reflection Attack
- **"Small request, large response"** → Amplification Attack
- **"Half-open connections" or "SYN without ACK"** → SYN Flooding
- **"Canvas.toDataURL()"** → Canvas Fingerprinting
- **"Third-party embedded in multiple sites"** → Third-party Tracking
- **"Set-Cookie header"** → HTTP Cookies
- **"Sequence number" + "sliding window"** → Anti-replay (ESP)
- **"SPI" + "Security Association"** → ESP/IPsec
- **"HMAC" + "before decryption"** → ESP ICV Verification
- **"Certificate + private key signature"** → TLS Client Authentication
- **"Three relays" + "nested encryption"** → TOR/Onion Routing
- **"Connection state table"** → Stateful Firewall
- **"Each packet independently"** → Stateless Firewall
- **"Known attack patterns"** → Signature-based IDS
- **"Deviation from baseline"** → Anomaly-based IDS

---

## Common Wrong Answer Traps

**DoS vs DDoS:**
- DoS = single source, DDoS = multiple sources (distributed)

**TLS vs IPsec:**
- TLS = transport layer, application-specific (HTTPS)
- IPsec = network layer, transparent to applications

**Authentication vs Confidentiality:**
- Authentication = verify identity (who sent it)
- Confidentiality = encryption (hide content)

**Stateless vs Stateful Firewall:**
- Stateless = no memory of previous packets
- Stateful = tracks connection state

**Cookies vs Fingerprinting:**
- Cookies = explicit identifiers, can be deleted
- Fingerprinting = implicit, persists across cookie clearing

**VPN vs TOR:**
- VPN = all traffic through one server, server knows identity
- TOR = traffic through multiple relays, no single relay knows both ends

**Encryption vs Hashing:**
- Encryption = reversible (decrypt to get original)
- Hashing = one-way (cannot get original back)

**Symmetric vs Asymmetric Crypto:**
- Symmetric = same key for encrypt/decrypt (faster, requires key exchange)
- Asymmetric = public/private key pair (slower, no key exchange needed)
