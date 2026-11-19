# Changelog

All notable changes to this project are documented in this file. The format is
based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this project
follows [Semantic Versioning](https://semver.org/).

## [0.1.0] - 2026-11-14

### Added

- Monorepo structure, MIT license, `.editorconfig` and `.gitattributes`.
- Design document v1.0 and ADRs 0001, 0011 and 0012.
- Pre-commit hooks: hygiene checks, gitleaks, ruff and Conventional Commits.
- Local infrastructure with Docker Compose (PostgreSQL with pgvector, Redis and
  RabbitMQ) and Task shortcuts.
- GitHub Actions CI (lint and infrastructure smoke test) and Dependabot.
- Contributing guide and pull request template.
