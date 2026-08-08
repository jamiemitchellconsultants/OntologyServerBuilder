# Prompt 36 — Add the engineer intake workbench

Using Prompt 34's durable `IntakeStore` and Prompt 35's qualified-user submission contracts, add
the engineer-only intake workbench in the separate `OntologyService` repository. Do not change
`OntologyServerBuilder` in this stage.

This stage lets an authorised engineer list, export, analyse, and record review dispositions for
durable intake evidence. It does not make a submission, a candidate, or a disposition into an
ontology fact; it does not merge or deploy anything; and it does not alter the delivery plane. A
later, human-reviewed registration task uses the exported evidence. When an engineer invokes that
separate workflow, they must supply the reviewed operator-template contents alongside the coding
task; the target repository neither contains nor reads that template.

The original source artifact is delivered separately to the engineer. Only the CLI receives its
explicit local path. Never store that path in an intake record, export, event, analysis report, log,
or task artifact.

## Read before editing

Read Prompts 33–35; the current `IntakeStore`, S3 adapter, capability authorization, MCP
registration and annotation conventions; the offline source adapters and matcher; compiler and
manifest workflow; tests; and Project Narrative rules. Adapt names and paths to the repository as
it exists. Do not assume a later prompt has introduced a shared intake adapter, automated
registration, a release workflow, or a model-backed reviewer.

## Engineer-only review operations

Register exactly these MCP tools, each requiring `ontology:intake:review` after authentication has
provided an already validated, attributable `AuthorisedPrincipal` and before any operation:

1. `ontology_intake_list`
2. `ontology_intake_export`
3. `ontology_intake_disposition`

Tool discovery and a valid token do not substitute for this capability. Annotate the tools using
the repository's MCP convention: list and export are read-only; disposition is state-changing,
non-destructive, and not idempotent unless the repository already has a sound event-idempotency
contract. Do not expose any of these operations to qualified users or unauthenticated callers.

`ontology_intake_list` returns bounded, paginated summaries only: opaque submission ID, submission
kind, received timestamp, current advisory status, payload digest, and enough bounded review-event
summary to select an ID. It must not return a full payload, original artifact, local path, or raw
unbounded event history. Validate a stable cursor and a finite page-size maximum; use a documented
default and reject an invalid or excessive page size.

`ontology_intake_export` accepts one opaque submission ID and returns the canonical export for that
submission: immutable canonical payload, receipt, complete append-only event history, payload and
provenance digests, and enough schema/version metadata to reproduce review. Its machine-readable
form is canonical JSON named `intake-export.json`; identical stored evidence must serialize to
byte-identical canonical output. An export event may be appended without changing the submission
payload. Do not add update, delete, payload replacement, artifact download, or retrieval tools.

`ontology_intake_disposition` accepts one submission ID, exactly one status from `accepted`,
`rejected`, or `superseded`, and a bounded non-empty human reason. It derives the actor from the
authorised principal and appends an event containing that actor, reason, status, and injected or
otherwise deterministic timestamp. It never rewrites the payload, receipt, prior events, or a
prior disposition. A disposition is an advisory intake-review record only: `accepted` does not
mean governed semantic acceptance, ontology registration, merge approval, compilation, release, or
deployment. Reject attempts to invent other statuses or caller-supplied actors/timestamps.

Add an engineer CLI wrapper for export. The wrapper uses the same authorization boundary or a
deliberately equivalent attributable engineer credential path; it must not bypass capability
authorization. Its export command takes a submission ID and an explicitly named local output path,
then writes `intake-export.json` only at that chosen path. It must not derive a filename from
untrusted metadata, accept a directory as the output target, or write a local artifact path into
any stored record. The CLI's separate artifact-analysis command receives the original artifact as
an explicit local path only.

## Offline artifact verification and re-parsing

The CLI analysis command takes explicit paths to an exported `intake-export.json`, the original
artifact, and an explicitly named `deterministic-analysis.json` output file. Before parsing, verify
the artifact's normalized filename, declared compatible format and media type, byte size, and
SHA-256 digest against the immutable exported provenance. Fail closed on any mismatch and do not
parse, amend, or replace the export. The local artifact path is process-local input: do not include
it in the analysis report or any persisted intake data.

Reuse the existing OpenAPI JSON, OpenAPI YAML, exported MCP catalog JSON, or WSDL adapters as
appropriate. Parsing must run with network access disabled: no URL retrieval, resolver, attachment
opening beyond the explicitly named local artifact, model call, repository mutation, or compiled
ontology mutation. Make network prohibition testable with sentinels around the repository's network
entry points. Do not fall back to a network resolver when a document refers to an external resource.

Compare the adapter's deterministic normalized result with the submitted normalized metadata. The
analysis report must identify omissions, additions, type differences, requiredness differences, and
provenance conflicts with stable identities and enough submitted-versus-parsed values for review.
Write a canonical, deterministic JSON report named `deterministic-analysis.json`; exclude local
paths, credentials, raw source bytes, live records, and any mutable machine-specific detail.

## Deterministic advisory matching

After verification and re-parsing, use the existing offline matcher to compare the re-parsed result
with the compiled ontology. Cover name, description, attributes, types, allowed values,
operations/tools, relationships, and governed synonyms. The matcher must be deterministic and
offline: no model, embedding service, network request, or repository mutation.

For every comparison, emit only one advisory status: `exact-candidate`, `likely-candidate`,
`review-required`, or `unmatched`. Never emit an accepted governed disposition, automatically add
a mapping, or treat an `exact-candidate` as approval. Sort every array in the report by stable ID,
including nested discrepancy and match arrays. Reordering otherwise identical submitted entities,
attributes, relationships, operations, allowed values, or matcher input must produce byte-identical
canonical analysis output.

## Tests

Add focused offline store, CLI, and linked in-memory MCP tests for:

- refusal of all three tools and the CLI export without `ontology:intake:review`, with success only
  for an attributable principal holding it;
- pagination default, maximum, invalid cursor/page-size rejection, and bounded list summaries;
- exact registered tool names, annotations, and canonical byte-identical `intake-export.json`;
- append-only disposition events with derived actor, reason, timestamp, and only `accepted`,
  `rejected`, or `superseded`; prove no disposition is a merge or deployment action;
- explicit local export target handling and proof that artifact paths never enter an intake row,
  event, export, or analysis report;
- filename, format/media-type, byte-size, and digest mismatch refusal before parsing;
- network-disabled parsing, no model call, and no repository, compiler, generated-artifact, or
  ontology mutation during export, analysis, and matching;
- parser discrepancies for omissions, additions, type differences, requiredness differences, and
  provenance conflicts;
- only the four permitted advisory match statuses, stable-ID sorting, and byte-identical reports for
  reordered equivalent inputs; and
- preservation of existing Prompt 35 qualified-user tools without adding qualified-user list,
  export, artifact retrieval, or disposition operations.

Keep tests offline. Run sentinels after fixture setup around network clients, model clients, source
retrieval, compiler writes, and repository-mutating commands. No test may reach a live endpoint.

## Acceptance criteria

- The exact three engineer review tools enforce `ontology:intake:review`; list is bounded, export
  is canonical, and disposition is append-only with the three stated advisory statuses.
- The CLI writes an export only to an explicit local path and receives the original artifact path
  only for local verification and analysis; neither path is persisted or reported.
- Artifact verification happens before parsing and rejects filename, format, size, media-type, or
  digest disagreement without network, model, or repository mutation.
- `deterministic-analysis.json` reports the stated parser differences and uses only the four stated
  advisory candidate statuses, with every report array stably sorted by ID.
- Candidates and dispositions remain review evidence, never a governed ontology acceptance,
  registration, merge, deployment, or generated-artifact edit.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This is a meaningful product, security, and architecture decision. If opening a pull request,
apply `narrative-required` and include substantive `## Narrative Context`, `## Narrative
Decision`, and `## Narrative Consequences` sections before merge. Never hand-edit generated
`Narrative.md`.

Commit locally with a focused engineer-intake-workbench message. Do not push.
