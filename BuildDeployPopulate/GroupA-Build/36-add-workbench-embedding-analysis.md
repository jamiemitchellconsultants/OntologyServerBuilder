# Prompt A-36 — Add engineer-workbench embedding analysis

Using the engineer intake workbench from Prompt A-24, the build-time embedding matcher
from Prompts A-25–A-30, and the bounded LLM analysis from Prompt A-31, give the engineer
workbench a working embedding pass in the separate `OntologyService` repository. Do not
change `OntologyServerBuilder` in this stage.

This stage closes known issue #89, recorded as a verified failure in the Prompt A-33
audit (`docs/qualified-user-intake-audit.md`, matrix row 5 and P0) and as known limit 3
of `docs/accounts-payable-mapping-example.md`: `analyzeSubmission` calls the matcher
without an embedding context, so `embeddingEvidence` is structurally unreachable, an LLM
request package can never carry it, and enabling `embedding.enabled` changes nothing
about an intake analysis.

The gap is not a missing argument. Intake analysis scores **ungoverned re-parsed
entities** against **compiled canonical entities**, and the committed governed cache
contains vectors only for the latter. Threading the existing cache through would leave
every newly submitted entity — precisely the entities lexical matching is worst at —
silently falling back to lexical scoring inside a report that claims an embedding pass.
Supplying the missing source-side vectors is the substance of this stage.

This stage depends on Prompts A-24, A-25–A-31, and A-33. Adapt names and paths to the
target repository as it exists. Do not assume Prompt A-34's registration skill or Prompt
A-35's server-connected supplier discovery has landed; where a later stage supplies the
verified artifact by a different route, use whichever verified-artifact path exists
rather than adding a second one.

## Explicit supersession of earlier prompts

Do not edit any earlier Builder prompt. This prompt is the sole authority for the
behavior it introduces, and supersedes only these earlier constraints, only for the new
command and the new report section:

- **Prompt A-27 / `docs/embedding-matcher.md`:** its "Deliberately unsupported" list forbids "any
  runtime, MCP, or CLI path that executes a model". That exclusion already tolerated the one
  deliberate refresh command it introduced. This stage adds a second, equally deliberate,
  equally maintainer-invoked refresh command, and the documentation must name both explicitly
  rather than leave the exclusion reading as absolute. Every other item on that list is unchanged:
  no implicit or automatic refresh during compile, check, test, startup, or request handling; no
  mixing of vectors across model identities; no manual cache repair; no automatic acceptance driven
  by semantic similarity.
- **Prompt A-30:** its evidence-visibility surfaces (`ontology_list_mapping_reviews`,
  `ontology://mapping-review`) project compiled evidence only. They remain unchanged, and must not
  learn about intake vectors, intake candidates, or the new report section.

Everything else from Prompts A-21–A-33 is preserved. In particular: intake material
remains evidence and never fact, the runtime and the MCP surface perform no network I/O,
and nothing here alters compilation, the source fingerprint, a compiled artifact, or a
deployed byte.

## Read before editing

Read the canonical repository agent instructions; `docs/intake-workbench.md`;
`docs/embedding-matcher.md`; `docs/qualified-user-intake-audit.md` (matrix row 5,
finding 3, and P0); known issue #89; the intake analysis module and its LLM packaging
module; the matcher and its fusion path; the compiler's cache-loading and coverage
checks; the embedding text, hash, cache, digest, and matching-context primitives; the
embedding refresh command and its HTTP client; the intake export document type; the
CLI's `intake-analyze` wiring; and the tests for all of them.

Two facts the compiled artifact makes non-obvious, and that the implementation must
respect:

- The compiled config deliberately **omits** the embedding block, so the intake path must read the
  model identity, weights, cache path, cache digest, and candidate cap from the compiled
  `embeddingProvenance`, never from `ontology.config`.
- The matcher treats an entity absent from a matching context as simply having no vector, and that
  is not an error there. The intake path must therefore establish complete coverage **before**
  calling the matcher, and must never rely on the matcher's per-source fallback.

## The two vector sources

Fusion at intake needs vectors on both sides of every comparison.

| Side | Source | Governance |
| --- | --- | --- |
| Canonical target entities | the committed governed cache named by the compiled `embeddingProvenance` | already a reviewed compiler input |
| Re-parsed source entities | a new, submission-scoped analysis vector set produced by this stage | evidence only; never a compiler input |

The analysis vector set is never written under `ontology/`, never committed as a
governed input, never enters the source fingerprint, never enters the intake store,
never becomes a submission event, and never appears in an export or an LLM request
package. It is an engineer's working file with the same standing as the deterministic
report itself.

## The new refresh command

Add one CLI mode, modelled exactly on the existing embedding refresh and reusing its
client, planner, validation, and atomic-write primitives rather than duplicating them:

```
node --import tsx src/cli.ts intake-embed \
  --export ./intake-export.json \
  --artifact ./supplier-catalog.json \
  --output ./submission-vectors.json \
  --endpoint https://your-embedding-provider.example/v1/embeddings \
  --credential-env EMBEDDING_API_KEY \
  --timeout-ms 30000 \
  --batch-size 32
```

Required behavior:

- **Verify before embedding.** Run the same provenance verification `intake-analyze` performs —
  normalized filename, compatible format and media type, byte size, SHA-256 — and stop before
  computing any text if it fails. Embedding an unverified artifact would produce vectors for a
  different artifact than the one under review.
- **Refuse when the deployment is not embedding-enabled.** If the loaded compiled ontology carries
  no `embeddingProvenance`, refuse with an actionable message. Canonical vectors would otherwise
  come from a cache the deployed artifact never consumed.
- **Stamp the exact model identity** from the compiled `embeddingProvenance`, and record it in the
  output. Vectors from two identities are never mixed, relabelled, or reused.
- **Reuse the governed cache where it already covers a re-parsed entity.** Content addressing makes
  this correct and free: a re-parsed entity whose canonical embedding text is byte-identical to a
  governed entity's has the same content hash and needs no request.
- **Treat provider responses as untrusted:** one vector per requested text, in order, finite,
  non-zero, of the configured dimension, or the command fails and writes nothing.
- **Assemble and validate the whole output before writing**, then install it atomically. A partial
  analysis vector set is never published.
- **Keep the credential environment-only.** Never commit, log, return, cache, or embed it in output,
  and redact failures so they can name the credential *variable* but never the endpoint or secret.
- **Never write into the governed cache path**, and never mutate the committed cache.

Nothing else may construct an embedding client. `intake-embed` and the existing refresh
command remain the only two.

## The analysis pass

Add an optional input to `intake-analyze`:

```
node --import tsx src/cli.ts intake-analyze \
  --export ./intake-export.json \
  --artifact ./supplier-catalog.json \
  --output ./intake-analysis.json \
  --embedding-vectors ./submission-vectors.json
```

When the flag is absent, behavior is exactly today's, and the report records
`embeddingAnalysisStatus: "not-requested"`.

When it is present, check all of the following **before** matching, and refuse the fused
run with a single actionable message naming what to refresh if any fails. Do not
degrade, warn, or fall back:

1. the compiled ontology carries `embeddingProvenance`;
2. the governed cache at the provenance's path is readable, valid, and identity-matched;
3. the analysis vector set is a known schema version and carries the identical model identity — id,
   version, and dimension — as both the governed cache and the compiled provenance;
4. every canonical entity matching will consider has a vector under its current content hash;
5. every re-parsed entity has a vector under its current content hash;
6. every consumed vector is finite, non-zero, and of the configured dimension.

Weights and the semantic-candidate cap come from the compiled `embeddingProvenance`.
This stage introduces no new tunable, no new threshold, and no intake-specific scoring
setting.

## Side by side, never reconciled

The deterministic pass stays authoritative for the report's structure, and the embedding
pass is strictly additive advice — the same shape Prompt A-31 established for the LLM
pass, for the same reason.

- Run the matcher **twice**: once with no embedding context, producing the deterministic candidates
  and their statuses exactly as today; once with the validated fused context.
- The deterministic report's candidates, statuses, ordering, and bytes are **unchanged** by the
  presence of the flag. Make this structural rather than asserted.
- Remove the unreachable `embeddingEvidence` field from the deterministic candidate type, and
  repoint the LLM packaging module's reads of it. Evidence now lives in a new top-level
  `embeddingAdvisory` section keyed by candidate stable id, sibling to `llmAdvisory`.
- For each candidate, `embeddingAdvisory` records the typed evidence that already exists — the
  independent lexical, embedding, and combined scores, the model identity, the contributing digests,
  and the source and target content hashes — plus the advisory status the fused score yields and an
  explicit agreement verdict against the deterministic status. Never emit a single merged answer.
- **The exact/likely split is decided on the lexical score, always.** A fused combined score at or
  above the exact-candidate floor must not produce `exact-candidate` when the pair's own lexical
  score is below it. An exact normalized name is a lexical fact and semantic similarity is not
  evidence of one.
- Record provenance for the section: model identity, the governed cache digest (comparable with the
  compiled `embeddingProvenance.cacheDigest`), a separate digest over the analysis vector set, and a
  merged consumed-context digest computed by the same rule the compiler uses — schema version, exact
  model identity, and each consumed entry's content hash, canonical text, and six-decimal rounded
  vector, in content-hash order.
- **No vector ever leaves the vector file.** Not into the report, the export, the LLM request
  package, a stored record, an event, or a log.
- `embeddingAnalysisStatus` is `not-requested`, `complete`, or `incomplete` with a stated safe
  reason. An unreadable, invalid, identity-mismatched, or incompletely covering vector set yields
  `incomplete` and the deterministic analysis stands intact, exactly as an absent LLM response does.

The LLM request package's candidate selection rule, its 200-candidate bound, its 2 MiB
cap, and its `selection` block are unchanged. Embedding evidence now populates the slots
that were dead, and an oversized package still refuses before writing rather than
truncating a field or choosing a different candidate set.

## Tests and boundaries

Add named offline tests, and mutation-test each invariant — break the rule deliberately
and confirm exactly one test fails. Almost every requirement here is a negative, which a
passing suite would otherwise assert vacuously.

- The deterministic report is byte-identical with and without `--embedding-vectors`, for a
  submission where fusion demonstrably changes candidate scores.
- Each of the six pre-matching checks fails closed, with an actionable message, and writes no output.
- A vector set built for one model identity is refused against a cache or provenance carrying
  another; identities are never mixed or relabelled.
- A vector set missing one re-parsed entity is refused rather than producing a report in which that
  entity silently scored lexically.
- A candidate whose combined score clears the exact-candidate floor on semantic strength alone is
  reported as `likely-candidate`, never `exact-candidate`.
- Fusion can raise a candidate the deterministic pass left `unmatched` into a fused
  `review-required`, and the agreement verdict names the disagreement rather than resolving it.
- An LLM request package carries embedding evidence for a fused candidate, contains no vector, and
  still obeys the selection rule and both bounds.
- `intake-embed` verifies provenance before computing any text, refuses when the compiled artifact
  carries no `embeddingProvenance`, preserves any prior output byte-for-byte on every failure path,
  and redacts endpoint and credential from transport failures.
- Sentinel tests: `analyzeSubmission`, the MCP module, the compiler, the check command, the server,
  and every ordinary CLI path neither import nor call the embedding HTTP client, reach no network,
  and read no credential. Use default imports rather than namespace imports in these tests — a
  namespace import cannot be patched.
- Analysis vector files are never written under `ontology/`, never enter the fingerprint, the intake
  store, an event, or an export.
- The committed `ontology/config.json` still has `embedding.enabled: false`, every committed
  compiled artifact is byte-identical, and the deployed MCP surface pin is unchanged — this stage
  adds no MCP tool, changes no tool signature, and touches no resource.

## Documentation

- `docs/intake-workbench.md`: replace the sentence describing "typed embedding evidence with
  compilation provenance when it already exists", which today reads as a conditional that is never
  satisfied. Document the two vector sources, the new command, the optional flag, the six
  fail-closed checks, the side-by-side reporting rule, and the lexical-only exact/likely split.
- `docs/embedding-matcher.md`: add an intake-scope section, and revise the "Deliberately unsupported"
  list so it names both deliberate refresh commands rather than reading as an absolute exclusion.
- `docs/accounts-payable-mapping-example.md`: replace known limit 3 with what now holds.
- Do **not** edit `docs/qualified-user-intake-audit.md`. An audit report is historical evidence; the
  closure belongs in this stage's Narrative entry and in the issue.
- Close issue #89 referencing the merged pull request.

## Acceptance criteria and verification

- An engineer can produce a submission-scoped vector set with one deliberate, explicitly configured
  command, and obtain a fused intake analysis; no other path in the service, CI, compiler, runtime,
  MCP server, or ordinary CLI invocation contacts a provider or costs anything.
- Enabling embeddings now demonstrably changes an intake analysis, and `embeddingEvidence` is
  reachable, populated, and carried into an LLM request package.
- The embedding pass cannot accept a mapping, cannot change an intake disposition, cannot alter the
  deterministic report, and cannot influence compilation, the fingerprint, a compiled artifact, a
  release manifest, or any runtime behavior.
- Every failure mode is fail-closed and stated; none degrades silently to lexical scoring.
- Compiled artifacts, the committed embedding configuration, and the deployed MCP surface pin are
  unchanged.

Run the repository's full check (currently `npm run check`, if still provided), the
focused intake workbench, LLM packaging, embedding matching, embedding compilation,
embedding refresh, and capability tests, plus `git diff --check`. Inspect
generated-artifact changes and explain every legitimate change; confirm
`ontology/compiled/` has no unreviewed manual edit and that the working tree contains
only changes belonging to this task. Commit locally with the focused message
`Add workbench embedding analysis`. Do not push.

## Governance

This is a decision-bearing architecture and governance implementation. Before merging a
target-repository pull request, apply `narrative-required` together with substantive
`## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences` sections
in the same action, before merge. The entry must record why intake vectors are evidence
rather than a governed compiler input, and why the two passes are reported side by side
rather than reconciled. Never hand-edit, hand-merge, or otherwise author generated
`Narrative.md`; use a reviewed fragment and the target repository's generation process.
The resulting Narrative-only pull request must not have `narrative-required`, or it
would recursively create another entry.