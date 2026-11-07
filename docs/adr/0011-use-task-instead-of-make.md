# ADR 0011: Use Task instead of make as the task runner

- **Status:** Accepted
- **Date:** 2026-11-07

## Context

The design document (v0.1) planned a `Makefile` with commands such as `make up`.
Development happens on native Windows (PowerShell and Docker Desktop). `make` is a
Unix tool: on Windows it needs extra installation, and recipes behave differently
depending on the shell in use, which makes them fragile.

## Decision

Use Task (`Taskfile.yml`) as the project's task runner, with the same shortcuts
(`task up`, `task down`, `task test`, ...).

## Consequences

- The same command works on Windows, Linux, macOS and in CI.
- Taskfiles are plain YAML and easy to read.
- Contributors must install one extra tool (`winget install Task.Task` on Windows).
- Some people know `make` better than Task.


## Alternatives considered

- `make` via WSL: rejected, because we develop on native Windows.
- `make` for Windows through third-party packages: rejected, fragile.
- PowerShell scripts: rejected, they do not run on Linux or in CI without changes.
- `just`: similar to Task, but less common in the projects we studied.
