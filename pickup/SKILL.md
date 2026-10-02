---
name: pickup
description: Start a session from the newest project handoff written by /handoff, checked against the repo before any work.
argument-hint: "[doc name, path, or bucket] What will this session focus on?"
disable-model-invocation: true
---

The handoff doc is **inheritance**: trust it, check only what git can disprove, and get to work.

## 1. Locate

```bash
root=$(git rev-parse --path-format=absolute --git-common-dir 2>/dev/null) && root=$(dirname "$root") || root=$PWD
dir="$HOME/.agents/handoffs/$(basename "$root")"
doc=$(ls "$dir"/[0-9]*.md 2>/dev/null | sort | tail -1)
```

- An argument naming a doc path, a filename or slug in `$dir`, or a bucket under `~/.agents/handoffs/` picks that doc or that bucket's newest. The rest of the argument is the focus and overrides the doc's.
- Outside git with `$doc` empty, take the newest doc across all buckets: `ls ~/.agents/handoffs/*/[0-9]*.md | awk -F/ '{print $NF"\t"$0}' | sort | tail -1 | cut -f2`.
- With no doc found, suggest `/handoff` in the previous session and stop.

## 2. Check and brief

Read the doc. For each entry under `repos` (`path: branch@head`), run `git -C <path> log --oneline <head>..HEAD`, `git -C <path> status --short`, and `git -C <path> branch --show-current`. Check nothing else yet: other claims get checked when the work reaches them. Older docs carry `repo`/`branch`/`head` frontmatter and different headings; read them the same way.

Brief in at most six lines: the doc's path, any drift git showed, and the answer to "What's next?" corrected for it. With no focus set, list the open items and ask which to take. Add one line if `find "$dir" -maxdepth 1 -name '*.md' -mtime +7` finds docs other than this one: "N docs older than 7 days; say `clean` to delete them."

Start the first action when the user says go, loading any skills the doc names for it.
