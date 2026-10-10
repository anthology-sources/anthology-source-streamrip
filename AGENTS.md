# AGENTS.md – anthology-source-streamrip

Instructions for AI coding agents (Claude Code, opencode and others) working in this repository.
`CLAUDE.md` only imports this file; keep all rules here.

## Project

`anthology-source-streamrip` is an optional **source service** for Anthology. It wraps [streamrip](https://github.com/nathom/streamrip) and exposes it through the Anthology **source service interface v1**. The Anthology core lives in a separate repository and talks to this service only over HTTP inside the Anthology pod.

- Contract (source of truth): [`docs/architecture/source-service-api.md`](https://github.com/anthology-music/anthology/blob/main/docs/architecture/source-service-api.md) in the core repository.
- Decision and rationale: [ADR 0017](https://github.com/anthology-music/anthology/blob/main/docs/adr/0017-source-services-outside-core.md).
- This service never decides about duplicates, quality or the library – the core does. It searches, resolves and downloads.

## Language

- Conversation: reply in the language the user writes in.
- Everything in the repository is English: code, identifiers, comments, commit messages and documentation.

## Rules (do not break)

- **Implement the contract exactly.** Endpoints, fields and semantics follow the core's `source-service-api.md`. Breaking changes need a new version prefix agreed in the core repository first.
- **Write only into the `imports` volume** (`/data/imports`), never into the library or any other path.
- **Credentials** are passed by the core per request or at login. Never log them, never return them, never persist them outside the service's own state volume.
- **Respect the upstream service:** rate limits, back-off on errors, no circumvention of DRM or access controls. Report rate limits so that the core can make items wait instead of failing.
- **Use streamrip as a library** where it offers an API; do not parse its command-line output unless there is no alternative.
- **Report progress** per item; downloads must be resumable or restartable after a container restart.
- **Health:** `GET /v1/health` reports version, upstream reachability and login state.

## Structure

```
anthology_source_streamrip/
  api/         FastAPI routes of the v1 interface (info, search, resolve, download, tasks, health)
  adapter/     thin wrapper around streamrip (search, resolve, download)
  tasks/       download tasks, progress, server-sent events, restart recovery
  core/        configuration, paths, logging, version
tests/
  unit/        tests of adapter mapping and task handling with streamrip mocked
  contract/    tests against the v1 contract schema of the core
deploy/
  quadlet/     Quadlet file for the Anthology pod
  compose.snippet.yaml
```

## Stack

- Python 3.12+, managed with **uv**; **FastAPI**, **pydantic** v2, **httpx**.
- Tests: **pytest** + **pytest-asyncio**; lint/format: **ruff**.
- Container: OCI image built from a `Containerfile`, published as `ghcr.io/anthology-sources/anthology-source-streamrip`.

## Commands

```
uv sync && uv run pytest && uv run ruff check . && uv run ruff format --check .
podman build -t anthology-source-streamrip -f Containerfile .
```

Run the tests and linters before finishing a task.

## Tests

- Never download real content in tests or CI. streamrip is mocked; media fixtures are generated (sine tones, silence).
- Contract tests validate every response against the schema published by the core.

## Git

- Conventional Commits (`feat(api): …`, `fix(adapter): …`).

## License

This repository is **GPL-3.0-only**. It must stay GPL-3.0-only because it uses streamrip. It is never imported by the MIT core.

## Never

- Commit credentials, tokens, cookies or `.env` files.
- Add copyrighted audio, video or cover art to the repository.
- Write outside `/data/imports`.
