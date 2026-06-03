---
name: express-security-performance
description: Review, refactor, and harden Node.js + Express backends for security and performance best practices. Use this skill whenever the user wants to audit, review, secure, harden, or speed up an Express/Node backend, asks "is my Express app secure", "review my Node backend", "check for security issues", "make this API faster", or shares Express server code, route handlers, middleware, or an app.js/server.js/index.js for feedback — even if they don't say "review". Also trigger when the user mentions Helmet, rate limiting, CORS, cookies/sessions, JWT, input validation, dependency vulnerabilities, TLS, clustering, compression, or production-readiness of an Express service. Grounded in the official Express production security and performance best practices guides.
---

# Express Security & Performance

Review, refactor, and harden Node.js + Express backends against a consistent set of best practices grounded in the official Express guides (security: https://expressjs.com/en/advanced/best-practice-security/, performance: https://expressjs.com/en/advanced/best-practice-performance/).

This skill does three separable things so the user can ask for any one:

1. **Review** — read the code and report findings (no edits), tiered by severity.
2. **Refactor** — rewrite the code to fix the findings, preserving behavior.
3. **Suggest** — recommend best practices and patterns for code the user is about to write.

Default to **review first**, then offer to refactor. Jump straight to refactoring only if the user explicitly asks to "fix/refactor/harden/secure" it.

## Workflow

1. **Detect the context first.** Before flagging anything, establish what's in play, because the advice bends to it:
   - **Express version** (Express 4 vs 5 — error handling for async, some defaults differ).
   - **Deployment shape**: behind a reverse proxy (nginx, ALB, Cloudflare)? containerized/Kubernetes? serverless (Lambda)? Some advice (TLS termination, clustering, `trust proxy`) depends entirely on this.
   - **What already exists**: is Helmet present? a validation layer (Joi/Zod/express-validator)? a session/auth strategy? a process manager (PM2/systemd) or orchestrator? Don't recommend what's already there.
   - **First-party vs library code**: don't flag a dependency's internals.
2. **Apply the concern areas.** Each has a dedicated reference file using the format Overview / When to Use / Common Rationalizations / Red Flags / Verification / Before-After. Read the relevant reference when a concern is in scope:
   - `references/security-headers-and-helmet.md` — Helmet, `X-Powered-By`, security headers
   - `references/input-and-injection.md` — input validation, query/command/NoSQL injection, payload limits
   - `references/auth-sessions-cookies.md` — sessions, cookie flags, JWT, secrets, brute-force/rate limiting
   - `references/transport-and-config.md` — TLS/HTTPS, `trust proxy`, CORS, env config, dependency hygiene
   - `references/error-handling-and-logging.md` — leaking stack traces, sync error handling, safe logging
   - `references/performance-runtime.md` — gzip compression, async (no sync calls in request path), avoiding heavy work in Node
   - `references/performance-deployment.md` — `NODE_ENV=production`, clustering/load, process managers, caching, reverse proxy
3. **Report findings tiered by severity** (see Output format).
4. **Offer the refactor.** If reviewing, end by offering to produce the corrected code.

## Severity tiers

Classify every finding so the review is actionable rather than a flat list:

- **Blocker** — a real, exploitable security hole or a production-breaking issue (e.g. no input validation on a query passed to the DB, secrets hardcoded in source, `X-Powered-By` plus a known-vulnerable dependency, stack traces returned to clients in production, a synchronous `fs.readFileSync` in a hot request path).
- **Warning** — works today but is a foreseeable risk or bottleneck (e.g. Helmet absent but app behind a WAF, no rate limiting on a login route, no gzip compression, missing `secure`/`httpOnly` on a cookie behind HTTPS).
- **Nit** — hardening or polish (e.g. could set a stricter CSP, could add `helmet` even though headers are set manually, could move a constant out of the handler).

Never invent blockers to pad a review. If the code is solid, say so. A clean Express app behind a properly configured proxy with Helmet, validation, and rate limiting deserves to hear it's clean.

## Context awareness (important)

The rules are not absolute — they yield to the deployment reality:

- **Behind a reverse proxy / managed platform**: TLS, compression, and even some headers may be handled at the proxy. Don't flag "no HTTPS in the Node code" if nginx/ALB terminates TLS — instead verify `trust proxy` is set correctly and `secure` cookies still work. Compression at the proxy is *preferable* to `compression` middleware for high-traffic apps.
- **Serverless (Lambda/Cloud Functions)**: clustering and process managers are irrelevant — the platform handles concurrency. Cold-start and per-invocation cost dominate instead.
- **Internal-only / non-public service**: rate limiting and brute-force protection matter less; injection and dependency hygiene still matter fully.
- **Express 5**: async errors propagate to error middleware automatically; don't recommend Express-4-era `try/catch` wrapping if they're on 5.

When unsure about the deployment shape, ask before flagging proxy/TLS/clustering items — the right answer flips entirely based on it.

## Output format

ALWAYS use this structure for a review:

```
## Express Security & Performance Review

**Context:** [Express version, deployment shape, what already exists — Helmet/validation/auth/proxy]

### Blockers
- [file:line or route] — [issue] → [fix, one line]

### Warnings
- ...

### Nits
- ...

### Summary
[1-2 sentences: overall health + what to prioritize first]
```

Pull concrete before/after snippets from the reference files into the report when they make a fix clearer. Keep security and performance findings in the same tiered list (an attacker and a load spike are both production risks) but label each finding `[sec]` or `[perf]` so the user can triage.

For a **refactor**, output the corrected code (as files if substantial, inline if short), then a short changelog mapping each change back to its concern area so the user can see what moved and why. Preserve behavior — never silently change an API contract while hardening it; call out any behavior change explicitly.

For **suggest** mode (user is writing new code, not fixing existing), give the recommended pattern up front with a minimal example, cite which concern area it covers, and note the common mistake it avoids.

## Reference files

- `references/security-headers-and-helmet.md`
- `references/input-and-injection.md`
- `references/auth-sessions-cookies.md`
- `references/transport-and-config.md`
- `references/error-handling-and-logging.md`
- `references/performance-runtime.md`
- `references/performance-deployment.md`

Read the ones relevant to the code in front of you rather than all seven every time.