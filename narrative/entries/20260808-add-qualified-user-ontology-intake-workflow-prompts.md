---
date: 2026-08-08
slug: add-qualified-user-ontology-intake-workflow-prompts
title: "Add qualified-user ontology intake workflow prompts"
summary: "Add a 15-stage Builder sequence that separates a mutable, capability-gated intake plane from the immutable delivery plane. Submitted normalized definitions and change proposals remain outside ontology compilation."
kind: product
status: accepted
sequence: 2026-08-08T09:12:23.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/26; merge commit 99c21a0cd027c6668d919de897f101ef4f4f85dc"
---

## Context

The existing Builder sequence supports governed source ingestion and a read-only compiled ontology, but qualified conversational users cannot durably register new API, MCP, OpenAPI, or WSDL definitions, inspect deployed ontology changes, or request reviewed entity-mapping tools through one controlled workflow.

## Decision

Add a 15-stage Builder sequence that separates a mutable, capability-gated intake plane from the immutable delivery plane. Submitted normalized definitions and change proposals remain outside ontology compilation. Engineers export and analyze them using deterministic, cached-embedding, and bounded coding-agent semantic passes, then apply reviewed changes through the OntologyService pull-request and CI/CD process. Approved declarative mappings compile into named, deterministic, pure MCP tools.

## Consequences

Qualified users gain durable submission receipts, deployed-release visibility, and access to approved mapping tools, but cannot retrieve pending submissions or apply ontology changes automatically. Engineers retain responsibility for evidence review, prompt execution, pull requests, deployment, and mapping approval. The initial SQLite intake store is explicitly single-instance; multi-instance deployment requires a later governed storage adapter. Runtime ontology and mapping operations remain offline from model providers and source-definition retrieval.
