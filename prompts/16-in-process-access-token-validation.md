# Prompt 16 — Validate access tokens in-process

Using the previously supplied repository contract, move HTTP MCP authentication into the service
process so the resource server validates the caller's bearer token itself.

Read before editing:

- `README.md`;
- `.env.example`, `Dockerfile`, and deployment configuration;
- `docs/architecture.md` and any security or deployment documentation;
- `src/server.ts` and the HTTP transport tests;
- the installed MCP SDK's bearer-auth types, middleware, and error behavior;
- `package.json` and the lockfile;
- `AGENTS.md`, `Narrative.md`, and the authoritative Narrative source fragments.

First inspect the repository and adapt names and file locations to its current structure. Preserve
stdio behavior, the stateless Streamable HTTP transport, host-header validation, and every existing
runtime trust boundary. Do not add an external identity service, database, session store, token
introspection call, authorization server, or runtime model call.

## Architectural decision

Authenticate every request to the HTTP MCP endpoint in-process through one middleware and a
verifier chosen once during HTTP startup. The fronting gateway remains responsible for TLS
termination, request-size limits, rate limiting, audit logging, and network policy, but it is no
longer the component trusted to validate access tokens.

Support exactly these modes through `MCP_AUTH_MODE`:

| Mode | Purpose | Required configuration |
|---|---|---|
| `entra` | Production validation of Entra ID (Azure AD) access-token JWTs | `ENTRA_TENANT_ID`, `ENTRA_AUDIENCE`; optional `ENTRA_ISSUER`, `ENTRA_JWKS_URI`, `ENTRA_REQUIRED_ROLES`, and `ENTRA_REQUIRED_SCOPES` |
| `static` | Trusted local or home-lab use | `MCP_AUTH_TOKEN`, at least 16 characters |
| `none` | Explicitly unauthenticated trusted local development | none |

An unset, blank, or unknown mode must make HTTP startup fail closed. `none` must be an explicit
choice and must log a clear warning. Stdio startup must not require HTTP authentication
configuration.

## Implementation requirements

Create a small authentication module with a common access-token verifier contract and keep the
HTTP route independent of verifier-specific details. Prefer the MCP SDK's standard bearer-auth
middleware and verifier types when the installed SDK provides them.

For `static` mode:

- accept only the exact configured bearer token;
- reject configured tokens shorter than 16 characters;
- compare equal-length token bytes with Node's constant-time comparison;
- handle unequal lengths without throwing;
- return only the minimal authentication metadata required by the MCP SDK;
- document that this mode is not suitable for an untrusted or shared production deployment.

For `entra` mode:

- use a maintained JOSE implementation such as `jose`;
- verify the JWT signature against the tenant JWKS;
- validate issuer, audience, and expiry;
- derive the default v2 issuer and JWKS URI from `ENTRA_TENANT_ID`, while allowing explicit
  overrides for controlled testing and non-default deployments;
- require every comma-separated role configured in `ENTRA_REQUIRED_ROLES` to appear in the
  token's `roles` claim;
- require every comma-separated scope configured in `ENTRA_REQUIRED_SCOPES` to appear in the
  space-delimited `scp` claim;
- derive a client identifier from `azp`, then `appid`, then `sub`, without trusting it as proof
  beyond the already-validated token;
- translate verification failures into the MCP SDK's invalid-token response without leaking the
  token or secrets to logs.

Create the verifier and middleware once when the HTTP application starts. Apply it to the MCP
endpoint before the MCP handler so missing or invalid credentials cannot create a server or
transport instance or invoke MCP logic. Keep `/health` unauthenticated for liveness probes, and keep
its response minimal and non-sensitive. Do not weaken the existing host allow-list or make it a
substitute for authentication.

Add or update `.env.example` with safe placeholders for every mode and variable. Never commit a real
token, tenant identifier, credential, or captured JWT.

## Tests

Add deterministic tests for the verifier and HTTP boundary. Do not call the live Entra service:
generate a test signing key, serve a local JWKS endpoint, and mint short-lived test JWTs.

Cover at least:

- the exact static token succeeds;
- wrong and differently sized static tokens fail;
- a configured static token shorter than 16 characters fails at startup;
- an unset, blank, or unknown mode fails closed for HTTP;
- explicit `none` returns a pass-through middleware;
- a correctly signed Entra token with the expected issuer, audience, expiry, roles, and scopes
  succeeds;
- expired tokens, wrong signatures, issuers, audiences, roles, and scopes fail;
- comma-separated configuration is trimmed and ignores empty entries;
- `/mcp` rejects missing, malformed, and invalid bearer credentials before MCP handling;
- `/health` remains unauthenticated and exposes no authentication configuration;
- stdio still starts without `MCP_AUTH_MODE`;
- existing host-header rejection behavior remains intact.

Ensure tests restore modified environment variables and close local HTTP/JWKS servers even when an
assertion fails.

## Documentation and governance

Update the README's local, Docker, and shared-deployment examples so authenticated commands are
copyable and callers send `Authorization: Bearer <token>`. Document all three modes, their intended
environments, required variables, the unauthenticated health endpoint, and the remaining gateway
responsibilities.

Update architecture, security, operations, roadmap, and agent instructions wherever they still say
authentication is delegated entirely to a gateway or is not implemented. Run a repository-wide
terminology search and remove contradictory current-state claims. Do not rewrite accepted
historical Narrative entries.

This is a meaningful architecture and security-boundary decision. If opening a pull request, apply
`narrative-required` and include substantive sections named exactly:

- `## Narrative Context`
- `## Narrative Decision`
- `## Narrative Consequences`

Explain the policy need, why the resource server validates tokens, why both Entra and fixed-token
verifiers exist, why HTTP fails closed, what remains at the gateway, and the operational cost of a
JOSE/JWKS dependency. Follow the repository's Narrative workflow; never hand-edit generated
`Narrative.md`.

## Acceptance criteria

- Every HTTP MCP request is authenticated in-process unless `MCP_AUTH_MODE=none` was explicitly
  selected.
- Production Entra validation checks signature, issuer, audience, expiry, and all configured
  roles/scopes.
- Static-token comparison is constant-time for equal-length values and enforces the minimum
  configured length.
- HTTP startup fails closed for missing or invalid authentication configuration; stdio is
  unaffected.
- `/health` remains unauthenticated and minimal.
- Host-header validation and all existing runtime security boundaries remain enforced.
- Documentation and examples agree with the implemented behavior and contain no stale claim that
  authentication is only a gateway responsibility or is unimplemented.
- Dependencies and the lockfile are updated intentionally, with no secrets or live tokens in the
  repository.
- `npm run check` and `git diff --check` pass.
- Generated ontology artifacts have no unexplained diff.

Commit locally with a focused architecture/security message. Do not push.
