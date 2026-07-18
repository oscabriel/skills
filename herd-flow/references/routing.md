# Model routing & reporting

Routing, reporting, and escalation rules for the herd-flow protocol. Consult this file
before writing any Workflow script, spawning workflow workers, or invoking `pi`.
Sections are in precedence order — on conflict, the earlier section wins.

Model names, costs, and effort tiers below reflect the author's setup. Adapt them to
the models available in your environment, but keep the shape: one head
orchestrator/reviewer, cheaper implementer models, and an explicit escalation ladder.
gpt-5.6-sol is only reachable via `pi`; Claude models run only via the Agent/Workflow
`model` parameter.

## Reporting changes for review

After any big change — the head's or a delegated agent's — report per-file summaries
instead of expecting the user to read the diff: files touched plus function signatures
added/removed/changed. Skip function bodies. Delegated agents (pi included) must return
this same shape, not raw diff dumps.

## Hard rules (nothing overrides these)

- **Fan-out is never Fable.** Anything spawning 2+ agents (workflows, `parallel()`/
  `pipeline()`, agent batches) uses only opus, sonnet, or gpt-5.6-sol-via-`pi`.
- **Workflow subagents are always pi in surfaced herdr panes** via the sonnet bridge —
  never headless, never bare Claude workers; every pi prompt carries the report-back
  instruction from `SKILL.md`. The only Claude agents in a workflow are bridges. An
  adversarial-verify pi wave may follow an implementation wave; the head always
  performs the final review of harvested reports and verdicts itself.
- **Fable is at most ONE agent at a time** — head orchestrator/reviewer or via the
  escalation ladder.
- **Dynamic-workflow orchestration model: always Opus at `effort: 'medium'`.**
- **Never Haiku. Never `xhigh`/`max` effort** — Fable runs at `high` at most.

## Routing — first match wins

Fable is the head orchestrator and reviewer; Opus and gpt-5.6-sol are the implementers.

1. Inside a Workflow script → pi-in-herdr-pane per the hard rules and `SKILL.md`.
2. Spawning 2+ agents outside a workflow → opus (default), sonnet, or gpt-5.6-sol-via-`pi`.
3. Bulk/mechanical work (clear-spec implementation, migrations, data analysis, long log
   digs, broad codebase scans) → gpt-5.6-sol via `pi -p`; it reports conclusions, not dumps.
4. Reviews of plans/implementations → the head reviews itself (per-file summaries +
   signatures), or at most one Fable review agent; optionally add opus-4.8 or gpt-5.6-sol
   as an independent second perspective.
5. Everything else (investigation, design, user-facing work) → opus-4.8.

Constraints:

- User-facing surface (UI, copy, API design) needs taste ≥ 7 — never ship gpt-5.6-sol
  output there without an opus-or-better pass.
- Mechanical subagents get `effort: 'low'`; reserve `high` for the hardest
  verify/judge/design stages.

## Escalation ladder — when output doesn't meet the bar

Standing permission to escalate without asking: judge the output, not the price tag —
but use cheaper models to gather information and try things first. Climb in order:

1. Re-run gpt-5.6-sol with a tighter prompt or higher thinking (suffix per `SKILL.md`).
   Via herd-flow, prefer a follow-up `pane run` in the same pi session over a
   from-scratch rerun.
2. Step up to opus-4.8, or get an independent attempt from whichever of opus/gpt-5.6-sol
   hasn't tried yet.
3. Fable — last resort (the standing reviewer role is routed above and doesn't need
   this ladder). Permitted only when ALL hold: exactly one agent; opus and gpt-5.6-sol
   already fell short on this specific task OR it's a single high-taste user-facing
   artifact where taste is the entire job; you state the justification out loud before
   spawning. Unsure whether Fable is justified → it isn't; use opus.

## Reference: model traits (single-agent quality — routing above always wins)

Higher = better. Cost reflects what the author actually pays, not list price.
Intelligence = how hard a problem the model handles unsupervised.
Taste = UI/UX, code quality, API design, copy.

| model       | cost | intelligence | taste |
| ----------- | ---- | ------------ | ----- |
| gpt-5.6-sol | 9    | 8            | 5     |
| sonnet-5    | 5    | 5            | 7     |
| opus-4.8    | 4    | 7            | 8     |
| fable-5     | 2    | 9            | 9     |

For anything that ships: intelligence > taste > cost — resolved within the routing and
escalation rules, never by jumping straight to Fable.
