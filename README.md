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
    └── sunbird-api-spec/
        ├── SKILL.md              # Main skill definition
        └── references/
            ├── error-catalog.md      # Standard error response examples
            ├── example-full-spec.md  # Complete worked API spec
            └── state-workflow.md     # State machine & lifecycle endpoints
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
