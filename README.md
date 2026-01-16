# Web Application Bug Bounty Methodology

**by 9mmPterodactyl**

Shoutout to NahamSec, JHaddix, Professor Messer, the Critical Thinking podcast crew, ars0n security, James Kettle aka albinowax and many others that put out so much great content for people to learn from.  

## Purpose and Scope

This repository documents my bug bounty–oriented web application vulnerability research methodology. It is optimized for long-running, competitive targets where the objective is to identify high-impact, exploitable vulnerabilities under strict program scope and stability constraints.

Unlike professional penetration testing engagements, bug bounty research allows extended time horizons and deep focus on specific vulnerability classes. 

As such, this methodology prioritizes depth over breadth, manual analysis over automated coverage, and exploitability over exhaustive security posture review.

This methodology is not intended to represent how I conduct client-facing security assessments, which are documented separately. All testing described here is performed exclusively under explicit authorization provided by bug bounty program scopes and rules.

## Time Horizon and Optimization Strategy

Bug bounty research is conducted over extended periods—often weeks or months per target—allowing focused exploration of complex attack surfaces and niche vulnerability classes. Testing decisions are optimized for:

High-impact vulnerability discovery (RCE, account takeover, authorization bypass)

Manual analysis of application logic and trust boundaries

Payload restraint to avoid service disruption

Strict adherence to in-scope assets and rate-limiting requirements

This differs from commercial penetration testing, where limited engagement timelines necessitate broader coverage and prioritized risk identification across the entire application.

As a result, lower-severity configuration issues or defense-in-depth findings may be deprioritized in favor of exploitable attack paths.

---

## Vulnerability Classification Framework

While vulnerability discovery in bug bounty research is driven by application-specific behavior and attack surface analysis, findings are evaluated and categorized using the OWASP Top 10 as a baseline framework.

The OWASP Top 10 provides a consistent reference for identifying common and systemic web application risks (e.g., broken access control, injection, authentication failures), while deeper research focuses on less common implementation-specific flaws that fall outside generic scanning coverage.

## Phase 1: Initial Information Gathering

Before touching anything, I read through the entire bug bounty program rules and scope.

**What I'm looking for:**
- Exact scope - which domains/subdomains are fair game
- Out of scope stuff (don't waste time on these)
- Rate limiting requirements (some programs are strict about this)
- What they consider in vs out of scope for specific vulnerabilities
- Suggested severity ratings for different bug types

I also check if there are any disclosed reports to see what's already been found and what the program actually pays for.

---

## Phase 2: Reconnaissance

Reconnaissance represents the majority of effort in my bug bounty workflow, as attack surface visibility directly correlates with finding quality.

### Subdomain Enumeration

**Goal:** Find every in-scope URL and subdomain

**My process:**
1. Run subdomain enumeration on all in-scope domains
2. HTTPx on everything found to see what's actually live
3. Naabu port scan on all enumerated IPs (if allowed in scope)

**Tools:** Subfinder, Assetfinder, Amass or shodan for passive discovery, then active enumeration if scope allows

### Port Scanning (When Allowed)

**Important:** Make sure you're within BBP scope and policy before doing this. Rate limiting at the network level is necessary - ping the service before and during scanning to make sure you're not affecting user experience.

**What I do:**
- Full Nmap scan on high priority targets
- Look for anything running on unexpected open ports
- Integrate Nmap with Naabu for simple automation

### Directory Enumeration

Two approaches here:

**Active enumeration:**
- ffuf for directory/file discovery (feroxbuster is also a good option)
- Look for hidden endpoints, admin panels, backup files, anything interesting

**Passive enumeration with Burp Suite:**
- Navigate through the app normally with Burp proxy running
- Let it map out all the endpoints
- Look for hidden endpoints in JavaScript files
- Check for API endpoints that aren't linked in the UI

---

## Phase 3: Decide What to Focus On

After recon, I decide which vulnerabilities I'm going to hunt for in this session.

**This depends on:**
- The tech stack (what frameworks, CMS, languages are they using)
- BBP scope (what's actually allowed and worth testing)
- What looks promising from recon

I don't try to test for everything at once. Pick 2-3 vulnerability types and go deep on those.

---

## Phase 4: Vulnerability Testing

Here's what I look for, depending on what I decided to focus on:

### Insecure Direct Object References (IDORs)

This is one of my main focuses because it's common and often overlooked.

**Process:**
1. Create two test accounts (User A and User B)
2. Map all authenticated endpoints in Burp
3. Look for any request with IDs (user_id, order_id, document_id, etc.)
4. Try accessing User B's resources with User A's session
5. Test in: profiles, orders, documents, messages, API endpoints, admin functions

### Server-Side Request Forgery (SSRF)

Look for anywhere the app makes requests based on user input:
- Import functions
- Webhooks
- PDF generators
- Image proxies

Test if you can make it request internal resources or use it for SSRF.

### Cross-Site Scripting (XSS)

Test in every input field, especially:
- Comments
- Profile fields
- Search functionality
- Anywhere user input is reflected

### SQL Injection

Still worth testing even though most modern frameworks protect against it:
- All input fields
- URL parameters
- Search and filter functions
- Manual checking for Blind Time-Based SQLi and Blind SQLi with Conditional-Errors

### Business Logic Flaws

This requires understanding how the app actually works:
- Map out workflows completely
- Look for ways to abuse the intended functionality
- Payment flows, discounts, race conditions
- Things like negative quantities, price manipulation

### Cross-Origin Resource Sharing (CORS)

Check if the CORS policy is too permissive:
- Allows untrusted origins
- Accepts null origin
- Wildcard subdomains

### HTTP Request Smuggling

For apps behind load balancers or proxies:
- Test for CL.TE and TE.CL desync
- Can be used to bypass security controls

### Remote Code Execution (RCE)

High severity, harder to find:
- File upload vulnerabilities
- Template injection
- Deserialization flaws
- Command injection

### API Enumeration/Exploitation

APIs are often less hardened than the main app:
- Find API endpoints through JS files and Burp history
- Test all endpoints for proper authentication
- Look for mass assignment
- Check for excessive data exposure
- Test older API versions (v1 when v2 exists)

### Misconfigurations

Sometimes the easiest finds:
- Exposed admin panels
- Directory listing enabled
- .git folders, .env files, backups
- Weak security headers
- Verbose error messages leaking info

---

## Tools I Use

**Recon:**
- Subfinder, Assetfinder, Amass, Shodan
- HTTPx, Naabu, Nuclei, Nmap
- Certificate transparency logs (crt.sh)

**Enumeration:**
- WayMore by xnl-h4ck3r
- ffuf, feroxbuster (directory brute forcing, vhost/subdomain bruteforcing)
- Burp Suite (main testing tool)

**Exploitation:**
- Burp Suite + extensions
- VPS for exploiting SSRF and other vuln types
- Custom scripts for specific tests
- Browser dev tools
- XSS Hunter

---

## Severity Assessment and Prioritization

Vulnerability severity is evaluated using the CVSS v3.1 framework as a baseline, as this aligns with how most bug bounty programs and triage teams assess technical impact and exploitability.

In practice, CVSS scoring is used to inform severity classification rather than dictate it rigidly. Final prioritization considers program-specific context, affected assets, exploit preconditions, and demonstrated impact.

Research focus is biased toward vulnerability classes that consistently score High or Critical under CVSS v3.1, as these findings typically represent meaningful security risk and align with higher payout tiers across bug bounty platforms.

## Depth-Focused Research vs Broad Assessments

Bug bounty research emphasizes deep investigation into selected attack vectors rather than comprehensive coverage of all application functionality. This approach enables the discovery of complex, high-value vulnerabilities that often require sustained analysis and iteration.

In contrast, professional penetration testing engagements typically prioritize broad coverage within limited timeframes, 

identifying a wider range of issues—including low and medium severity findings—rather than pursuing extended exploit development.

---

## Platforms I'm Active On

- HackerOne
- Bugcrowd
- Intigriti
- YesWeHack


---

**Note:** This is only for authorized testing in bug bounty programs where you're given explicit permission to test. Always follow program rules and scope.
