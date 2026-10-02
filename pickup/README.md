# Handoff and pickup

Two user-invoked skills that carry work across sessions and harnesses. `/handoff` writes a doc at the end of a session, and `/pickup` reads it at the start of the next one.

```text
/handoff [relay] <focus>   →   ~/.agents/handoffs/<project>/YYYY-MM-DD-HHMM-<topic>.md   →   /pickup
```

## How they fit

- **One bucket per project.** Docs live in `~/.agents/handoffs/<project>/`, where `<project>` is the main repo's basename (worktrees share it). A session run outside git that changed one repo files under that repo; otherwise it uses the working directory's name. The directory sits outside every repo and every harness, so pi, Claude Code, Codex, and OpenCode all share it.
- **Four questions.** [`handoff/TEMPLATE.md`](../handoff/TEMPLATE.md) holds the doc's four `##` questions and a `repos` frontmatter list of `path: branch@head`.
- **Pickup is short.** `/pickup` runs git against each listed repo, briefs in at most six lines, and starts on your go. It checks other claims only when the work reaches them. Outside git it takes the newest doc across all buckets.
- **Docs stand alone.** Each doc carries forward everything still live from the one before, so older docs are safe to delete. `/pickup` deletes docs older than 7 days when you say `clean`.
- **Relay.** `/handoff relay` writes the doc, starts the same agent with the same flags in a new herdr pane, prompts it to pick up that exact doc, and closes the old pane once the new one is working. Outside herdr it prints the doc path for a manual `/pickup`.

## Invoking

| Harness | Write | Resume |
| --- | --- | --- |
| pi | `/skill:handoff` | `/skill:pickup` |
| Claude Code | `/handoff` | `/pickup` |
| Codex | `$handoff` | `$pickup` |

## Prerequisites

- Both skills installed side by side in `~/.agents/skills/`. Symlink them to a clone of this repo so edits apply without a copy step. Relay prompts the successor with `~/.agents/skills/pickup/SKILL.md`.
- Relay only: `herdr` and `jq` on PATH, with the session running in a herdr pane.

`/handoff` began as [mattpocock's `handoff` skill](https://github.com/mattpocock/skills).
