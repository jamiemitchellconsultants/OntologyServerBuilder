# Prompt 34 — Add durable intake and capability authorization

Using the approved qualified-user intake plan from Prompt 33, implement the first intake-plane
foundation in the separate `OntologyService` repository. Do not change `OntologyServerBuilder` in
this stage. Prompt 32b remains active and its authentication failure distinction must be preserved.

This stage adds provider-neutral capability authorization and durable, disabled-by-default intake
storage. It must not add an MCP tool, change the compiled ontology, make intake material into
ontology facts, or change the delivery plane. The runtime must never retrieve a source, open a user
attachment, or call a model. A source locator is inert provenance, not a retrieval instruction.

## Read before editing

Read the Prompt 33 plan and inspect the current authentication middleware, verified-principal
contract, configuration, server startup, proposal workflows, tests, deployment material, homelab
guide, Prompt 31 AWS baseline, and Project Narrative rules. Adapt names and paths to the repository
as it exists; do not assume future prompts have created files or interfaces.

## Capability authorization

Introduce provider-neutral capabilities named exactly:

- `ontology:read`
- `ontology:propose`
- `ontology:intake:review`

Define `Capability` and `AuthorisedPrincipal` so that a handler receives the already validated
principal and can make an explicit capability decision. Authentication remains responsible for token
validation; authorization follows token validation. Tool discovery, route presence, or a
successfully validated token must never be treated as authorization.

Assign and enforce the capabilities now for every current MCP handler. Existing ontology query,
description, list, and resource operations require `ontology:read`. Existing stateless
proposal-preparation tools require `ontology:propose`. Each handler must make that capability check
after token validation and before executing its operation. Do not change the handler's behavior
beyond this authorization requirement. `ontology:intake:review` has no handler until later stages,
but its configured claim mapping must be parsed and validated now so later handlers cannot invent a
second authorization scheme.

Preserve every existing authentication mode and the Prompt 32b distinction between invalid tokens
and unavailable signing keys. Intake is an additional capability, not a change to which credentials
are valid. Refuse to enable intake in `none` or `static` mode because neither supplies an
attributable qualified-user identity. For `keycloak` and `entra`, require explicit, configured
claim-to-capability mapping. When intake is enabled, absent, malformed, or non-matching capability
configuration must fail closed at startup; do not infer capability names from unconfigured roles,
scopes, or claims.

This stage establishes the contracts and enforcement needed by later intake handlers; it adds no MCP
tools.

## Durable intake contract

Create a focused storage module that owns intake persistence and exports the conceptual operations
below, adapting supporting input and output types to the service's conventions:

```ts
interface IntakeStore {
  submit(input: NewSubmission): Promise<SubmissionReceipt>;
  list(query: IntakeListQuery): Promise<IntakeSummary[]>;
  export(id: string): Promise<IntakeExport>;
  appendEvent(id: string, event: NewSubmissionEvent): Promise<SubmissionEvent>;
}
```

Define and export `IntakeStore`, `SubmissionReceipt`, and `SubmissionEvent`. Canonicalize every
submission payload to canonical JSON and record its SHA-256 payload digest. A stored submission
object is immutable: there is no update or delete operation. Lifecycle history consists only of
append-only event objects. An export may record an event without changing the submission payload.

Implement the first adapter with S3-compatible object storage. Set `INTAKE_MODE=disabled|s3`,
defaulting to `disabled`; require `INTAKE_S3_BUCKET` when mode is `s3`. Support an optional
`INTAKE_S3_PREFIX` for key namespacing, an optional `INTAKE_S3_REGION`, and an optional
`INTAKE_S3_ENDPOINT` together with `INTAKE_S3_FORCE_PATH_STYLE` so the same adapter targets a
LocalStack or other S3-compatible endpoint for home-lab and test use while targeting real AWS S3 in
production. Read AWS credentials only through the SDK's standard credential chain; never accept a
caller-supplied or hand-rolled credential. In disabled mode, do not construct a client, resolve a
bucket, accept an intake write, or expose a configuration that could persist a submission.

Store each submission as one object at a stable, content-addressed key so the payload and its
initial `received` event are written together in a single request: the object body carries the
canonical payload, its digest, and the received timestamp as one immutable unit. Write it with a
create-only conditional request (an `If-None-Match: *` precondition or the adapter's equivalent) so
a colliding key can never silently overwrite an existing submission; treat a precondition failure on
a freshly generated key as a defect, not a normal conflict path. This makes the
submission-plus-initial-event write atomic without a multi-object transaction: there is exactly one
object, written exactly once.

Enforce uniqueness on authenticated subject plus caller-provided idempotency key with a separate
lookup object, keyed only by that subject and key, written with the same create-only precondition
*before* the submission object itself; do not write a submission object when the lookup precondition
fails. If the precondition fails, fetch the existing lookup object and its recorded submission ID:
an identical replay, determined by the canonical payload digest, returns the original receipt. A
replay using that subject and key with a different digest fails with a clear conflict. The store
must not accept a caller-supplied subject in place of the validated, attributable principal.

Record every later lifecycle event (`exported`, `accepted`, `rejected`, `superseded`) as its own new
object under a submission-scoped event-key prefix, ordered so listing that prefix returns events in
append order; write each with the same create-only precondition and never edit or delete an existing
submission or event object. Implement `list` over a sorted key prefix so its cursor is the
object-storage list operation's own continuation token, never an offset.

Validate every object's schema, digest, and content on read and fail closed on a malformed,
truncated, or digest-mismatched object; never repair, replace, or silently skip it. Object storage
enforces no schema, so this validation is the adapter's responsibility, not the platform's.

A create-only conditional write is a server-side atomic guarantee, not a client-side lock, so this
adapter is safe under concurrent writers from multiple service instances without the single-writer
restriction a local embedded database would need. Do not add an artificial single-instance
restriction; prove concurrent-writer safety directly, as the tests below require.

## Deployment and recovery boundary

Keep compiled ontology artifacts inside the immutable image. They remain compiler-owned and are not
placed in, restored from, or coupled to the intake bucket. Require a configured, access-restricted
`INTAKE_S3_BUCKET` only when S3 intake is enabled. With intake disabled, retain the current
no-volume, no-bucket deployment behavior.

Update the homelab guidance to cover pointing `INTAKE_S3_ENDPOINT` at a local S3-compatible service
(such as LocalStack) for development parity with production, alongside bucket versioning, a
least-privilege IAM policy or access-key scope, server-side encryption at rest, lifecycle/retention
rules, and capacity monitoring. Specify recovery without inventing a per-record delete endpoint:
operator-controlled bucket versioning, replication, and lifecycle rules operate on the bucket as a
whole. Document that this adapter may be enabled under Prompt 31's multi-instance AWS baseline,
because object-storage writes are safely concurrent across instances; a deployment still keeps
intake disabled by default and enables it deliberately once the bucket, its access scope, and
monitoring are provisioned and verified.

## Tests

Add focused automated tests for:

- claim-to-capability mapping and missing capabilities;
- the explicit `ontology:read` assignment for every current ontology query, description, list, and
  resource handler, and the explicit `ontology:propose` assignment for every current stateless
  proposal-preparation handler;
- post-token-validation capability checks that refuse a validated principal lacking the required
  capability before the current handler executes;
- disabled mode and refusal to enable intake in `none` or `static` mode;
- S3 client initialization against a fake or local S3-compatible endpoint, and object persistence
  across a simulated restart;
- atomic receipt creation and idempotent identical replay;
- conflicting replay for the same authenticated subject and idempotency key;
- immutable stored payloads and append-only events, including rejection of a precondition-failed
  overwrite attempt;
- malformed, truncated, or digest-mismatched object handling; and
- concurrent-writer safety: two simulated instances submitting with the same idempotency key produce
  exactly one submission and a shared receipt, and submitting with different idempotency keys never
  collides, against a fake or local S3-compatible store.

Keep tests offline. They must prove no runtime source retrieval, attachment opening, or model call
is introduced. Inject clocks and ID generation where the repository's test conventions need stable
assertions. No MCP tool is added in this stage, so the registered MCP surface must remain unchanged.

## Acceptance criteria

- The service has the exact three provider-neutral capability names. Every current ontology query,
  description, list, and resource handler enforces `ontology:read`; every current stateless
  proposal-preparation handler enforces `ontology:propose`; and all checks happen after token
  validation. `ontology:intake:review` mapping is validated now for its later handler.
- Intake is disabled by default, cannot run in `none` or `static` mode, and fails closed without
  explicit Keycloak or Entra claim-to-capability configuration.
- S3-compatible object storage stores canonical JSON and SHA-256 digests, atomically creates durable
  receipts through create-only conditional writes, preserves immutable submissions, and records
  append-only events.
- Idempotency is unique per authenticated subject and key: identical replays return the original
  receipt, while conflicting replays fail.
- S3 intake requires a configured, access-restricted bucket and remains safe under concurrent
  writers from multiple service instances without a single-instance restriction.
- The homelab recovery guidance covers a LocalStack-compatible endpoint override, bucket versioning,
  access scope, encryption, lifecycle rules, and capacity; the Prompt 31 AWS multi-instance baseline
  may enable intake once its bucket and access scope are provisioned and verified.
- No MCP tool, delivery-plane behavior, compiled ontology artifact, source retrieval, attachment
  opening, or model call is added by this stage.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This is a meaningful architecture, security, and operational decision. If opening a pull request,
apply `narrative-required` and include substantive `## Narrative Context`, `## Narrative Decision`,
and `## Narrative Consequences` sections before merge. Record the disabled-by-default migration, the
move to shared S3-compatible object storage with concurrent-writer safety, capability attribution,
and the retained immutable-image boundary. Never hand-edit generated `Narrative.md`.

Commit locally with a focused intake-foundation message. Do not push.
