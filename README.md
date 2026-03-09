# Claude Code Skills for PocketBase and Colab

> Teach your AI assistant domain-specific workflows for PocketBase and Colab.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PocketBase](https://img.shields.io/badge/PocketBase-0.25+-blueviolet)](https://pocketbase.io)
[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-green)](https://docs.anthropic.com/en/docs/claude-code)

## What This Is

[Claude Code Skills](https://docs.anthropic.com/en/docs/claude-code/skills) are markdown instruction files that auto-activate when your AI assistant encounters relevant tasks. This repository is now a multi-skill collection, not a PocketBase-only package: it houses skill packs for PocketBase work and Colab-related workflows under the same repo.

The currently documented skills cover PocketBase end-to-end — from schema design to production deployment — so Claude writes correct PocketBase code on the first try.

Think of them as expert knowledge, distilled into rules that fire at the right moment.

PocketBase is the first fully documented skill set here, and Colab skills live alongside it as part of the broader repository scope.

## The Problem

Without these skills, AI assistants consistently make the same PocketBase mistakes:

- **Using `=` instead of `?=`** for multi-relation filter fields — silently matches nothing
- **Writing ES6+ syntax** (`=>`, `import`) in pb_hooks — goja only supports ES5
- **Setting API rules to `""`** (empty string) thinking it means "locked" — it means **everyone has access**
- **Missing `proxy_buffering off`** in nginx config — breaks SSE realtime subscriptions
- **Wrong `maxSelect` semantics**: `1` returns a string, `0`/`>1` returns an array
- **Creating nullable fields** — only `json` type can be null in PocketBase
- **Forgetting `e.next()`** in Execute hooks — silently skips the actual action
- **Using `$app.findRecordById` in loops** instead of batch queries — N+1 performance disaster
- **Hardcoding `127.0.0.1`** in Docker — PocketBase needs `0.0.0.0` to bind correctly

These 6 skills prevent all of the above and many more.

## PocketBase Skills Included

| Skill | Lines | What It Covers | Example Triggers |
|-------|------:|----------------|------------------|
| **pb-collections** | 125 | Schema design, field types, relations, collection types | "create a posts collection", "add a relation field" |
| **pb-api-rules** | 208 | Access control, filter expressions, rule debugging | "set permissions", "only owner can edit", "why am I getting 403" |
| **pb-hooks** | 597 | Server-side JS, custom routes, event hooks, cron, email | "add a hook", "custom endpoint", "send email on signup" |
| **pb-migrations** | 258 | Schema versioning, automigrate, snapshot import | "create a migration", "sync schema between environments" |
| **pb-sdk** | 432 | JS SDK, auth, realtime, file uploads, batch ops | "authenticate user", "subscribe to changes", "upload a file" |
| **pb-deploy** | 363 | Docker, systemd, nginx/Caddy, TLS, S3, backups | "deploy PocketBase", "set up nginx", "configure backups" |

**Total: 1,983 lines** of battle-tested PocketBase knowledge.

This repo is no longer limited to PocketBase skills; it also serves as the home for Colab skills and other skill packs as they are added.

## Installation

### One-liner

```bash
git clone https://github.com/bolkhovsky/pocketbase-skills.git
cd pocketbase-skills && ./install.sh
```

### Manual

Copy the skill directories into your Claude Code skills folder:

```bash
cp -r skills/pb-* ~/.claude/skills/
```

### Uninstall

```bash
./install.sh --uninstall
```

## Usage

Skills auto-activate — just talk to Claude Code naturally:

```
You: "Create a posts collection with title, body, and author relation"
     → pb-collections activates: correct field types, maxSelect, cascade rules

You: "Only the author should be able to edit their posts"
     → pb-api-rules activates: proper filter expression with @request.auth.id

You: "Add a hook that sends a welcome email when a user signs up"
     → pb-hooks activates: ES5 syntax, e.next(), correct event binding

You: "Deploy this to production with Docker and nginx"
     → pb-deploy activates: Dockerfile, nginx config with SSE support, systemd unit
```

## Skill Details

<details>
<summary><strong>pb-collections</strong> — Schema & collection design</summary>

Covers the three collection types (base, auth, view), all field types with their zero-value defaults, relation setup (`maxSelect` semantics), and cascading behavior. Prevents nullable field mistakes and incorrect relation cardinality.

**Sections:** Collection Types · Field Types & Defaults · Relations · Indexes · View Collections

</details>

<details>
<summary><strong>pb-api-rules</strong> — Access control & filter expressions</summary>

Covers all 5 rule types (list/view/create/update/delete), the critical difference between `null` (locked) and `""` (everyone), filter syntax with operators, `@request.*` macros, field modifiers (`:each`, `:length`), and multi-relation `?=` gotcha.

**Sections:** Rule Types · Filter Operators · Request Macros · Collection Macros · Field Modifiers · Common Patterns

</details>

<details>
<summary><strong>pb-hooks</strong> — Server-side JavaScript reference</summary>

The largest skill. Covers the goja ES5 runtime constraints, route registration, middleware, all event hooks (Record/Auth/Realtime/Cron/Mail), database queries, record CRUD, file handling, and the critical `e.next()` pattern for Execute hooks.

**Sections:** Runtime Basics · Routing · Middleware · Event Hooks · DB Queries · Record Operations · File Handling · Cron · Email · HTTP Client

</details>

<details>
<summary><strong>pb-migrations</strong> — Schema versioning</summary>

Covers both automigrate and manual migration workflows, the migration file format, snapshot import for bootstrapping, the `_migrations` tracking table, and environment sync strategies.

**Sections:** CLI Commands · Automigrate · Manual Migrations · Snapshot Import · Environment Sync

</details>

<details>
<summary><strong>pb-sdk</strong> — JavaScript SDK usage</summary>

Covers initialization, CRUD operations, auth flows (email/password, OAuth2, OTP), the `authStore` lifecycle, realtime SSE subscriptions, file uploads/downloads, batch operations, and query syntax (`filter`, `sort`, `expand`, `fields`).

**Sections:** Setup · CRUD · Authentication · AuthStore · Realtime · Files · Batch · Query Syntax

</details>

<details>
<summary><strong>pb-deploy</strong> — Production deployment</summary>

Covers single-binary deployment, Docker setup (with correct `0.0.0.0` binding), systemd unit files, nginx and Caddy reverse proxy configs (with `proxy_buffering off` for SSE), TLS, SMTP configuration, S3 storage, automated backups, and security hardening.

**Sections:** Binary Deploy · Docker · Systemd · Nginx · Caddy · TLS · SMTP · S3 · Backups · Hardening

</details>

## PocketBase Version

These skills are written for **PocketBase v0.25+** — the major rewrite that landed in the v0.23→0.25 cycle. Key changes from older versions include the new `$app` API surface, updated hook signatures, and the revised auth system. If you're on an older PocketBase version, some patterns may not apply.

## Contributing

**Improve existing skills:** Edit the relevant skill under `skills/`, test with Claude Code by having a conversation that triggers it, and submit a PR.

**Add new skills:** Create `skills/<skill-family>-<name>/SKILL.md` with YAML frontmatter:

```yaml
---
name: "<Skill Topic>"
description: "<When to activate this skill — be specific about trigger conditions>"
---
```

**Upstream:** These skills are compatible with the [claude-code-templates](https://github.com/davila7/claude-code-templates) ecosystem. Contributions welcome there too.

Issues and PRs are welcome.

## License

[MIT](LICENSE)
