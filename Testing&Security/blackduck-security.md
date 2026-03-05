# BlackDuck Security Scan + Application Security

## Question
What is BlackDuck and why do you run it on every PR? What other security practices does your team follow?

---

## My Answer
I was not confident on this topic during the interview. The answer below is what I learned after.

---

## Model Answer
We use BlackDuck on every PR to scan third-party dependencies for known CVEs and license violations. On the application security side, we protect against XSS by avoiding unsafe HTML rendering in React and setting CSP headers. We protect against SQL injection through TypeORM's parameterized queries. Our JWT tokens are stored in HttpOnly Secure cookies so JavaScript cannot access them, protecting against XSS token theft. We keep access tokens short-lived to limit the damage window if a token is ever compromised.

---

## Key Things to Remember

**What is BlackDuck:**
- Software Composition Analysis (SCA) tool
- Scans all third-party dependencies (package.json) for known security vulnerabilities (CVEs)
- Also checks license compliance — some open source licenses don't allow commercial use
- CVE = Common Vulnerability and Exposure — a publicly disclosed security vulnerability

**Why scan on every PR:**
- Vulnerabilities are discovered daily
- A library safe last month may have a CVE reported today
- Scanning on every PR ensures no vulnerable dependency reaches production

**The second security check (confirm with team):**
- Likely SonarQube — static code analysis for code quality and security issues
- Or OWASP Dependency Check — similar to BlackDuck for dependency scanning

**Common security threats and protections:**

XSS — Cross Site Scripting
- Attacker injects malicious JavaScript into your page
- Protection: never use `dangerouslySetInnerHTML` without sanitizing, set Content Security Policy headers

CSRF — Cross Site Request Forgery
- Attacker tricks a logged-in user's browser into making unauthorized requests
- Protection: `SameSite=Strict` on cookies, CSRF tokens

SQL Injection
- Attacker sends malicious SQL through input fields
- Protection: TypeORM parameterized queries protect automatically, never concatenate raw user input into SQL

JWT Security
- Store in HttpOnly cookies — not localStorage (XSS can steal localStorage)
- Always use Secure flag — cookies only travel over HTTPS
- Keep access token expiry short — 15 minutes
- Validate token signature on every request server-side
- Never store sensitive data in JWT payload — it is encoded not encrypted

**Cookie security flags:**
- `HttpOnly` — JavaScript cannot read the cookie
- `Secure` — cookie only sent over HTTPS
- `SameSite=Strict` — cookie not sent with cross-site requests, prevents CSRF

---

## Things to Improve
- Find out what the second security check on PRs is — ask the team
- Learn what a CVE looks like and how to read one on the NVD database
- Understand what SonarQube does and how it differs from BlackDuck
- Practice explaining XSS and CSRF with a simple example story
