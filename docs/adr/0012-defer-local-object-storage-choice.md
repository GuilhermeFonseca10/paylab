# ADR 0012: Defer the choice of local object storage to Phase 8

- **Status:** Accepted
- **Date:** 2026-11-07

## Context

The design document planned MinIO as local S3-compatible storage. The MinIO
community edition stopped publishing Docker images and binaries in October 2025,
and its repository was archived in 2026. A dependency we cannot pull is a risk
for every local and CI environment. PayLab only needs object storage in Phase 8
(documents for the AI copilot and exports).

## Decision

Do not include object storage in the local infrastructure for now. Choose a
replacement in Phase 8, among SeaweedFS, Garage and RustFS. Access the storage
only through an S3 client (such as boto3) with a configurable endpoint, so that
changing the provider is a configuration change.

## Consequences

- The Phase 0 Docker Compose has PostgreSQL, Redis and RabbitMQ only.
- Where the design document says MinIO, read "S3-compatible storage".
- The final choice needs a new evaluation (maintenance, license, image
  availability) before Phase 8.

## Alternatives considered

- Build MinIO from source: rejected, upstream is archived and we would own the
  maintenance.
- Use AWS S3 directly in development: rejected, it needs an account and can cost
  money.
