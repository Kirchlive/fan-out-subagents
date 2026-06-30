# fan-out-subagents

A standalone Claude Code plugin for **deliberate parallel subagent dispatch**. It makes Claude choose the *right* subagent type for each task before fanning work out — `fork` for coding, `Explore` for codebase search, `research` for external lookups — and it scales external research by tier.

It is a focused, independently maintained alternative to the `dispatching-parallel-agents` skill in [Superpowers](https://github.com/obra/superpowers). Install it on its own; it does not replace or override Superpowers.

---

## Why this exists

The common failure when dispatching parallel agents is **picking the wrong subagent type**. In current Claude Code:

- A **fork** (`subagent_type: fork`) inherits the *entire* conversation — the real files, decisions, and state of your session. It's the correct worker for coding/fixing/refactoring.
- An **isolated named subagent** (`general-purpose`, `Explore`, `research`) starts from a *fresh* context window and works off a delegation summary. Correct for self-contained exploration and research.
- **Omitting** the type does **not** produce a fork — it produces `general-purpose`, which will confidently code from an incomplete summary. This is the single most common, most expensive mistake.

Skills written before forks became generally available tend to bias every dispatch toward `general-purpose`, including coding. This plugin fixes that by forcing an explicit type decision *before* the dispatch, and by treating external research as a tiered fan-out.

## What's inside

```
fan-out-subagents/                          # marketplace root
├── .claude-plugin/marketplace.json
└── plugins/fan-out-subagents/
    ├── .claude-plugin/plugin.json
    ├── skills/fan-out-subagents/
    │   ├── SKILL.md                         # the orchestration skill
    │   └── references/research-tiers.md     # tiered research (loaded on demand)
    └── agents/
        ├── Research.md                       # Sonnet research worker (read-only)
        └── Debug-Explore.md                  # inherited-model root-cause investigator (read-only)
```

- **`fan-out-subagents` skill** — when to fan out, which subagent type per task, how to dispatch concurrently (fan-out → fan-in), and a pointer into the research tiers.
- **`research-tiers.md`** — read once when external research comes up; selects light / moderate / advanced and the reconcile strategy. Loaded only when needed (progressive disclosure), so it never costs context on a pure coding fan-out.
- **`Research` agent** — a read-only Sonnet worker for external research.
- **`Debug-Explore` agent** — a read-only investigator that runs on the *inherited main model* (not Haiku) for genuine root-cause debugging; returns a diagnosis, not a fix.

Both ship with the plugin; see *Worker naming* below.

## Requirements

- **Claude Code v2.1.117 or later** for forks. Claude-spawned forks roll out in stages; `CLAUDE_CODE_FORK_SUBAGENT=1` is only an explicit override and isn't required once the rollout is active in your session.
- `fork` and `Explore` are native Claude Code subagent types. The `Research` and `Debug-Explore` workers are provided by this plugin.

## Install

```text
/plugin marketplace add Kirchlive/fan-out-subagents
/plugin install fan-out-subagents@fan-out-subagents
```

The first argument is the plugin name; the second is the marketplace name (both are `fan-out-subagents` for this repo). To pin a tag or branch, append `@ref` to the marketplace `add` shorthand.

To try it locally without installing, clone the repo and start Claude Code with the plugin directory:

```text
claude --plugin-dir ./plugins/fan-out-subagents
```

## How it works

The skill triggers automatically (by its description) whenever you're about to dispatch 2+ independent tasks. It then walks a fixed order:

1. **Pick the type per task.** Coding → `fork`; locate in codebase → `Explore` (Haiku); root-cause a bug → `Debug-Explore` (inherited model, read-only); external research → `Research`; `general-purpose` only as a fallback for mixed tool work that is none of these.
2. **Fan out.** Issue all dispatches in one response so they run concurrently. Parallel forks that write files should use `isolation: "worktree"` to avoid clobbering each other.
3. **Fan in.** Review summaries, run tests, integrate. For advanced research, reconcile via a full-context `fork`.

### Research tiers

When external research comes up, the skill reads `references/research-tiers.md` once. Surfaces (repositories / state-of-the-art / documentation / context7 / web) describe **what** to find, not **where** — an agent ranges across sources to hit its goal.

| Tier | Buckets | Orchestration | Reconcile |
| :-- | :-- | :-- | :-- |
| light | 1–2 | none (single `Research` agent) | — |
| moderate | 2–4 | fan out `research` per surface | — |
| advanced | 5–6 | fan out, then reconcile | full-context `fork` |

### Worker naming

`fork` and `Explore` are native types and need no setup. The `Research` and `Debug-Explore` workers ship with this plugin and, when installed as a plugin, register under **scoped names** — `fan-out-subagents:Research` and `fan-out-subagents:Debug-Explore`. Use those as the `subagent_type`, or copy the agent files to `~/.claude/agents/` (user scope) to use the bare `Research` / `Debug-Explore` names. The skill's examples write the bare names for readability.

## Configuration notes

- The `Research` agent runs on **Sonnet** and is **read-only** (`disallowedTools: Write, Edit`). Everything else is inherited on purpose, so your ambient stack (e.g. codebase-memory, context-mode, the context7 plugin) stays available to it. It deliberately does **not** pin tools or MCP servers.
- The `Debug-Explore` agent runs on the **inherited main model** (`model: inherit`) and is **read-only**. It exists because `Explore` (Haiku) is too shallow for real debugging; it investigates and returns a diagnosis, and the fix is a separate `fork` dispatch.
- If you want research workers to be able to write files (e.g. drop a findings file), add `Write` back — but note that loosens the read-only guarantee; the intended pattern is that code/file changes are a separate `fork` dispatch.
- **Restricting what a worker may spawn:** an `Agent(typeA, typeB)` allowlist inside a subagent's `tools` field is ignored at runtime — it only enforces when an agent runs as the main thread via `claude --agent`. To stop a worker from spawning nested subagents (possible since Claude Code v2.1.172), use `disallowedTools: Agent` (as this plugin's workers do) or a `settings.json` permissions `deny`.

## Relationship to Superpowers

This plugin is **adapted from** the `dispatching-parallel-agents` skill in [Superpowers](https://github.com/obra/superpowers) (MIT, © Jesse Vincent). It was renamed to `fan-out-subagents` to avoid any collision and is shipped as an independent plugin so it can be installed, versioned, and updated without touching your Superpowers installation. It is not affiliated with or endorsed by the Superpowers project.

Key changes from the original: explicit subagent-type selection before dispatch, corrected fork mechanics (`subagent_type: fork`), worktree isolation guidance, and the tiered research workflow with a fork-based reconcile.

## Credits

- Original `dispatching-parallel-agents` skill: [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent (obra), MIT License.
- Adaptation and extensions: [Kirchlive](https://github.com/Kirchlive).

## License

MIT — see [LICENSE](./LICENSE). Includes the upstream Superpowers attribution.
