# fan-out-subagents (plugin)

Deliberate parallel subagent dispatch for Claude Code: pick the right subagent type per task (`fork` for code, `Explore`/`Debug-Explore` for codebase, `Research` for external), fan out concurrently, and reconcile. Includes a tiered research workflow.

This is the plugin itself. For installation, usage, and the full write-up, see the [repository README](../../README.md).

## Components

- `skills/fan-out-subagents/SKILL.md` — the orchestration skill.
- `skills/fan-out-subagents/references/research-tiers.md` — tiered research, loaded on demand.
- `agents/Research.md` — read-only Sonnet research worker (registers as `fan-out-subagents:Research`).
- `agents/Debug-Explore.md` — read-only root-cause investigator on the inherited main model (registers as `fan-out-subagents:Debug-Explore`).

Adapted from the `dispatching-parallel-agents` skill in [Superpowers](https://github.com/obra/superpowers) (MIT, © Jesse Vincent). MIT licensed.
