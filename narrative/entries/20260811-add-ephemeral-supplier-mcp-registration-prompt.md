---
date: 2026-08-11
slug: add-ephemeral-supplier-mcp-registration-prompt
title: "Add ephemeral supplier MCP registration prompt"
summary: "Add Prompt 49 as a separate stage and leave Prompt 48 unchanged."
kind: product
status: accepted
sequence: 2026-08-11T04:26:32.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/35; merge commit 3b6a34c5b7234eb80ab14cae7aa04117ff27d291"
---

## Context

Prompt 48 packages a client-side skill for users who already connected to a supplier MCP server, but a separate product option is needed for OntologyService to perform one attributable discovery attempt itself. That option conflicts with previously played no-fetch and separately supplied artifact constraints. Earlier prompts are historical stages and cannot be rewritten after they have been applied, so the new stage must state exactly which clauses it supersedes and preserve every unaffected boundary.

## Decision

Add Prompt 49 as a separate stage and leave Prompt 48 unchanged. Prompt 49 specifies two capability-gated tools, a short-lived browser handoff for client credentials or authorization code with PKCE, bounded and SSRF-resistant supplier MCP discovery, immutable content-addressed catalog evidence, structural normalization, and review-required intake submission. Credentials and tokens exist only for the registration attempt. Prompt 49 explicitly supersedes the conflicting behavior from Prompts 18, 33, 34, 35, 36, and 48; Prompt 47 remains the historical pre-change audit. The rejected alternatives were editing Prompt 13 retroactively, replacing Prompt 48, accepting secrets in MCP tool arguments, persisting supplier credentials, or widening network access across the delivery plane.

## Consequences

Future builders gain a direct-registration option without erasing the existing client-captured path or rewriting completed stages. Applying Prompt 49 will deliberately expand OntologyService's intake architecture with outbound MCP/OIDC traffic, browser callbacks, ephemeral state, and raw catalog evidence custody, which increases implementation and operational complexity and requires substantial security testing. The compiler, SPARQL, mapping tools, ordinary delivery handlers, and all non-registration paths retain their no-network boundary. Prompt 47 cannot be cited as assurance for the new path; Prompt 49 carries its own acceptance evidence and will require later independent review if that assurance is desired.
