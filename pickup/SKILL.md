---
name: pickup
description: Resume work from a handoff document written by /handoff. Run in a fresh session to read the newest handoff and get briefed; run mid-session to hand off and relaunch into a fresh session automatically (via herdr when available).
argument-hint: "What will you pick up with in the next session?"
disable-model-invocation: true
---

`/pickup` has two modes, chosen by an unambiguous signal:

- **Read mode** — `/pickup` is the opening message of this conversation. Read the handoff and brief the user.
- **Relay mode** — `/pickup` was invoked mid-session, with prior context. Hand off this session's work and relaunch into a fresh session that picks it up.

In both modes: `<project>` is the basename of the git repo root (or of the current working directory when not in a repo), and handoff docs live in `~/.agents/handoffs/<project>/`.

## Relay mode

Goal: this session ends; a fresh session in a new herdr pane continues from a handoff doc.

### 1. Write the handoff

Follow the instructions in `~/.claude/skills/handoff/SKILL.md` exactly; if that skill is not installed, use the bundled copy at `references/handoff/SKILL.md` next to this file. Treat any arguments as the description of what the next session will focus on.

### 2. Check for herdr

```bash
test "${HERDR_ENV:-}" = 1
```

If this fails, or any herdr command below fails, use the fallback: print the handoff doc path, tell the user to start a fresh session in this project and run `/pickup`, and stop. Do not attempt any other way to spawn or kill sessions.

### 3. Spawn the successor

Split a new pane to the right of this one, launch the fresh agent with the pickup prompt as its launch argument, and verify it started before tearing anything down. The prompt must be passed at launch — after this pane closes, nothing is left to send it.

Launch the successor with the same command that started this session: from a Bash call, `ps -o args= -p $PPID` prints this agent process's command line (the shell's parent), with any alias already expanded and flags included. Drop any positional prompt argument and keep the command plus its flags. If the output doesn't look like an agent CLI invocation, fall back to `claude`.

```bash
launch=$(ps -o args= -p $PPID)   # e.g. "claude --some-flag"; fall back to "claude" if implausible
new=$(herdr pane split --current --direction right --no-focus --cwd "<project-root>" | jq -r '.result.pane.pane_id')
herdr pane rename "$new" "<project> pickup"
herdr pane run "$new" "$launch '/pickup <focus args, if any>'"
herdr wait agent-status "$new" --status working --timeout 45000
```

- Parse the pane ID from the JSON response; never construct or guess IDs.
- Quote the launch command carefully if the focus args contain quotes.
- If the wait times out, do NOT close anything. Inspect `herdr pane get "$new"` and `herdr pane read "$new" --source recent-unwrapped --lines 40`, report what happened, and leave both panes open for the user.

### 4. Hand over

Only after the successor is confirmed working, close this pane as the final act — focus falls to the remaining pane:

```bash
herdr pane close "$HERDR_PANE_ID"
```

Nothing may be scheduled after this command; this session dies with the pane.

## Read mode

### Locate

- No argument → pick the newest document in `~/.agents/handoffs/<project>/` by modification time.
- Argument matching a filename or topic slug in that directory → pick that document. An explicit path → read it directly. Any remaining argument text is the focus for this session; tailor the briefing toward it.
- Directory missing or empty → check the OS temp directory for legacy `*handoff*.md` files matching the project. If still nothing, list which projects under `~/.agents/handoffs/` do have documents, suggest `/handoff`, and stop.

### Read and verify

Read the chosen document in full, and follow the paths/URLs it references when they matter to the next step. The doc may be stale: compare its claims against reality — `git log` and `git status` since the doc's timestamp, tracker/issue state if referenced — and note any drift.

If the doc has a "suggested skills" section, invoke the skills relevant to the work being resumed.

### Brief

Report back, concisely:

1. **Where we are** — current state of the work, corrected for any drift found.
2. **What we learned** — key decisions and gotchas the doc carries forward.
3. **What's next** — the recommended next step, as a concrete action.

Mention how many older handoffs exist for the project. Do not start implementing until the user confirms the direction.
