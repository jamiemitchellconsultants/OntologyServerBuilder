# Prompt A-30 — Complete embedding matcher delivery

Implement only the evidence visibility, documentation, and final acceptance work for the build-time
embedding matcher in the separate `OntologyService` repository. Prompts A-25–A-29 have already added
canonical embedding text and cache primitives, disabled-by-default configuration and evidence
contracts, the explicit refresh command, governed matcher fusion, and deterministic compiler cache
integration. This prompt deliberately combines Prompt 6, "Adversarial determinism and boundary
tests", and Prompt 7, "Documentation and the narrative entry", from the current `docs/step4.md`
guide into one final delivery stage.

Read Prompts A-25–A-29, the current guide, `AGENTS.md`, Project Narrative rules, the MCP server and
mapping-review resource/tool implementation, generated-artifact loading, `docs/owl-profile.md`,
`docs/architecture.md`, `docs/registering-systems.md`, operational documentation, and the relevant
compiler, matcher, runtime, MCP, refresh, and no-I/O tests before editing. Adapt names and paths to
the target repository as it exists. Do not assume a later prompt has added LLM analysis, intake
reports, release manifests, or a semantic-query surface.

This is a completion and assurance stage. It must not change the matcher algorithm, lexical or
embedding thresholds, cache format, compiler cache-validation rules, refresh implementation,
credential transport, ontology facts, OWL or SHACL reasoning, or the committed disabled setting.
It must not hand-edit `ontology/compiled/` or generated Project Narrative output.

## Existing mapping-review evidence only

Expose the compilation-level `embeddingProvenance` and a selected candidate's complete typed
`embeddingEvidence` through both existing mapping-review surfaces: the
`ontology_list_mapping_reviews` MCP tool and the `ontology://mapping-review` resource. Both must
faithfully project already compiled review data and return the same evidence fields for the same
records. Do not replace either surface or add another mapping-review identifier.

Define one deterministic, bounded review-queue projection and serializer shared by both surfaces;
do not implement parallel formatting paths. Its only population is compiled records whose
disposition is `review-required` or `unmatched`, sorted by stable ID. The resource takes no new
input and returns the first 100 records from that queue.

The tool accepts an optional `limit` integer from 1 through 100, defaulting to 100. With no
disposition filter and the default limit, it must return a byte-equivalent record array and
byte-equivalent `{ total, returned, truncated }` metadata to the resource. When the existing tool
disposition filter is supplied, filter within that review-queue population before stable-ID sorting
and limiting; do not invent a second population. `total` is the filtered-population size, `returned`
is the selected-record count, and `truncated` is true exactly when `returned` is less than `total`.
For unsupported or non-review dispositions, use the existing validation-error conventions when they
allow it; do not silently select a different population. Use the existing authorization and
generated-artifact loading conventions outside this shared projection.

- Do not add an embedding creation, refresh, query, search, similarity, or free-form semantic MCP
  tool, resource, endpoint, CLI command, or runtime model client.
- Do not load a cache, calculate a vector or score, read credentials/environment settings, resolve
  an endpoint, or reach a network service at runtime or during MCP registration or request handling.
- Return only bounded stored review evidence through the specified shared projection. Do not expose
  cache vectors, arbitrary cache entries, or an unbounded artifact dump.
- Keep the MCP tool/resource read-only and idempotent. The response must neither refresh nor repair
  a cache, mutate artifacts, mutate review state, write files, or create a model-side effect.
- Omit embedding provenance and candidate evidence in disabled and lexical-fallback cases rather
  than inventing empty values. Preserve manual-decision precedence and do not imply that evidence is
  OWL inference or an automatic acceptance decision.

Add in-memory MCP and runtime tests that prove both named surfaces expose identical enabled compiled
evidence through the shared bounded projection. Assert exact default tool/resource equivalence,
shared queue membership, filtered metadata, the tool's minimum, maximum, and invalid limits, the
100-record cap, stable-ID selection, exact `{ total, returned, truncated }` metadata, resource
selection of the same first 100 records, and identical evidence fields returned by both surfaces.
Also prove disabled output remains compatible without placeholders, and
repeated calls are equal and cause no writes, cache reads, environment/credential reads,
network/model-client construction, refresh dispatch, or model execution. Install sentinels after
fixture setup so any prohibited operation fails immediately.

## Operator, governance, and architecture documentation

Add `docs/embedding-matcher.md`, matching the tone and structure of `docs/owl-profile.md`. Update
the architecture and operational/registering documentation where the current service directs an
operator to matching or cache maintenance. The documentation must state all of the following
plainly:

- Embeddings are build-time compilation evidence, disabled by default. Normal compilation,
  `ontology:check`, CI, server startup, runtime requests, MCP registration, and MCP calls remain
  offline and never execute a model.
- `npm run ontology:embed` is the sole explicit, maintainer-operated network-capable refresh path.
  It refreshes before compilation; it is never invoked implicitly by compile, check, runtime, or
  MCP. Credential values are environment-only, are never committed, logged, returned, or placed in
  generated artifacts, and safe errors may name only the credential-variable name.
- The cache is a reviewed, committed, content-addressed compiler input. Describe normal refresh,
  the deliberate cache review and commit step, and that no one manually edits, relabels, or merges
  cache entries. State the atomic-refresh preservation of the previous cache on failure.
- `ontology/config.json` governs the exact model ID, version, dimension, cache path, weights, and
  semantic-candidate bound. An intentional model identity upgrade requires a deliberate
  `npm run ontology:embed -- --full`, review and commit of the replacement cache, then a separate
  enabled-configuration/recompile review; mixed or relabelled model caches fail closed.
- Explain cache digest, exact fingerprinting, deterministic replay, byte-identical equivalent
  compiles, and fail-closed recovery for stale, missing, invalid, or identity-mismatched cache data.
  Recovery directs the operator to the explicit refresh and review/commit procedure, never manual
  cache repair or a runtime fallback.
- State cost ownership: embedding-provider usage and model upgrades are deliberate maintainer
  decisions; no ordinary build, test, runtime, or MCP operation incurs model cost. Record the model
  identity and cache change in the review evidence appropriate to the repository's governance.
- Semantic scores improve ranking and can promote a candidate only to `review-required`. An
  embedding-only or semantically high candidate never receives automatic acceptance: the exact pair
  must independently clear the lexical auto-accept threshold, and a manual mapping remains
  authoritative.

Do not document an unsupported provider default, manual cache editing, a live semantic-search API,
or a promise that a model is available at runtime. Keep endpoint and credential examples
secret-safe.

## Final adversarial acceptance matrix

Add or consolidate focused, deterministic tests. All tests must use fixtures, fake injected clients,
or sentinels; none may require a live model, live endpoint, or real credential. The matrix below is
mandatory. Name tests so each scenario can be identified in command output.

1. **Disabled compatibility:** disabled matching, fingerprints, generated JSON, OWL, SHACL, and
   mapping-review behavior remain byte-identical to lexical behavior; no cache is required or read.
2. **Enabled ranking:** valid in-memory evidence reranks by configured fusion weights with stable
   ties; semantic promotion stops at `review-required` unless the exact pair independently clears
   lexical auto-accept.
3. **Offline boundaries:** compile, check, ordinary CLI, tests, server startup and requests, MCP
   registration and calls do not import, construct, or call the HTTP client or endpoint, and do not
   read credentials.
4. **Cache freshness:** missing, corrupt, unsupported, incomplete, stale-text/hash,
   wrong-dimension, non-finite, zero-vector, or identity-mismatched consumed cache data fails before
   matching with the explicit refresh instruction.
5. **Fingerprint change:** a changed used vector, text hash, model identity, dimension, weight, or
   maximum semantic-candidate setting changes the digest/fingerprint and makes `ontology:check`
   report stale artifacts; an unused valid entry does not.
6. **MCP evidence:** both surfaces share one serializer over the `review-required` and `unmatched`
   queue, sorted by stable ID. The resource returns its first 100 records; the default tool response
   is byte-equivalent. Tool filters apply within that queue before sorting and limiting and return
   filtered `{ total, returned, truncated }` metadata. Both omit disabled/fallback placeholders and
   remain read-only and idempotent.
7. **Deterministic artifacts:** two equivalent enabled compiles produce byte-identical JSON, OWL,
   SHACL, mapping-review, and fingerprint output; reordered cache entries do not change them.
8. **Manual precedence:** manual mappings override automatic rank, disposition, and evidence; no
   misleading automatic embedding evidence is attached to a manual decision.
9. **No embedding-only acceptance:** a high semantic or combined score without the exact pair's
   lexical auto-accept score is `review-required`, never `accepted-auto`.

Include provider-response failure coverage from the guide: missing, extra, non-finite, or
wrong-dimension in-memory response vectors and failed refresh each fail safely, preserve the old
cache, and never reveal endpoint or credential values in errors, logs, generated artifacts, or test
snapshots. Run `npm run check` twice and show the named tests and commands that establish the
matrix.

## Acceptance criteria and verification

- `ontology_list_mapping_reviews` and `ontology://mapping-review` expose only already compiled
  embedding evidence through one deterministic, bounded review-queue projection and serializer;
  there is no new model-executing or free-form semantic interface.
- Runtime and MCP delivery are read-only, idempotent, and offline; compilation remains the only
  consumer of a committed cache, and explicit refresh remains the only network-capable route.
- Documentation covers disabled default, refresh, cache review/commit, upgrades, costs, credentials,
  deterministic replay, fail-closed recovery, and review-only semantic confidence.
- Every final-matrix scenario has an offline deterministic test, including adversarial provider,
  secret-safety, cache-preservation, and no-I/O boundary checks.
- The normal committed configuration remains disabled. Do not hand-edit generated artifacts or
  generated Narrative output.

Run `npm run check` twice, the focused compiler, matcher, refresh, runtime, MCP, stale-artifact,
secret-safety, and no-I/O boundary tests, plus `git diff --check`. Inspect generated artifacts and
explain every legitimate change. Confirm no generated artifact changes merely from disabled mode.
Commit locally with the focused message `Complete embedding matcher delivery`. Do not push.

## Governance

This is a decision-bearing product, architecture, governance, and operational implementation.
Before merging its target-repository pull request, apply `narrative-required` together with
substantive `## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences`
sections in the same action, before merge. Never hand-edit, hand-merge, or otherwise author
generated
`Narrative.md`; use a reviewed fragment and the target repository's generation process. The
resulting Narrative-only pull request must not have `narrative-required`, or it would recursively
create another entry.
