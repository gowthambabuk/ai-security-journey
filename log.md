# Learning Log

Goal: Full-stack developer (8 yrs) → AI Security / AppSec Engineer
Plan: 30 weeks · 1 hr weekdays · 2 hrs weekends · started 29 Oct 2026

---

## Day 1 — Thu 29 Oct 2026
- Created PortSwigger Web Security Academy account
- Read the Web LLM attacks topic
- Installed Burp Suite; intercepted and modified requests in Repeater
- Key idea: an LLM is a new entry point; the damage comes from the APIs/tools it can call

## Day 2 — Fri 30 Oct 2026
- Created this repo and learning log
- Setup: Homebrew blocked by a Command Line Tools error; installed Burp directly from portswigger.net instead

## Day 3 — Sat 31 Oct 2026
- ✅ Solved PortSwigger lab: Exploiting LLM APIs with excessive agency
- Asked the bot which APIs it had → found a Debug SQL API
- Bot refused a raw `SELECT * FROM users`, but ran it when asked to "call the Debug SQL API"
- It hid Carlos's password, yet executed `DELETE FROM users WHERE username='carlos'`
- Lesson: refusals ≠ security. The guardrail protected data but allowed a destructive action.
  The real risk is the tool's permissions → OWASP LLM Top 10: Excessive Agency
- Fix in a real system: no raw SQL tool, least-privilege DB user, human approval for destructive actions

## Day 4 — Sun 1 Nov 2026
- Mapped OWASP Top 10:2025 to my own experience (below)
- Updated LinkedIn headline toward AI Application Security

### OWASP Top 10:2025 — mapped to my experience
- **A01 Broken Access Control** → Not seen. In my APIs, changing an ID returned 404/no data
  because ownership was checked server-side (user/tenant derived from the token, not the request).
  No features fetched user-supplied URLs, so no SSRF exposure.
- **A02 Security Misconfiguration** → Public-facing API gateway (Azure APIM) behind an
  edge layer (Cloudflare) with MFA via Microsoft identity; backends internal and VPN-only,
  so the gateway was the only public entry point. Learned: next step beyond network
  isolation is zero trust (services authenticate each request even inside the network).
- **A03 Software Supply Chain Failures** → Used npm audit for known-vulnerable packages,
  pinned dependencies with package-lock.json, secrets in Azure Key Vault fetched at runtime
  by CI/CD (never in code). Learned: next steps are passwordless pipeline auth (managed
  identity / OIDC) to solve "secret zero", least-privilege pipeline RBAC, automated
  dependency scanning per PR, and SBOMs.
- **A04 Cryptographic Failures** → Early on used SHA-256 for passwords, then moved auth to
  Microsoft identity (no passwords stored). TLS everywhere; encryption at rest handled by
  managed DBs (Cosmos DB, MongoDB). Learned: SHA-256 is too fast for passwords; use
  Argon2id/bcrypt. Provider encryption at rest doesn't protect against app/account
  compromise; customer-managed keys and field-level encryption go further.
- **A05 Injection** → Worked with APIs where database queries were built dynamically from
  client requests (SQL & NoSQL). Learned: dynamic queries need parameterization, schema
  validation, and whitelisted fields/operators; otherwise NoSQL operator injection
  ($ne, $gt, $regex) and SQL injection become possible. Never reached shell/eval.
  Follow-up: practise SQLi/NoSQLi in PortSwigger labs (Week 5).

Moving to Microsoft identity was the right call, and there’s a key lesson in it.
Why SHA-256 is wrong for passwords: it’s built to be fast. A modern GPU can try billions of SHA-256 guesses per second, so leaked hashes get cracked quickly. Passwords need deliberately slow, salted algorithms: Argon2id (the current recommendation), bcrypt, or scrypt. Delegating authentication to Microsoft identity avoids the problem entirely, because you don’t store passwords at all. This is a classic interview question: “How would you store passwords?” Now you have a real story to tell.
About encryption at rest by default: it’s good, but know its limits. Provider encryption protects against stolen disks or backups. It does not help if someone gets into the app or the database account, because the data decrypts automatically for them. The next levels are customer-managed keys (in Key Vault) and field-level encryption for very sensitive fields.
Log line for A04:
- **A04 Cryptographic Failures** → Early on used SHA-256 for passwords, then moved auth to
  Microsoft identity (no passwords stored). TLS everywhere; encryption at rest handled by
  managed DBs (Cosmos DB, MongoDB). Learned: SHA-256 is too fast for passwords; use
  Argon2id/bcrypt. Provider encryption at rest doesn't protect against app/account
  compromise; customer-managed keys and field-level encryption go further.
A05 Injection
In your Flask and Go APIs, did you build SQL with string concatenation, or use parameterized queries / an ORM?
With MongoDB/Cosmos, were queries ever built directly from request JSON? (NoSQL injection, like {"$ne": null} tricks)
Any place where user input reached a shell command, template, or eval?
queries were built based on the request by the client. it was static queries. Then moved to dynamic query building ways. It was built directly from request. no didn't reach
This one’s worth pausing on, because queries built directly from the client’s request are a classic injection risk. Whether it was actually exploitable depends on the details.
How NoSQL injection works. Say a login query is built from the request body:
js
db.users.find({ username: req.body.username, password: req.body.password })
An attacker sends:
json
{ "username": "admin", "password": { "$ne": null } }
Now the query means “password is not null”, which matches every user, so they log in as admin. The same trick works with $gt, $regex, or $where. Cosmos SQL-API queries built with string concatenation have the classic SQL injection problem too.
It was probably safe if:
Input was validated against a schema (types enforced, so objects rejected where strings are expected)
Only whitelisted field names and operators were allowed
Values went in as parameters, such as Cosmos parameterized queries (@param)
Important, given what you just told me:
Don’t write this one in your public repo as a real finding. If any version of that dynamic query code is still running, describing it publicly is risky for Ericsson and for you.
If you think current code might be vulnerable, check it privately at work and raise it through Ericsson’s normal internal channels (your team or the security team). Doing that well is real AppSec experience, and it’s an excellent interview story afterwards: “I spotted a potential injection risk in our own code and got it fixed.”
Log line for A05 (generic and safe):
- **A05 Injection** → Worked with APIs where database queries were built dynamically from
  client requests (SQL & NoSQL). Learned: dynamic queries need parameterization, schema
  validation, and whitelisted fields/operators; otherwise NoSQL operator injection
  ($ne, $gt, $regex) and SQL injection become possible. Never reached shell/eval.
  Follow-up: practise SQLi/NoSQLi in PortSwigger labs (Week 5).
A06 Insecure Design
Did your APIs have rate limiting? APIM has policies for this.
In multi-tenant systems, was tenant isolation designed in from the start, or added later?
Did anyone do threat modeling or security reviews before building features?
yes APIs have rate limiting to avoid throttling, and usually its not a static, its dynamic based on server resources and capacity. isolation designed in from start. I did
That’s strong, and the last part matters a lot. You did security reviews yourself. That’s direct AppSec experience. Put it on your resume and LinkedIn experience as something like “Conducted security design reviews for new features”. Recruiters for security roles look for exactly that.
One nuance on rate limiting (good interview material). Capacity-based dynamic limits protect availability. Security rate limiting has a different goal: stopping abuse per user, per IP, or per API key. A capacity limit won’t stop an attacker trying 5 passwords a minute on one account, since that never stresses the server. Strong designs layer both:
Global or dynamic limits to protect capacity
Per-identity limits on sensitive endpoints (login, OTP, password reset) to stop brute force and enumeration
Log line for A06:
- **A06 Insecure Design** → APIs had dynamic rate limiting based on server capacity;
  tenant isolation designed in from the start; I conducted security reviews of features
  before build. Learned: capacity limits protect availability, but abuse needs
  per-user/IP/key limits on sensitive endpoints (login, OTP, reset).
- **A07 Authentication Failures** → MFA via Microsoft identity; 1h access tokens with
  proactive refresh before expiry; logout invalidated sessions server-side; built a custom
  password reset flow. Learned: JWTs stay valid until expiry, so pair short lifetimes with
  refresh-token revocation; custom reset flows need random single-use short-lived tokens
  (stored hashed), no user enumeration, session invalidation, Host-header protection.
- **A08 Software or Data Integrity Failures** → IoT firmware updates were signed and
  verified on-device before install; build artifacts signed/verified before deployment;
  no untrusted deserialization. Learned: in AI, pickle-based model files can execute
  code on load, so prefer safetensors and scan models from public hubs.
- **A09 Security Logging and Alerting Failures** → Centralized logging in Log Analytics /
  App Insights with end-to-end tracing (client IP → server); alerts for traffic spikes and
  failed logins; minimal personal data (email for tracing only). Learned: email and IP
  count as personal data under GDPR/DPDP, so use retention limits, access controls,
  pseudonymous IDs; never log tokens or auth headers.
- **A10 Mishandling of Exceptional Conditions** → Clients saw generic error messages (no
  stack traces); no fail-open paths; timeouts and retries tuned to backend behavior.
  Learned: retries need exponential backoff + jitter, circuit breakers, and idempotent
  writes to avoid retry storms and duplicate actions.

---

## Progress
- PortSwigger labs solved: 1
- Blog posts: 0
- Portfolio projects: 0/3
