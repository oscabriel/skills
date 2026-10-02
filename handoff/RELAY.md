# Relay

This session ends, and a fresh session in a new herdr pane picks up the doc you just wrote. `$root`, `$dir`, and the doc path carry over from the handoff steps.

## 1. Check for herdr

```bash
test "${HERDR_ENV:-}" = 1 && kind=$(herdr agent get "$HERDR_PANE_ID" | jq -r '.result.agent.agent')
```

`kind` is this agent's herdr kind (`pi`, `claude`, `codex`, ...). The relay continues only with a real kind and every herdr command below succeeding. On any failure, take the **fallback**: print the doc path, tell the user to start a fresh session in this project and run `/pickup`, and stop with both panes as they are.

## 2. Spawn the successor

Relaunch the same kind with the same flags. `ps -o args= -p $PPID` prints this agent's command line (the shell's parent). Its flags are everything after the executable, minus any positional prompt.

```bash
new=$(herdr pane split --current --direction right --no-focus --cwd "$root" | jq -r '.result.pane.pane_id')
herdr pane rename "$new" "<project> pickup"
herdr agent start "<project>-pickup" --kind "$kind" --pane "$new" -- <flags>
herdr agent prompt "$new" "Read ~/.agents/skills/pickup/SKILL.md and follow it to pick up <doc path>."
herdr agent wait "$new" --until working --timeout 45000
```

- Take the pane ID from the JSON response every time.
- The prompt names the skill file and the exact doc in plain words. Every harness reads it the same way, and the successor gets this doc even if another session writes a newer one.
- If `start`, `prompt`, or `wait` fails, keep both panes open. Show `herdr pane read "$new" --source recent-unwrapped --lines 40` to the user, report what happened, and stop.

## 3. Hand over

Once the successor is working, close this pane as the final command:

```bash
herdr pane close "$HERDR_PANE_ID"
```

This session ends with the pane, so this is the last tool call.
