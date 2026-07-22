# Prompt 1 — Project scaffold and domain model

Using the previously supplied repository contract, implement Stage 1: project scaffold and semantic
domain model.

Create a Node.js 22 TypeScript ESM project with:

- strict TypeScript configuration;
- `src`, `test`, `docs`, `ontology/sources`, `ontology/systems`, and `ontology/compiled` directories;
- `build`, `test`, `check`, `ontology:compile`, `ontology:check`, `dev`, and `start` scripts;
- stable JSON-compatible types for:
  - system manifests;
  - entities, workflows, and operations;
  - attributes and provenance;
  - governed relationships;
  - compiler configuration;
  - semantic mapping candidates;
  - manual mapping decisions;
  - compiled ontology and review artifacts.

Stable entity IDs must use `<system-id>.<entity-slug>`.

Mapping dispositions must include:

- `accepted-auto`
- `accepted-human`
- `review-required`
- `unmatched`
- `rejected`
- `confirmed-unmatched`

Create these initial canonical finance concepts:

- FinancialDocument
- PurchaseOrder
- Invoice
- PaymentInstruction

Create these governed canonical relationships:

- PurchaseOrder “is billed by” Invoice
- Invoice “is settled via” PaymentInstruction

Add validation utilities and deterministic JSON serialization. Avoid timestamps or
environment-specific data in generated output.

## Acceptance criteria

- `npm install` succeeds.
- `npm run build` succeeds.
- Model and stable-ID unit tests pass.
- The repository contains no runtime database or external service dependency.

Commit the completed stage locally with a focused commit message, but do not push.
