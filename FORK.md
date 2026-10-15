# FORK.md — rpmalouin/hindsight

**This is `rpmalouin/hindsight`**, a public fork of
[`vectorize-io/hindsight`](https://github.com/vectorize-io/hindsight) — Hindsight, the agent memory
server (facts, observations and mental models in Postgres + pgvector, with LLM-based fact extraction
and multi-strategy retrieval) — kept by Ron Malouin as the **build source of a self-hosted deployment**.

**Standing policy: no upstream pull requests.** This fork is a destination, not a staging area.
Upstream issues and fixes belong upstream. Nothing here is a candidate for upstream review unless
asked for at that moment.

| | |
|---|---|
| **Fork** | `rpmalouin/hindsight` (public fork, default branch `main`) |
| **Upstream** | `vectorize-io/hindsight`, branch `main` |
| **Fork point** | upstream `main` @ `6a9fa03a1`, 0 ahead / 0 behind, 2026-10-07 |
| **License** | MIT, upstream's (© Vectorize AI, Inc.) — unchanged |

## Why this fork exists

The deployment runs Hindsight as **its own container built from this checkout** — image built from
`docker/standalone/Dockerfile`, `target: standalone`, which is the API, the web Control Panel and an
embedded Postgres in one image — rather than from a published artifact. That makes the running server
a pinned, auditable snapshot of a known upstream SHA, and makes a rebuild reproducible on the
deployment's own machine.

The point is that the code is **upstream's, untouched**: this deployment wants the real project at a
known revision, not a patched derivative.

> Deployment specifics — host paths, ports, bank names, compose files, credentials — are deliberately
> **not** in this repository. They live in the untracked `MEMORY.md` operator notes and in the
> deployment's own stack directory. Placeholders here: `<repo>`, `<stack-dir>`, `<upstream-url>`.

## Fork delta

**No code delta.** No upstream file is modified, no patch is carried, no file is vendored.

| Ref | Content |
|---|---|
| `main` | upstream `main` @ `6a9fa03a1` **+ one docs commit adding this file** |
| working tree | two *uncommitted* ignore-file edits (see below) |

Two consequences worth knowing before touching anything:

- **`.github/workflows/*` are upstream's and stay exactly as they are.** Deleting or disabling them
  would conflict on every sync and would break upstream's CI-contract tests. GitHub does not run
  workflows in a fork until someone enables them, and this fork has **0 workflow runs** — no
  automation here, and nothing billed.
- **The two local-only edits stay uncommitted on purpose**: `.dockerignore` and `.gitignore` both
  exclude `MEMORY.md`, and `.dockerignore` also excludes `.code-review-graph/`. They are
  deployment-convenience excludes, not fork content — upstream touches neither file, so they are
  simply re-checked after a sync.

## Syncing with upstream

`main` is **not** a plain fast-forward mirror any more — it carries this file, so a sync is a rebase:

```sh
git fetch <upstream-url> main
git -c rebase.autoStash=true rebase FETCH_HEAD     # autoStash keeps the uncommitted ignore edits
git tag pre-upstream-sync-<YYYYMMDD> ORIG_HEAD     # rollback point, optional
git push --force-with-lease origin main            # the rebase rewrites main: --force-with-lease, never bare --force
```

The rebase rewrites this fork's `main` (upstream's history plus this one commit), so the push is always a
forced update — `--force-with-lease` is the safe form: it refuses if anyone else moved the branch.
`git merge --ff-only` and a plain `git push` both fail by design here.

`FORK.md` does not exist upstream, so this rebase is conflict-free unless upstream adds a file of the
same name. The delta must stay this small: adding fork-owned code to `main` means re-applying it on
every future sync, which is the cost this repository is deliberately avoiding.

Never `git add -A` here — the tree carries local-only artifacts (below).

## Deliberately NOT included

- **`MEMORY.md`** — host-local operator notes: clone paths, build and deploy gotchas, verification
  recipes, the deployment's exact values. Untracked and gitignored.
- **`.code-review-graph/`** — the local code knowledge graph (hundreds of MB, absolute host paths).
- **Any credential, token, key or `.env` value**, and the deployment's compose file and its secrets.

## Related

- Upstream docs of record: `AGENTS.md` → `CLAUDE.md` (project documentation and coding conventions).
- Local operator notes: `MEMORY.md` (untracked; read it before changing anything in a checkout).
