---
date: 2026-08-07
slug: withdraw-prompt-32a-add-32b-for-jwks-retrieval-failures
title: "Withdraw prompt 32a; add 32b for JWKS retrieval failures"
summary: "Withdraw 32a by supersession rather than deletion, and replace it with a prompt for a defect that is real in the deployment that actually exists."
kind: product
status: accepted
sequence: 2026-08-07T20:11:14.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/24; merge commit cacb0169686b83c1d777467c334def91f94cfaab"
---

## Context

Prompt 32a was merged in #22. It was written on a premise that does not hold: that
`OntologyService/deploy/homelab/` describes how the service is deployed.

It does not. The deployment lives in the LocalAI repository — `docs/setup-ontology-mcp-windows.ps1`
stands the ontology container up on the external `mcp-public` Docker network behind **one shared
Caddy ingress** at `C:\mcp-host` that owns ports 80 and 443, with the Keycloak realm created by
`setup-mcp-host-windows.ps1`. That script already deploys prompt 32's contract — issuer, audience,
JWKS URI, and `MCP_RESOURCE_URI` — with `keycloak` as its default mode.

Checked against that, each of 32a's three premises fails:

- **The proxy gap it centres on is not real.** Caddy's drop-in is a bare
  `reverse_proxy ontology-service:3000` with no path matchers, so the protected-resource metadata
  paths are already forwarded. The exact-match `location` list belongs to OntologyService's own
  nginx file, which nothing runs.
- **A second Compose model would fight the real ingress for ports.** The LocalAI script explicitly
  stopped running its own Caddy because only one process can bind 80 and 443.
- **The egress deliberation was moot.** `mcp-public` is how the container reaches Keycloak, so the
  `internal: true` question and the `identity` network invented to answer it solve nothing.

The error was one of scope, not detail: a prompt in this repository directed OntologyService to
build ingress, TLS termination, and identity-provider wiring that another repository owns and had
already built.

## Decision

Withdraw 32a by supersession rather than deletion, and replace it with a prompt for a defect that is
real in the deployment that actually exists.

**Withdrawn, not deleted.** 32a merged and its narrative entry was accepted, so removing the file
would destroy the evidence that the framing ever needed correcting — the same rule this repository
applies to narrative entries. The banner states what the real deployment is and how each premise
fails, and the body is left intact: its reasoning about what a deployment must *prove*, and about
the honest limits of validating an OAuth flow with no authorization server, survives the premise it
was attached to.

**Boundary, stated in 32b so it is not re-crossed.** Ingress, TLS, and Keycloak are LocalAI's
responsibility; this repository's prompts direct what the container does with a token. 32b forbids
touching anything under `deploy/` and names 32a's mistake as the reason, because the next agent
reading `deploy/homelab/` will find a complete, self-consistent, unused deployment and draw the same
conclusion.

**32b addresses the split-horizon failure.** The realm's issuer is a public HTTPS name, pinned so
every relying party validates one canonical `iss`. The container sits behind the same router, so
resolving that name from inside needs a hairpin many routers do not do — which is precisely why
`KEYCLOAK_JWKS_URI` exists and why the deployment sets it. Key retrieval therefore has a failure
surface entirely independent of the token, and today every failure on it surfaces as
`401 invalid_token`: indistinguishable from a forgery, and aiming every diagnostic instinct at the
token, the realm, and the client, none of which are wrong.

The prompt draws the boundary rather than describing a fix. Token problems stay `401`, including the
case that most looks like retrieval and is not — a `kid` still unknown after a *successful* refresh
is a statement about the token. Retrieval problems become a retryable status whose challenge must
not claim `invalid_token`, because a client reading that discards a good token and starts a sign-in
that will also fail. The diagnostic detail goes to the log, never to a caller who is unauthenticated
by definition at that moment.

It also carries the prohibitions that matter more than the fix: no startup prefetch, because
container start must not depend on the realm; no hand-rolled retry against a realm already having a
bad day; no identity state in `/health`; no fallback. And it requires discovery to survive an
unreachable realm — the metadata documents are served from configuration, so an agent can still find
out where to sign in.

## Consequences

The prompt sequence now contains a withdrawn entry, and the README says so rather than quietly
renumbering. A reader working through in order meets the correction where the mistake was made.

32b changes `entra` as well as `keycloak`, which revises shipped behaviour from prompt 20 rather than
adding to it. That is stated in the prompt so the implementing stage argues it instead of presenting
it as additive.

What this does not resolve: `deploy/homelab/` still describes a deployment nobody runs, and now
contradicts the real one on image provenance (digest-pinned and tag-rejecting, against a box that
builds from a git build context and tags per git ref) and on the static token (a mounted secret,
against an environment literal). Deployment definitions and instructions are maintained in the
LocalAI repository, which generates changes to the MCP server repositories as needed, so that
reconciliation is that repository's to sequence — not this one's.
