# Changelog

All notable changes to this repository will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-07-05

### Added

- **Definition-of-Ready checklist** — a story may only reach `state: Ready` (or join a dispatch wave) after passing six mechanical checks: vertical slice, sized (≤5 AC, ≤10 scope files), AC are assertions, scope resolves to real files, unblocked, and identifiable. Enforced automatically in `create`, `validate`, and `waves`; failing stories stay in `Backlog` with the failing check number recorded.
- **Worked story example** in the agent system prompt showing a story file that passes all six checks, plus a counter-example that fails on sight (horizontal slice, prose AC).
- **Scout output acceptance gate** — before trusting any scout's table, the agent verifies table shape, row-count parity against its own Glob/Grep, and the zero-result rule; a failed gate triggers one re-dispatch, then a fallback to inline recon.
- **Board self-verification** — `generate-board` now runs five structural checks (card-per-story parity, checkbox semantics, column/state agreement, file structure) before reporting success, and reports the failing check verbatim after 2 failed regeneration attempts instead of shipping a broken board.
- README: Quickstart, a worked Walkthrough (plan → stories), Definition-of-Ready section, and a Troubleshooting table.
- Plugin `author.email` (`ben@meipath.com`) alongside `author.name`.

### Changed

- **Model policy**: agent frontmatter no longer pins `model: opus`. The agent and its scout runners inherit the session model — always the strongest available Claude — instead of a hardcoded Opus generation. All "Opus" references in prose ("Opus scout protocol", "Dispatch Opus board scanner", etc.) replaced with generic "session model" language throughout the agent prompt, skill, README, `plugin.json`, and `marketplace.json`.
- `SKILL.md` gained an explicit "Execution mode" note: scouts inherit the session model with no separate fallback tier; small recon may run inline instead of via scout dispatch when the session model is already the strongest available.
- Stale reference to the disabled `feature-dev` plugin replaced with "the dev plugin" in the skill description.
- `plugin.json` / `marketplace.json` author and owner fields corrected to `TheMizeGuy` (previously the bare handle `mize`); `LICENSE` copyright holder corrected to match.

## 2026-04-16

- Marketplace/README description text bumped `Opus 4.6` → `Opus 4.7` (cosmetic; no plugin functionality change).

## 2026-03-02

- Upgraded all scout runners from `model: sonnet` to `model: opus`.
- Restructured the repository as a marketplace: plugin moved under `scrum-master/` with a root `.claude-plugin/marketplace.json`.

## [0.1.0] - 2026-02-14

### Added

- Initial public release: `scrum-master` agent and skill, Kanban story-per-file source of truth, Obsidian Kanban board generation, Mermaid dependency DAGs, dispatch-wave planning, flow metrics, and session-scoped stop-hook generation.

[0.2.0]: https://github.com/TheMizeGuy/scrum-master-public/releases/tag/v0.2.0
[0.1.0]: https://github.com/TheMizeGuy/scrum-master-public/releases/tag/v0.1.0
