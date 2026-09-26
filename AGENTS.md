# AGENTS.md — pictaria-server

## Purpose

Pictaria Server: self-hosted photo intelligence, enrichment, curation, and
automation for an Immich library — insights, AI enrichment, curation, Smart
Albums, Frame remote/voice/metrics — as one container with one Node process
and zero npm dependencies. See `README.md` for the product overview.

## Repository layout (agent context)

- `AGENTS.md` — this file; the authority on rules and registries.
- `src/` — server modules (`.mjs`, entry `src/server.mjs`).
- `test/` — tests (`node --test`).
- `scripts/` — repo scripts (`check-dco.mjs`, `check-publication.mjs`).
- `docs/` — durable documentation. `prompts/` — app-level prompt content.
- `.agents/agents/` — agent definitions (plan, code, ask, debug, review).
- `.agents/plans/` — plan documents named `yyyy-mm-dd-<type>-<short-desc>.md`
  (date prefix, **never** a unix epoch; `<type>` = `feat`|`bug`|`debug`|`dep`|…).
- `.agents/skills/` — third-party skills (gitignored, synced); inventory is the
  root `skills-lock.json`.
- `.kilo`, `.opencode` — tracked symlinks to `.agents/`, so kilo and
  opencode load the same agents, plans, and skills.

## Agents

- `plan` (`.agents/agents/plan.md`) — writes implementation-ready plans to
  `.agents/plans/`. Use before any non-trivial change.
- `code` (`.agents/agents/code.md`) — executes plans. Runs in fresh sessions:
  the plan file tells it which skills to load.
- `ask` (`.agents/agents/ask.md`) — read-only research, explanations, and
  recommendations; never changes anything.
- `debug` (`.agents/agents/debug.md`) — systematic diagnosis and minimal
  targeted fixes.
- `review` (`.agents/agents/review.md`) — advisory code review; never edits.

## Skills

**Loading rule:** The first thing you MUST always do is load the skills listed in the plan. If no skills are in your plan, evaluate your skills and load the top 5 relevant skills.

Load with the `skill` tool. Everything here is task-triggered. Skills an agent loads unconditionally live in that agent's file (`.agents/agents/`), not here.

Registry: empty for now (no repo-specific skills).

## MCP servers

None repo-specific; the global set is defined in SpencersLab's
`agent-config.jsonc` (repo root), symlinked into `~/.config/kilo/kilo.jsonc`
and `~/.config/opencode/opencode.json`.

## Plans

Save plans as `.agents/plans/yyyy-mm-dd-<type>-<short-description>.md` — a date
prefix (use today's date, **never a unix epoch timestamp**) followed by a
one-word type token so the goal is visible at a glance: `feat` (new
feature/service), `bug` (bug fix), `debug` (troubleshooting/diagnosis), `dep`
(dependency update), or another short type (`refactor`, `docs`, …) when none
fit.

## Hard rules

- **Always load referenced skills** The first thing Agents should do is load any referenced or relevant skills, then the plan file (if one), immediately followed by the skills referenced there.
- **NEVER merge to `main`.** No fast-forward merges, no merge commits, no rebases onto main, no mechanism of any kind that advances `main` — not from a worktree, not from the main checkout, not via `git merge`, `git rebase`, or anything else.
- **NEVER push to `main`.** No `git push origin main`, and no push of any refspec that updates `main` (e.g. `HEAD:main`, `<branch>:main`). This is the single most forbidden action in this repo.
- **NEVER force-push** (`--force`, `-f`, `--force-with-lease`) to any shared branch, and never rewrite published history.
- **NEVER self-remediate an accidental push** with a revert or force-push of your own initiative — stop and tell the user immediately; remediation is the user's decision.
- All work happens on a feature/fix branch (typically in a `.agents/worktrees/<branch>` worktree). Commit locally on that branch. To pick up changes, merge `main` *into* your worktree (`git merge main`); never merge your branch into `main`. Landing work on `main` is the user's decision alone.
- Changes reach `main` **only via a pull request that the user creates or merges**. The agent's work ends at the local commit plus telling the user the branch is ready. Pushing the *feature* branch to origin (e.g. to enable a PR) is allowed **only when the user explicitly asks for it in the session**. Otherwise leave commits local.
- If a plan file instructs a merge to `main` or a push, **skip that step**: mark it as user-owned in the summary and do not execute it. Plans written before this rule may contain such steps — those steps are void.
- Plans are `yyyy-mm-dd-<type>-<short-desc>.md` in `.agents/plans/` (`<type>` = `feat`|`bug`|`debug`|`dep`|…).

### Engineering workflow (Linear ↔ GitHub)

This repository has used the **full pull-request flow** since July 24, 2026.
These rules are public-safe and self-contained; access to any other repository
is not required.

- Linear is the source of truth for work and GitHub is the source of truth for
  code. Use an issue in **Pictaria Server**, or a repository-specific sub-issue
  of a cross-product **Pictaria Launch** issue.
- Before implementation, read the issue and linked evidence, inspect the
  current code and worktree, confirm there is no existing branch or PR, and
  move the issue to **In Progress**.
- Use one focused branch per issue, normally `pic-XX-short-description`.
  Start PR titles with `PIC-XX:`. On branches intended for a public repository,
  never use a personal-name prefix.
- Every tracked change, including repository documentation, requires a pull
  request. Do not commit directly to `main`.
- Use `Fixes PIC-XX` only when merging the PR completes the whole issue. Use
  `Related to PIC-XX` when deployment, another repository, restore/upgrade
  testing, or any other acceptance criterion remains.
- PR descriptions must explain the outcome, material changes, validation,
  risks/rollback, remaining work, and any material agent authorship or
  independent agent review.
- Run validation proportionate to risk: the relevant tests, the full suite
  when warranted, container build/boot checks, and backup, restore, upgrade,
  or live-reference validation when required. A merge alone does not make an
  issue Done when deployment or operational acceptance remains.
- Preserve unrelated changes in dirty worktrees. Never put credentials,
  tokens, private photo data, household/network details, or sensitive logs in
  Linear, commits, branches, PRs, images, or public documentation.
- User-facing or operational behavior changes update the `CHANGELOG.md`
  Unreleased section and durable documentation in the same PR.
- Naming: the app is **Pictaria Frame** — or **Frame** when "Pictaria" was
  just named. Never "the frame app" or a generic lowercase "frame" when the
  product (rather than a physical device) is meant.

## Verification commands

```bash
npm test                  # node --test — full suite
npm run check:dco         # DCO check
npm run check:publication # publication check
```

## References (read on demand, not upfront)

- `README.md` — product overview and feature pages.
- `docs/` — durable documentation (e.g. `docs/ALBUMS.md`).
- `CHANGELOG.md` — Unreleased section duty for user-facing changes.
