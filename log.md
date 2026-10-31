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
- **A01 Broken Access Control** (now includes SSRF) → ...
- **A02 Security Misconfiguration** → ...
- **A03 Software Supply Chain Failures** → ...
- **A04 Cryptographic Failures** → ...
- **A05 Injection** → ...
- **A06 Insecure Design** → ...
- **A07 Authentication Failures** → ...
- **A08 Software or Data Integrity Failures** → ...
- **A09 Security Logging and Alerting Failures** → ...
- **A10 Mishandling of Exceptional Conditions** → ...

---

## Progress
- PortSwigger labs solved: 1
- Blog posts: 0
- Portfolio projects: 0/3