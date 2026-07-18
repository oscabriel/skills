# Herdr Bridge

Herdr Bridge is a Claude Code skill for delegating work to [pi](https://pi.dev) while keeping every long-running worker visible and controllable through herdr.

It gives Claude a concrete protocol for:

- choosing when to delegate a task to pi;
- launching pi in a surfaced herdr pane instead;
- using pi workers from dynamic workflow fan-outs;
- monitoring worker state and recovering blocked or failed sessions;
- collecting structured reports for Claude to review before accepting changes.

Claude remains the orchestrator and final reviewer. Pi performs delegated investigation or implementation, while herdr provides the pane lifecycle and status bridge that makes those workers observable.

## Integrating it with Claude Code

### 1. Install the skill

Install this repository with your preferred skills installer, or copy this directory to:

```text
~/.claude/skills/herdr-bridge/
```

Claude Code should then be able to load `herdr-bridge/SKILL.md` whenever it plans to invoke pi, create herdr panes, or run a dynamic Workflow.

### 2. Install and configure pi and herdr

Both commands must be available in the environment where Claude Code runs:

```bash
pi --version
herdr --version
```

Configure pi with access to the model named in `SKILL.md`, or update the skill to use the pi model available in your environment. Start Claude Code from inside a herdr-managed session so `HERDR_ENV`, `HERDR_PANE_ID`, and the herdr socket are available to Claude and inherited by Workflow workers.

### 3. Add the companion Claude instructions

[`references/CLAUDE.example.md`](./references/CLAUDE.example.md) contains the global routing and reporting rules this skill was designed to work with. Merge the relevant sections into your global Claude Code instructions:

```text
~/.claude/CLAUDE.md
```

Do not blindly replace an existing file. Preserve your own instructions and adapt model names, effort levels, and routing rules to your setup. Keep the blocking requirement that Claude must load `herdr-bridge/SKILL.md` before invoking pi or creating Workflow workers.

### 4. Use it in dynamic Workflows

Ask Claude Code to use a dynamic Workflow when a task benefits from parallel investigation, implementation, or verification. With the companion instructions installed, Claude should:

1. load the Herdr Bridge skill;
2. create and name the required herdr panes;
3. spawn thin Workflow bridge agents that drive pi in those panes;
4. give each pi worker a self-contained brief and report path;
5. wait for terminal worker states and harvest the reports;
6. review the reports and resulting diffs itself;
7. close successful panes while leaving blocked panes open for inspection.

For example:

```text
Use a dynamic Workflow to investigate these failures in parallel. Follow the
herdr-bridge protocol, run each pi worker in a visible herdr pane, and review the
harvested reports before proposing changes.
```

The skill intentionally treats missing herdr context as a blocker for Workflow fan-out. If Claude Code was not launched under herdr, restart it in a herdr-managed pane rather than silently falling back to invisible long-running workers.
