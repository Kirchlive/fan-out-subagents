# Research Tiers

Read this once, the first time external research comes up in a session. It selects how much
to fan out and whether to reconcile. It is the orchestration layer for research; the worker
behavior lives in the `Research` (Sonnet) subagent, and the advanced reconcile is a `fork`.

## Source surfaces (lenses, not locations)

Surfaces describe **WHAT** to find, not **WHERE** to find it. Each is a lens on the question,
not a fenced-off place. A surface agent uses whatever sources best answer its goal and must
not confine itself to one location — surfaces overlap, and the same topic legitimately shows
up across several of them. Treat the "what you're after" column as a hint, never a rule.

| Surface (lens) | What you're after |
| :-- | :-- |
| **repositories** | how real projects actually implement, use, or diverge on the topic — code, issues, PRs, and any docs that live in repos |
| **sota** | what the current best/leading approach is — may surface from papers, preprints, or benchmarks, OR from comparing what leading repositories do |
| **documentation** | authoritative usage — official docs, guides, API references, wherever they live (doc sites, READMEs, in-repo docs) |
| **context7** | up-to-date, version-pinned library/framework knowledge (applied via the context7 plugin) |
| **web** | practitioner reality — blogs, discussions, caveats, gotchas |

Do not prescribe tools or locations when dispatching a surface agent. State the goal; let the
agent range. E.g. a `sota` agent may end up reading repos; a `documentation` agent may find
the real answer in a GitHub README. That's intended.

## Tiers

| Tier | Buckets | Surfaces used | Fan-out | Reconcile |
| :-- | :-- | :-- | :-- | :-- |
| **light** | 1–2 | repositories / context7 / web | **No** | No |
| **moderate** | 2–4 | repositories / sota / documentation / context7 / web | Yes | No |
| **advanced** | 5–6 | 2× repositories / sota / documentation / context7 / web | Yes | **Fork** (full context) |

### light — no orchestration

A bounded, one- or two-surface lookup. **Do not orchestrate.** Dispatch one `Research` agent
directly and stop; the main session reads its summary. This tier is not a parallel dispatch
and does not belong to this skill's fan-out machinery. Use it for "what's the current
recommended config for X", "does repo Y expose Z", and similar bounded questions.

### moderate — fan out, no reconcile

Pick 2–4 surfaces the question actually spans. Dispatch one `Research` (Sonnet) agent per
surface **in the same response** so they run concurrently. Each returns its own narrow
summary; the main session reads them side by side. No reconcile — surfaces are expected to be
complementary, not contradictory. Frame each task by goal, not by tool/location:

```text
Agent (subagent_type: Research): "repositories: how does <lib> implement <feature>? cite files/PRs"
Agent (subagent_type: Research): "documentation: authoritative guidance on <feature>, incl. version notes"
Agent (subagent_type: Research): "context7: current API surface for <lib> for <feature>"
```

### advanced — fan out, then reconcile via fork

Use all five surfaces, with **repositories doubled** (two parallel agents split by repo set or
by search angle) → 5–6 agents. Dispatch all `Research` (Sonnet) agents in the same response.

When they return, **dispatch a `fork` to reconcile** — not an isolated agent. The fork inherits
the full session, so it already holds the returned summaries AND the project's own constraints
and decisions; that context lets it consolidate better than an isolated synthesizer could.

```text
# Fan-out (same response):
Agent (subagent_type: Research): "repositories (primary): <repo set A> — implementation of <topic>"
Agent (subagent_type: Research): "repositories (secondary): <repo set B> — alternative implementations"
Agent (subagent_type: Research): "sota: leading/current approach to <topic> (papers, benchmarks, or repos)"
Agent (subagent_type: Research): "documentation: authoritative docs on <topic>, wherever they live"
Agent (subagent_type: Research): "context7: version-pinned library knowledge for <topic>"
Agent (subagent_type: Research): "web: practitioner caveats and gotchas on <topic>"

# Fan-in (after summaries are back in main context):
Agent (subagent_type: fork): "Reconcile the research findings now in context. Weigh source
quality, resolve contradictions across surfaces, surface tradeoffs explicitly, and give one
justified recommendation that fits this project's constraints."
```

Reconcile rules (these live with the fork, since the fork carries them out):
- The fork already has the summaries in context — **don't paste them**; just tell it to reconcile.
- Weigh source quality; resolve contradictions across surfaces; surface tradeoffs explicitly.
- Produce ONE justified recommendation, fitted to the project's own constraints and decisions.
- Make the judgment auditable: which sources were decisive, and what would change the call.
- Read-only synthesis — no file edits. Code changes are a separate fork dispatch.

## Discipline

- **Narrow summaries.** Every fan-out summary lands in the main context. Bind each surface
  agent to a tight, link-backed summary — many detailed returns bloat main context fast.
- **Consolidate, don't accumulate.** On advanced runs, route findings into the fork reconcile
  rather than reading six raw blocks in main. On moderate runs, keep surfaces honest (2–4).
- **Don't over-tier.** A light question dispatched as advanced wastes tokens and a reconcile
  pass on nothing to resolve. Match the tier to the actual breadth of the question.
- **Lens, not location.** Never confine a surface agent to one source; state the goal and let
  it range.
