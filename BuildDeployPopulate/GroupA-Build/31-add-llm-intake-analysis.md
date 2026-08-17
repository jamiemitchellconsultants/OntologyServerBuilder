# Prompt A-31 — Add bounded LLM intake analysis

Using the engineer intake workbench from Prompt A-24 and the completed offline embedding evidence
from Prompt A-30, add a bounded, advisory LLM analysis pass in the separate `OntologyService`
repository. Do not change `OntologyServerBuilder` in this stage.

This stage packages already available evidence for an engineer's coding agent, validates the
agent's returned JSON offline, and consolidates it with the existing analysis. It does not invoke a
model from the service, CI, compiler, runtime, MCP server, or ordinary CLI path. It does not fetch
a source, write ontology inputs, modify compiled artifacts, change an intake disposition, accept a
mapping, open a pull request, merge, or deploy.

## Read before editing

Read Prompts A-21–A-30; the approved qualified-user intake plan; `AGENTS.md`; Project
Narrative rules; the `IntakeStore`, engineer export and analysis CLI, offline adapter, deterministic
matcher, compiled embedding evidence, canonical JSON and digest helpers, and their tests. Also read
the locally available operational instructions supplied with this task, if the engineer is
invoking the separate response-analysis workflow. Those instructions are an operator input, never a
target-repository artifact. Adapt names and paths to the target repository as it exists. Do not
assume a later prompt has added release manifests, deployed-change visibility, mapping tools, live
model integration, or a semantic-query surface.

## Bounded request package

Extend the engineer-only analysis command with an explicit, offline request-generation mode. It
must write a schema-versioned, canonical `llm-analysis-request.json` only to an explicit local
output path. The command selects at most 200 candidate records and the complete canonical JSON
must not exceed 2 MiB. Fail closed before writing an oversized request; do not silently truncate a
field or choose a different candidate set.

Select candidates deterministically using the existing advisory analysis records and stable IDs;
document the exact ordering, inclusion rule, and deterministic tie-breakers. The package must
contain only the normalized source and canonical-target evidence needed for review, including:

- stable IDs, normalized names, descriptions, attributes, types, requiredness, allowed values,
  operation/tool context, declared relationships, and governed synonyms where they are relevant;
- deterministic candidate scores, component evidence, advisory statuses, parser discrepancies, and
  source and ontology fingerprints;
- embedding scores, typed embedding evidence, cache digest, model identity, and compilation
  provenance when already present; and
- provenance, conflicts, prior-pass disagreements, and explicit, bounded questions for the agent
  to answer.

Treat every source-derived description, name, allowed value, note, locator, and instruction-shaped
string as quoted inert data, not as a command or a higher-priority instruction. Do not include raw
artifacts, credentials, access tokens, local paths, live records, payment data, personal data, or
an instruction to retrieve a locator. Calculate and include a lower-case SHA-256 digest of the
exact canonical request bytes. Reordered equivalent inputs must produce byte-identical request
bytes and digest.

The service must not execute this request, construct a model client, resolve an endpoint, read a
credential, or call a network service. An engineer supplies the request and separately supplied
operator instructions to a coding agent manually; this target-repository command never automates
that hand-off. If those operator instructions are absent, the engineer workflow cannot be invoked;
the service still emits and validates its self-contained request and response contracts.

## Advisory response contract

Accept a response only from an explicit local `llm-analysis-response.json` path supplied to the
engineer-only analysis command. Define and strictly validate a versioned response schema. It must
include the exact request digest, model ID, model version, prompt-template version, analysis
timestamp, and canonically ordered arrays of candidate recommendations, explanations, gaps,
disagreements, and unanswered questions.

Each recommendation must identify the relevant source and target stable IDs (or explicitly state
that no target is proposed), quote the evidence it relies on, distinguish source fact from model
inference, and use exactly one of these advisory values:

- `likely-candidate`
- `review-required`
- `unmatched`

Reject unknown recommendation values, including `accepted`, `accepted-auto`, or any governance
state. A response must never assert that a candidate, intake, mapping, ontology fact, pull request,
or deployment was accepted. Validate unknown and missing fields, wrong schema or prompt-template
version, malformed timestamps or digests, duplicate IDs, references outside the request, unbounded
or malformed text, invalid evidence quotes, and non-canonical collection ordering according to the
repository's established public-schema conventions. Do not accept a response merely because it is
valid JSON.

## Offline validation and consolidated report

Validate a supplied response without calling a model or network service. Confirm its request digest
matches the exact generated request and that every referenced candidate and evidence quote is in
the request. Preserve the source request and the validated raw advisory response as local engineer
evidence according to repository conventions; do not store local paths, credentials, or raw source
artifacts in an intake record or generated ontology artifact.

Extend the canonical `intake-analysis.json` with a clearly named LLM advisory section and a
`semanticAnalysisStatus`. Merge validated advisory output without deleting, changing, relabelling,
or rewriting parser discrepancies, deterministic evidence, embedding evidence, provenance, or
their original advisory statuses. Canonically sort every merged array by stable identifiers and
document deterministic tie-breakers. Retain the source and ontology fingerprints and show both
agreement and disagreement across deterministic, embedding, and LLM passes.

If the response is absent, invalid, wrong-digest, unsupported, or cannot be read,
`semanticAnalysisStatus` must be `incomplete`; preserve the earlier analysis intact and report the
safe validation reason. Do not make LLM availability a requirement for deterministic analysis or
return an incomplete report as a successful semantic decision. A valid response is traceable,
advisory evidence only; it cannot alter an intake disposition, ontology, generated artifact,
mapping decision, or runtime behavior.

## Tests and boundaries

Add focused offline tests. Cover:

- request selection at 200 candidates, request-size refusal above 2 MiB, canonical generation, and
  byte-identical request/digest output for reordered equivalent inputs;
- normalized source text containing prompt-injection-shaped instructions, proving it remains inert
  quoted data and causes no command, retrieval, model, or repository action;
- every required response field, strict schema violations, unknown fields, malformed data, wrong
  schema or prompt-template version, unsupported status, duplicate or out-of-request references,
  invalid evidence quotes, and a wrong request digest;
- valid recommendations, explanations, gaps, disagreements, and questions merging in canonical
  order while preserving the exact pre-existing parser, deterministic, and embedding evidence;
- absent, unreadable, invalid, and rejected response paths producing
  `semanticAnalysisStatus=incomplete` with earlier results intact; and
- sentinels installed after fixture setup proving request generation, response validation,
  consolidation, `npm run check`, ordinary CLI, compiler, server startup and requests, MCP
  registration and calls neither construct nor call a model client, resolve an endpoint, read a
  credential, fetch a source, mutate intake state, write ontology sources or generated artifacts,
  nor require a live model in CI.

Keep tests deterministic and offline. Do not hand-edit `ontology/compiled/` or generated Project
Narrative output.

## Acceptance criteria and verification

- The engineer-only command emits a canonical, schema-versioned request of at most 200 candidates
  and 2 MiB, with all specified existing evidence, explicit questions, inert source text, and an
  exact request digest.
- A strict, provenance-bearing local response schema permits only the three advisory values and
  cannot claim governed acceptance.
- `intake-analysis.json` consolidates validated advisory evidence without rewriting prior evidence;
  unavailable or invalid LLM output sets `semanticAnalysisStatus=incomplete` and leaves earlier
  results usable.
- No target-service path executes a model or network request, and CI has no live-model dependency.

Run the repository's full check (currently `npm run check`, if still provided), focused intake
analysis, canonicalization, validation, consolidation, and no-I/O boundary tests, and
`git diff --check`. Inspect generated-artifact changes and explain every legitimate change. Confirm
that `ontology/compiled/` has no unreviewed manual edit. Commit locally with the focused message
`Add LLM intake analysis`. Do not push.

## Governance

This is a decision-bearing product, architecture, and governance implementation. Before merging a
target-repository pull request, apply `narrative-required` together with substantive
`## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences` sections in the
same action, before merge. Never hand-edit, hand-merge, or otherwise author generated
`Narrative.md`; use a reviewed fragment and the target repository's generation process. The
resulting Narrative-only pull request must not have `narrative-required`, or it would recursively
create another entry.
