---
date: 2026-08-02
slug: add-prompt-32-keycloak-access-tokens-and-the-mcp-oauth-challenge
title: "Add prompt 32: Keycloak access tokens and the MCP OAuth challenge"
summary: "Add `keycloak` as a fourth mode rather than change any existing one, and specify the discovery path as the substantive part of the stage rather than a detail of it."
kind: product
status: accepted
sequence: 2026-08-02T12:30:04.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/20; merge commit 18199327d01e91addc438178a22e80115b6a8046"
---

## Context

The home lab's users reach the ontology server from a tier-1 conversational agent — Claude, ChatGPT, Copilot, Gemini, and the mainstream open-source desktop and web agents. Prompt 30 was written when the callers were assumed to be tools that could be handed a bearer token in a configuration file, and it fixed `static` as the only acceptable home-lab mode on that basis.

That assumption does not survive the actual clients. Such an agent cannot be given a token: it expects to be refused, told where the authorization server is, register itself, send the user to sign in, and retry. A server that only knows how to compare a fixed string is unreachable from all of them.

The deployment side of this is already built — the LocalAI cluster now stands up a Keycloak realm whose tokens carry the audience and claims this mode needs (jamiemitchellconsultants/LocalAI#29) — so the constraint here is a real one with a real consumer waiting on it, not a speculative capability.

## Decision

Add `keycloak` as a fourth mode rather than change any existing one, and specify the discovery path as the substantive part of the stage rather than a detail of it. A `keycloak` verifier without the challenge and the metadata document would authenticate nobody who was not already holding a token from somewhere else, which is precisely the population that made `static` insufficient.

Two specifics were decided against the obvious alternative:

Role and scope checks default to empty, and the prompt states explicitly that this is an ordinary supported configuration and not a degraded one, with no warning logged. The home lab has no role differentiation. The obvious alternative — treat an empty check as suspicious and warn — produces a server that logs a warning on every correct deployment, which trains operators to ignore the log.

The metadata document must be served at both the RFC 9728 path-suffixed location and the bare well-known path. Serving only the specified one is defensible and loses every client that requests the other, silently: the user sees an agent that will not connect and no error anywhere useful.

## Consequences

`static` becomes the fallback rather than the home-lab norm, kept for clients that run no OAuth flow and for when the identity provider is unavailable. The two remain modes and not a fallback chain — an unreachable Keycloak does not silently accept the static token.

The audience the mode is deployed with is shared by every MCP server in that home lab, so `aud` no longer distinguishes one server from another there. The prompt neither depends on nor entrenches that: it is the deployment's choice, a dedicated audience is one setting away, and the prompt forbids describing the shared audience as a security property.

Prompt 30's text is not rewritten, so a reader of Prompt 30 alone will still find the static-only rule. Prompt 32 and the README both state the supersession; that is the cost of the append-only prompt convention and is accepted rather than worked around.

Deliberately left open: the ontology server does not become an authorization server, and dynamic client registration remains entirely the identity provider's concern.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
