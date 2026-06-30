# Changelog

All notable changes to this plugin are documented here.
This project adheres to [Semantic Versioning](https://semver.org/).

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
