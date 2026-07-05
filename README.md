# scrum-master

[![Plugin Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](https://github.com/TheMizeGuy/scrum-master-public/releases)

Kanban board manager for [Claude Code](https://claude.com/claude-code). Creates stories from plans, manages per-story markdown files, generates [Obsidian Kanban](https://github.com/mgmeyers/obsidian-kanban) board views, maps dependencies, audits backlogs, reconciles board state with git, and plans parallelizable dispatch waves.

The plugin is methodology-strict: Kanban (pull systems, WIP limits, cycle time, classes of service, Little's Law). Despite the name, it does NOT run Scrum ceremonies (sprints, velocity, burn-down). The name is a convenient handle.

## Installation

```bash
# 1. Add this repo as a marketplace
claude plugin marketplace add https://github.com/TheMizeGuy/scrum-master-public.git

# 2. Install the plugin
claude plugin install scrum-master@scrum-master-public

# 3. Restart Claude Code for the plugin to load
```

After restart, verify with `claude plugin list`. Updates ship through the same channel: when a new release lands, run `claude plugin marketplace update scrum-master-public` then `claude plugin update scrum-master@scrum-master-public`, or accept the update prompt in `/plugin`.

Manual alternative: `git clone https://github.com/TheMizeGuy/scrum-master-public.git` and load with `claude --plugin-dir <path>`.

## Install

Install via your Claude Code plugin marketplace, or place the plugin folder at `~/.claude/plugins/scrum-master/` (or any directory listed in your marketplace config).

If you cloned this repo directly, point your marketplace at the local directory.

## Quickstart

1. **Install** the plugin (above).
2. **First invocation** — run `/scrum-master` (no argument) or say "have the scrum master look at the backlog" in a Claude Code session inside your project. With no story files yet, the agent proposes a board path and ID prefix and asks you to confirm.
3. **Confirm the scaffold.** The agent creates the story directory, an empty `board.md`, and `.claude/scrum-master.local.md` with your confirmed config.
4. **Create your first stories** — after writing or reviewing an implementation plan, say "have the scrum master create stories from this plan" (or run `/scrum-master create-stories`). The agent reads the plan, proposes a table of candidate stories, and writes the approved ones as per-story markdown files.
5. **Check status any time** with `/scrum-master status` or "what's next in the pipeline" — the agent scans the board and reports counts, aging work-in-progress, and blocked items as tables.

What to expect: every response is table-first, not prose. The agent never edits `board.md` by hand — it is always regenerated from the story files, which remain the single source of truth.

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

## Walkthrough — creating stories from a plan

**Query:** you finish writing an implementation plan in the conversation, then say "have the scrum master map out stories from this."

**What happens:**

1. The agent reads the plan you just wrote from conversation context (no need to re-paste it).
2. It dispatches a read-only board scanner to see existing stories (avoids duplicates) and a codebase scanner to resolve file:line anchors for symbols the plan mentions.
3. It decomposes the plan into vertical slices — each story delivers end-to-end observable value, sized to at most 5 acceptance criteria and 10 scope files. Oversized or horizontal candidates get split before you see them.
4. Each candidate's acceptance criteria are written as mechanically verifiable assertions (test names, grep patterns, exit codes) — never prose like "works correctly."
5. It runs the [Definition-of-Ready checklist](#definition-of-ready) on every candidate; only stories that pass all six checks are proposed at `state: Ready`, the rest land in `Backlog` with the failing check noted.
6. It dedupes each candidate against the existing board by title similarity and scope overlap, flagging any match.

**Output shape:** a table for your approval —

```
| # | ID | Title | Epic | Effort | Deps | AC count | Dedupe flag |
```

On approval, the agent writes one markdown file per story and regenerates `board.md`. If more than 3 stories have cross-dependencies, it also writes a Mermaid dependency graph.

## Definition of Ready

A story may hold `state: Ready` — and join a dispatch wave — only if it passes all six checks: vertical slice, sized (≤5 AC, ≤10 scope files), AC are assertions, scope resolves to real files, unblocked, and identifiable (unique ID, priority, effort set). The checklist runs automatically during `create`, `validate`, and `waves`; any failing story stays in `Backlog` with the failing check number recorded in its body. Full schema and a worked example are in the agent system prompt at `agents/scrum-master.md`.

## Architecture

| Component | Role |
|---|---|
| **Agent** | Runs on the session model (always the strongest available Claude). All decisions and file writes |
| **Scout runners** | Read-only reconnaissance (board scanning, git reconciliation, AC verification), dispatched without a model override so they inherit the same session model |
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

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Agent says "No story files found" on first use | No board scaffolded yet | Run `/scrum-master` with no argument to trigger first-run scaffolding, or set `board_path` explicitly in `.claude/scrum-master.local.md` |
| Story stuck in `Backlog`, never reaches `Ready` | Fails one of the six Definition-of-Ready checks | Run `/scrum-master validate` — it reports the failing check number per story |
| `board.md` looks stale or hand-edited changes vanished | Board view is always regenerated from story files, never the master | Edit the story file's frontmatter/body instead, then run `/scrum-master generate-board` |
| Dependency graph won't generate | A dependency cycle exists (A blocks B blocks A) | Run `/scrum-master deps` — it reports the exact cycle and suggests which edge to remove |
| Story marked Done but `update` keeps flagging it | `evidence.commit` is empty, or `git log --stat` shows zero code changes ("art-only false-done") | Backfill `evidence.commit` from the actual merge commit, or move the story back to `In Progress` if the work isn't real |
| GoodMem / serena / obsidian steps seem skipped | Optional integration not configured or its MCP server isn't installed | This is expected — the plugin degrades gracefully. Configure `goodmem_learnings_space` etc. in `.claude/scrum-master.local.md` to enable |
| Board scanner (or another scout) returns 0 rows unexpectedly | Scout output failed the acceptance gate, or genuinely nothing matched | The agent double-checks with its own Glob/Grep before trusting a zero-result scout; if it still reports zero, the board path is probably empty or misconfigured |

## Key rules

1. The main agent owns all writes. Scout runners are read-only.
2. Stories are the source of truth. Board view is always generated.
3. Done requires evidence (commit hash).
4. AC must be mechanically verifiable assertions.
5. Dedupe before creating new stories.
6. Vertical slices only. Size guard: AC > 5 or files > 10 = split.
7. Ready + dispatch waves require passing the [Definition-of-Ready](#definition-of-ready) checklist.

## License

MIT — see [LICENSE](LICENSE).

## Author

Created by [TheMizeGuy](https://github.com/TheMizeGuy).
