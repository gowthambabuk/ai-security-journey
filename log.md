# Learning Log

   Goal: Going deeper into application & AI security — labs, notes, projects
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

### Key takeaways from the mapping
- Strongest areas: authentication, API gateway security, secrets management, IoT update integrity
- To deepen: injection (SQL/NoSQL), password reset security, zero trust, pipeline identity

---

## Day 5–6 — Mon 2 / Tue 3 Nov 2026
- Set up Hashnode blog: gowthambabuk.hashnode.dev
- Fixed Mac setup: Command Line Tools for Xcode 26 (matched to macOS 26.0.1; 26.4.1 was incompatible), then Homebrew + git, python, jq
- Published my first LinkedIn post: the "Excessive Agency" lab lesson
- ✅ Week 1 complete

## Day 7 — Wed 4 Nov 2026 (Week 2 head start)
- ✅ Solved PortSwigger lab: Username enumeration via different responses
- Used Burp Intruder (Sniper) to find a valid user (different error message), then brute-forced the password (302 vs 200)
- Lesson: generic login errors + per-account rate limiting + MFA stop this

## Day 8 — Thu 5 Nov 2026 (Week 2)
- ✅ Solved PortSwigger lab: 2FA simple bypass
- After password login the session was already valid; browsing straight to /my-account skipped 2FA
- Lesson: enforce MFA server-side on every protected route; treat post-password sessions as
  "partially authenticated". IdP-based MFA (e.g. Microsoft identity) avoids this by only
  issuing tokens after MFA.

## Day 9 — Fri 6 Nov 2026 (Week 2)
- ✅ Solved PortSwigger lab: Password reset broken logic
- Reset token wasn't bound to the user; server trusted the `username` field in the body
- Changed username=wiener → carlos (empty token) → reset Carlos's password → logged in
- Lesson: bind reset tokens to the account server-side; single-use, random, short-lived, hashed
- Ties to my custom reset flow: check token-to-user binding

## Progress
- PortSwigger labs solved: 4
- OWASP Top 10:2025 mapped: ✅
- Blog: set up (0 posts)
- LinkedIn posts: 1
- Portfolio projects: 0/3
