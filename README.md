# claude-brain

A Claude Code skill that installs an **LLM Wiki** (Carpati/Obsidian-style knowledge base) in any project. One command and Claude sets up everything: an interconnected markdown wiki with session nodes, a dense index, an append-only log, and a slash command (`/brain`) for ingest, query, and lint operations.

## What you get

After running the skill, your project has:

```
brain/                        ← wiki directory (name is configurable)
├── CLAUDE.md                 ← operational schema (rules for Claude)
├── index.md                  ← dense node catalog (LLM cursor)
├── log.md                    ← append-only operations bitácora
├── sources.md                ← external sources registry
└── sessions/                 ← one markdown file per session
.claude/commands/brain.md     ← /brain slash command
.vscode/settings.json         ← Foam wikilinks configuration
```

The wiki is maintained by Claude through three operations:

| Command | What it does |
|---------|-------------|
| `/brain ingest` | Creates a session node from the current conversation. Extracts decisions, outputs, cross-refs, and pending items. Updates the index and log automatically. |
| `/brain query <question>` | Answers using the wiki as source, with `[[wikilink]]` citations. Fetches fresh content from external sources when needed. |
| `/brain lint` | Health check: broken wikilinks, orphan nodes, stale claims, missing cross-refs, emerging concept candidates. Read-only. |

## How to install

### Option 1 — Direct copy (recommended)

Copy the skill to your project or global Claude Code skills directory:

```bash
# For a specific project
git clone https://github.com/AgustinGoniDev/claude-brain .agents/skills/claude-brain

# Or globally (available in all projects)
git clone https://github.com/AgustinGoniDev/claude-brain ~/.claude/skills/claude-brain
```

Then open Claude Code and run:

```
/claude-brain
```

Claude will ask you a few questions and generate everything.

### Option 2 — Manual copy

1. Download or clone this repo
2. Copy the contents to `.agents/skills/claude-brain/` inside your project (or `~/.claude/skills/claude-brain/` for global use)
3. Run `/claude-brain` in Claude Code

## Setup questions

When you run `/claude-brain`, Claude will ask:

1. **Wiki directory name** — what to call the folder (default: `brain`, or use `cerebro`, `wiki`, etc.)
2. **Slash command name** — what to call the command (default: same as directory name)
3. **Project name** — used in the command description
4. **Content language** — Spanish, English, or custom (affects section names in nodes)
5. **Project areas** — your team's areas (e.g. `marketing, ops, dev`). These become the `area` frontmatter enum and index sections.
6. **Source of truth backend**:
   - **Notion** — wiki stores short IDs; Claude fetches fresh content via MCP when needed
   - **Local files** — source of truth lives in the repo; Claude reads files directly
   - **Other tool** — Claude adapts instructions for your tool (Confluence, Linear, Airtable, etc.)
   - **None** — standalone wiki, no external source
7. **Legacy sessions** — whether you have existing session logs to migrate

## How it works

Each session node is a markdown file with YAML frontmatter:

```yaml
---
type: session
area: marketing
date: 2026-04-10
slug: email-infrastructure-setup
title: "Email capture infrastructure: Brevo + n8n + Nginx"
tags: [email, lead-magnet, brevo, n8n, nginx]
status: active
related:
  - 2026-04-04-cold-email-sequence
sources:
  - notion:email-strategy
  - repo:.claude/plans/email-infra.md
superseded_by: null
---
```

And a body with six fixed sections:

```markdown
# Title

## Context
## Decisions
## Output
## Pending
## Cross-refs
- [[2026-04-04-cold-email-sequence]] — same date, complementary piece
## Sources
- [[sources#email-strategy]] (Notion)
```

Internal links use Foam wikilink syntax (`[[slug]]`), which the [Foam VSCode extension](https://foambubble.github.io/foam/) renders as a navigable graph.

## VSCode extension

Install [Foam](https://marketplace.visualstudio.com/items?itemName=foam.foam-vscode) to get:
- Clickable `[[wikilinks]]` in your editor
- Graph visualization of the knowledge base
- Backlinks panel
- Link autocompletion

The skill configures `.vscode/settings.json` automatically to exclude legacy session folders from the Foam graph (no duplicate nodes).

## Pattern

This skill implements the **LLM Wiki / Carpati pattern**: a persistent, self-compiling knowledge base where the LLM is both the writer and the reader. The key insight is that Claude reads the wiki index first on every operation, making retrieval O(index) not O(all-files). Cross-references are bidirectional and enforced on ingest.

The three-layer architecture:
1. **Raw sources** — Notion pages, repo files (immutable, fetched fresh)
2. **Wiki** — `brain/` — session nodes with frontmatter, dense index, cross-refs
3. **Schema** — `brain/CLAUDE.md` — the operational manual Claude reads before every operation

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- [Foam VSCode extension](https://foambubble.github.io/foam/) (optional but recommended)
- If using Notion backend: Notion MCP configured in your `.mcp.json`

## License

MIT
