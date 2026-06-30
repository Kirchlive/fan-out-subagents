# Changelog

All notable changes to this plugin are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

## [1.0.1] - 2026-06-30

### Changed
- Read-only workers (`Research`, `Debug-Explore`) now also disallow the `Agent` tool,
  keeping them as clean leaves now that Claude Code v2.1.172+ lets subagents spawn
  their own subagents (depth limit 5, fixed). Restricting spawning via `Agent(...)`
  in `tools` is ignored at runtime — `disallowedTools: Agent` is the correct lever.

### Added
- Nesting note (v2.1.172+) in `SKILL.md`: orchestration stays in the main session
  (flat fan-out); nesting is opt-in per surface, not the default.
- Staggered-launch guidance: for external / rate-limited surfaces (web, docs,
  context7/MCP), launch in batches of 2–3 across consecutive turns instead of one
  simultaneous burst; retry failed surfaces sequentially. `[1302]` rate-limits are
  transient infrastructure, not a skill error.
- `context7` surface scoped to its lane: include it only for library/framework topics
  with versioned docs; drop it for product/feature/tool-behavior topics.
- README "Configuration notes": documents the `Agent()` allowlist gotcha
  (ignored at runtime; enforced only via `claude --agent`).

## [1.0.0] - 2026-06-30

### Added
- Initial standalone release of the `fan-out-subagents` skill, adapted and
  extended from the `dispatching-parallel-agents` skill in Superpowers.
- Explicit **subagent-type selection step** before any dispatch: coding → `fork`
  (full context), codebase exploration → `Explore`, external research → `research`.
- Corrected fork mechanics for current Claude Code: forks are requested
  explicitly via `subagent_type: fork` (omitting the type yields `general-purpose`).
- `isolation: "worktree"` guidance for parallel forks that edit files concurrently.
- **Tiered research workflow** (`references/research-tiers.md`): light (no
  orchestration), moderate (fan-out), advanced (fan-out + full-context fork
  reconcile), with source surfaces treated as lenses (what to find), not locations.
- `Research` worker subagent (Sonnet, read-only) shipped with the plugin.
- `Debug-Explore` worker subagent (inherited main model, read-only) for genuine
  root-cause debugging, distinct from the Haiku-based native `Explore`. Returns a
  diagnosis; the fix is a separate `fork` dispatch.
- Subagent types use capitalized names (`Research`, `Debug-Explore`) to match the
  native `Explore` convention.
