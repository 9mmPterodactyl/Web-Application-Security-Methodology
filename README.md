# Web Application Security Testing Methodology For Bug Bounty

**by 9mmPterodactyl**

This is my approach to bug bounty hunting that I've developed over the past year. It's focused on deep recon and systematic testing. Shoutout to NahamSec, JHaddix, Professor Messer, the Critical Thinking podcast crew and many others that put out so much great content for people to learn from, they are the reason I am able to write this today.  

## Why This Methodology

Coming from HTB machines, I had to completely change how I think about testing. CTFs are designed to be vulnerable - real apps aren't. You need more recon, you need to understand the business logic, and you need to respect scope and rate limits. This methodology reflects that shift.

---

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

This is where I spend most of my time. The better your recon, the more you find.

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

### Indirect Object References (IDOR)

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
- Subfinder, Assetfinder, Amass
- HTTPx, Naabu, Nuclei, Nmap
- Certificate transparency logs (crt.sh)

**Enumeration:**
- WayMore by xnl-h4ck3r
- ffuf, feroxbuster (directory brute forcing)
- Burp Suite (main testing tool)

**Exploitation:**
- Burp Suite + extensions
- VPS for exploiting SSRF and other vuln types
- Custom scripts for specific tests
- Browser dev tools

---

## What I'm Still Working On

This methodology keeps evolving. I'm still learning and adjusting my approach based on:
- Studying disclosed bug bounty reports
- Understanding what actually gets accepted vs rejected
- Learning from failed attempts
- New vulnerability research

The goal is to go deep rather than wide. Take a week just to try and understand the infastructure and logic of the website before diving into pre-structured methodology. Better to thoroughly understand and test one application than superficially poke at ten.

---

## Platforms I'm Active On

- HackerOne
- Bugcrowd
- Intigriti
- YesWeHack


---

**Note:** This is only for authorized testing in bug bounty programs where you're given explicit permission to test. Always follow program rules and scope.
