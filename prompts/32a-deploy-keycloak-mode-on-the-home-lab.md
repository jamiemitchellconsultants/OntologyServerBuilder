# Prompt 32a — Make the home-lab deployment actually run `keycloak` mode

Using the previously supplied repository contract, extend the checked-in home-lab deployment so the
`keycloak` mode added in Prompt 32 can be deployed as shipped, rather than by hand-editing the
Compose model. This stage changes deployment configuration, the proxy configuration, the operator
scripts, and their gates. It must not change `src/`, the MCP surface, ontology facts, or generated
artifacts, and it must not weaken any control the `static` deployment already proves.

This stage depends on Prompt 30 (the home-lab deployment) and Prompt 32 (the `keycloak` mode and the
OAuth challenge).

## Why this stage exists

Prompt 32 made `keycloak` the ordinary home-lab mode and left the deployment behind. What is checked
in still deploys `static` and nothing else:

- `deploy/homelab/compose.yaml` hard-codes `MCP_AUTH_MODE: static`, mounts the `mcp_auth_token`
  secret, and requires `MCP_AUTH_TOKEN_FILE_HOST` at render time — there is no token file in
  `keycloak` mode, so the model cannot even render for it;
- `deploy/homelab/nginx/ontology.conf` proxies exactly two locations, `= /health` and `= /mcp`, and
  returns 404 for everything else. **The protected-resource metadata paths are not proxied.** The
  server publishes them; the deployment hides them; a client discovers nothing and gives up. This is
  the single most consequential gap, and it is invisible from the server's own tests;
- `Validate.ps1` and `test/deployment.test.ts` assert the `static` model specifically, so they would
  pass unchanged while the deployment served a mode nobody could reach.

The result is a documented mode with no shipped way to run it. Prompt 32 recorded the required edits
in `docs/homelab-deployment.md` prose; prose is not a deployment.

## Read before editing

Inspect the repository first and adapt every name and path below to its current structure:

- `deploy/homelab/compose.yaml`, `deployment.env.example`, `nginx/ontology.conf`,
  `expected-mcp-surface.json`, `manual-gates.md`, and every script under `deploy/homelab/scripts/`;
- `test/deployment.test.ts`, and what each of its assertions is actually protecting;
- `src/auth.ts` and `src/server.ts` as they stand after Prompt 32 — in particular which paths are
  served unauthenticated and which settings fail startup closed;
- `docs/homelab-deployment.md`, especially §10 (the static fallback), §11 (Keycloak), and the
  "Deliberately unsupported" table;
- the repository's agent instructions and Narrative rules.

## The proxy

The metadata paths must reach the server. Add them to the proxied set, exactly and no more widely
than that: the two well-known paths Prompt 32 serves, plus the existing `/health` and `/mcp`.
Everything else keeps returning 404 — the point of that `location /` is that the deployment publishes
the MCP surface and nothing adjacent to it, and a prefix match over `/.well-known/` would give away
more than the two documents.

The `Host` header handling, the TLS settings, the body-size and timeout limits, and the security
headers are unchanged and must stay unchanged.

Note that this is safe in `static` mode too: the server publishes no metadata outside `keycloak`
mode, so the proxied paths simply 404 from the backend. Prefer that over making the proxy
configuration mode-dependent — one file that behaves correctly in both modes is one fewer thing to
get wrong at 2am.

## The Compose model

Both modes must be deployable from what is checked in, and neither may be able to silently render
the other's configuration. Compose has no conditional secret, so a single file cannot both require
`MCP_AUTH_TOKEN_FILE_HOST` and run without a token file. Choose a shape that respects that — a base
plus one overlay per mode, or two complete models — and state the choice and its cost in the guide.

Whatever the shape:

- Every interpolation keeps the `${VAR:?message}` form. `docker compose config` against an empty
  environment must still fail rather than render an empty string; that property is load-bearing and
  is asserted today.
- The `keycloak` model sets `MCP_AUTH_MODE: keycloak`, `KEYCLOAK_ISSUER`, `KEYCLOAK_AUDIENCE` and
  `MCP_RESOURCE_URI`, and carries **no** `mcp_auth_token` secret and no `MCP_AUTH_TOKEN*` variable of
  any kind. A model that leaves the static token mounted "in case" defeats the mode: the server
  validates one mode's credential, and an unused mounted secret is a credential still sitting on the
  host with a reason to be forgotten.
- `MCP_RESOURCE_URI` is derived from the same `SERVICE_DNS_NAME` and `LAN_TLS_PORT` the rest of the
  model already uses, or checked against them. Three copies of a name is three chances to disagree,
  and this one disagrees silently at the client.
- `ALLOWED_HOSTS` must admit the host in `MCP_RESOURCE_URI`, or the metadata documents 403 for the
  exact clients that need them.
- No new persistent state, no new published port, no new capability, and the ontology container keeps
  its read-only root filesystem, dropped capabilities, resource limits, and `internal: true` network.
  The `keycloak` mode needs **no** egress from the ontology container: the JWKS is fetched from
  Keycloak, so if the realm is not reachable on the internal network this is the one place the
  deployment's egress posture genuinely changes. Work out what that requires, state it explicitly,
  and do not quietly relax `internal: true` without saying so in the guide and the Narrative.

The `KEYCLOAK_ISSUER`, `KEYCLOAK_AUDIENCE` and `MCP_RESOURCE_URI` values are configuration, not
secrets, and belong in the environment file with everything else. No secret enters that file.

## The scripts

`Preflight.ps1`, `Start.ps1`, `Validate.ps1`, `Stop.ps1`, `Upgrade.ps1` and `Rollback.ps1` must all
work in both modes, and must refuse a configuration that is half of each. Specifically:

- Preflight rejects a `keycloak` configuration carrying a token path, and a `static` configuration
  missing one; it checks `MCP_RESOURCE_URI` for the same properties the server checks at startup, so
  the failure arrives before the containers do; and it verifies `MCP_RESOURCE_URI` agrees with
  `SERVICE_DNS_NAME`, the TLS port, and the certificate SAN.
- Start reports the mode it started, alongside the digest and `sourceFingerprint` it already prints.
- Upgrade and Rollback keep working without knowing which mode is deployed, and keep recording what
  they recorded before.

## Validation, and its honest limit

`Validate.ps1` currently starts a throwaway stack with a fixture token and proves the boundary. In
`keycloak` mode there is no fixture credential to mint, because minting one means standing up an
authorization server. Do not stand one up, and do not add a test-only token path to the server — that
would be a second credential in the product, which is exactly what this mode removes.

What the throwaway stack **can** prove without any identity provider, and must:

- an unauthenticated `POST /mcp` returns 401 with a `WWW-Authenticate: Bearer` header whose
  `resource_metadata` names a URL that resolves through the proxy;
- both metadata paths return 200 over TLS, unauthenticated, with the same document, naming the
  configured resource and issuer;
- the resource in the document is byte-for-byte the URL a client on this LAN would dial;
- a garbage bearer token is refused with 401, and the refusal does not echo the token;
- a disallowed `Host` is still refused with 403, including on the metadata paths;
- `GET` and `DELETE` on `/mcp` still return 405, and no path outside the four proxied ones answers;
- every container control the `static` validation already asserts.

What it cannot prove, and must therefore print as a manual gate with the exact command and the exact
expected result: a real token from the real realm being accepted, and a real agent completing
discovery, sign-in, and a tool call. Record both in `deploy/homelab/manual-gates.md` as unexecuted.
**Do not report an unrecorded gate as passing**, and do not let the throwaway validation's success
read as evidence that the flow works end to end.

## Tests

`test/deployment.test.ts` gains the same class of static gates for the `keycloak` model that it
already applies to the `static` one, and keeps every existing `static` assertion working. Where an
existing assertion is genuinely about the static model, keep it pointed at that model rather than
generalising it into something that asserts less.

New gates worth having:

- the `keycloak` model carries no `MCP_AUTH_TOKEN`, no `MCP_AUTH_TOKEN_FILE`, and no secret;
- `MCP_AUTH_MODE=none` appears in neither model, and neither model can render it;
- the proxy configuration proxies exactly the four expected paths and 404s the rest;
- `MCP_RESOURCE_URI` in the example environment is `https`, ends in the MCP path, and agrees with
  `SERVICE_DNS_NAME` and the TLS port;
- both models still fail to render against an empty environment;
- `expected-mcp-surface.json` is unchanged, and the surface check still runs in both modes.

## Documentation

Update `docs/homelab-deployment.md` and `deploy/homelab/README.md` so the guide describes what is
checked in rather than an edit the reader has to make:

- replace §11's "What the Compose model still ships" with the real deployment procedure;
- state which files a `keycloak` deployment uses and which a `static` one uses, and that choosing is
  an operator decision in configuration, not a runtime fallback;
- state what an operator configures on the Keycloak side, as requirements — a realm, a client whose
  tokens carry the configured audience, users, and a redirect-URI policy admitting the agents in use;
- keep the audience-sharing note as the deployment's choice, not a security property;
- record the egress conclusion from "The Compose model" above, whichever way it went;
- keep the roles-and-scopes-empty case documented as ordinary, not as a weakness.

## Acceptance criteria

- A `keycloak` home-lab deployment can be brought up from the checked-in files and an environment
  file, with no edit to a tracked file.
- The protected-resource metadata is reachable through the proxy at both paths, and the 401 challenge
  names a URL that resolves.
- The `static` deployment behaves exactly as it did, and every control it proved is still proved.
- No token, key, or secret appears in the image, repository, Compose model, environment example,
  logs, or generated documentation.
- `src/`, the MCP surface, ontology facts, and generated artifacts are unchanged by this stage.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This is a meaningful operational decision. If opening a pull request, apply `narrative-required` and
include substantive `## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences`
sections recording why the deployment had to change for a mode the server already supported, why the
proxy's exact-match location list was the invisible failure, what shape the two models take and what
that costs, what the throwaway validation can and cannot prove without an authorization server, and
the egress conclusion. Never hand-edit generated `Narrative.md`.

Commit locally with a focused message. Do not push.
