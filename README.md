# guard-skills

[![skills.sh](https://skills.sh/b/amElnagdy/guard-skills)](https://skills.sh/amElnagdy/guard-skills)

Focused **guard skills** for coding agents: second-pass quality gates that catch the systematic failure modes of AI-generated code, tests, and docs before they ship.

Best use: let your agent do the work, then invoke the relevant guard on the diff before you present, commit, or merge it. These skills can guide writing when you explicitly ask for that, but they are strongest as reactive review passes.

## Install

Browse the package first:

```bash
npx skills add amElnagdy/guard-skills --list
```

Install the package:

```bash
npx skills add amElnagdy/guard-skills
```

Or install one guard:

```bash
npx skills add amElnagdy/guard-skills --skill clean-code-guard
npx skills add amElnagdy/guard-skills --skill test-guard
npx skills add amElnagdy/guard-skills --skill docs-guard
npx skills add amElnagdy/guard-skills --skill wp-guard
npx skills add amElnagdy/guard-skills --skill woo-guard
```

Install for a specific agent:

```bash
npx skills add amElnagdy/guard-skills --skill clean-code-guard --agent codex
npx skills add amElnagdy/guard-skills --skill test-guard --agent claude-code
npx skills add amElnagdy/guard-skills --skill '*' --agent cursor
```

Install globally:

```bash
npx skills add amElnagdy/guard-skills --global
```

Works with Claude Code, Codex, Cursor, OpenCode, and other supported agents via the [Skills CLI](https://github.com/vercel-labs/skills).

## Updating

Skills install as a copy, so a new version here does not reach your agent until you update. Refresh them with the Skills CLI:

```bash
npx skills update                 # all installed skills (alias: upgrade)
npx skills update clean-code-guard
```

Add `--global` for global installs or `--project` for project installs. Re-running `npx skills add amElnagdy/guard-skills` also re-fetches the latest.

## How to use them

Run a guard after your agent produces work:

```text
Use $clean-code-guard on the diff you just produced.
Use $test-guard on the tests you just wrote.
Use $docs-guard on this README update before we ship it.
Use $wp-guard on this WordPress plugin change.
Use $woo-guard on this WooCommerce checkout change.
```

Use a guard up front only when you want that constraint active while writing:

```text
Use $wp-guard while implementing this REST endpoint, then self-check before delivery.
```

## Which guard to run

| Guard | Use after the agent changed | Catches | Pair with |
| --- | --- | --- | --- |
| `clean-code-guard` | Production code in any language | LLM code smells, over-abstraction, broad error swallowing, bad names, SOLID/DRY/KISS/YAGNI violations | Platform-specific guards |
| `test-guard` | Test code | Mock abuse, duplicate tests, implementation-detail assertions, tests that catch nothing | `clean-code-guard` when test helpers contain real logic |
| `docs-guard` | READMEs, docstrings, API docs, changelogs, tutorials | Hallucinated symbols, broken samples, docs-vs-code drift, unverifiable claims | Any guard whose behavior is documented |
| `wp-guard` | WordPress plugin, theme, block, REST, AJAX, shortcode, query, or WP-CLI code | Escaping, sanitization, nonces, capabilities, prepared queries, i18n, query/caching mistakes | `clean-code-guard`; `woo-guard` when Woo APIs appear |
| `woo-guard` | WooCommerce extension, checkout, order, product, gateway, shipping, or HPOS code | Direct order meta, HPOS breakage, missing compatibility declarations, checkout bypasses, money errors | `wp-guard` for the WordPress layer |

## The skills

### clean-code-guard

Applies Clean Code, SOLID, DRY/KISS/YAGNI to generated or changed code in any language, plus an AI-specific layer most rule packs miss: catch-all error swallowing, hardcoded "success" returns, hallucinated APIs, premature abstraction, comment pollution, and copy-from-similar bugs.

Why the AI layer matters: the skill references published research on duplication growth, package hallucination, and agents declaring success despite failed tests.

**You'll feel it when:** your agent refactors without silently changing behavior, asks before changing a contract, justifies what it deliberately left out, and stops wrapping everything in `try/catch -> return ok`.

### test-guard

A quality gate for generated or changed test code in any language: pytest, PHPUnit/Pest, Jest/Vitest, Go tests, WordPress/WooCommerce tests, and more. Nine universal rules catch AI test bloat: mock only at system boundaries, never mock your own state objects, parametrize instead of copy-pasting, delete tests that catch nothing, and treat production regression tests as sacred.

Framework specifics live in progressive-disclosure references the agent loads only when relevant, including a dedicated reference for LLM applications.

**You'll feel it when:** a generated test file with `MagicMock()` state, duplicated test bodies, and log-message assertions comes back as "do not merge" with rule-by-rule fixes.

### docs-guard

A documentation accuracy gate for READMEs, API references, docstrings, PHPDoc/JSDoc, changelogs, and tutorials. Its core move: treat documentation as a list of claims and verify every one against the codebase.

**You'll feel it when:** a generated README stops referencing functions that do not exist, `@param` tags match the real signature, samples run on a clean machine, and "blazingly fast" leaves the building.

### wp-guard

A WordPress shipping guard for generated or changed plugins, themes, blocks, REST endpoints, AJAX handlers, shortcodes, WP-CLI commands, and queries. It enforces the layer generic clean-code guidance misses: escaping and sanitization, nonce plus capability checks, prepared database queries, core APIs before custom plumbing, translation-ready strings, and query/caching discipline.

**You'll feel it when:** a generated plugin stops echoing raw request data, a writing REST route gets a real permission callback, every string ships translation-ready, and a million-row site does not get handed `posts_per_page => -1`.

### woo-guard

A WooCommerce shipping guard for generated or changed extensions, payment and shipping integrations, checkout customizations, and order/product logic. It sits on top of `wp-guard` and enforces what is WooCommerce-specific: HPOS-safe order access, CRUD over direct meta, truthful feature-compatibility declarations, server-side checkout validation, money-handling discipline, and hooks over template overrides.

**You'll feel it when:** order code survives HPOS, stock updates stop racing, checkout rules hold without JavaScript, and `'$' . $amount` never reaches a store that sells in dirhams.

## How this differs

This is not a full process framework and not a broad platform catalog. Repos like [WordPress/agent-skills](https://github.com/WordPress/agent-skills) teach agents how to build across the WordPress ecosystem. `guard-skills` is narrower: it gives agents review gates to run after they have produced work, so common AI failure modes get caught before the work reaches your repo.

## Repository shape

Each skill is a folder with a `SKILL.md` entrypoint, lightweight agent metadata, and progressive-disclosure references:

```text
skills/
├── clean-code-guard/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   └── references/
├── docs-guard/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   └── references/
├── test-guard/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   └── references/
├── woo-guard/
│   ├── SKILL.md
│   ├── agents/openai.yaml
│   └── references/
└── wp-guard/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
```

The `SKILL.md` stays small so it loads cheaply; deeper guidance loads only when the task needs it.

## Trust and validation

This package is intentionally inspectable:

- Skill content is Markdown plus lightweight `agents/openai.yaml` metadata.
- There are no executable scripts, network calls, MCP server dependencies, or credentials.
- External source URLs live in each skill's `references/sources.md`.

Maintainer checks before publishing:

```bash
npx skills add . --list --full-depth
```

This lists every skill the CLI discovers by scanning `skills/`, with its references, so you can confirm structure and discovery before publishing.

## License

MIT — see [LICENSE](LICENSE).
