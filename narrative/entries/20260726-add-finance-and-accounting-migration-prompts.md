---
date: 2026-07-26
slug: add-finance-and-accounting-migration-prompts
title: "Add finance and accounting migration prompts"
summary: "Add a nine-stage prompt sequence that plans first, introduces additive standards-alignment layers before compatibility-sensitive migration, and concludes with an independent audit."
kind: product
status: accepted
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/14; merge commit b48c69ad72d9fd699c5d2bab80df41a7864ea0cb"
---

## Context

OntologyService's thin finance reference model needs a governed path to a richer finance and accounting semantic layer for a UK-based group with international operations. A single implementation prompt would make compatibility, licensing, reporting-profile, provenance, and trust-boundary decisions difficult to review independently.

## Decision

Add a nine-stage prompt sequence that plans first, introduces additive standards-alignment layers before compatibility-sensitive migration, and concludes with an independent audit. Pin FIBO to the Q1 2026 `master_2026Q1` release and require the implementation to record its resolved commit and selected module hashes rather than following the moving `master` branch. Treat any later FIBO release as a separate governed upgrade.

## Consequences

The sequence gives implementers bounded changes, explicit review and rollback points, and testable governance expectations. It is intentionally longer than a single migration prompt and requires release references to be maintained. Standards packages remain reference-only until licensing and snapshot approval are recorded, and subsequent FIBO upgrades require their own reviewed decision.
