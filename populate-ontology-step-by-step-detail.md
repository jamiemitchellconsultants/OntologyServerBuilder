# Populate the ontology from a supplier MCP server — step by step

This guide expands [`add-new-source-to-ontology.md`](add-new-source-to-ontology.md) into the individual steps
implemented in the separate `OntologyService` repository, including which MCP tool, CLI command, or
file each step uses, what crosses each trust boundary, and what gate stops it. It also documents the
three-tier semantic-matching step that advises a human when no deterministic match exists.

This is a description of the implemented workflow as it stands in `OntologyService` at the time of
writing. File paths and line references are illustrative evidence, not a contract of this repository.

## Stage A — Outside the system (steps 1–3)

**1. The supplier announces the server.** An email, a vendor portal note, a message: "our Rebate
Management MCP server is live." Nothing in `OntologyService` learns anything from this on its own.
There is no registry poll, no discovery endpoint, no health probe of the supplier — its canonical
agent instructions state that untrusted source URLs and live MCP endpoints are not fetched during
compilation or runtime. The announcement enters the system only because a human carries it in.

**2. The finance user asks their agent what the ontology accepts.** The agent calls
`ontology_supported_sources` (capability `ontology:read`). The answer is a fixed list:
`openapi`, `json-schema`, `xsd`, `wsdl`, `csv`, `xlsx`, `mcp-catalog`, `normalized`. `mcp-catalog` is
the relevant one here.

**3. The agent produces a catalog snapshot — client-side.** The user's agent (or another approved
client-side process) connects to the supplier's MCP server *in the user's own environment* and
exports a JSON snapshot containing any combination of:

- `resources` — `name`, `uri`, `description`, optional `schema.properties`;
- `tools` — `name`, `description`, `inputSchema`, optional `outputSchema`;
- `entities` — explicit normalized entities, preferable when the supplier publishes a domain
  catalog.

Prohibited in the snapshot: credentials, tokens, live responses, personal data, bank coordinates,
transaction records. The agent computes the file's SHA-256 digest and byte size here; both values
are carried forward and verified again later.

## Stage B — Proposal plane, read-only (steps 4–8)

**4. The agent calls `ontology_prepare_system_registration_request`.** Capability `ontology:propose`.
What crosses the wire is normalized metadata only, never the file itself:

| Block | Contents |
|---|---|
| `objective` | Why the system should be registered |
| `system` | `suggestedId`, `name`, `description`, `integrationType`, `sourceFormat`, `sourceFilename`, `sourceVersion`, `sourceSha256` |
| `entities[]` | `name`, `meaning`, `attributes[]` (`name`, `type`, `required`, `meaning`, `allowedValues[]` each with a `meaning`) |
| `proposedRelationships[]` | `from`, `to`, `predicate`, `description`, `cardinality`, `evidence` |
| `businessRules[]`, `unresolvedQuestions[]` | Evidence-bearing rules; questions left open |

**5. The tool runs structural validation and may reject outright.** Rejection is for unsafe
structure only: a system ID already registered; a filename containing path components, control
characters, or bidirectional formatting controls that disguise its displayed order; an extension
conflicting with the declared format; a malformed SHA-256; missing systems, entities, attributes,
names, or datatypes; entity or attribute names colliding on their stable ID; duplicate or
meaning-less allowed values; relationship endpoints that are neither proposed nor registered;
relationships not involving the proposed system; duplicate relationship IDs; empty descriptions,
predicates, rules, or objective. Filenames are trimmed and normalized to Unicode NFC before the
extension check and before being placed into a suggested repository path.

**6. The tool preserves everything merely unfamiliar.** Supplier-specific datatypes, non-English
names, unknown status values, missing meanings, and even XML, formulas, code fragments, or
instruction-like wording inside descriptions are all retained as inert quoted data. Unfamiliarity is
never a reason to discard — that is the deliberate split between unsafe *structure*, which is
refused, and unfamiliar *business content*, which is escalated for review.

**7. The tool reports gaps without inventing answers.** Entities or attributes with no business
meaning; non-standard datatypes needing owner review; entities with no obvious required identifier;
status/state fields with no allowed values; relationships and rules with no evidence; plus
deterministically generated unresolved questions. An MCP catalog's `inputSchema` rarely carries field
meanings, so this step usually surfaces real gaps.

**8. The tool returns a fingerprint-bound proposal.** An `ontology-system-registration-request`
document: deterministic request ID, recommended filename, `basedOnSourceFingerprint`, normalized
definitions with stable proposed IDs `<system-id>.<entity-slug>`, gaps, questions, suggested source
and manifest paths, a deterministic manifest body, owner application instructions, and
`proposedStatus: "review-required"`.

No semantic matching happens anywhere in Stage B. The runtime does not compare ungoverned proposal
entities against canonical concepts. The server also does not write the file — the user's agent
saves it.

## Stage C — Intake plane, the only write boundary (steps 9–13)

Skippable: where intake is disabled — the default — the user carries the JSON to GitHub by hand and
Stage C is skipped entirely; go directly to Stage E.

**9. Preconditions checked at startup, not at request time.** `INTAKE_MODE` must be `s3`, with
`INTAKE_S3_BUCKET` set, and capability authorization is checked before the server accepts a
connection. There is no in-memory fallback: a submission accepted and then lost is worse than one
refused. In disabled mode the storage SDK is never even imported.

**10. The agent calls `ontology_submit_system_registration`.** Capability `ontology:propose`, plus
an attributable subject — refused when authentication mode is `static` or `none`, where a shared
token authenticates without identifying anybody. Input carries: the exact `registrationRequest` from
step 8; `provenance` (`sourceFormat` restricted here to `openapi`, `mcp-catalog`, or `wsdl`,
`mediaType`, `sourceByteSize`, `sourceSha256`, `normalizedFilename`, an optional `sourceLocator`
explicitly marked inert — the runtime never opens, resolves, or fetches it); `extraction`
(`extractorId`, `extractorVersion`, `extractedAt`, optional `notes`); and an `idempotencyKey`.

**11. Bounds are enforced at the store boundary.** Payload capped at 2 MiB; at most 500 entities,
1000 attributes per entity, 1000 relationships, 1000 allowed values per attribute; free text capped
at 16,000 scalars; filenames at 255 scalars.

**12. One request writes the payload, its digest, and the `received` event durably.** The submission
is written exactly once and is immutable. There is no update and no delete operation at any layer.

**13. The caller gets an opaque receipt.** `{ id, payloadDigest, receivedAt, status: "received" }`
and nothing else — no payload echo, no queue position, no ETA. The compiled ontology has not moved a
single byte: the compiler never reads this store, and no submission enters the fingerprint.

## Stage D — Engineer review (steps 14–19)

Requires an intake-review capability, granted only through an explicit claim mapping and unreachable
under `static`/`none` authentication. No qualified user holds it, including for their own submission.

**14. Find it.** A list tool returns cursor-paginated summaries — page size default 50, hard cap 200,
an out-of-range request refused rather than silently clamped so a caller cannot mistake 200 rows for
the whole queue. Returns opaque id, kind, subject, timestamp, payload digest, latest status, event
count. No payloads.

**15. Export it.** An export tool or an equivalent CLI command reaching the same enforcement point.
The engineer identity is supplied through configuration, never inferred. The export carries the
immutable payload, the receipt, the complete append-only event history, and both digests. Exporting
itself appends an `exported` event, so it is not marked read-only — a tool that writes must not carry
a hint that hosts use to decide what may run unattended.

**16. Verify the artifact before parsing it.** The catalog file reaches the engineer out of band; only
the CLI ever learns its local path, and that path is never written into any stored record, event,
export, or report. Checked against the immutable exported provenance, in order, before anything is
parsed: normalized filename → declared format and media type as a compatible combination → byte size
→ SHA-256. Any disagreement stops the command. An artifact that fails verification is a different
artifact, and a report over it would look like review of the submission while being review of
something else.

**17. Re-parse offline through the compiler's own adapters.** The same parser the compiler uses, so
the engineer's view cannot drift from the compiler's. For an MCP catalog:

| Catalog member | Produces |
|---|---|
| each `resources[]` | one entity from `schema.properties` |
| each `tools[]` | one `operation` entity from `inputSchema` |
| each `tools[].outputSchema` | one additional entity named `<tool> result` |
| each `entities[]` | one entity with its explicit attributes |

No URL is fetched, no external reference is resolved, no second attachment is opened, no model is
called. A document naming an external schema has that reference recorded by name rather than
followed.

**18. Compare claim against evidence.** Omissions, additions, type differences, requiredness
differences, provenance conflicts. Direction is fixed: `*-omitted` means the artifact declares
something the submission does not; `*-added` means the submission declares something the artifact
does not. An omitted record is reported once, at the record level, not once per field.

**19. Record an outcome.** `accepted`, `rejected`, or `superseded`, with a required bounded reason.
The actor comes from the authorised principal and the timestamp from the server; neither is
caller-supplied. `accepted` means accepted for governed engineering work — not semantic acceptance,
registration, merge approval, compilation, release, or deployment.

## Stage E — Governed source and compilation (steps 20–28)

**20. Owner-side preconditions.** Compare `basedOnSourceFingerprint` with the current compiled
ontology and rebase a stale proposal. Confirm the catalog is schema only. Review meanings with the
relevant data owners. Resolve material questions or preserve them as explicit review blockers.

**21. Add the source snapshot** under the ontology's sources directory, byte-identical to what was
reviewed.

**22. Add the manifest**, naming the format as `mcp-catalog` and pointing at the source path and
version. This is the first moment anything about the supplier's server becomes a candidate fact.

**23. Compile.** The compilation pipeline runs in a fixed order, each stage a hard gate: load
configuration → parse every manifest into entities → validate governed relationships → validate
mapping instructions → compile mapping-tool descriptors → load and validate manual mapping decisions
(a duplicate decision, a non-existent source, an `accept` with no target, an `unmatched` with a
target, or a non-existent canonical target each fail compilation) → standards → external alignments
→ lifecycle → business paths → replacement paths → apply lifecycle to entities → load the embedding
context.

**24. The fingerprint is computed — before matching runs.** It hashes governed *inputs*, never
rendered artifacts, and it is computed before semantic matching runs: matching results are derived
output, never fingerprint input.

**25. Matching runs.** Detailed in the semantic-matching section below.

**26. Only accepted mappings become traversable edges.** A "maps to" relationship is emitted only for
`accepted-auto` and `accepted-human` dispositions with a canonical target. `review-required` and
`unmatched` produce no edge — the new system's entities are queryable, but deliberately unjoined to
the canonical graph.

**27. Four artifacts are written, canonically:** the compiled ontology, the mapping review queue, the
OWL projection, the SHACL shapes. None is ever hand-edited; a check command recompiles and compares
byte-for-byte, failing loudly when stale.

**28. Work the review queue.** Every `review-required` and `unmatched` item is closed by a decision
in the manual mappings file — `accept` (with a canonical target), `reject`, or `unmatched`
(confirmed, no target) — each carrying a rationale that cites the organisation's approval evidence.
Governed relationships, mapping instructions, mapping-tool definitions, and OWL axioms are added
where required. Recompile, then run the full local check.

## Stage F — Merge, release, deploy (steps 29–35)

**29. Open the pull request.** Registering a new business system is a decision-bearing change:
apply the narrative label and the three required narrative body sections — Context, Decision,
Consequences — in the same action. Labelling first and filling in the sections later is a permanent
failure if the pull request merges in that window, because the narrative automation reads the body
from the merge event payload.

**30. CI gates.** Build, full test suite, artifact-freshness check, dependency audit, and the
narrative-sections check. Do not merge past a red result.

**31. Merge.** Post-merge automation proposes a separate draft pull request containing a new
narrative fragment and a regenerated narrative document. Human review of that draft accepts the
wording into the project's record.

**32. Cut the release metadata.** Reviewed release metadata — release ID, deployment timestamp, and
the SHA-256 and fingerprint of the previous release's exact compiled bytes — is reviewed and merged
like any other governed change. There is no implicit empty prior release; a first deployment
declares a real, reviewed baseline artifact and a rationale.

**33. Generate the release manifest.** Both declared previous values are verified before a single
record is classified; a mismatch fails generation rather than degrading it. Changes are classified
across seven governed classes: system, entity, attribute, relationship, semantic mapping, mapping
definition, named mapping tool. The new supplier server appears as one system addition, one entity
addition per parsed resource, tool result, or explicit entity, attribute additions, and a
semantic-mapping addition for each accepted mapping decision.

**34. Deploy.** The release manifest ships with the digest-pinned image as delivery metadata: never
placed under the compiled-artifact directory, never an ontology source, never in the fingerprint. The
runtime recomputes the manifest's own digest on load — a manifest is evidence to check, not a file to
trust.

**35. The finance user finally sees it.** The health endpoint publishes the new fingerprint. A
release-changes tool and resource say what changed. Then the read tools serve the compiled facts:
listing systems, searching entities, describing an entity, explaining a relationship, listing the
mapping review queue, retrieving mapping instructions, running a bounded SPARQL query, and any named
mapping tool compiled from an approved definition.

**The gate, stated once:** a receipt, an engineer export, an analysis report, an `accepted`
disposition, and an unmerged pull request are each *not deployment*. A release-delta test proves it
directly — comparing the pre-stage artifact with itself reports no change at all, whatever evidence
was submitted in the meantime.

---

## Yes — there is a semantic-matching step, in three tiers

Every tier answers the same question — "what canonical concept is this probably?" — and none of
them can accept a mapping on its own. They exist to route the non-deterministic cases to a named
human.

### Tier 1 — deterministic lexical, always on, in the compiler

Runs at step 25, for every non-canonical entity of kind `entity` against every live canonical
concept (`systemId === "canonical"`, lifecycle not retired). Operations derived from an MCP tool's
`inputSchema` are never scored — only resources, tool-result entities, and explicit entities enter
matching.

Short-circuits first: a retired source keeps any human decision as provenance and is scored
`unmatched` without re-scoring. A source carrying a manual decision returns that decision
immediately, confidence 1 or 0, and is never annotated with automatic evidence it did not rest on —
a human decision outranks every automated score.

For everything else, tokens are lower-cased and expanded through a governed synonym table (for
example `supplier→vendor`, `bill→invoice`, `po→purchase order`, `remittance→payment`), then filtered
against a stop-word list. Three component scores combine into one:

| Component | Method | Weight |
|---|---|---|
| Name | 1.0 on an exact normalized slug match, else the larger of token-Jaccard or 0.9 × bigram-Dice | 0.65 |
| Description | token-Jaccard over descriptions | 0.20 |
| Attributes | token-Jaccard over attribute-name tokens | 0.15 |

Candidates rank by descending combined score, ties broken on ascending canonical ID — deterministic,
never locale-dependent. Disposition then follows two configured thresholds (a high-confidence
threshold and a lower review threshold):

| Condition | Disposition | Becomes an edge? |
|---|---|---|
| lexical score and combined score both clear the high-confidence threshold | `accepted-auto` | yes |
| combined score clears the review threshold | `review-required` | no |
| below the review threshold | `unmatched` | no |

Every candidate carries a `reasons` array quoting each component score verbatim. Non-accepted items
land in the mapping review queue, projected at runtime through a bounded, filterable read tool and a
matching read-only resource — this is the "advise where there is no deterministic match" surface.

### Tier 2 — embedding fusion, build time only, disabled by default

This tier exists precisely for candidates lexical scoring cannot separate. A maintainer runs one
deliberate, explicitly configured, network-capable refresh command — the only such command in the
service — naming an endpoint and a credential environment variable with no built-in provider
default. Entity text is canonical and deterministic, and each cache entry is keyed by the SHA-256 of
that exact text, so a vector is reused only when the model identity matches configuration and the
entity's current content hash is already present. The whole next cache is assembled and validated
before an atomic install; any failure leaves the previously committed cache byte-for-byte intact, and
there is no partial cache and no manual repair.

The reviewed, committed cache becomes a governed compilation input. Fusion combines the lexical and
embedding scores by configured weights, and coverage is all-or-nothing: a cache missing one canonical
vector disables fusion for the entire compilation run rather than letting sources rank
inconsistently, and compilation fails closed — never degrading silently to lexical scoring — when
coverage is incomplete.

The governance ceiling is the point of the whole tier: `accepted-auto` still requires the selected
pair's *own lexical score*, independently, to clear the high-confidence threshold, in addition to the
combined score. A pair that is only semantically strong is `review-required`, always. A semantic
score from one pair never qualifies a different pair, and a human decision is never overridden.
Enabling embeddings therefore only ever moves work toward human review — on a fresh cache it
typically grows the review queue rather than shrinking it. Evidence is recorded even on `unmatched`,
naming the nearest concept the embeddings actually saw.

### Tier 3 — LLM advisory, the engineer's own agent, never the service

The service never calls a model. An engineer's offline analysis command can package a bounded request
— at most 200 candidates, preferring the most uncertain dispositions first, capped at 2 MiB,
digest-sealed — built entirely from evidence that already exists: re-parsed source evidence, the
submitted claim, declared relationships, governed synonyms, deterministic scores and their component
reasons, the canonical target's evidence, and typed embedding evidence where it already exists.
Locators, extraction notes, raw bytes, credentials, and local paths are excluded outright.

The engineer hands that package to their own coding agent — nothing in the service, CI, compiler,
runtime, or ordinary CLI path constructs a model client, resolves an endpoint, or reads a credential.
The response validates only against the exact request it answers: matching digest, known schema and
prompt-template versions, model identity, canonically ordered recommendations whose every id resolves
inside the request and whose every quoted "source fact" actually occurs there. Only three advisory
values are permitted — `likely-candidate`, `review-required`, `unmatched` — and any response
asserting a governed state is rejected outright. The consolidated report shows both passes side by
side with an explicit agreement verdict, never reconciled into a single answer. A missing, unreadable,
or mismatched response leaves the deterministic analysis intact.

### The intake-side vocabulary is deliberately different

At the engineer workbench, matching results carry four advisory statuses, distinct from the
compiler's `accepted-auto` / `accepted-human` vocabulary so an intake candidate can never be misread
as an accepted mapping:

| Status | Meaning |
|---|---|
| `exact-candidate` | a high-confidence match at or above the matcher's exact-name floor — not approval |
| `likely-candidate` | a high-confidence match below that floor |
| `review-required` | a plausible match a human must judge |
| `unmatched` | nothing above the review threshold |

### One known gap

At the time of writing, the engineer workbench has no working embedding path: intake analysis never
reaches the embedding matcher, so typed embedding evidence is structurally unreachable there, and
enabling embeddings changes nothing about an intake report. Semantic fusion applies only at compile
time, after a source is already governed — so at intake time, the available semantic tiers are Tier 1
plus the optional Tier 3. [`BuildDeployPopulate/GroupA-Build/36-add-workbench-embedding-analysis.md`](BuildDeployPopulate/GroupA-Build/36-add-workbench-embedding-analysis.md)
proposes closing this gap.
