---
name: replicant
description: Source-first external repository research using durable, human-findable local clones. Use when the user asks about an open-source repo, GitHub/GitLab URL, library internals, framework behavior, architecture, implementation details, or examples grounded in source code.
allowed-tools: Bash(git:*), Bash(gh:*), Bash(mkdir:*), Bash(test:*), Bash(find:*), Bash(rg:*), Bash(grep:*), Bash(ls:*), Bash(pwd:*), Bash(date:*), Read, Glob, Grep, Edit, Write
---

# Replicant

Use durable, human-findable local clones of external repositories as source context. Prefer real source code over stale docs, generated summaries, or web snippets.

Replicant is a clone shelf, not a hidden cache, generated-docs system, or custom CLI.

<!-- REPLICANT_CONFIG
configured: true
clone_root: ~/clones
default_update_policy: auto-clean-only
default_clone_depth: full
preferred_transport: ssh
inventory_file: ~/clones/README.md
last_setup_at: 2026-06-05
REPLICANT_CONFIG -->

## Required order of operations

Always follow this order when the user asks about an external repo/library/project:

1. Resolve config from the `REPLICANT_CONFIG` block above, or fallback config if needed.
2. Extract repo clues from the user request: explicit URL, `owner/repo`, package name, project name, keywords, framework/library names.
3. Search locally in the configured clone shelf **before web search**:
   - check the inventory file if present;
   - search clone directory names under `clone_root`;
   - prefer exact repo-name or `owner/repo` matches over fuzzy keyword matches.
4. If local search yields one confident repo, use that clone.
5. If local search yields multiple plausible repos, disambiguate using local paths/inventory first; ask the user only if still ambiguous.
6. If no confident local match exists, resolve the canonical upstream repo with web/code search, then map it to `clone_root/<host>/<owner>/<repo>`.
7. Check whether the clone exists: `test -d "$LOCAL/.git"`.
8. Clone if missing, using configured transport/depth.
9. If clone exists, update according to policy.
10. Search and read source directly.
11. Answer with source evidence: commit SHA, file paths, and line ranges when practical.

Do not use web search as the first tool call for an ambiguous project name unless local clone-shelf search is impossible.

## Config and layout

The configured `clone_root` is authoritative. Defaults and examples are only fallbacks.

Current configured layout:

```text
~/clones/<host>/<owner>/<repo>
```

Examples:

```text
~/clones/github.com/facebook/react
~/clones/github.com/tanstack/router
~/clones/github.com/davis7dotsh/better-context
```

If unconfigured, use `~/clones` as the default root during first-run setup.

## Local-first repo resolution

For ambiguous names like “mole”, “react router”, or “the auth library”, search the local shelf first.

Use commands like:

```bash
CLONE_ROOT="$HOME/Developer/clones"
INVENTORY="$HOME/Developer/clones/README.md"
KEYWORD="mole"

# Inventory first, if present.
test -f "$INVENTORY" && rg -i -- "$KEYWORD" "$INVENTORY"

# Then human-findable clone paths.
find "$CLONE_ROOT" -mindepth 3 -maxdepth 3 -type d \
  | grep -i -- "$KEYWORD"

# Confirm candidate repos.
test -d "$CLONE_ROOT/github.com/owner/repo/.git" \
  && git -C "$CLONE_ROOT/github.com/owner/repo" rev-parse HEAD \
  && git -C "$CLONE_ROOT/github.com/owner/repo" status --porcelain
```

Selection rules:

- Exact `repo` name match beats substring match.
- Exact `owner/repo` match beats repo-only match.
- A locally cloned repo beats a web result unless evidence shows it is the wrong project.
- If two local clones are equally plausible, ask the user to choose.
- If local search finds no plausible clone, use web/code search to identify the canonical repo.

## Normalize repository inputs

Accept:

```text
owner/repo
https://github.com/owner/repo
https://github.com/owner/repo/tree/main/path
https://github.com/owner/repo/blob/main/file.ts
git@github.com:owner/repo.git
```

Normalize to:

```text
host/owner/repo
```

Strip URL suffixes like `/tree/...`, `/blob/...`, `/issues/...`, `/pull/...` and clone the base repo.

## Clone and update policy

Policies:

- `ask`: ask before fetching/updating an existing clone.
- `auto-clean-only`: if `git status --porcelain` is empty, fetch/update safely; if dirty, do not modify and mention the dirty checkout.
- `never`: do not update automatically.

Before using a clone, record:

```bash
git -C "$LOCAL" rev-parse HEAD
git -C "$LOCAL" status --porcelain
```

If the clone is missing:

- use configured transport (`ssh` or `https`);
- use configured depth (`full` or shallow depth);
- ask before full-cloning an obviously large repo or monorepo, even if `default_clone_depth: full`.

## Research rules

- Treat external clones as read-only context by default.
- Do not commit, push, branch, reset, run `git clean`, delete, or otherwise mutate clone contents unless the user explicitly asks.
- Preserve local modifications.
- Do not install dependencies or run builds/tests inside external clones unless necessary for the answer.
- Search and read source files directly; prefer implementation evidence over README claims when explaining behavior.
- Include the commit SHA used for evidence.
- Cite source file paths; include line ranges when practical.

## First-run setup

Before using this skill, inspect the `REPLICANT_CONFIG` block above.

If `configured: false`:

1. Tell the user Replicant keeps durable, human-findable source clones in `~/clones` by default.
2. Confirm or ask preferences for:
   - clone root: default `~/clones`
   - update policy: `ask`, `auto-clean-only`, or `never`
   - clone depth: `1` shallow clone by default, or `full`
   - transport: `https` by default, or `ssh`
3. Create the clone root and inventory file.
4. Edit this `SKILL.md` config block to set `configured: true` and record the chosen values.

If this skill file cannot be edited, write the same config to `~/clones/REPLICANT.md` and read it on future uses.

See [setup reference](references/setup.md) for exact setup prompts and fallback config.

## References

See [workflow reference](references/workflows.md) for expanded command recipes.
