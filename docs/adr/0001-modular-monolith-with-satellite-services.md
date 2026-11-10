# ADR 0001: Modular monolith with satellite services

- **Status:** Accepted
- **Date:** 2026-11-07

## Context

PayLab has a core domain (accounts, charges, ledger, refunds, webhooks) that must
stay strongly consistent. It also has parts with different needs: a payment
provider simulator, a risk scorer and an AI service, which use different
libraries and scale differently. One developer will build and operate everything.
Full microservices would multiply deployments, pipelines, contracts and failure
modes before the domain is even understood.

## Decision

Build the core as a modular monolith in Django: one deployable unit with clear
internal modules. Extract only the parts with a real reason to be separate
(`psp-simulator`, `risk-service` and `ai-service`) as small FastAPI services that
communicate through events and HTTP.

## Consequences

- Local database transactions keep the ledger consistent, with no distributed
  transactions in the core flows.
- One pipeline and one deployment for the core means fewer moving parts.
- Module boundaries must be enforced by convention and by tests, because the
  language does not enforce them.
- A module can be extracted later if a real need appears (scale, ownership).
- Satellite services need explicit contracts (OpenAPI and event schemas).

## Alternatives considered

- Full microservices: rejected, too much operational overhead for one developer
  and a domain that is not yet proven.
- A single monolith that also contains the AI and the PSP simulator: rejected,
  it mixes different runtime needs and lets AI experiments put the core at risk.
