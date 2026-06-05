# Replicant setup

Replicant has no CLI and no hidden map. First-run state lives in the skill's `REPLICANT_CONFIG` block when possible.

## First-run prompt

Use this concise setup prompt:

```text
Replicant keeps durable source clones for humans and agents. I recommend `~/clones` as the central clone shelf.

Confirm these defaults?
- Clone root: ~/clones
- Update policy: ask before pulling existing clones
- Clone depth: shallow (`--depth 1`)
- Transport: HTTPS
```

If the user wants details, explain options:

- `clone_root`
  - default: `~/clones`
  - should be human-findable and outside project repos
- `default_update_policy`
  - `ask`: ask before pulling existing clones
  - `auto-clean-only`: auto-pull only when working tree is clean
  - `never`: do not update unless user explicitly asks
- `default_clone_depth`
  - `1`: shallow clone, fast and small
  - `full`: full history, useful for archaeology/blame/tag-heavy work
- `preferred_transport`
  - `https`: default, simplest for public repos
  - `ssh`: useful for private repos and configured SSH keys

## Setup commands

For default setup:

```bash
mkdir -p "$HOME/clones"
test -f "$HOME/clones/README.md" || cat > "$HOME/clones/README.md" <<'EOF'
# Local Source Clones

This directory contains durable external source clones for humans and coding agents.

Default layout:

```text
~/clones/<host>/<owner>/<repo>
```

Examples:

```text
~/clones/github.com/facebook/react
~/clones/github.com/tanstack/router
```

## Policy

- Clones are source context, not working copies.
- Agents should treat them as read-only unless explicitly told otherwise.
- Agents may clone missing repositories here.
- Agents should ask before updating existing clones unless configured otherwise.
- Agents should not hide external clones inside project directories.

## Inventory

Add useful repos below.
EOF
```

## Edit skill config

After setup, edit `SKILL.md`:

```text
<!-- REPLICANT_CONFIG
configured: true
clone_root: ~/clones
default_update_policy: ask
default_clone_depth: 1
preferred_transport: https
inventory_file: ~/clones/README.md
last_setup_at: YYYY-MM-DD
REPLICANT_CONFIG -->
```

Use the current date for `last_setup_at`.

## Fallback config

If the skill cannot be edited, create `~/clones/REPLICANT.md`:

```md
# Replicant Config

configured: true
clone_root: ~/clones
default_update_policy: ask
default_clone_depth: 1
preferred_transport: https
inventory_file: ~/clones/README.md
last_setup_at: YYYY-MM-DD
```

On future uses, read this file after the skill config block.
