# Qualified-user ontology intake and governed mapping tools

Status: approved in design review on 2026-08-07.

## Summary

Extend the OntologyServerBuilder sequence so that the built OntologyService can accept bounded,
review-required proposals from qualified users without incorporating them into the deployed
ontology. A separate intake plane stores immutable proposals. An engineer exports a proposal,
verifies the separately delivered source artifact, runs deterministic and semantic analysis, and
uses reviewed prompt templates to author normal repository changes. Only pull-request review,
CI/CD, and deployment can change the ontology or add an executable mapping tool.

Approved entity mappings compile into separately named, pure MCP tools. Qualified users can inspect
the changes in the current deployment, submit further proposals, retrieve source records through
the relevant system tools, invoke a pure mapping tool, and call the target system separately.

## Goals

- Let an authorized conversational agent submit an OpenAPI, exported MCP catalog, or WSDL system
  registration proposal using normalized metadata, provenance, and an artifact digest.
- Keep proposed material outside the compiled ontology and runtime relationship graph.
- Give engineers deterministic parsing and matching results, embedding candidates, LLM analysis,
  disagreements, and unresolved gaps.
- Let qualified users propose corrections to deployed ontology facts without applying them.
- Expose the changes in the current deployment to qualified users through a pull-based MCP surface.
- Compile each approved entity-to-entity mapping into a separately named, schema-specific, pure MCP
  tool.
- Preserve the existing rule that governed repository inputs, review, CI/CD, and deployment are the
  only path to new runtime ontology behavior.

## Non-goals

- OntologyService does not retrieve URLs, connect to live MCP servers, or open user attachments.
- The intake queue does not store raw OpenAPI, MCP, or WSDL bytes.
- Proposal submission does not perform runtime model calls or create accepted ontology facts.
- Qualified users cannot browse, retrieve, edit, or disposition pending submissions.
- Mapping tools do not retrieve source records or invoke target systems.
- The initial design does not add subscriptions, webhooks, or push notifications.
- The initial design does not implement a multi-instance intake database adapter.
- The workflow does not perform a real payment in tests or examples.

## Existing foundation

The existing sequence already teaches:

- deterministic adapters for OpenAPI, JSON Schema, XSD/WSDL, CSV/XLSX, MCP catalogs, and normalized
  JSON;
- deterministic lexical matching and governed mapping review;
- a compiled, read-only ontology delivered through MCP;
- reviewed declarative mapping instructions;
- stateless, review-required system-registration and ontology-refinement proposal packages;
- in-process access-token validation and Keycloak MCP OAuth discovery; and
- a seven-stage guide for adding an offline, cached embedding matcher.

The requested workflow adds four material capabilities: durable intake, engineer-side combined
matching, deployed change visibility, and compiled named mapping tools.

## Architectural decision

Use two planes in one OntologyService product:

1. The **intake plane** is the only new write-capable boundary. It accepts bounded proposals and
   stores immutable payloads plus append-only lifecycle events.
2. The **delivery plane** continues to serve compiled, governed artifacts. It exposes release
   changes and approved named mapping tools but never reads proposal content as ontology facts.

The planes share validated identity infrastructure but use separate authorization checks, data
models, storage, and services. An intake failure cannot fall back to writing an ontology source.
An ontology compilation cannot read the intake database.

```text
Qualified user's agent
  -> retrieve source using the user's existing access
  -> extract bounded normalized metadata
  -> submit review-required proposal
  -> isolated durable intake plane

Engineer
  -> export proposal and obtain original artifact separately
  -> verify digest and deterministically re-parse
  -> deterministic match, cached embeddings, bounded LLM analysis
  -> author governed sources, decisions, relationships, and mapping definitions
  -> lead review, pull request, CI/CD, deployment

Qualified user's agent
  -> inspect deployed release changes
  -> retrieve source records through system tools
  -> invoke a named pure mapping tool
  -> invoke the target API separately
```

### Alternatives not selected

- A separate intake service provides stronger deployment isolation but duplicates authentication,
  deployment, observability, and teaching work before it is needed.
- A Git-native queue provides immediate review history but couples qualified users to Git hosting
  and does not satisfy the requirement for an OntologyServer intake queue.
- Runtime source retrieval or runtime semantic matching would weaken the existing offline compiler
  and read-only ontology trust boundary.

## Identity and authorization

Introduce three explicit capabilities derived from already validated identity claims:

- `ontology:read` permits access to deployed ontology and release-change tools.
- `ontology:propose` permits system-registration and ontology-change submissions.
- `ontology:intake:review` permits engineer queue listing, export, and disposition.

Authentication without the relevant capability is insufficient. Static, Entra, and Keycloak modes
must map their existing claims configuration to these provider-neutral capabilities. The server
must fail closed when a required mapping is absent.

Qualified users receive a submission receipt but cannot retrieve a pending proposal. Engineer MCP
tools and their CLI wrapper require `ontology:intake:review`. Every handler repeats authorization;
tool discovery alone is never treated as an access control.

Submission tools are state-changing, non-destructive, and idempotent. Read and export tools are
read-only. Disposition appends an event and is state-changing but does not edit or delete the
submission.

## Intake data contracts

Support two proposal types.

### System registration

A system-registration submission contains:

- the ontology fingerprint visible to the caller;
- proposed system ID, name, description, and source version;
- source filename, format, media type, byte size, and SHA-256 digest;
- an optional source locator retained only as inert provenance and never followed by the server;
- extractor identity and version, extraction time, and extraction notes;
- normalized entities, attributes, operations or tools, and relationships;
- types, requiredness, allowed values, meanings, and evidence;
- known gaps, warnings, and questions for the source owner; and
- a caller-generated idempotency key.

The initial accepted source formats are OpenAPI JSON/YAML, exported MCP catalog JSON, and WSDL.
The contract reuses the existing filename canonicalization and structural-validation policy.

### Deployed ontology change

An ontology-change submission contains:

- the ontology fingerprint on which the proposal is based;
- referenced deployed system, entity, attribute, relationship, semantic-mapping, mapping-tool, or
  requirement IDs;
- the proposed correction, addition, deprecation, clarification, or mapping refinement;
- evidence and expected workflow consequences;
- unresolved questions and declared assumptions; and
- a caller-generated idempotency key.

Stable references are validated against the deployed ontology. A stale base fingerprint creates a
warning and cannot be promoted automatically, but it does not discard otherwise valid evidence.

### Prohibited content

Neither contract accepts credentials, access tokens, live business records, transaction instances,
personal data, payment details, local filesystem paths, source URLs for the server to follow, or raw
artifact bytes. A source URL may be recorded only as inert provenance, not as a retrieval request.
Arbitrary text is inert data. Structural controls cannot claim to detect every secret or personal
record; the caller and engineer remain responsible for excluding them.

## Intake storage and lifecycle

Define an `IntakeStore` interface and ship SQLite as the initial single-instance adapter. The
database is outside `ontology/`, is never compiled, and is not copied into the container image.
Its configured path must be on a persistent deployment volume.

Store canonical JSON payloads and their SHA-256 digests. A submission row is immutable. Lifecycle
changes are separate append-only events. The service API exposes no update or delete operation for
a stored payload.

The lifecycle is:

```text
received -> exported -> accepted | rejected | superseded
```

Repeated export may append another `exported` event. Terminal dispositions do not imply that a
repository change has merged. An `accepted` disposition means only that an engineer accepted the
intake item for governed engineering work.

A successful submission returns a receipt with the server-generated intake ID, payload digest,
received timestamp, and `received` status. The server issues no receipt until the transaction is
durable. A queue failure returns an error.

The uniqueness key is authenticated subject plus idempotency key. Reuse with an identical payload
digest returns the original receipt. Reuse with a different digest fails. Tests inject the clock
and ID generator; production timestamps and IDs need not be deterministic.

The deployment guide must cover file permissions, volume ownership, backup, restore, retention,
capacity monitoring, and encryption at rest. Per-record deletion is not exposed through MCP;
operator-controlled archival or database rotation applies the configured retention policy. A later
decision may add a multi-instance adapter without changing the `IntakeStore` contract.

## Engineer intake workbench

Add engineer-scoped list, export, and disposition operations plus a CLI wrapper suitable for a
coding-agent session. Export creates a canonical package but does not include the raw source.

The engineer obtains the original artifact through an approved separate channel and passes its
local path explicitly to the analysis command. The command must:

1. verify filename, format, byte size, and SHA-256 digest;
2. re-parse the artifact with the existing offline adapter;
3. compare parsed output with submitted normalized metadata;
4. report omissions, additions, type conflicts, and provenance conflicts;
5. run deterministic entity and attribute matching;
6. run cached embedding matching when configured;
7. create a bounded LLM request package;
8. validate the returned LLM report; and
9. emit one consolidated, canonical analysis report.

The command never fetches the source, edits ontology files, opens a pull request, or changes an
intake disposition.

## Deterministic and semantic matching

### Deterministic pass

Reuse the existing matcher and extend its evidence where necessary for interface definitions. The
pass considers normalized names, descriptions, attribute names, types, requiredness, allowed
values, operation or tool context, declared relationships, and governed synonyms.

The intake report uses advisory dispositions:

- `exact-candidate`;
- `likely-candidate`;
- `review-required`; and
- `unmatched`.

No intake candidate has a governed `accepted` disposition. Existing compiler rules apply only after
an engineer checks in a reviewed source and runs normal compilation.

### Embedding pass

Implement the existing Step 4 guide as separately verified stages. Embedding text, vectors, content
hashes, model identity, model version, dimension, cache digest, fusion weights, and component scores
are retained as evidence. Compilation and runtime remain offline. An embedding result may surface a
candidate for review but cannot make it accepted automatically.

### LLM advisory pass

The workbench emits a schema-versioned, bounded request containing only the relevant normalized
entities, deterministic candidates, embedding candidates, provenance, and explicit questions. An
engineer's coding agent uses the reusable Builder prompt to produce a JSON response.

The response records the model, model version, prompt version, input digest, and analysis time. It
may explain candidates, identify gaps, or disagree with earlier passes. It cannot remove
deterministic findings, assign a governed acceptance status, write repository files, or execute
source text. Invalid or absent output marks semantic analysis incomplete while leaving the earlier
passes usable.

### Consolidated report

The final report contains:

- parser agreement and discrepancy sections;
- deterministic scores and component evidence;
- embedding scores and cache provenance;
- LLM recommendations and provenance;
- agreement and disagreement across passes;
- collisions, missing meanings, relationship gaps, unresolved requirements, and questions;
- suggested governed repository destinations; and
- the source and ontology fingerprints used.

Equivalent normalized inputs and a fixed embedding cache produce byte-identical non-LLM sections.
LLM output is traceable advisory evidence, not deterministic evidence.

## Governed repository promotion

OntologyServerBuilder will provide reusable operational prompt templates for:

- registering an exported intake package;
- resolving its mapping-review report;
- creating or revising a named mapping tool; and
- applying a qualified user's deployed-ontology change proposal.

The templates instruct a coding agent working in OntologyService. They never edit either repository
automatically. The engineer reviews the report, selects evidence, and authorizes a bounded task.

Accepted work follows the existing source, manifest, manual mapping, relationship, mapping
instruction, compilation, pull-request, Narrative, CI/CD, and deployment workflow. Generated
ontology artifacts are compiler-owned. Decision-bearing pull requests carry `narrative-required`
and the three required Narrative sections before merge.

## Deployment release visibility

CI generates a release manifest by comparing the candidate compiled ontology with an explicitly
supplied previously deployed artifact and explicit release metadata. The manifest is deterministic
from those inputs and does not contribute to the ontology fingerprint.

The manifest contains:

- release ID and deployment timestamp;
- current and previous ontology fingerprints;
- added, changed, deprecated, and removed systems, entities, attributes, relationships, semantic
  mappings, mapping definitions, and named mapping tools;
- compatibility-impact classification; and
- governed provenance for each change.

The delivery plane exposes the current manifest through a bounded read-only MCP tool and resource.
It does not promise an unbounded release history. Qualified users can use the current delta to
prepare an ontology-change submission. Pending intake content and engineer dispositions are never
included.

## Named deterministic mapping tools

Extend governed mapping definitions with:

- a stable mapping ID and stable MCP tool name;
- source and target entity IDs;
- explicit input and output JSON Schemas;
- preconditions and an allow-listed transformation program;
- required supporting lookup inputs;
- failure behavior, lifecycle status, and semantic version;
- evidence, unresolved requirements, and review provenance; and
- reviewed positive, negative, boundary, and ambiguity examples.

An approved and structurally complete definition compiles into a thin, separately named MCP adapter
backed by the restricted mapping evaluator. The generator does not accept arbitrary source code,
expressions, callbacks, imports, network operations, or template escapes. Review-required,
deprecated, incomplete, or invalid definitions do not register tools.

A mapping tool receives the source record and all required lookup records. It returns an envelope
containing either a schema-valid target record or structured failures such as:

- `precondition-failed`;
- `missing-input`;
- `ambiguous-lookup`; or
- `target-validation-failed`.

The envelope also identifies the mapping ID, mapping version, and ontology fingerprint. A tool is
pure, deterministic, idempotent, and isolated from filesystem, network, environment, clock, and
randomness. The conversational agent retrieves source and lookup records and invokes the target API
separately.

## Reference workflow

The end-to-end assurance example uses an invoice-management MCP server and an accounts-payable
OpenAPI definition.

1. A qualified user's agent retrieves the invoice MCP catalog, extracts normalized metadata, and
   submits a system-registration proposal.
2. The agent later retrieves the accounts-payable OpenAPI definition and submits another proposal.
3. An engineer exports both, obtains the original artifacts separately, verifies their digests, and
   runs the workbench.
4. Deterministic, embedding, and LLM reports identify entity candidates and unresolved gaps.
5. The engineer uses the Builder registration prompt to author governed OntologyService changes.
6. The engineer lead reviews the decision-bearing pull request; CI compiles and checks the change;
   the change is merged and deployed.
7. The new release manifest makes the deployed systems and semantic changes visible to qualified
   users.
8. An engineer uses the mapping-tool prompt and analysis evidence to author an invoice-to-payment
   mapping definition and examples.
9. Review, CI/CD, and deployment register a named pure mapping tool.
10. A user asks an agent to pay an invoice. The agent retrieves the invoice and required supporting
    records, invokes the mapping tool, presents or applies the required approval policy, and calls
    the accounts-payable API separately.

The automated test stops before the final target-system call and uses synthetic records only.

## Failure handling

- Structurally invalid, malformed, or oversized submissions fail closed.
- Unfamiliar but safe business semantics remain inert review gaps.
- Queue unavailability fails without a receipt.
- Artifact mismatch stops analysis before parsing or matching.
- Parser disagreement remains visible and cannot be overwritten by semantic analysis.
- Missing, malformed, or rejected LLM output leaves deterministic and embedding evidence intact.
- Stale proposals remain reviewable but cannot be promoted automatically.
- An invalid mapping definition fails compilation and prevents deployment rather than silently
  omitting an approved mapping.
- Release comparison fails when the declared previous artifact or provenance is missing.

## Testing and acceptance strategy

Each stage adds focused unit, store, CLI, compiler, and in-memory MCP tests as appropriate. Required
coverage includes:

- capability enforcement for every new operation;
- receipt idempotency, digest mismatch, atomicity, recovery, and append-only events;
- filename, ID, digest, size, and bounded-array validation;
- user inability to list or retrieve submissions;
- engineer export and disposition behavior;
- artifact verification and offline adapter execution;
- deterministic reordered-input and byte-identical report tests;
- embedding cache validation, stable ranking, provenance, and no-runtime-network tests;
- strict LLM request and response schema tests without requiring a live model in CI;
- release-delta determinism and compatibility classification;
- mapping-tool schema validation, purity, no-I/O sentinels, stable failures, and golden examples;
- proof that submissions and analysis never mutate the compiled ontology; and
- the synthetic invoice-to-payment workflow without a target API side effect.

Every implementation prompt requires `npm run check` and `git diff --check`. The final independent
audit may report a failure but may not weaken a control or acceptance check to make the scenario
pass.

## Builder prompt sequence

Add these stages after Prompt 32b:

33. Plan qualified-user intake and governed mapping-tool delivery.
34. Add capability authorization and the durable append-only intake store.
35. Add qualified system-registration and ontology-change submissions.
36. Add the engineer intake workbench and deterministic analysis report.
37. Implement Step 4 embedding text and cache primitives.
38. Add embedding configuration and evidence contracts.
39. Add the explicit embedding refresh command and network isolation.
40. Fuse embeddings into matching without automatic acceptance.
41. Integrate the validated embedding cache into deterministic compilation.
42. Complete embedding documentation, MCP evidence, and acceptance checks.
43. Add bounded coding-agent LLM analysis and consolidated intake reports.
44. Add deterministic deployment release manifests and qualified-user visibility.
45. Compile approved mapping definitions into named pure MCP tools.
46. Register the accounts-payable example and approve the invoice-to-payment mapping tool.
47. Independently audit the complete qualified-user-to-deployment workflow.

Prompt 33 changes no runtime behavior. The current `docs/step4.md` in the built repository is a
seven-stage guide: Prompts 37–41 preserve its first five stages, while Prompt 42 deliberately
combines stages 6 and 7 into the final embedding delivery and acceptance stage. Prompt 47 is an
audit stage and cannot repair defects inside the audit task.

Each prompt will update the README sequence in the same change that adds it. Every prompt will state
its dependencies, scope exclusions, executable acceptance evidence, Project Narrative
classification, and local commit boundary.

## Consequences

The product gains a controlled contribution path for qualified non-engineers without turning the
runtime ontology into a self-modifying system. Engineers receive richer evidence and reusable
coding-agent prompts, while deployment remains the only activation boundary.

The cost is a new stateful intake subsystem, capability authorization, operational backup and
retention duties, a longer teaching sequence, and additional generated-tool assurance. SQLite is a
deliberate single-instance baseline; multi-instance production storage remains a later governed
decision.
