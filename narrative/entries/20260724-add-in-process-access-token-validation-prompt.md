---
date: 2026-07-24
slug: add-in-process-access-token-validation-prompt
title: "Add in-process access-token validation prompt"
summary: "Add a new post-positioning prompt that directs a coding agent to introduce one in-process bearer-authentication middleware with verifiers selected at HTTP startup."
kind: product
status: accepted
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/12; merge commit dce655d5598b28ddf3541a37aa0796d2acebf2da"
---

## Context

OntologyService now validates HTTP access tokens inside the MCP resource server because its production security policy does not permit delegating that trust decision entirely to a gateway. OntologyServerBuilder currently ends before teaching this later architecture change, leaving learners with the earlier gateway-authentication boundary and no staged path for reproducing the Entra and local-development behavior.

The new stage must remain usable across repositories reconstructed from the earlier prompts, preserve their deterministic read-only runtime and existing transport protections, and provide testable security requirements without depending on live Entra infrastructure or real credentials.

## Decision

Add a new post-positioning prompt that directs a coding agent to introduce one in-process bearer-authentication middleware with verifiers selected at HTTP startup. The prompt requires Entra JWT validation for production, a constant-time fixed token for trusted local deployments, an explicit `none` mode, and fail-closed HTTP startup when authentication configuration is missing or invalid.

It also requires local-JWKS tests for token signature and claim failures, HTTP-boundary tests proving rejection occurs before MCP handling, preservation of stdio and host validation, copyable deployment documentation, and removal of contradictory current-state claims. A narrowly implementation-specific patch was rejected in favor of a repository-adaptive prompt, while retaining explicit acceptance criteria so the security boundary cannot be weakened by interpretation.

## Consequences

Learners can now reproduce the service's current authentication architecture through the same staged, evidence-driven workflow as earlier milestones. The sequence grows by one decision-bearing stage and asks learners to understand JWT validation, JWKS behavior, role and scope enforcement, environment configuration, and the division of responsibilities between the service and its gateway.

Implementations produced from the prompt gain a JOSE dependency and stricter HTTP startup requirements. Static and unauthenticated modes remain available only for trusted local use, while production deployments must supply correct Entra configuration and retain TLS and other edge controls. The generated `Narrative.md` remains untouched; the repository's post-merge Narrative workflow will propose the authoritative entry separately.
