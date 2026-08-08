# Prompt 45 — Compile named mapping tools

Using the governed mapping instructions from Prompt 17 and the current compiled entity and
attribute schemas, add approved named mapping tools in the separate `OntologyService` repository.
Do not change `OntologyServerBuilder` in this stage. This is the delivery-plane activation of a
reviewed mapping definition: it must not make intake content, matching evidence, or a coding-agent
recommendation executable.

## Read before editing

Read Prompts 17, 33–36, 43, and 44; the approved qualified-user intake plan; `AGENTS.md`; Project
Narrative rules; governed mapping instructions; ontology source and compiler code; compiled schema
and fingerprint code; MCP registration, authorization, and resource conventions; canonical JSON
and schema-validation helpers; and their tests. An engineer may separately supply the reviewed
named-mapping-tool operator-template contents when invoking a governed definition change; it is not
a target-repository artifact and this stage must not read or create it. Adapt names and paths to the
target repository as it exists. Do not assume Prompt 46 has registered an accounts-payable system or
approved an invoice-to-payment mapping.

## Governed mapping-tool definitions

Add a reviewed source such as `ontology/mapping-tools.json`. Each mapping-tool definition must
contain, at minimum:

- a stable mapping-tool ID and a stable MCP tool name;
- the approved mapping-instruction ID it implements, source and target entity IDs, and semantic
  version;
- explicit input and output JSON Schemas, plus JSON Schemas for every named supporting lookup
  input;
- preconditions, a declarative transformation program using only the already governed operation
  allow-list, and the mapping's structured failure contract;
- lifecycle status, evidence, unresolved requirements, review provenance, and the ontology
  fingerprint or governed source context on which review occurred; and
- reviewed positive, negative, boundary, and ambiguity examples with their expected envelope.

Definitions are governed data, not source code. A definition may reference only a complete,
approved mapping instruction and the compiled source and target entities and attributes that it
declares. Do not accept arbitrary expressions, source code, callbacks, imports, module names,
template escapes, executable templates, generated functions, or a mechanism to select an operation
outside the reviewed allow-list. Do not treat deterministic, embedding, or LLM matching evidence
as approval.

## Compile and register only complete approved definitions

Extend deterministic compilation to validate every definition before deployment. Fail compilation
for a missing, duplicate, or invalid stable ID; invalid or colliding MCP name; unknown mapping
instruction, entity, attribute, relationship, operation, or schema keyword; source/target or
lookup-schema disagreement; an incomplete schema; an unresolved requirement; malformed review
provenance; unsupported status; malformed example; or any example whose expected result is
inconsistent with the declared contract. Enforce the repository's JSON-Schema dialect strictly;
reject unknown keywords or features that would make evaluation ambiguous.

Validate that the referenced mapping instruction is approved, structurally complete, and compatible
with the declared entities, lookups, preconditions, transformations, and failure behavior. A
`review-required`, deprecated, incomplete, invalid, or unresolved definition or instruction must
fail closed: it must not produce a tool descriptor or MCP registration. Do not silently omit an
entry that claims to be approved; make the compilation error actionable.

Compile only complete approved definitions into canonical, stable-ID-sorted descriptors that are
part of the reviewed compiled ontology and fingerprint according to existing conventions. Each
descriptor must retain the mapping-tool ID, stable MCP name, version, source and target schema
identity, input shape, failure contract, review provenance, and enough governed evidence for
runtime provenance. Repeated compilation of equivalent reordered source objects must produce
byte-identical descriptors and fingerprint inputs. Never hand-edit compiled artifacts.

Integrate descriptors with Prompt 44's existing release-manifest generator and its optional,
canonical named-mapping-tool descriptor collection. Do not replace the manifest schema, generator,
or its forward-compatible empty-class rule. The candidate artifact must expose the stable-ID-sorted
compiled descriptors through that interface, so the union-based release classifier emits added,
changed, deprecated, and removed named-tool records when either artifact has descriptors. Mapping
descriptors remain part of the ontology fingerprint according to the existing conventions; the
release manifest and release metadata remain non-participating delivery metadata.

At startup, load only validated compiled descriptors. Register exactly one separately named MCP
tool closure for each descriptor using its stable MCP name. Tool discovery must therefore show only
approved, complete mapping tools and must change deterministically when the compiled descriptors
change. Do not register a generic execute-any-mapping tool, dynamically discover definitions from
the filesystem, intake store, network, or a mutable runtime source, or generate code at startup.

Every generated closure must authenticate to the existing `AuthorisedPrincipal` and require
`ontology:read` using the repository's established authorization path before it validates tool
input, selects a descriptor, or invokes the evaluator. Reject an authenticated principal lacking
that capability with the existing authorization error convention, without revealing tool data,
schema details, descriptor provenance, mapping outcomes, or validation errors. Tool discovery,
where the existing MCP framework authorizes it, and every invocation must preserve the same
fail-closed policy; a missing capability must produce no evaluation and no I/O.

## Restricted pure evaluator and result contract

Back each closure with one restricted evaluator over the compiled descriptor and its supplied
input. The evaluator may interpret only the reviewed declarative operation allow-list; it must not
generate or execute arbitrary source code, expressions, callbacks, imports, templates, shell
commands, or dynamic modules. It must not read or write the filesystem, use the network,
environment, clock, randomness, process state, source API, target API, intake store, release
manifest, or any external system.

The tool input must contain exactly one source record and a named collection of all required lookup
records, validated against the descriptor's source and lookup-input schemas. The caller retrieves
those records through the appropriate system tools before invocation; this mapping tool never does
so itself. Validate input before evaluation and validate every successful target record against the
compiled output schema.

Every call returns a deterministic envelope containing mapping-tool ID, mapping-instruction ID,
semantic version, ontology fingerprint, and exactly one outcome:

- a schema-valid `targetRecord`; or
- `precondition-failed`;
- `missing-input`;
- `ambiguous-lookup`; or
- `target-validation-failed`.

Define canonical error details that identify the declared input, lookup, precondition, or target
schema rule involved without exposing source artifacts, credentials, local paths, intake records,
or live business data beyond the caller's input. The same semantically equivalent input, including
reordered object keys, must yield byte-equivalent output. The evaluator is pure, deterministic,
idempotent, and has no side effects. A mapping result is an instruction for the caller, not
permission for the service to make a source or target API call.

## Tests and boundaries

Add focused, offline tests covering:

- every required definition field, reference, status, schema keyword, operation, provenance,
  unresolved-requirement, and example validation rule;
- omission of non-approved, deprecated, incomplete, invalid, and unresolved definitions, and
  actionable failure for a definition falsely claiming approval;
- stable MCP name collision rejection, stable-ID ordering, repeated byte-identical compilation,
  compiled-descriptor provenance, and fingerprint participation;
- deterministic Prompt 44 release deltas for added, changed, deprecated, and removed named mapping
  tools, including a descriptor collection absent in one artifact, while proving release-manifest
  and release-metadata changes do not alter the ontology fingerprint;
- dynamic MCP discovery and registration of exactly one named closure per approved descriptor,
  with no generic execution endpoint or mutable runtime discovery;
- an allowed qualified principal invoking a named tool, and an authenticated principal without
  `ontology:read` rejected through the existing authorization error before schema validation,
  evaluation, descriptor disclosure, or any I/O sentinel can run;
- source, lookup, and target JSON-Schema validation; stable results for reordered object keys; and
  positive, negative, boundary, and ambiguity examples for every compiled mapping tool;
- each structured failure outcome: `precondition-failed`, `missing-input`, `ambiguous-lookup`, and
  `target-validation-failed`, including deterministic, non-sensitive failure provenance;
- sentinels installed after fixture setup proving compilation, server startup, MCP discovery, and
  tool invocation cannot generate or execute code, load a module, read or write a file, use an
  environment value, consult time or randomness, query an intake store or release manifest, fetch
  a source, call a model, resolve an endpoint, or call a source or target API; and
- repeated invocation idempotence and no side effect, including proof that the synthetic workflow
  ends with a target record rather than a target-system call.

Keep tests deterministic and offline. Do not hand-edit `ontology/compiled/` or generated Project
Narrative output.

## Acceptance criteria and verification

- Only complete, reviewed, approved mapping-tool definitions that reference approved governed
  instructions compile into stable, fingerprinted descriptors and one named MCP tool each.
- The evaluator is a restricted, deterministic interpreter with schema-validated records and
  structured outcomes; it cannot execute generated code or perform I/O.
- Tool discovery exposes the exact approved descriptor set, while review-required, deprecated,
  incomplete, invalid, and unresolved definitions cannot become callable by omission or fallback.
- Every named mapping-tool invocation requires `ontology:read` on an authenticated
  `AuthorisedPrincipal` before it can disclose a descriptor, validate input, evaluate a mapping, or
  cause any side effect.
- Mapping envelopes provide deterministic version and ontology provenance and either a valid target
  record or a canonical structured failure, leaving source retrieval and target calls to the
  conversational agent's separate system tools.
- Compiled descriptors flow through Prompt 44's established release-manifest interface, producing
  deterministic added, changed, deprecated, and removed named-tool deltas without changing the
  ontology fingerprint for manifest-only changes.

Run the repository's full check (currently `npm run check`, if still provided), focused compiler,
schema-validation, mapping evaluator, MCP discovery/registration, determinism, failure-envelope,
and no-I/O boundary tests, plus `git diff --check`. Inspect generated-artifact changes and explain
every legitimate change. Confirm `ontology/compiled/` has no unreviewed manual edit.
Commit locally with the focused message `Add named mapping tools`. Do not push.

## Governance

This is a decision-bearing product, architecture, governance, and operational implementation.
Before merging a target-repository pull request, apply `narrative-required` together with
substantive `## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences`
sections in the same action, before merge. Never hand-edit, hand-merge, or otherwise author
generated `Narrative.md`; use a reviewed fragment and the target repository's generation process.
The resulting Narrative-only pull request must not have `narrative-required`, or it would
recursively create another entry.
