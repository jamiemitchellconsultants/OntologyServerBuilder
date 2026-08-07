---
date: 2026-08-07
slug: add-prompt-32a-deploy-keycloak-mode-on-the-home-lab
title: "Add prompt 32a: deploy keycloak mode on the home lab"
summary: "Add a separate prompt rather than amend Prompt 32."
kind: product
status: accepted
sequence: 2026-08-07T19:32:09.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/22; merge commit e2ffc2f9be3fc40c7187d34401911eb99396e9a0"
---

## Context

Prompt 32 added a `keycloak` authentication mode and the RFC 9728 challenge to OntologyService, and
declared `keycloak` the ordinary home-lab mode with `static` as the fallback. Applying it exposed a
gap the prompt did not cover: the mode is reachable in the server and unreachable in the deployment.

Three concrete findings from that application, all in `deploy/homelab/`:

- `nginx/ontology.conf` proxies exactly `= /health` and `= /mcp` and returns 404 for everything else,
  so the protected-resource metadata paths the server now publishes are hidden by the proxy. The
  server's own tests cannot see this, and the client-side symptom is an agent that silently declines
  to connect.
- `compose.yaml` hard-codes `MCP_AUTH_MODE: static`, mounts the `mcp_auth_token` secret, and requires
  `MCP_AUTH_TOKEN_FILE_HOST` at render time. There is no token file in `keycloak` mode, so the model
  cannot render for it at all.
- `Validate.ps1` and `test/deployment.test.ts` assert the static model specifically, so both would
  stay green while the deployment served a mode nobody could reach.

Prompt 32's application recorded the required edits in the deployment guide's prose. Prose is not a
deployment, and a documented mode with no shipped way to run it is the kind of gap that is discovered
by an operator at the point of use rather than by a gate.

## Decision

Add a separate prompt rather than amend Prompt 32. Prompt 32 is scoped to authentication and
discovery in `src/`, and it landed correctly against that scope; widening it retrospectively would
destroy the evidence that the deployment consequence was not seen at the time. 32a carries the
deployment change on its own terms, and the `a` suffix keeps it adjacent to the prompt it completes
without renumbering 33 onwards.

The prompt states requirements and leaves shape to inspection, consistent with the rest of the
series. Where a decision is genuinely open it says so rather than pre-deciding:

- the Compose shape is left to the author, because Compose has no conditional secret and neither a
  base-plus-overlay nor two complete models is obviously right — but whichever is chosen must keep
  the `${VAR:?message}` fail-to-render property that is currently load-bearing;
- the proxy change is pinned down precisely — the two well-known paths as exact matches, never a
  prefix match over `/.well-known/` — because the failure it prevents is invisible and the wrong fix
  publishes more surface than the deployment intends;
- the ontology container's `internal: true` network is called out as the one place `keycloak` mode
  may genuinely change the egress posture, since the JWKS is fetched from Keycloak. The prompt
  requires that conclusion be reached explicitly and recorded, either way, rather than relaxed
  quietly to make the mode work.

On validation, the prompt refuses the tempting shortcut. There is no fixture credential for
`keycloak` mode without standing up an authorization server, and a test-only token path in the server
would reintroduce the second credential the mode exists to remove. So the throwaway stack proves
everything reachable without an identity provider — the 401 and its `WWW-Authenticate` target, both
metadata documents over TLS, host validation on those paths, the 405s, the container controls — and
the real-token and real-agent checks are printed as manual gates and recorded as unexecuted.

## Consequences

The home-lab deployment becomes two deployable configurations rather than one, chosen in the
environment file. That is more to keep consistent, and the prompt accepts the cost explicitly:
choosing is an operator decision in configuration, never a runtime fallback, because an unreachable
Keycloak must refuse callers rather than accept a shared token.

`static` keeps every control it proves today; the prompt forbids generalising an existing assertion
into one that asserts less about the static model.

What remains deliberately open after 32a: the end-to-end flow is still unproven. No realm exists to
test against, so a real agent completing discovery, sign-in, and a tool call stays an unexecuted
manual gate. The guide's convention holds — an unrecorded gate is not a pass, and the throwaway
validation passing must not be read as evidence the flow works.

The AWS production path (Prompt 31) is untouched. Production remains `entra`.
