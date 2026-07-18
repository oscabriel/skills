---
name: handoff
description: Compact the current conversation into a handoff document for another agent to pick up.
argument-hint: "What will the next session be used for?"
disable-model-invocation: true
---

Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save to `~/.agents/handoffs/<project>/` — never the OS temp directory and never the current workspace. `<project>` is the basename of the git repo root (or of the current working directory when not in a repo). Create the directory if it doesn't exist.

Name the file `YYYY-MM-DD-HHMM-<topic>.md`, where `<topic>` is a short kebab-case slug of the session's focus (e.g. `2026-07-15-2150-testing-v0.1.0.md`). After writing, print the full path. The companion `/pickup` skill resumes from the newest document in this directory.

Include a "suggested skills" section in the document, which suggests skills that the agent should invoke.

Do not duplicate content already captured in other artifacts (specs, plans, ADRs, issues, commits, diffs). Reference them by path or URL instead.

Redact any sensitive information, such as API keys, passwords, or personally identifiable information.

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the doc accordingly.
