# scrum-master

Opus 4.6 kanban board manager for [Claude Code](https://claude.com/claude-code). Creates stories from plans, manages per-story markdown files, generates [Obsidian Kanban](https://github.com/mgmeyers/obsidian-kanban) board views, maps dependencies, audits backlogs, reconciles board state with git, and plans parallelizable dispatch waves.

The plugin is methodology-strict: Kanban (pull systems, WIP limits, cycle time, classes of service, Little's Law). Despite the name, it does NOT run Scrum ceremonies (sprints, velocity, burn-down). The name is a convenient handle.

## Install

Install via your Claude Code plugin marketplace, or place the plugin folder at `~/.claude/plugins/scrum-master/` (or any directory listed in your marketplace config).

If you cloned this repo directly, point your marketplace at the local directory.

## Usage

### Slash commands

| Command | What it does |
|---|---|
| `/scrum-master` | Interactive — agent reads board, asks what to do |
| `/scrum-master create-stories` | Decompose a plan/spec into stories with dependencies and AC |
| `/scrum-master status` | Summarize board state, aging WIP, blocked items, pipeline |
| `/scrum-master audit` | Find stale stories, missing evidence, schema violations |
| `/scrum-master update` | Reconcile board vs git history, backfill evidence |
| `/scrum-master deps` | Build and render Mermaid dependency graph |
| `/scrum-master plan-waves` | Group Ready stories into parallelizable dispatch waves |
| `/scrum-master generate-board` | Regenerate Obsidian Kanban board view from story files |
| `/scrum-master retro` | Cycle time, throughput, percentile analysis |
| `/scrum-master validate` | Check all stories against schema |
| `/scrum-master prioritize` | Suggest WSJF-style priority ordering |
| `/scrum-master generate-hook` | Write session-scoped stop-hook for board validation |

### Natural language

Works with conversation context:

- "have the scrum master map out stories" (after writing a plan)
- "scrum master check the backlog for outdated stories"
- "what's next in our pipeline"
- "update the board with what we just shipped"

## Architecture

| Component | Role |
|---|---|
| **Opus 4.6 agent** | All decisions and file writes |
| **Sonnet scout runners** | Read-only reconnaissance (board scanning, git reconciliation, AC verification) |
| **Per-story markdown files** | Source of truth (YAML frontmatter with structured fields) |
| **Generated Obsidian Kanban board** | Human-readable view, always derived from story files |

## Configuration

Auto-detects project conventions. Optional explicit config at `.claude/scrum-master.local.md`:

```yaml
---
# Required if you want to override auto-detection
board_path: vault/Backlog/current
board_view_path: vault/Backlog/board.md

# ID format
id_prefix: PROJ
id_format: "<PREFIX>-<EPIC>-<SEQ>"
epic_list: [GOV, NET, AUTH, READ]

# Board columns
columns: [Backlog, Ready, "In Progress", "In Review", Done, Blocked]
wip_limits:
  "In Progress": 3
  "In Review": 2

# OPTIONAL: GoodMem integration (only if you have a GoodMem instance)
# When configured, the agent searches before operations and writes learnings after.
# When not configured (or GoodMem MCP not installed), these steps are skipped silently.
goodmem_learnings_space: "<your-goodmem-learnings-space-uuid>"
goodmem_project_space:   "<your-goodmem-project-space-uuid>"
goodmem_reranker_id:     "<your-goodmem-reranker-uuid>"

# OPTIONAL: Local Kanban methodology knowledge base
# When configured, the agent reads from this directory for methodology references
# and cites specific files when making recommendations.
# When not configured, the agent relies on methodology baked into its prompt.
kanban_knowledge_path: "/path/to/your/kanban/notes"
---
```

Everything except `board_path` and `board_view_path` is optional. Auto-detection covers most cases.

## Story file format

Each story is a markdown file with YAML frontmatter:

```yaml
---
id: PROJ-NET-01
title: Implement APIClient
state: Ready
priority: P1
effort: M
acceptance:
  - "file src/Networking/APIClient.swift exists"
  - "xcodebuild test -only-testing:NetworkingTests returns exit 0"
dependencies:
  blocks: [PROJ-READ-01]
  blocked-by: [PROJ-GOV-01]
evidence:
  commit: ""
---
```

Full schema and required-fields-by-state matrix are documented in the agent system prompt at `agents/scrum-master.md`.

## Optional MCP integrations

The plugin works standalone. These integrations add capability when present but are never required:

| MCP server | What it adds |
|---|---|
| [goodmem](https://github.com/anthropics/goodmem) | Cross-project semantic memory for prior decisions and learnings |
| [serena](https://github.com/oraios/serena) | Symbol-level code navigation for create/waves modes |
| [obsidian](https://github.com/MarkusPfundstein/mcp-obsidian) | Vault search and note reading when board lives in an Obsidian vault |

If a configured MCP integration is missing at runtime, the agent skips those steps silently.

## Key rules

1. Opus owns all writes. Sonnet scouts are read-only.
2. Stories are the source of truth. Board view is always generated.
3. Done requires evidence (commit hash).
4. AC must be mechanically verifiable assertions.
5. Dedupe before creating new stories.
6. Vertical slices only. Size guard: AC > 5 or files > 10 = split.

## License

MIT — see [LICENSE](LICENSE).

## Author

Created by [mize](https://github.com/TheMizeGuy).
