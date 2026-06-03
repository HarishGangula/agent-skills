# Agent Skills

**Production-grade engineering skills for AI coding agents.**

Skills encode the workflows, quality gates, and best practices that senior engineers use when building software. These are packaged so AI agents follow them consistently across every project.

---

## What Are Skills?

Skills are structured Markdown files (`SKILL.md`) that teach AI coding agents *how* to do a task — not just *what* to do. Each skill includes:

- **When to trigger** — clear activation phrases so the agent knows when to apply the skill
- **Step-by-step workflow** — an ordered process the agent follows end-to-end
- **Conventions & templates** — standardized formats, naming rules, and output structures
- **Verification checklists** — quality gates the agent checks before delivering output
- **Reference files** — extended examples, error catalogs, and edge-case documentation

---

## Skills

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [sunbird-api-spec](skills/sunbird-api-spec/SKILL.md) | Design and document RESTful APIs following the Sunbird convention — consistent URL patterns, request/response envelopes, error formats, and naming rules | Designing new API endpoints, producing formal API documentation, defining contracts between frontend and backend or between microservices |
| [css-code-quality](skills/css-code-quality%20/%20SKILL.md) | Review, refactor, and optimize CSS/SCSS against W3C/MDN best practices — catches hardcoded colors, `!important` abuse, px-heavy sizing, inline styles, and more | Reviewing CSS, refactoring stylesheets, cleaning up styles, enforcing CSS conventions, or checking CSS code quality |
| [express-security-performance](skills/express-security-performance/SKILL.md) | Review, refactor, and harden Node.js + Express backends for security and performance — covers Helmet, input validation, auth, TLS, clustering, and compression | Auditing Express apps for security holes, hardening production backends, optimizing API performance, or reviewing server/route handler code |

---

### sunbird-api-spec

A comprehensive skill for designing and documenting REST APIs that follow the [Sunbird](https://sunbird.org/) convention. It covers:

- **URL pattern** — `/{resource}/{version}/{verb}/{resourceId}` with consistent plural/singular naming
- **Request/response envelope** — symmetric `id`, `ver`, `ts`, `params`, `request`/`result` structure
- **Verb-to-HTTP mapping** — standard CRUD (`create`, `read`, `update`, `delete`, `list`, `search`) plus domain-specific verbs (`publish`, `retire`, `flag`, etc.)
- **Error handling** — standardized `ERR_<DOMAIN>_<DETAIL>` codes with full HTTP status mapping
- **Endpoint documentation template** — a ready-to-use format for consistent API specs
- **Verification checklist** — 16-point quality gate to validate every spec before delivery

**Includes reference files:**

| File | Purpose |
|------|---------|
| [error-catalog.md](skills/sunbird-api-spec/references/error-catalog.md) | Full JSON examples for every standard error response |
| [example-full-spec.md](skills/sunbird-api-spec/references/example-full-spec.md) | Complete worked example — an "Order" resource with all CRUD + search endpoints |
| [state-workflow.md](skills/sunbird-api-spec/references/state-workflow.md) | Draft → Review → Live → Retired state machine and transition endpoints |

---

### css-code-quality

Review, refactor, and optimize CSS against a consistent set of best practices grounded in W3C/MDN guidance. The skill operates in three separable modes:

1. **Review** — read CSS and report findings (no edits), tiered by severity
2. **Refactor** — rewrite the CSS to fix findings, preserving visual output
3. **Optimize** — reduce redundancy, collapse selectors, remove dead rules

**Key capabilities:**

- **Framework-aware** — adapts rules for Tailwind, Bootstrap, Angular Material, and plain CSS/SCSS
- **Token system detection** — conforms to existing `:root` variables, SCSS `$variables`, or proposes one if absent
- **Severity-tiered output** — findings classified as Blocker, Warning, or Nit for actionable reviews

**Includes reference files:**

| File | Purpose |
|------|---------|
| [inline-styles.md](skills/css-code-quality%20/references/inline-styles.md) | Rules and examples for eliminating inline CSS |
| [sizing-units.md](skills/css-code-quality%20/references/sizing-units.md) | `rem` vs `px` guidance with documented exceptions |
| [colors-and-tokens.md](skills/css-code-quality%20/references/colors-and-tokens.md) | Hardcoded color detection and design token conventions |
| [specificity-and-important.md](skills/css-code-quality%20/references/specificity-and-important.md) | `!important` elimination and specificity management strategies |
| [additional-best-practices.md](skills/css-code-quality%20/references/additional-best-practices.md) | Magic numbers, logical properties, nesting depth, `gap`, `clamp()` |

---

### express-security-performance

Review, refactor, and harden Node.js + Express backends against best practices grounded in the official [Express security](https://expressjs.com/en/advanced/best-practice-security/) and [performance](https://expressjs.com/en/advanced/best-practice-performance/) guides. The skill operates in three separable modes:

1. **Review** — read the code and report findings (no edits), tiered by severity
2. **Refactor** — rewrite the code to fix findings, preserving behavior
3. **Suggest** — recommend patterns for code being written, with minimal examples

**Key capabilities:**

- **Context-aware** — adapts advice based on Express version (4 vs 5), deployment shape (reverse proxy, serverless, Kubernetes), and existing middleware
- **Security + performance unified** — findings labeled `[sec]` or `[perf]` in a single tiered list so both production risks are visible
- **Severity-tiered output** — findings classified as Blocker, Warning, or Nit for actionable reviews

**Includes reference files:**

| File | Purpose |
|------|---------|
| [security-headers-and-helmet.md](skills/express-security-performance/references/security-headers-and-helmet.md) | Helmet configuration, `X-Powered-By`, and security header best practices |
| [input-and-injection.md](skills/express-security-performance/references/input-and-injection.md) | Input validation, query/command/NoSQL injection prevention, payload limits |
| [auth-sessions-cookies.md](skills/express-security-performance/references/auth-sessions-cookies.md) | Session management, cookie flags, JWT, secrets, brute-force and rate limiting |
| [transport-and-config.md](skills/express-security-performance/references/transport-and-config.md) | TLS/HTTPS, `trust proxy`, CORS, env configuration, dependency hygiene |
| [error-handling-and-logging.md](skills/express-security-performance/references/error-handling-and-logging.md) | Preventing stack trace leaks, sync error handling, safe logging |
| [performance-runtime.md](skills/express-security-performance/references/performance-runtime.md) | Gzip compression, async patterns, avoiding heavy work in the Node event loop |
| [performance-deployment.md](skills/express-security-performance/references/performance-deployment.md) | `NODE_ENV=production`, clustering, process managers, caching, reverse proxy |

---

## Quick Start

<details>
<summary><b>Gemini CLI</b></summary>

Install as native skills for auto-discovery. See the [Gemini CLI skills docs](https://github.com/google-gemini/gemini-cli/blob/main/docs/skills.md).

**Install from the repo:**

```bash
gemini skills install https://github.com/HarishGangula/agent-skills.git --path skills
```

**Install from a local clone:**

```bash
gemini skills install ./agent-skills/skills/
```

</details>

<details>
<summary><b>Claude Code</b></summary>

**Local / development:**

```bash
git clone https://github.com/HarishGangula/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

</details>

<details>
<summary><b>Cursor</b></summary>

Copy the `SKILL.md` files into `.cursor/rules/`, or reference the full `skills/` directory in your Cursor rules configuration.

</details>

<details>
<summary><b>Windsurf</b></summary>

Add skill contents to your Windsurf rules configuration.

</details>

<details>
<summary><b>Kiro IDE & CLI</b></summary>

Skills for Kiro reside under `.kiro/skills/` and can be stored at the project or global level. Kiro also supports `Agents.md`. See [Kiro docs](https://kiro.dev/docs/skills/).

</details>

<details>
<summary><b>Other Agents</b></summary>

Skills are plain Markdown — they work with any agent that accepts system prompts or instruction files. Clone the repo and point your agent at the `skills/` directory.

</details>

---

## Repo Structure

```
agent-skills/
├── README.md
└── skills/
    ├── css-code-quality/
    │   ├── SKILL.md                          # Main skill definition
    │   └── references/
    │       ├── additional-best-practices.md      # Magic numbers, logical props, nesting, gap, clamp()
    │       ├── colors-and-tokens.md              # Design token conventions & hardcoded color rules
    │       ├── inline-styles.md                  # Inline CSS elimination rules
    │       ├── sizing-units.md                   # rem vs px guidance
    │       └── specificity-and-important.md      # !important elimination & specificity management
    ├── express-security-performance/
    │   ├── SKILL.md                          # Main skill definition
    │   └── references/
    │       ├── auth-sessions-cookies.md          # Sessions, cookies, JWT, rate limiting
    │       ├── error-handling-and-logging.md      # Stack trace prevention & safe logging
    │       ├── input-and-injection.md             # Input validation & injection prevention
    │       ├── performance-deployment.md          # NODE_ENV, clustering, caching, proxy
    │       ├── performance-runtime.md             # Compression, async patterns, event loop
    │       ├── security-headers-and-helmet.md     # Helmet & security headers
    │       └── transport-and-config.md            # TLS, trust proxy, CORS, env config
    └── sunbird-api-spec/
        ├── SKILL.md                          # Main skill definition
        └── references/
            ├── error-catalog.md                  # Standard error response examples
            ├── example-full-spec.md              # Complete worked API spec
            └── state-workflow.md                 # State machine & lifecycle endpoints
```

---

## Contributing

Want to add a new skill? Follow this structure:

1. Create a new directory under `skills/` with your skill name (lowercase, hyphenated)
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`, `license`, `metadata`)
3. Include a clear **When to Use** section with trigger phrases
4. Define a step-by-step **Workflow**
5. Add a **Verification Checklist** for quality gates
6. Place any supporting files in a `references/` subdirectory
7. Update this README to list your new skill in the table above

---

## License

MIT
