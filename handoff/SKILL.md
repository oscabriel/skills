---
name: handoff
description: Write a handoff doc so a fresh session can resume this work with /pickup, or relay into a fresh herdr pane.
argument-hint: "[relay] What will the next session be used for?"
disable-model-invocation: true
---

Write a handoff doc that a fresh agent resumes with `/pickup`. The doc passes the **cold start** test: an agent holding only this doc and the repo can take the first action without asking anyone anything.

Arguments describe the next session's focus. If the first argument is `relay`, drop it from the focus and follow [RELAY.md](RELAY.md) after saving.

## 1. Locate the bucket

```bash
root=$(git rev-parse --path-format=absolute --git-common-dir 2>/dev/null) && root=$(dirname "$root") || root=$PWD
```

Outside git, when the session changed exactly one repo, set `root` to that repo instead. Then:

```bash
dir="$HOME/.agents/handoffs/$(basename "$root")"; mkdir -p "$dir"
```

## 2. Write and save

Fill [TEMPLATE.md](TEMPLATE.md), keeping its headings verbatim. List every repo the session touched under `repos`, each at `git -C <path> rev-parse --short HEAD`.

- Point at artifacts that already hold the content (specs, issues, commits, diffs). The doc carries only what lives nowhere else.
- Carry forward every live item from the newest earlier doc in `$dir`, so earlier docs become safe to delete.
- Name where a secret lives, never its value.

Save to `$dir/YYYY-MM-DD-HHMM-<topic>.md` in local time, `<topic>` a short kebab-case slug of where the work stands and what comes next. Print the full path.
