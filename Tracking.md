# Essay Topic 1: Tracking Methods

## Reference: L8_Privacy.pdf

---

## Two Common Ways to Track Users

### 1. Cookie-Based Tracking (HTTP Cookies)

#### What to Track:
- User identity across sessions
- Browsing behavior across different websites
- User preferences and login state
- Shopping cart contents
- Cross-site user activity

#### How to Track:

**Basic Mechanism:**
- Server sends `Set-Cookie` header in HTTP response
- Browser stores the cookie
- Browser automatically includes cookie in subsequent HTTP requests to that domain
- Cookie contains key-value pairs with user identifiers

**Third-Party Tracking:** (L8 p.3-4)
- Website A embeds content from Tracker.com (e.g., ad, image, iframe)
- Tracker.com sets a cookie in the user's browser
- When user visits Website B (also has Tracker.com content), browser sends the same cookie
- Tracker.com can now link user's activity across Website A and Website B

**Cookie Syncing:** (L8 p.6)
- Multiple trackers synchronize their user IDs
- Tracker1.com and Tracker2.com exchange user mappings
- Allows trackers to combine their tracking data
- Creates more complete user profiles across different tracking networks

**Technical Details:**
- Cookies stored in browser's cookie jar
- Automatically sent with HTTP requests matching domain/path
- Can be persistent (with expiration date) or session cookies
- Third-party cookies: set by domains different from the website being visited

---

### 2. Browser Fingerprinting

#### What to Track:
- Device configuration and characteristics
- Browser type, version, and settings
- Installed fonts and plugins
- Screen resolution and color depth
- Hardware capabilities (GPU, audio system)
- System timezone and language
- Unique device identifiers

#### How to Track:

**Canvas Fingerprinting:** (L8 p.5, 7)
- JavaScript draws hidden text/graphics on HTML5 Canvas element
- `canvas.toDataURL()` extracts pixel data as image
- Pixel rendering varies slightly between devices (due to GPU, drivers, fonts)
- Hash of the image creates unique fingerprint
- Technique is invisible to user
- Works without cookies or user consent

**Canvas Font Fingerprinting:** (L8 p.7)
- Measures how fonts are rendered on canvas
- Different OS/browsers render fonts differently
- Tests multiple fonts to create unique signature
- More stable than general canvas fingerprinting

**WebRTC IP Leaking:** (L8 p.7)
- WebRTC API reveals local IP addresses
- Can bypass VPN/proxy protections
- Provides network-level identification

**AudioContext Fingerprinting:** (L8 p.7)
- Uses Web Audio API to generate audio signal
- Audio processing varies by hardware (sound card, audio stack)
- Analyzes output to create fingerprint
- Works even if device has no speakers

**Battery API:** (L8 p.8)
- Reads battery level and charging status
- Battery level can serve as temporary identifier
- Combined with other attributes for tracking

**Other Fingerprinting Attributes:** (L8 p.5)
- User-Agent string (browser/OS version)
- Screen resolution and color depth
- Installed plugins (Flash, Java, etc.)
- Timezone offset
- HTTP Accept headers
- Do Not Track setting
- Installed fonts (via CSS/JavaScript detection)

---

## Key Differences Between Methods

| Aspect | Cookie-Based | Browser Fingerprinting |
|--------|--------------|------------------------|
| **User Awareness** | Users can see/delete cookies | Invisible to users |
| **Persistence** | Until cookie expires or deleted | Survives cookie clearing |
| **Accuracy** | High (unique ID assigned) | Medium (collisions possible) |
| **Regulation** | Subject to GDPR/consent laws | Less regulated |
| **Circumvention** | Clear cookies, incognito mode | Difficult to prevent |
| **Cross-browser** | Separate cookies per browser | Can track across browsers |

---

## Research Findings (from L8)

- **Prevalence:** Canvas fingerprinting found on top websites (L8 p.7)
- **Third-party tracking:** Widespread across popular websites (L8 p.3-4)
- **Cookie syncing:** Enables tracking networks to share user data (L8 p.6)
- **Fingerprinting combinations:** Multiple techniques used together increase uniqueness
- **GDPR impact:** Cookie consent requirements, but fingerprinting less affected (L8 p.36-38)

---

## Essay Answer Structure

**For "2 common ways to track (what to track and how to track)":**

1. **Cookie-Based Tracking**
   - What: User identity, browsing behavior, cross-site activity
   - How: HTTP Set-Cookie headers, third-party cookies, cookie syncing
   - Example: Third-party tracker embedded in multiple websites

2. **Browser Fingerprinting**
   - What: Device configuration, hardware characteristics, unique identifiers
   - How: Canvas fingerprinting, font rendering, WebRTC, AudioContext
   - Example: Canvas toDataURL() creates unique device signature

**Key Points to Include:**
- Cookies are explicit identifiers, fingerprinting is implicit
- Cookies can be deleted, fingerprints persist
- Third-party tracking enables cross-site correlation
- Fingerprinting combines multiple attributes for uniqueness
