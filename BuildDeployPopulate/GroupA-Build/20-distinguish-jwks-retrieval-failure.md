# Prompt A-20 — Tell a key-retrieval failure apart from an invalid token

Using the previously supplied repository contract, make the JWT authentication modes distinguish a
failure to **retrieve the signing keys** from a failure to **validate a token**. Today both surface
as `401 invalid_token`, which sends an operator to inspect the token, the realm, and the client —
none of which are wrong when the real fault is that the key set could not be fetched.

This stage changes error handling, logging, and the documentation of those two failure classes. It
must not change the MCP surface, ontology facts, generated artifacts, or which tokens are accepted.
A token that is valid today must still be accepted, and a token that is rejected today must still be
rejected, with the same status.

This stage depends on Prompt A-18 (the verifier chain) and Prompt A-19 (`keycloak` mode).

## What this stage is not

It is not a deployment change, and **nothing under `deploy/` may be touched**.

Prompt B-04 assumed `deploy/homelab/` describes the deployment and was withdrawn for it. The real
deployment lives in the LocalAI repository: the ontology container joins the external `mcp-public`
Docker network behind one shared Caddy ingress that owns ports 80 and 443, and the Keycloak realm is
created there too. Ingress, TLS, and the identity provider are LocalAI's responsibility. This
repository owns the container and what it does with a token — which is exactly where this defect
lives.

## Why this stage exists

The realm's issuer is a public HTTPS name, pinned deliberately so every relying party validates one
canonical `iss` regardless of which name it reached the realm by. The service container runs on the
same host as Keycloak, behind the same router. Resolving that public name from inside gives the
site's public address, and reaching it requires the router to hairpin the connection back inside —
which many do not.

So the deployment sets `KEYCLOAK_JWKS_URI` to an internally reachable address while `KEYCLOAK_ISSUER`
stays the public canonical one. That pairing is correct and supported; it also means key retrieval
has its own failure surface, independent of anything about the token.

When that surface fails — DNS, a refused connection, a timeout, a 502 from the proxy in front of the
realm, a body that is not a JWKS — the caller currently gets `401 invalid_token`, indistinguishable
from presenting a forged one. Every diagnostic instinct that 401 triggers is aimed at the wrong
half of the system. The whole cost of the incident is in that misdirection, because the fix
(hairpin, a container alias, or the `KEYCLOAK_JWKS_URI` override) is trivial once seen.

## Read before editing

Inspect the repository first and adapt every name below to its current structure:

- the authentication module: the verifier contract, both JWT verifiers, and how each translates a
  failure into the SDK's error types;
- the installed JOSE version's remote key-set helper — which errors it raises for a retrieval
  problem, which for a key that does not match, and what its refresh and cooldown behaviour is;
- the MCP SDK's bearer middleware: which error classes it maps to which status, and what it puts in
  `WWW-Authenticate`;
- `src/server.ts`, and which routes are unauthenticated;
- the authentication tests, and the local JWKS fixture they already stand up;
- `README.md`, `docs/architecture.md`, and the operations documentation;
- the repository's agent instructions and Narrative rules.

## The distinction

Two classes, and the boundary between them is the point of the stage.

**A token problem — unchanged, `401`, `invalid_token`.** Bad signature, wrong issuer, wrong or
missing audience, expired or not yet valid, an unacceptable algorithm, a malformed token, a missing
required role or scope. Also: a `kid` that is still unknown after the key set has been refreshed.
That last one looks like a retrieval failure and is not — the keys were fetched successfully and do
not contain the one the token names, which is a statement about the token.

**A key-retrieval problem — new, and must not be `invalid_token`.** DNS failure, connection refused,
TLS failure, timeout, a non-200 response, or a body that does not parse as a key set. The server
could not form an opinion about the token at all, and saying "invalid token" asserts something it
does not know.

Requirements:

- Answer a retrieval failure with a status that means *the server cannot serve this request right
  now*, not one that means *your credential is bad*. `503` is the honest one. Whatever is chosen,
  the `WWW-Authenticate` challenge must not claim `error="invalid_token"`, because an OAuth client
  reading that will discard a perfectly good token and start a new sign-in that will also fail.
- **The response body must not name the JWKS URI**, the issuer, or anything else about the internal
  topology. The caller is unauthenticated by definition at that moment. A generic retryable error is
  what it gets.
- **The server log must name the URI it tried**, say that the key set could not be retrieved, and
  include the underlying reason. That is the whole diagnostic value of this change, and it belongs
  on the operator's side of the boundary. Never log the presented token, any key material, or an
  end-user claim the repository does not already log.
- Apply the same treatment to **both** JWT modes. `entra` has the identical defect for the identical
  reason, and one shared code path is the honest fix; two would drift. This deliberately revises
  Prompt A-18's error behaviour, and the Narrative should say so rather than presenting it as
  additive.

## Prohibitions

Each of these is a plausible-looking move that makes the deployment worse:

- **Do not fetch the key set at startup.** Container start must not depend on the realm being up at
  that moment; the deployment sets the JWKS URI explicitly rather than discovering it for exactly
  that reason. Validation-time fetching with the library's cache is the intended behaviour.
- **Do not hand-roll caching, retry, or backoff.** The JOSE remote key-set helper already has a
  refresh cooldown; a per-request retry loop turns a brief realm outage into a stampede against it.
- **Do not make `/health` report identity-provider state.** It stays minimal and exposes nothing
  about the mode, the issuer, or the audience. An operator diagnosing this reads the log, and a
  liveness probe must not start failing because a *dependency* is down.
- **Do not fall back to another mode, another key source, or an unauthenticated path.** A server
  that cannot verify serves nobody.
- **Do not add a dependency.** No HTTP client, no second JWT library, no circuit-breaker package.

## Availability while the realm is down

State and test what still works, because it is more than it looks and it is what an agent needs:

- `/health` still answers, unauthenticated.
- In `keycloak` mode both protected-resource metadata documents still answer, unauthenticated, and
  still name the configured resource and issuer. Discovery is served from configuration and must not
  depend on the realm being reachable — an agent can still discover where to sign in, and Keycloak
  itself will tell it when it is back.
- `GET` and `DELETE` on `/mcp` still return 405.
- Only token validation fails, and it fails legibly.

Once the key set becomes retrievable again, a valid token must be accepted with no restart. A
transient failure must not poison the verifier.

## Tests

Extend the existing local-JWKS fixture rather than adding a new harness:

- an unreachable JWKS endpoint (nothing listening) produces the retrieval outcome, not `401`;
- a JWKS endpoint answering `500`, and one answering `200` with a body that is not a key set, both
  produce the retrieval outcome;
- the response body carries no URI, issuer, or audience;
- the log line names the attempted URI and the reason, and contains neither the presented token nor
  key material;
- a token whose `kid` is absent from a **successfully retrieved** key set is still `401`;
- every existing rejection — wrong audience, wrong issuer, expired, `alg: none`, unknown key,
  missing role or scope — still returns `401` with the same error code;
- a valid token is accepted after the endpoint recovers, with no restart;
- `/health` and both metadata paths still answer while retrieval is failing;
- `static` and `none` modes are untouched.

## Documentation

Update the README's authentication section, `docs/architecture.md`, and the operations
documentation to name the two failure classes and what each means for an operator. Cover the
split-horizon pairing explicitly: `KEYCLOAK_ISSUER` and `KEYCLOAK_JWKS_URI` are allowed to name
different hosts, that is not a misconfiguration, and the override exists for the case where the
canonical public issuer is not reachable from inside the network it lives on. Say plainly that a
`503` here means the server could not reach the realm, and that the token is not the thing to look
at.

## Acceptance criteria

- A key-retrieval failure is distinguishable from an invalid token in the response status, in the
  challenge, and in the log — and the log alone is enough to find the cause.
- No token, key, or internal URI reaches an unauthenticated caller.
- Every token accepted before this stage is still accepted; every token rejected before it is still
  rejected with the same status and code.
- `/health` and the metadata documents are unaffected by the realm's availability.
- Nothing under `deploy/` changes, and the MCP surface, ontology facts, and generated artifacts are
  unchanged.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This corrects shipped behaviour. If opening a pull request, apply `narrative-required` and include
substantive `## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences`
sections recording why the two failure classes were conflated, why the fix changes `entra` as well
as `keycloak`, why the diagnostic detail goes to the log and not the response, why startup does not
prefetch, and that discovery deliberately survives an unreachable realm. Never hand-edit generated
`Narrative.md`.

Commit locally with a focused message. Do not push.
