# Prompt 49 — Add ephemeral supplier MCP discovery and registration

Using the qualified-user intake tools from Prompt 35 and the engineer workbench from Prompt 36,
add a direct supplier-MCP discovery workflow to the separate `OntologyService` repository. A
qualified user supplies the supplier endpoint and authentication configuration, completes any
secret entry or interactive authorization in a short-lived browser handoff, and receives only the
review-required intake receipt. Do not change `OntologyServerBuilder` in this stage.

This is a deliberate, narrowly bounded exception to the earlier rule that the runtime never opens a
live MCP endpoint. The exception belongs only to the attributable, capability-gated intake
registration plane introduced here. Compilation, ontology reads, SPARQL, release projection,
mapping tools, and every other delivery-plane handler remain unable to perform network I/O.

Prompt 48 remains an independent, supported alternative and must keep working unchanged. Its skill
captures a catalog in the user's own MCP client and submits normalized evidence through the Prompt
35 tools. This stage adds a server-connected path; it does not replace, edit, or auto-install that
skill.

This stage depends on Prompts 20, 32, 35, and 36. Adapt paths and types to the repository as it
exists. Do not assume an embedding, LLM, release, named-mapping-tool, or accounts-payable stage is
present.

## Explicit supersession of earlier prompts

Do not edit Prompt 13 or any other earlier Builder prompt. Those stages are historical inputs that
have already been applied. This prompt is the sole authority for the later behavior it introduces.

For the two new tools, browser routes, attempt store, evidence store, and supplier-discovery client
added by this stage, this prompt explicitly supersedes only these earlier constraints:

- **Prompt 18:** its preparation tool accepts caller-extracted normalized metadata and performs no
  network or raw-artifact handling. Keep that public tool unchanged. The new direct workflow may
  obtain and structurally normalize one bounded MCP catalog before calling the same internal
  preparation validation.
- **Prompt 33:** its plan states that the runtime never retrieves an interface source. The direct
  workflow may retrieve discovery metadata from the one validated supplier MCP endpoint named in
  its attributable attempt. The no-attachment and no-model rules remain unchanged.
- **Prompt 34:** its intake foundation performs no source retrieval. The new workflow is the only
  intake operation allowed to perform the bounded MCP/OIDC requests specified here. Existing
  submission immutability, append-only events, capability attribution, and disabled-by-default
  behavior remain unchanged.
- **Prompt 35:** its two public submission handlers never fetch a locator, receive raw bytes, or
  accept credentials. Keep both handlers and schemas compatible and unchanged. The new start and
  completion tools acquire credentials only through the browser handoff, capture raw catalog
  evidence internally, and then use Prompt 35's canonical submission builder.
- **Prompt 36:** its original-artifact workflow requires an engineer to supply a separate local
  artifact and its export contains no artifact download. Preserve that behavior for caller-captured
  submissions. A direct registration created by this stage instead carries an immutable evidence
  reference whose bytes are integrity-checked and included in the authorized engineer export.
- **Prompt 48:** its skill requires the user's own client to connect, capture, hash, normalize, and
  submit without server-side retrieval. Preserve the skill byte-for-byte. Prompt 49 adds a separate
  server-connected alternative and does not reinterpret Prompt 48.

Prompt 47 remains the historical audit of the boundary that existed before this stage; do not edit
its report or represent it as having audited Prompt 49. Preserve every requirement from every
earlier prompt unless this section identifies the exact conflicting behavior and replacement.

## Read before editing

Read the canonical repository agent instructions; authentication and capability code; Prompt 35's
preparation/submission contracts; Prompt 36's export and event contracts; HTTP transport and host
validation; intake configuration, store, and S3 adapter; MCP catalog parser; deployment guidance;
security documentation; tests; and Project Narrative rules.

Read `skills/register-supplier-mcp-server/SKILL.md` if Prompt 48 has been applied. Preserve it
byte-for-byte and retain both of its exact tool references. The two paths must coexist.

## Explicit product boundary

This workflow means "connect once, capture discovery metadata, and submit it for review." It does
not create a persistent supplier integration, periodically refresh a catalog, register the system,
or make any submitted fact traversable.

Credentials exist only for one registration attempt. Do not persist, cache, refresh, replay, or
reuse a client secret, authorization code, access token, refresh token, browser session credential,
or PKCE verifier after that attempt succeeds, fails, is cancelled, or expires.

Support remote HTTPS Streamable HTTP MCP only in this first release. Do not add stdio, SSE fallback,
WebSocket, local-command execution, arbitrary proxying, or a generic "connect to URL" facility.

## MCP tools and authorization

Register exactly these two qualified-user tools:

1. `ontology_start_supplier_mcp_registration`
2. `ontology_complete_supplier_mcp_registration`

Add both to the explicit tool-capability map. Each handler's first statement must require
`ontology:propose` from an already validated, attributable principal. Bind every attempt to that
principal's subject. A valid bearer token, a browser handoff token, possession of an attempt ID, or
tool discovery is never sufficient authorization.

`ontology_start_supplier_mcp_registration` is state-changing, non-destructive, and idempotent. Its
input contains:

- proposed system ID, name, description, and optional source version;
- the exact supplier MCP HTTPS endpoint;
- one of the bounded authentication configurations below;
- a caller-generated idempotency key under Prompt 35's existing contract.

It returns only an opaque attempt ID, expiry, state, authenticated browser-handoff URL, and a
separate opaque completion proof. It never accepts a client secret, authorization code, access
token, refresh token, password, cookie, or private key in MCP input.

`ontology_complete_supplier_mcp_registration` is read-only and idempotent. It accepts the attempt ID
and completion proof and returns exactly one of:

- `authentication-required`, with no supplier-controlled content;
- `processing`;
- a bounded failure envelope;
- the Prompt 35 receipt shape: opaque ID, payload digest, received timestamp, and `received` status.

The completion tool may return the same receipt until the attempt expires. It must not list
attempts, return evidence, disclose supplier tokens, or become a qualified-user queue/status tool.

## Authentication configurations

Use a discriminated input schema with exactly these modes:

### OAuth client credentials

Require a supplier issuer or discovery URL, client ID, requested scopes, optional audience/resource
parameter, and an allowed token-endpoint client-authentication method. The client secret is entered
only in the browser handoff over HTTPS.

The browser POST handler holds the secret only for the request, exchanges it for a short-lived
supplier access token, immediately runs discovery, and clears secret/token references before
responding. Never request `offline_access`. Discard any refresh token the authorization server
returns despite that restriction.

### OIDC authorization code with PKCE

Require the supplier issuer, client ID, requested scopes including `openid`, optional
audience/resource, and whether the supplier client is public or confidential. A confidential client
collects its client secret through the browser handoff; a public client never supplies one.

Generate PKCE S256 material, `state`, and `nonce`. Use an exact configured callback URI. Validate
issuer, callback state, code exchange, ID-token signature, audience, expiry, and nonce before using
the supplier access token. Reject mix-up, replay, missing state, unsupported algorithms, and a
callback for another attempt.

Existing supplier SSO is this same authorization-code flow benefiting from the user's current
identity-provider browser session. Never forward or exchange the bearer token issued for
OntologyService merely because both services use SSO. The supplier access token must be issued for
the supplier resource/audience through the selected supplier flow.

## Browser handoff and identity binding

Add a minimal HTTPS browser surface under an isolated supplier-registration route namespace. It
needs separate callbacks for the OntologyService browser login and the supplier authorization flow.
Do not turn the service into a general web application.

The browser must authenticate through the deployment's production OIDC provider and resolve the
same attributable subject that started the MCP attempt. A valid one-time handoff token for a
different subject fails. The page displays, before continuing:

- the sanitized supplier origin;
- proposed system identity;
- authentication mode;
- requested scopes and audience/resource;
- the fact that successful discovery submits evidence for review rather than registering a system.

Use high-entropy, single-use handoff and callback tokens; CSRF protection; `Secure`, `HttpOnly`, and
appropriate `SameSite` cookies; restrictive CSP; `Cache-Control: no-store`; and a no-referrer
policy.
Redact callback query strings and request bodies from application, proxy, tracing, and error logs.
Escape all rendered values. Do not render supplier-controlled catalog content in this surface.

The browser may cancel its own attempt. Closing a page without completion leaves the attempt to
expire; it never leaves credentials usable in the background.

## Attempt lifecycle and storage

Implement a separate `RegistrationAttemptStore` inside the intake plane with atomic transitions:

```text
awaiting-authentication -> authenticating -> discovering -> submitting -> received
```

Terminal alternatives are `failed`, `cancelled`, and `expired`. Reject every invalid or repeated
transition. Bind a subject/idempotency-key pair to one active attempt or final receipt.

Attempt records may contain only:

- opaque ID and owner subject;
- non-secret endpoint and OIDC configuration;
- creation/expiry timestamps and current state;
- hashes of one-time browser/callback/completion proofs;
- bounded failure code or final receipt;
- durable evidence reference and digest after discovery succeeds.

They must not contain client secrets, authorization codes, access tokens, refresh tokens, raw
catalog bytes, cookies, or PKCE verifiers. Keep the PKCE verifier in authenticated encrypted browser
state only for the authorization round trip, then discard it.

Default expiry is 15 minutes. Production requires a shared, TTL-capable adapter with conditional
state transitions so browser callbacks and MCP completion may reach different instances. An
in-memory adapter is allowed only for deterministic tests and explicitly local development. Attempt
records are ephemeral coordination state, not intake submissions; isolate their mutable/delete
operations from the immutable submission and append-only event APIs.

## Fail-closed feature configuration

Ship the entire capability disabled by default. Enabling it requires an HTTP deployment using an
attributable JWT mode; refuse it under `static`, `none`, and stdio.

Require deliberate configuration for:

- public browser/callback base URL;
- OntologyService browser-session OIDC issuer, client, callback, and signing/encryption material;
- shared production attempt-store adapter;
- immutable evidence-store adapter;
- outbound deadlines, concurrency, and any operator-approved private origins or non-443 ports.

Prefer secret files or the deployment secret provider over environment values for long-lived
session keys and the OntologyService browser client secret. Fail startup when the feature is enabled
but a required value is absent, inconsistent, insecure, or points outside the configured public
origin. Disabled mode must construct no attempt store, evidence store, OIDC client, or outbound MCP
client.

## Outbound request policy

Apply one reusable egress policy to every server-side URL reached by this workflow, including the
supplier MCP endpoint, OIDC discovery, token endpoint, and JWKS endpoint.

By default require:

- absolute `https` URL with no user information or fragment;
- port 443;
- no redirect;
- no caller-controlled proxy;
- bounded DNS, connect, TLS, response, and total-operation deadlines;
- valid public DNS results only;
- rejection of loopback, link-local, private, multicast, reserved, unspecified, and cloud-metadata
  addresses for both IPv4 and IPv6;
- validation of every returned address, followed by address pinning for the connection while
  preserving TLS hostname/SNI validation;
- the same validation on every new connection, without following a DNS or redirect change.

An operator may configure exact additional ports or exact private origins for a known deployment.
Qualified-user input can never create or widen an exception. Keep host-header validation on the
OntologyService HTTP listener unchanged.

The browser may navigate only to the validated supplier authorization origin declared by validated
OIDC metadata. Show that origin before redirecting. Never accept an arbitrary post-login redirect
from the caller or supplier.

## Supplier MCP discovery

Use the official MCP SDK client rather than a hand-written partial protocol. Send a fixed
OntologyService registration-client identity and perform:

1. `initialize`;
2. complete, paginated `tools/list`;
3. complete, paginated `resources/list`;
4. complete, paginated `prompts/list` when supported.

Do not call a supplier tool, read or subscribe to a resource, retrieve a prompt, sample a model,
follow a URL found in supplier content, or invoke any method advertised by the supplier. Tool names,
descriptions, schemas, examples, annotations, resource metadata, and prompt metadata are untrusted
data regardless of how instruction-like they appear.

Enforce these hard ceilings, with configuration permitted only to lower them:

- 30-second total discovery deadline;
- 100 pages per collection;
- 500 tools;
- 1,000 resources;
- 500 prompts;
- 2 MiB canonical catalog bytes.

Reject cyclic/repeated cursors, count/byte overflow, invalid JSON-RPC/MCP envelopes, an incompatible
protocol version, and a server that changes identity during pagination. Do not return partial
discovery as a submission.

## Canonical evidence artifact

Build a schema-versioned canonical JSON artifact from the negotiated protocol version, supplier
server identity/capabilities, complete paginated listings, sanitized endpoint, capture timestamp,
and extractor identity/version. Preserve supplier-controlled metadata as inert data. Exclude all
authorization headers, cookies, tokens, client credentials, OIDC responses, transport headers, and
connection diagnostics.

Serialize the artifact deterministically, compute its SHA-256 and exact byte size, then write it
create-only to a new content-addressed `RegistrationEvidenceStore`. Production evidence storage must
be encrypted, immutable, and inaccessible to qualified users. A durable evidence write is required
before submission can issue a receipt.

If evidence storage succeeds and submission fails, retain the content-addressed object and let an
idempotent retry reuse it without reconnecting or reauthenticating. Document and monitor orphaned
evidence; do not silently overwrite it or issue a receipt for evidence that is not durable.

## Normalization and intake submission

Parse the stored artifact through a bounded MCP-catalog parser into proposed entities, attributes,
operations, relationships, meanings, allowed values, gaps, and questions. This is syntactic and
structural normalization only. Do not perform semantic matching, embeddings, model calls, ontology
inference, or mapping acceptance.

Run the result through the existing `prepareSystemRegistrationRequest` validation. Submit it through
the Prompt 35 canonical system-registration builder and `IntakeStore`, preserving Prompt 35's
attributable subject, payload limits, idempotency, and receipt contract.

Extend the system-registration payload compatibly with an optional captured-evidence reference
containing artifact kind, digest, byte size, media type, and opaque evidence ID. Include that
reference in the canonical payload digest. The existing public
`ontology_submit_system_registration` schema and Prompt 48 skill must remain compatible; their
caller-captured submissions simply omit the new reference.

A receipt means only "submitted for review." Do not register a system, update a source, create a
relationship, activate a mapping, change the compiled fingerprint, or claim deployment.

## Engineer export and evidence integrity

Extend the Prompt 36 engineer export path, still requiring `ontology:intake:review`, so a direct
registration export carries the immutable catalog artifact or a bounded export representation of
it. Recompute and verify the evidence digest and byte size before returning any export. Verify the
submission payload digest, receipt identity, submission identity, and evidence reference as one
consistent envelope; fail closed on disagreement.

Do not add a qualified-user evidence, download, list, status, or export operation. The compiler must
never read the evidence or attempt stores, and neither store participates in the ontology
fingerprint.

## Stable failure contract and partial failures

Return deterministic, value-safe failures with codes including:

- `endpoint-not-allowed`;
- `oidc-discovery-failed`;
- `authentication-denied`;
- `authentication-failed`;
- `supplier-unreachable`;
- `supplier-protocol-invalid`;
- `catalog-limit-exceeded`;
- `evidence-write-failed`;
- `intake-write-failed`;
- `attempt-expired`;
- `attempt-owner-mismatch`.

Name the failed stage and safe corrective action, but never include secrets, tokens, authorization
codes, raw supplier bodies, resolved internal addresses, stack traces, or SDK error objects.

Authentication or discovery failure writes no evidence and no submission. Evidence-write failure
writes no submission. Intake-write failure returns no receipt and preserves the immutable evidence
for retry. If intake succeeds but result publication fails, recover the same receipt through the
existing idempotency record. Cancellation, deadline, and disconnect must close outbound activity and
clear credential material.

Limit active attempts per subject and apply separate gateway/service rate limits to MCP start,
browser, callback, and completion surfaces. Audit only subject, attempt ID, sanitized supplier
origin, authentication mode, state transition, evidence digest, receipt ID, and outcome.

## Documentation and canonical instructions

Update README/tool listings, system-registration documentation, architecture and security
boundaries, HTTP/deployment configuration, engineer export guidance, and expected deployed MCP
surface.

Do not edit Prompt 13. Update the built `OntologyService` repository's current canonical `AGENTS.md`
in this stage so it states the exact new exception: qualified-user supplier registration may
perform bounded outbound MCP/OIDC access only through this disabled-by-default intake workflow.
Keep the prohibition absolute for the compiler, ordinary delivery handlers, SPARQL, mapping tools,
submitted locators, and every other runtime path. Keep every agent-specific instruction file as a
pointer; do not duplicate policy.

State the compatibility relationship plainly: Prompt 48 remains the no-outbound, client-captured
option, while Prompt 49 is the direct, ephemeral-credential option.

## Tests

Use only local synthetic OIDC and MCP servers. Never contact a real supplier, identity provider,
public DNS name, or cloud metadata endpoint.

Add focused unit, linked in-memory MCP, HTTP, store-adapter, and deployment tests covering:

- explicit capability-map coverage and first-statement `ontology:propose` enforcement;
- attributable owner binding across both tools and every browser/callback route;
- exact tool names, annotations, input/output schemas, and expected deployment surface;
- client-credentials success without secret/token persistence, output, snapshot, trace, or log;
- authorization code with PKCE, public/confidential clients, state, nonce, issuer, audience,
  signature, expiry, denial, cancellation, and an existing SSO session;
- callback replay, CSRF, handoff theft, owner mismatch, expiry, and cross-instance completion;
- IPv4/IPv6 private ranges, metadata addresses, DNS rebinding, redirects, alternate-address DNS,
  unapproved ports, proxy attempts, and operator-configured exact exceptions;
- proof that only `initialize` and the three list methods are invoked;
- instruction-shaped supplier content remaining inert;
- pagination success plus cursor loops, page/count/byte overflow, timeout, disconnect, malformed
  MCP, and supplier identity change;
- canonical artifact determinism, create-only evidence writes, encryption configuration, and digest
  verification on engineer export;
- subject/idempotency replay, concurrent state transitions, duplicate callbacks, and each partial
  failure boundary;
- disabled mode constructing no outbound client or registration store, plus fail-closed startup for
  every missing/insecure production setting;
- multi-instance callback/completion behavior using the production attempt-store adapter;
- existing Prompt 35 submissions and the Prompt 48 skill path remaining compatible;
- byte-for-byte proof that attempts, failures, evidence, submission, export, and receipt retrieval
  do not alter compiled ontology artifacts;
- network sentinels proving the compiler, ontology reads, SPARQL, release projection, mapping tools,
  and all non-registration handlers still perform no external I/O.

Run the repository's full local gate and `git diff --check`. Confirm generated ontology artifacts
have no unexplained diff and that no fixture, snapshot, log assertion, or documentation example
contains a real credential, endpoint, user identity, or supplier catalog.

## Acceptance criteria

- Prompt 48's installed skill and existing Prompt 35 tool schemas remain compatible and unchanged.
- Exactly the two new tools exist, require `ontology:propose`, bind attempts to an attributable
  subject, and never accept credentials through MCP input.
- Browser handoff supports client credentials and authorization code with PKCE; existing SSO is
  correctly treated as the authorization-code flow, never bearer-token forwarding.
- Every credential and token is used only during one attempt and is absent from durable state,
  evidence, intake payloads, receipts, logs, traces, errors, and generated output.
- Supplier/OIDC connections pass the shared SSRF/egress policy, and qualified users cannot widen
  deployment exceptions.
- Discovery is bounded to `initialize`, `tools/list`, `resources/list`, and `prompts/list`; no
  supplier tool, resource, prompt, model, or discovered URL is invoked.
- Canonical catalog evidence is content-addressed, create-only, durable before receipt, digest-bound
  to the submission, and exportable only to an authorized engineer with integrity verification.
- Normalization remains structural and every submission remains review-required and outside the
  compiled ontology and fingerprint.
- The feature is disabled by default, unavailable under static/none/stdio, and fails startup closed
  when enabled without complete secure configuration.
- Attempt state is bounded, atomic, shared in production, contains no credentials, and expires.
- Documentation, canonical agent instructions, deployment surface, and threat model state the new
  exception without weakening any non-registration boundary.
- All stated local synthetic tests, the full repository gate, `git diff --check`, and generated-file
  freshness checks pass.

This is a major product, security, architecture, governance, and operational decision. If opening a
pull request, apply `narrative-required` together with substantive `## Narrative Context`,
`## Narrative Decision`, and `## Narrative Consequences` sections before merge. Never hand-edit
generated `Narrative.md`.

Commit locally with a focused direct-supplier-registration message. Do not push.
