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
submission payload to canonical JSON and record its SHA-256 payload digest. A stored submission row
is immutable: there is no update or delete operation. Lifecycle history consists only of append-only
events. An export may record an event without changing the submission payload.

Create a receipt atomically with the submission. Its successful return means both the immutable
submission and its initial received event are durable. Enforce uniqueness on authenticated subject
plus caller-provided idempotency key. An identical replay, determined by the canonical payload
digest, returns the original receipt. A replay using that subject and key with a different digest
fails with a clear conflict. The store must not accept a caller-supplied subject in place of the
validated, attributable principal.

Implement the first adapter with SQLite. Set `INTAKE_MODE=disabled|sqlite`, defaulting to
`disabled`; require `INTAKE_SQLITE_PATH` when mode is `sqlite`. In disabled mode, do not create a
database, accept an intake write, or expose a path that could persist a submission. Use startup
validation that fails before serving requests for invalid combinations or database initialization
errors. Handle a corrupt or incompatible database explicitly and fail closed; never replace it,
truncate it, or continue with an empty database.

SQLite supports exactly one intake-enabled service instance. Define and validate the repository's
declared instance-mode configuration so startup fails if SQLite intake is combined with a declared
multi-instance configuration. Do not weaken the constraint by relying on a file lock, best-effort
coordination, or a deployment convention. A later shared adapter may satisfy the `IntakeStore`
contract without changing it.

## Deployment and recovery boundary

Keep compiled ontology artifacts inside the immutable image. They remain compiler-owned and are not
placed on, restored from, or coupled to the intake volume. Require a persistent intake volume only
when SQLite intake is enabled, with `INTAKE_SQLITE_PATH` located on that volume. With intake
disabled, retain the current no-volume deployment behavior.

Update the homelab guidance to cover SQLite backup, tested restore, file ownership, encryption at
rest, database rotation, and capacity monitoring. Specify recovery without inventing a per-record
delete endpoint: operator-controlled backup, restore, and rotation operate on the database as a
whole. Document plainly that Prompt 31's multi-instance AWS baseline keeps intake disabled until a
shared durable adapter is deliberately added and verified.

## Tests

Add focused automated tests for:

- claim-to-capability mapping and missing capabilities;
- the explicit `ontology:read` assignment for every current ontology query, description, list, and
  resource handler, and the explicit `ontology:propose` assignment for every current stateless
  proposal-preparation handler;
- post-token-validation capability checks that refuse a validated principal lacking the required
  capability before the current handler executes;
- disabled mode and refusal to enable intake in `none` or `static` mode;
- SQLite initialization and restart persistence;
- atomic receipt creation and idempotent identical replay;
- conflicting replay for the same authenticated subject and idempotency key;
- immutable stored payloads and append-only events;
- corrupt database handling; and
- startup prohibition of SQLite intake in a declared multi-instance configuration.

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
- SQLite stores canonical JSON and SHA-256 digests, atomically creates durable receipts, preserves
  immutable submissions, and records append-only events.
- Idempotency is unique per authenticated subject and key: identical replays return the original
  receipt, while conflicting replays fail.
- SQLite intake requires a persistent volume, supports exactly one intake-enabled service instance,
  and fails at startup with a declared multi-instance configuration.
- The homelab recovery guidance covers backup, restore, ownership, encryption, rotation, and
  capacity; the Prompt 31 AWS multi-instance baseline documents intake as disabled.
- No MCP tool, delivery-plane behavior, compiled ontology artifact, source retrieval, attachment
  opening, or model call is added by this stage.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This is a meaningful architecture, security, and operational decision. If opening a pull request,
apply `narrative-required` and include substantive `## Narrative Context`, `## Narrative Decision`,
and `## Narrative Consequences` sections before merge. Record the disabled-by-default migration, the
single-instance SQLite limit, capability attribution, and the retained immutable-image boundary.
Never hand-edit generated `Narrative.md`.

Commit locally with a focused intake-foundation message. Do not push.
