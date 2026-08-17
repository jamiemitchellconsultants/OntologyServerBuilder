# Prompt A-19 — Accept Keycloak access tokens and answer the MCP OAuth challenge

Using the previously supplied repository contract, add a `keycloak` authentication mode to the HTTP
MCP transport and publish the OAuth protected-resource metadata that lets an MCP client obtain a
token for itself. This stage changes authentication and discovery only. It must not widen the MCP
surface by a single tool, resource, or argument, must not change ontology facts or generated
artifacts, and must not weaken any existing runtime trust boundary.

This stage depends on the in-process authentication from Prompt A-18 and the deployment path from
Prompt B-02.

## What this supersedes

Prompt B-02 states that `MCP_AUTH_MODE=static` is *the only acceptable homelab mode*. That is no
longer true, and this stage replaces it: `keycloak` becomes the ordinary home-lab mode and `static`
becomes the fallback. Prompt A-18's mode table is extended, not rewritten — `static`, `entra` and
`none` keep their current behaviour exactly, including the fail-closed rule for an unset or unknown
mode. Do not edit Prompt A-18 or 30; this prompt is the later instruction and says so.

Nothing about production changes. `entra` remains the production mode, and the corporate Entra
instance remains the only production identity provider.

## Why this stage exists

The home lab's users reach this server from a tier-1 conversational agent — Claude, ChatGPT,
Copilot, Gemini, and the mainstream open-source desktop and web agents. Those clients cannot be
handed a bearer token in a configuration file. They expect to be refused, told where the
authorization server is, register themselves, send the user to sign in, and retry. A server that
only knows how to compare a fixed string is unreachable from all of them.

So this stage is as much about the 401 as about the token: the challenge and the metadata document
are the parts that make the mode usable, and a `keycloak` verifier without them would authenticate
nobody who was not already holding a token from somewhere else.

## Read before editing

Inspect the repository first and adapt every name and path below to its current structure rather
than assuming these are the names it uses:

- `src/auth.ts`, or whatever module Prompt A-18 created for the verifier chain and the bearer
  middleware;
- `src/server.ts` and the HTTP transport wiring, including where Express routes are registered and
  which of them are deliberately unauthenticated;
- the installed MCP SDK's bearer-auth middleware, its protected-resource metadata router, and its
  helper for deriving the metadata URL — prefer these over hand-written equivalents, and read the
  installed version's types rather than assuming an older or newer shape;
- `.env.example`, `Dockerfile`, and the deployment configuration and scripts;
- `README.md`, `docs/architecture.md`, and the security and operations documentation;
- the HTTP transport, host-validation and authentication tests;
- `package.json`, the lockfile, and the repository's agent instructions and Narrative rules.

## The mode

Extend the mode table with one row:

| Mode | Purpose | Required configuration |
|---|---|---|
| `keycloak` | Home-lab validation of Keycloak-issued access-token JWTs | `KEYCLOAK_ISSUER`, `KEYCLOAK_AUDIENCE`; optional `KEYCLOAK_JWKS_URI`, `KEYCLOAK_REQUIRED_ROLES`, `KEYCLOAK_REQUIRED_SCOPES` |

Names follow the `ENTRA_*` convention the repository already uses, so the two JWT modes read the
same way.

Validation is the ordinary RS256 bearer path and must reuse the existing verifier contract and
middleware rather than introducing a parallel one: fetch and cache the JWKS keyed by `kid`, verify
the signature, then check `iss`, `aud` and `exp`. Use the maintained JOSE implementation already in
the dependency tree. Do not hand-roll key rotation, clock-skew tolerance, algorithm pinning, or
`kid` cache invalidation — those are precisely the details a bespoke validator gets wrong.

`KEYCLOAK_JWKS_URI` defaults to `${KEYCLOAK_ISSUER}/protocol/openid-connect/certs`. Derive it, and
allow the override for controlled testing.

`KEYCLOAK_REQUIRED_ROLES` and `KEYCLOAK_REQUIRED_SCOPES` are optional and **default to empty**, and
a deployment that sets neither is a supported, ordinary configuration — not a degraded one. The home
lab has no role differentiation: every authenticated user is equally authorised, and the realm
mints the same constant claims for everyone. Implement the checks for the deployments that will
want them later, honour them when set, and do not warn, log, or document the empty case as a
weakness. A valid token from the configured issuer, for the configured audience, is the whole
authorization decision in that configuration, and that is the intended design.

Derive the client identifier the same way `entra` mode does — `azp`, then `appid`, then `sub` —
without trusting it as proof beyond the already-validated token.

## The challenge and the metadata

This is the substantive addition, and the part that has no counterpart in `static` mode.

Add one more required setting for `keycloak` mode:

| Variable | Meaning |
|---|---|
| `MCP_RESOURCE_URI` | The canonical, externally reachable HTTPS URI of this server's MCP endpoint, ending in the transport's fixed path (`https://<host>/mcp`) |

Requirements:

- An unauthenticated request to the MCP endpoint must be answered `401` with a `WWW-Authenticate:
  Bearer` header naming the protected-resource metadata document, per RFC 9728. The SDK's bearer
  middleware takes the metadata URL as an option; pass it rather than assembling the header by hand.
- Serve the protected-resource metadata document itself, unauthenticated. It must advertise
  `MCP_RESOURCE_URI` as the resource and `KEYCLOAK_ISSUER` as the authorization server.
- Serve it at **both** the path-suffixed form the SDK's metadata router registers for a resource
  with a path (`/.well-known/oauth-protected-resource/mcp`) **and** the bare
  `/.well-known/oauth-protected-resource`. RFC 9728 specifies the first; a number of clients request
  the second and give up if it 404s. Serving both is a few lines and removes a class of failure that
  presents as an agent silently refusing to connect.
- `MCP_RESOURCE_URI` must be an absolute `https` URI with no credentials, query, or fragment, whose
  path is the MCP endpoint path. Reject anything else at startup with a message that says what was
  wrong. A client compares this value against the URL it actually dialled and abandons the flow when
  they differ, so a plausible-looking wrong value fails in a way that is very hard to read from the
  client end.
- Startup in `keycloak` mode must fail closed when `MCP_RESOURCE_URI`, `KEYCLOAK_ISSUER` or
  `KEYCLOAK_AUDIENCE` is missing or malformed, in the same style as the existing mode validation.
- The metadata endpoints and `/health` stay unauthenticated. Every other route keeps whatever
  protection it has today.

Do not implement an authorization server, a token endpoint, dynamic client registration, a session
store, or token introspection. This server is a resource server. Keycloak is the authorization
server and the client talks to it directly.

## Trust boundary

Unchanged, and worth restating because this stage is the one that invites a misreading: the MCP
server validates the caller's token in-process. The fronting proxy terminates TLS and forwards; it
authenticates nobody, and no configuration in this stage may delegate validation to it or trust a
header it sets.

The audience this mode is deployed with is shared by every MCP server in the home lab, so a token
minted for one is accepted by all of them. That is a deliberate home-lab decision made in the
deployment, not a property of this code, and this stage neither depends on it nor entrenches it: a
deployment that wants a dedicated audience sets `KEYCLOAK_AUDIENCE` to one and nothing here changes.
Document it as the deployment's choice, and do not describe the shared audience as a security
property.

## Tests

- A token signed by a test JWKS, with the configured issuer and audience, is accepted.
- Wrong audience, wrong issuer, expired, `alg: none`, and a signature from an unknown key are each
  rejected, and the rejection does not leak the token or key material into logs.
- Required roles and scopes are enforced when configured, and their absence is accepted when they
  are not configured.
- An unauthenticated MCP request returns 401 carrying a `WWW-Authenticate` header whose
  `resource_metadata` points at the served document.
- Both metadata paths return the same document, unauthenticated, and it names the configured
  resource and issuer.
- Startup fails closed on each missing or malformed `keycloak` setting, with a distinguishable
  message for each.
- The existing `static`, `entra` and `none` tests still pass unchanged. If any needed editing to
  accommodate this stage, that is a signal the change was not additive — reconsider rather than
  adjust the test.

## Documentation

Update `.env.example`, the README's authentication section, the architecture document, and the
deployment guide to cover the new mode, including:

- the exact settings, their defaults, and which are required;
- that `keycloak` is the ordinary home-lab mode and `static` remains available for clients that run
  no OAuth flow and for when the identity provider is unavailable;
- that these are two modes and not a fallback chain — an unreachable Keycloak does not silently
  accept the static token;
- that production remains `entra`, unchanged;
- what an operator must configure on the Keycloak side (a realm, a client whose tokens carry the
  configured audience, and users), stated as requirements rather than as a click-through, since the
  identity provider is not this repository's to script.

State plainly that Prompt B-02's static-only rule is superseded, and where.

## Acceptance criteria

- A conversational agent given only `https://<host>/mcp` can complete the flow: refused, discovers
  the authorization server from the metadata document, obtains a token, and calls the MCP endpoint
  successfully.
- `static`, `entra` and `none` behave exactly as before, and an unset or unknown mode still fails
  closed.
- No token, key, or secret appears in the image, repository, Compose model, logs, or generated
  documentation.
- The MCP surface, ontology facts, and generated artifacts are unchanged by this stage.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This is a meaningful security decision. If opening a pull request, apply `narrative-required` and
include substantive `## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences`
sections recording why a conversational agent needs the challenge and not just the verifier, why the
role and scope checks are empty by default in this deployment, that the audience is shared across
the home lab by deployment choice, and that Prompt B-02's static-only rule is superseded. Never
hand-edit generated `Narrative.md`.

Commit locally with a focused message. Do not push.
