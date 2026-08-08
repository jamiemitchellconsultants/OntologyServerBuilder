# Prompt 35 — Add qualified-user intake submissions

Using the durable `IntakeStore` and capability authorization from Prompt 34, add the two
qualified-user submission tools in the separate `OntologyService` repository. Do not change
`OntologyServerBuilder` in this stage.

This stage accepts review-required evidence into the intake plane. It does not make submissions
into ontology facts, alter the compiled ontology, add an engineer workbench, or change the delivery
plane. Preserve the existing read-only preparation tools from Prompt 18; the submit tools consume
their normalized output rather than replacing them.

The runtime must never fetch a source locator, open an attachment, receive raw artifact bytes, or
call a model. A locator is inert provenance only, never an instruction to retrieve content.

## Read before editing

Read Prompt 33's approved intake plan; Prompt 34; the existing proposal-preparation tools and their
validators; the compiled-ontology models and fingerprint code; MCP registrations and annotations;
the `IntakeStore` contract and SQLite adapter; capability authorization; tests; and Project
Narrative rules. Adapt names and paths to the built repository as it exists. Do not assume a later
prompt has added a queue workbench, export, disposition, release manifest, mapping tool, or shared
intake adapter.

## Submission tools and authorization

Register exactly these two MCP tools:

1. `ontology_submit_system_registration`
2. `ontology_submit_change_proposal`

Both handlers require `ontology:propose` after authentication has supplied an already validated,
attributable `AuthorisedPrincipal`, and before validation or store submission. A valid token or tool
discovery must not substitute for this capability check. Derive the submission subject solely from
that principal; never accept a caller-supplied subject.

Annotate both tools using the repository's MCP annotation convention as state-changing,
non-destructive, and idempotent. A successful tool result is a receipt, not a representation of a
pending queue item. Do not add a qualified-user list, get, retrieval, status, update, delete, or
disposition tool. Engineer queue operations remain a later stage.

## Shared canonical submission and receipt contract

Define and export the canonical submission types and receipt schema that Prompt 36 will consume.
The handler constructs one canonical payload, records its canonical JSON SHA-256 digest through
`IntakeStore`, and submits it with the authenticated subject and idempotency key. Do not permit a
caller to provide the canonical payload digest or receipt fields.

On successful durable submission, return only this receipt shape, adapting property casing only to
the service's established public schema convention:

```text
{
  id: opaque server-generated identifier,
  payloadDigest: SHA-256 digest of canonical payload,
  receivedAt: timestamp,
  status: "received"
}
```

The receipt is returned only after the immutable row and initial `received` event are durable.
An identical replay by the same authenticated subject with the same idempotency key returns the
original receipt. Reuse of that subject and key with a different canonical payload digest fails with
a clear conflict. Store failure, including an unavailable queue, fails without a receipt. Never
silently retry a submission in a way that can duplicate an accepted write.

Apply these exact limits at the MCP boundary and again at the store boundary. Count Unicode scalar
values, not UTF-16 code units, where a character limit is stated. Validate all nested content before
canonicalization and durable write.

```text
canonical payload: 2 MiB
normalized filename: 255 Unicode scalar values
entities: 500
attributes per entity: 1,000
operations/tools: 1,000
relationships: 1,000
allowed values per attribute: 1,000
individual free-text field: 16,000 Unicode scalar values
idempotency key: 8–128 characters from [A-Za-z0-9._:-]
SHA-256: 64 lower-case hexadecimal characters
```

Use the existing Unicode NFC filename canonicalization and structural filename policy from Prompt
19. Reject path separators, path-like names, empty names, control characters, deceptive directional
controls, and a post-normalization filename beyond the stated limit. Preserve ordinary Unicode.
Reject malformed IDs, duplicate canonical identities, invalid digests, invalid idempotency keys,
oversized structures, and structurally prohibited content. Do not turn unfamiliar but safe
datatypes, languages, status conventions, incomplete meanings, missing evidence, or instruction-like
text into a rejection: retain them as inert review gaps, warnings, or questions.

## System-registration submission

`ontology_submit_system_registration` accepts the normalized output of
`ontology_prepare_system_registration_request` together with source provenance and
durable-submission metadata. Its schema must require:

- proposed stable system ID, name, description, source version, entities, attributes,
  operations/tools, relationships, meanings, requiredness, allowed values, evidence, gaps, warnings,
  and questions from the existing preparation workflow;
- source format, media type, byte size, SHA-256 digest, normalized filename, and inert source
  locator;
- extractor identity, version, extraction timestamp, and extraction notes;
- the compiled ontology fingerprint visible to the caller; and
- the caller-generated idempotency key.

Accept only OpenAPI JSON, OpenAPI YAML, exported MCP catalog JSON, and WSDL. Validate the declared
format, filename extension, and media type as a compatible combination under the repository's
existing conventions. The declared byte size is provenance that must be a non-negative safe integer;
it is not an uploaded payload size.

Never accept raw document bytes, attachments, credentials, access tokens, local filesystem paths,
live business records, transaction instances, payment data, personal data, or a URL for the server
to fetch. A safely formed source locator may be retained as inert provenance only. The tool must not
open, resolve, dereference, parse, or otherwise use it.

Validate the proposed structure with the existing registration validators. Relationships must have
valid endpoints and involve the proposed system where that validator requires it. Do not register
the system, add a runtime relationship, invoke semantic matching, call a model, or mutate compiled
ontology bytes. Submit the validated canonical payload to `IntakeStore` as a system-registration
submission.

## Ontology-change submission

`ontology_submit_change_proposal` accepts the normalized output of
`ontology_prepare_mapping_update_request` and records a correction proposal against deployed facts.
Its schema must require:

- stable deployed references to the relevant system, entity, attribute, relationship,
  semantic-mapping, mapping-tool, or requirement IDs;
- the caller's base ontology fingerprint;
- a declared change kind: correction, addition, deprecation, clarification, or mapping refinement;
- the proposed change, evidence, expected workflow effect, declared assumptions, gaps, warnings,
  and unanswered questions; and
- the caller-generated idempotency key.

Validate every reference against the current compiled ontology before storage. Reject unknown,
malformed, or internally inconsistent references and structural content that cannot be represented
by the submitted change kind. Do not accept caller-provided replacement ontology objects, source
bytes, credentials, records, payment data, personal data, local paths, or retrieval URLs.

Compare the submitted base fingerprint with the current compiled ontology fingerprint. A mismatch is
not a reason to discard otherwise valid evidence: store the submission with a `stale-base` warning
that is visible in the canonical payload. A stale submission is review-required evidence and must
never be promoted, rewritten, or automatically rebased by this tool. A matching fingerprint does not
promote it either.

The handler does not alter a deployed system, relationship, mapping, requirement, compiled ontology
artifact, or delivery-plane behavior. Submit the validated canonical payload to `IntakeStore` as an
ontology-change submission.

## Tests

Add focused store-level and linked in-memory MCP tests. Keep them offline and ensure they use an
intake-enabled test store only where required. Cover:

- authorization failure for a validated principal without `ontology:propose`, and success only for
  an attributable principal that has it;
- every stated bound at its accepted edge and rejection one unit beyond it, including Unicode scalar
  counting, filename NFC normalization, nested array limits, individual free-text fields,
  idempotency keys, SHA-256 form, and the 2 MiB canonical-payload limit;
- identical idempotent replays returning the original receipt, duplicate subject/key conflicts for
  a different canonical payload, and caller inability to choose the submission subject;
- submitted provenance digest mismatch or malformed digest rejection, including incompatible source
  format, media type, or filename-extension combinations;
- current-fingerprint acceptance and stale-fingerprint preservation with `stale-base`, without
  automatic promotion or rebase;
- preservation of unfamiliar but structurally safe metadata as inert review gaps;
- rejection of prohibited structural content, paths, raw bytes, credentials, records, payment data,
  personal data, and retrieval instructions where represented structurally;
- queue failure producing no receipt or partial durable submission;
- both tool annotations and the exact registered names;
- preservation of all existing preparation tools and proof that no qualified-user retrieval, list,
  status, update, delete, or disposition operation is registered; and
- byte-for-byte proof that submitting accepted, rejected, replayed, and stale proposals leaves the
  compiled ontology unchanged.

Install sentinels appropriate to the repository's test conventions around filesystem access,
network entry points, and model clients after the fixture ontology is loaded. Prove neither handler
fetches a source locator, opens an attachment, receives bytes, nor calls a model. Exercise MCP
cases through linked in-memory transports, never a live endpoint.

## Acceptance criteria

- Exactly the two named qualified-user submission tools exist, both require `ontology:propose`, and
  both are state-changing, non-destructive, and idempotent.
- System-registration submissions accept only bounded normalized metadata plus declared provenance
  for the three source families, while keeping source material outside the runtime boundary.
- Ontology-change submissions validate deployed stable references and preserve, but never
  automatically promote, a `stale-base` submission.
- Durable receipts are atomic, opaque, digest-bound, idempotent per authenticated subject and key,
  and contain `received` status; queue failure returns no receipt.
- Existing preparation tools remain available. No qualified-user queue retrieval, list, status,
  update, delete, or disposition operation exists.
- All stated bounds, Unicode normalization, provenance validation, and structural rejection rules
  have store-level and in-memory MCP evidence.
- Submissions perform no retrieval, attachment opening, raw-byte handling, model call, or compiled
  ontology mutation.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This is a meaningful product, security, and architecture decision. If opening a pull request,
apply `narrative-required` and include substantive `## Narrative Context`, `## Narrative Decision`,
and `## Narrative Consequences` sections before merge. Never hand-edit generated `Narrative.md`.

Commit locally with a focused qualified-intake-submission message. Do not push.
