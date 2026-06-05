# Agent Skills

A small collection of agent skills I use locally.

This repo is intentionally simple: each skill lives in a top-level directory with its own `SKILL.md` and any supporting reference files beside it.

## Install

Install these skills with [skills.sh](https://skills.sh):

```bash
npx skills@latest add oscabriel/skills
```

## Skills

### Replicant

[`replicant`](./replicant/SKILL.md) is a source-first research skill for external repositories and libraries. It teaches an agent to use durable, human-findable local clones as its primary evidence source instead of relying on stale documentation, web snippets, or generated summaries.

Replicant is useful when you ask about:

- a GitHub (or GitLab, etc) repository;
- an open-source library or framework;
- package internals;
- implementation details;
- architecture or design decisions in a codebase;
- examples that should be grounded in real source code.

The skill uses a local clone shelf with paths like:

```text
~/clones/<host>/<owner>/<repo>
```

#### Setup

On first use, configure the clone shelf by choosing:

- **Clone root** — where repositories should live, for example `~/clones`.
- **Update policy** — whether clean existing clones can be updated automatically, whether the agent should ask first, or whether it should never update.
- **Clone depth** — full history or shallow clones.
- **Transport** — SSH or HTTPS.

After setup, Replicant records those choices in its `SKILL.md` config block. If a local inventory file exists, the skill checks it before searching clone paths.

#### Usage

Ask the agent about an external repo, library, or source-backed behavior, for example:

```text
Use replicant to look up facebook/react. How does Suspense handle thrown promises?
```

Replicant should then:

1. search the local clone shelf first;
2. clone the repository if it is missing;
3. update an existing clean clone according to the configured update policy;
4. inspect the source directly;
5. answer with evidence, including the commit SHA, file paths, and line ranges when practical.

### Architecture Spine

[`architecture-spine`](./architecture-spine/SKILL.md) converts clarified domain context into compile-visible architecture before business behavior is implemented. It is meant for the moment after a plan, grill session, ADR, or domain discussion has produced durable decisions, but before the agent starts coding product workflows.

The skill helps turn approved context into:

- domain types, schemas, brands, and invariants;
- discriminated unions and state models;
- smart constructors and parsers at system edges;
- service and adapter seams;
- typed result and error families;
- composition, layer, and module topology;
- production and test call stacks;
- dependency-direction checks where practical.

There is no setup step for this skill. Use it when the project already has enough agreed context to encode structure safely, for example:

```text
Use architecture-spine to turn these grill-with-docs decisions into domain types, service seams, typed errors, and dependency boundaries. Do not implement business behavior yet.
```

Architecture Spine should not create product workflows, fake persistence, fake network behavior, or speculative implementation logic. Its job is to make the intended architecture hard to accidentally violate before feature work begins.
