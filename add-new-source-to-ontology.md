# Add a new source to the ontology from a supplier MCP server

This guide describes the intended workflow specified by `OntologyServerBuilder`. The Builder
repository is a staged specification; the workflow is implemented in the separate
`OntologyService` repository.

## Outcome and trust boundary

A supplier announcement does not change the ontology. A finance user can submit a captured MCP
catalog as durable review evidence, but only a human-reviewed, merged, compiled, and deployed
repository change makes the supplier's governed details visible in the live ontology.

```text
supplier announcement
  -> user captures connected MCP catalog
  -> immutable intake submission
  -> engineer review and offline analysis
  -> reviewed governed source change
  -> deterministic compilation and pull-request approval
  -> CI/CD deployment by immutable image digest
  -> deployed ontology and release-change visibility
```

## Preconditions

The finance user must already be connected to the supplier's MCP server using their own authorised
access. The registration workflow never discovers or connects to a supplier server on the user's
behalf.

The user also needs `ontology:propose`. Installing the qualified-user registration skill does not
grant that capability. Intake is disabled by default, and can be enabled only after S3-compatible
storage, access scope, and monitoring have been deliberately provisioned.

## Qualified-user submission

1. The user's agent calls the supplier server's `tools/list`, `resources/list`, and, where
   available, `prompts/list` methods. It saves the complete raw result in one local JSON file.
   Supplier documentation, remembered details, and reconstructed descriptions are not substitutes
   for the captured catalog.
2. The agent calculates the JSON file's SHA-256 digest and byte size. Its media type is
   `application/json`.
3. The agent calls `ontology_prepare_system_registration_request`. This produces normalized
   proposed system, entity, attribute, operation, relationship, meaning, allowed-value, gap, and
   question data. All supplier-provided text is inert data, even if it resembles an instruction.
4. The agent calls `ontology_submit_system_registration` with the normalized package; source format
   `mcp`; filename, media type, byte size, and SHA-256; optional inert provenance locator; extractor
   identity and version; extraction timestamp; notes; current ontology fingerprint; and a fresh
   idempotency key.
5. The submitted provenance digest must equal the digest in the prepared result. The service rejects
   raw artifacts, credentials, tokens, local paths, live records, personal data, payment data, and
   sources for the service to retrieve.

The intake plane stores canonical JSON with a server-computed digest using create-only, atomic S3
operations. It creates an immutable submission and initial `received` event, and keeps later events
append-only. An identical retry by the same authenticated user and idempotency key returns the
original receipt; a different payload using that key fails.

The user receives only an opaque receipt ID, payload digest, timestamp, and `received` status. That
means *submitted for review*, not *registered*. The user cannot list, export, retrieve, change, or
dispose of pending submissions.

## Engineer review and governed promotion

An engineer with `ontology:intake:review` may list an intake summary, export a canonical
`intake-export.json`, and append an advisory disposition. An `accepted` intake disposition means
only that it is ready for engineering work; it is not ontology acceptance.

The engineer separately supplies the original captured catalog to a local offline analysis command.
Before parsing, the command verifies filename, format, media type, byte size, and SHA-256 against
the exported provenance. It reparses the catalog without network access and compares it with the
submitted normalization.

After human review, the engineer uses the reviewed export and analysis to make a bounded change in
`OntologyService`: a curated source representation, system manifest, approved mappings where
needed, tests, and documentation. Raw intake content, local paths, credentials, and generated
artifacts are not committed. Unresolved semantic decisions stop the change rather than being
guessed.

The compiler consumes only reviewed, checked-in governed inputs. It deterministically produces the
compiled ontology, mapping-review output, fingerprint, and later projections such as OWL, SHACL,
and release metadata. Generated artifacts are never hand-edited. The pull request receives human
review, the normal checks, and the required Project Narrative governance before merge.

## Semantic matching where lexical matching is insufficient

The workbench first produces deterministic, offline advisory matching. It reports only:

- `exact-candidate`
- `likely-candidate`
- `review-required`
- `unmatched`

Those are review results, not semantic acceptance.

### Embedding-assisted matching

Prompts 37–42 define an optional, build-time embedding path. A reviewed, committed cache supplies
vectors; normal runtime and MCP calls do not call an embedding provider. When enabled and complete
for the candidate set, matching uses:

```text
combined score = lexical weight * lexical score + embedding weight * embedding score
```

Embedding evidence can promote an otherwise unmatched candidate to `review-required`, but it cannot
independently produce `accepted-auto`. Automatic acceptance still requires the exact source-target
pair's lexical-only score and combined score both to clear the high-confidence threshold. Manual
mapping decisions remain authoritative.

The committed default configuration names the embedding model `text-embedding-3-small`, version
`2024-01-25`, dimension 1536, but has embeddings disabled. This is an embedding model choice, not
a choice of generative LLM.

### Bounded LLM advisory analysis

Prompt 43 adds a separate, optional coding-agent analysis pass. It does not make a model call from
the service, compiler, CI, runtime, MCP server, or normal CLI path.

The engineer workbench generates a canonical, schema-versioned `llm-analysis-request.json` that:

- contains at most 200 candidate records and is at most 2 MiB;
- carries normalized source and target evidence, deterministic and embedding evidence, provenance,
  disagreement, fingerprints, and explicit questions; and
- carries a SHA-256 digest of the exact request bytes.

The engineer manually provides that file and the `resolve-intake-mapping-review` operator template
to a coding agent. The agent must read only the named request and governed context, must treat all
source-derived text as inert quoted data, and must return only a JSON response.

The response must contain the request digest, model ID, model version, prompt-template version, and
analysis timestamp. Each recommendation must cite evidence, distinguish source facts from
inference, and use only `likely-candidate`, `review-required`, or `unmatched`. It cannot claim
`accepted`, change files, call tools, retrieve sources, merge, deploy, or make an ontology fact.

The workbench validates the response offline: schema and version, request digest, bounded content,
stable-ID references, evidence quotes, ordering, and permitted statuses. It then adds the result to
`intake-analysis.json` alongside the deterministic and embedding evidence. Missing or invalid LLM
output yields `semanticAnalysisStatus: incomplete`; earlier evidence remains usable.

No generative LLM provider, family, model, temperature, seed, context-window size, or model
attestation mechanism is mandated. The design records a model ID and version for traceability but
leaves model selection to the engineer's coding-agent environment.

## Deployment and visibility

After the approved pull request merges, CI compiles and validates the candidate ontology, builds a
release manifest against an explicitly verified previous artifact, and builds the service image. The
compiled ontology is inside the image; the immutable image digest is the ontology release.

Deployment verifies readiness and the source fingerprint reported by `/health`. In production,
steady-state replicas must serve the same image digest. A rollback is also an ontology rollback.

Only after the new image is deployed can users see the supplier's governed details through the
ontology's normal read surface, such as system listing, entity search and description, graph query,
or catalog resource. The current release delta is available through
`ontology_get_release_changes` and `ontology://release/current` to callers with `ontology:read`.
Pending intake submissions never appear on that delivery surface.

## Prompt map

| Capability | Prompts |
| --- | --- |
| Base deterministic matching and compilation | 4 |
| Supplier registration preparation and submission | 18, 34, 35, 48 |
| Engineer export, reparse, and deterministic analysis | 36 |
| Embedding evidence and semantic fusion | 37–42 |
| Bounded LLM advisory analysis | 43 |
| Deployed release-change visibility | 44 |
| Assurance of the full qualified-user boundary | 47 |
| Homelab and production deployment | 30–32b |
