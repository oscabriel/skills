# Pickup

`/pickup` resumes work from handoff documents. It is a companion skill to [mattpocock's `/handoff`](https://github.com/mattpocock/skills) ([skill source](https://github.com/mattpocock/skills/blob/main/skills/productivity/handoff/SKILL.md)): `/handoff` compacts a session into a document; `/pickup` reads that document and continues the work.

## Modes

- **Read mode** — run `/pickup` as the opening message of a fresh session. It finds the newest handoff for the project, verifies the doc's claims against the current repo state, and briefs you before any work starts.
- **Relay mode** — run `/pickup` mid-session. It writes a handoff, spawns a fresh session in a new herdr pane launched with the same command that started the current session (alias-expanded, flags included), and closes its own pane once the successor is confirmed running. Outside herdr it falls back to printing the handoff path so you can relaunch manually.

## Prerequisites

- **The `/handoff` skill**, installed at `~/.claude/skills/handoff/`. Either:
  - install upstream: `npx -y skills add mattpocock/skills --skill handoff --agent claude-code`, or
  - copy the bundled version from [`references/handoff/`](./references/handoff/) into `~/.claude/skills/handoff/`.

  The bundled copy is lightly adapted from upstream: it saves handoffs to `~/.agents/handoffs/<project>/` — the directory read mode checks first — instead of the OS temp directory. Read mode falls back to the temp directory, so unmodified upstream `/handoff` works too.
- **Relay mode only:** `herdr` and `jq` on PATH, with the session running inside a herdr-managed pane. Read mode needs neither.
