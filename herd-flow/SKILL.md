---
name: herd-flow
description: Mandatory protocol for all pi (gpt-5.6-sol) delegation — one-off `pi` CLI calls and every workflow subagent (pi in a surfaced herdr pane via a sonnet bridge). Load BEFORE writing a Workflow script, spawning workflow workers, delegating to pi/gpt-5.6-sol, or driving herdr panes. Covers pi invocation, pane placement, report harvesting, and failure handling.
---

# herd-flow: pi delegation through visible herdr panes

## Routing reference

Before writing a Workflow script or launching a dynamic workflow, read
[`references/routing.md`](./references/routing.md) — the model-routing, reporting, and
escalation rules this protocol operates under. This file covers HOW to run pi workers;
the routing reference covers WHO does which work.

## pi CLI mechanics

gpt-5.6-sol is only reachable via `pi`; Claude models run via the Agent/Workflow
`model` parameter, never through pi. Default pi model: `openai-codex/gpt-5.6-sol:low` —
an example of the author's setup; substitute the model configured in your pi
environment, keeping the `:low`/`:medium`/`:high` thinking-suffix pattern.
Calibration: staged pinned-contract implementation briefs (~1–2k lines, multi-package)
run reliably at `:medium` (~15–30 min); keep `:low` for small mechanical tasks.
Smoke-test with a one-line `pi -p` before fanning out a wave.

- Implementation task (pi has read/bash/edit/write tools):
  `pi --model openai-codex/gpt-5.6-sol:low --no-session -p "<self-contained prompt>"`
- Read-only investigation/review: add `--tools read,grep,find,ls`.
- Harder problems: raise the thinking suffix, e.g. `openai-codex/gpt-5.6-sol:high`.
- Attach files positionally before the message: `pi ... -p @src/auth.ts "Review this."`
- Prompts must be fully self-contained — pi cannot see your conversation. State the
  goal, constraints, exact file paths, and expected output or diff format.
- pi resolves ambiguity silently rather than flagging it — wherever correctness hinges
  on exact semantics, pin them to the letter; it will not ask.
- After pi edits files, review the diff yourself before accepting.

Headless pi via Bash hard-caps at 10 minutes and a killed pi loses its work — reserve
it for calls confidently under ~8 minutes; anything longer runs in a surfaced pane
(protocol below). Workflow subagents never run headless.

## Workflow subagents — the herdr bridge (MANDATORY for all workflow workers)

The Workflow `model` parameter only takes Claude models, so each worker is a thin
Claude bridge (`model: 'sonnet', effort: 'low'`, label prefixed `pi:`, `schema` on the
bridge for structured output) driving pi in a **visible herdr pane** — never headless,
never a bare Claude worker. Subagents inherit `HERDR_ENV`/`HERDR_SOCKET_PATH` from the
head session; herdr push-reports pi status (`working`/`blocked`/`idle`).

Report-back instruction — append verbatim to every workflow pi prompt:
"When finished, write your REPORT to <report-path>: per-file summaries with function
signatures added/removed/changed, plus your conclusions and anything you're unsure
about. For read-only tasks (investigation, review, verification) skip the signature
section and report your findings instead. Your report goes to the head orchestrator,
who reviews and verifies your work after you finish — flag open questions rather than
papering over them."

For long briefs, instruct pi to APPEND to the report file after each stage — a dead or
blocked pi then leaves a harvestable trail instead of nothing.

Protocol (plain Bash; parse ids from JSON responses, never guess them):

1. The HEAD splits, renames, and assigns all panes BEFORE spawning bridges — one split
   at a time (concurrent splits race the layout) — and bakes each pane id into its
   bridge's prompt: `herdr pane split --pane "$HERDR_PANE_ID" --direction right|down
--focus --cwd <dir>` (`--direction` is required), then
   `herdr pane rename <pane> <task>`. Verify pane existence with
   `herdr pane get <pane>` — never `herdr agent get`, which queries the agent registry
   and returns agent_not_found for a bare shell pane; before pi is started that error
   is expected, not a missing pane. Wave panes spawn in the foreground
   (`--focus`); past ~4 agents, create one dedicated wave tab and build the grid there.
   Parallel implementers each get their own checkout via workflow
   `isolation: 'worktree'` or `git worktree add` — not `herdr worktree create`, which
   opens a whole new workspace.
2. `herdr pane run <pane> "pi --model openai-codex/gpt-5.6-sol:low --no-session"`, then
   `herdr agent wait <pane> --status idle --timeout 30000` to confirm pi is up.
3. `herdr pane run <pane> "<task + report-back instruction>"`, wait
   `--status working --timeout 20000` (guards against matching the pre-task idle), then
   wait for completion in a bounded loop: `herdr agent wait <pane> --status idle
--timeout 540000`, re-issuing on timeout (`idle` also matches done; Bash calls cap
   at 10 min — never wait unbounded). Every wait runs as a FOREGROUND Bash call —
   never `run_in_background`, which ends your turn before pi finishes. `working` is
   not an end state: a slow pi is neither done nor blocked, so keep re-issuing the
   wait; a bridge may only report back once the agent is `idle` (harvest), `blocked`
   (failure handling below), or dead.
4. Harvest and sanity-check the report file — pi's TUI runs on the alternate screen, so
   the report file is the source of truth (fallback: `pane read --source visible`).
5. Close the pane (or wave tab) after harvest. Leave failed/blocked panes OPEN for
   inspection.

Failure handling:

- `blocked` = pi needs input or gave up retrying. Read `--source visible`, then
  `pane run` a follow-up (same pi session, context intact) before rerunning from
  scratch, or report failure upward.
- Once pi has been started, check `herdr agent get <pane>` before every `pane run`: a
  dead pi drops the pane back to a shell, and your text lands in the shell.
  agent_not_found at this stage means pi died; pre-launch it is normal (see step 1).
- Agent/tab names must be unique; on workflow resume, adopt an existing pane instead of
  respawning.
- Always target panes explicitly (`--pane`/`--tab`) — never the user's focused pane.
  `--focus` only when spawning; never steal focus on later commands.
- Workflow token budgets count only Claude tokens; pi work is invisible to
  `budget.spent()`.
- `HERDR_ENV` unset is a blocker for workflow workers — tell the user and get their
  go-ahead before running headless.
