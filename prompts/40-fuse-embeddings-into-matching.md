# Prompt 40 — Fuse embeddings into governed matching

Implement only Prompt 4, "Fuse embeddings into the matcher (disabled by default)", from the
current `docs/step4.md` in the separate `OntologyService` repository. Prompts 37–39 have supplied
the offline embedding text, cache, configuration, evidence, and explicit refresh contracts. Read
those prompts, the current guide, `AGENTS.md`, Project Narrative rules, `src/matcher.ts`, shared
normalization helpers, `src/model.ts`, configuration validation, manual-mapping handling, and
relevant matcher and compiler tests before editing. Adapt names and paths to the repository as it
exists; do not assume a later prompt has added compiler cache loading or embedding integration.

This stage lets matching consume a validated, in-memory embedding context. It does not load a
cache, read the filesystem, read environment variables, call a model or network endpoint, change
the refresh command, or integrate embeddings into compilation, runtime, CLI, or MCP paths. Keep
the committed configuration disabled; disabled matching and generated ontology output must remain
byte-identical.

## Injected, validated matching context

Extend `buildMappings`, its similarity helper, or an equivalent focused matcher boundary to accept
an optional embedding context. The caller supplies the already validated model identity, cache
digest, source and canonical content hashes, and vectors. The matcher must neither reconstruct nor
guess this data, and it must validate any context crossing its own public boundary according to the
typed contracts from Prompt 38.

When embeddings are enabled and both the source and candidate vectors are present, calculate:

```text
embeddingScore = normalizeToUnitScore(cosineSimilarity(sourceVector, candidateVector))
combinedScore = lexicalWeight * lexicalScore + embeddingWeight * embeddingScore
```

Use the validated configured weights exactly. Rank eligible candidates by `combinedScore`, then by
stable canonical ID in ascending order for an exact tie. Retain the current lexical ranking,
selection, reasons, and output byte-for-byte when embeddings are disabled or either vector is
missing. A missing vector is not an error and must never receive an invented zero score, partial
fusion, or placeholder evidence.

For each selected pair scored with both vectors, preserve complete structured embedding evidence:
the independent lexical score, embedding score, combined score, exact model ID/version/dimension,
cache digest, and source and target content hashes. Use the typed `embeddingEvidence` contract from
Prompt 38 rather than a reason string. Evidence must agree exactly with the injected context and be
absent whenever fusion did not occur.

## Governed dispositions and overrides

Manual mappings retain their existing precedence over every automated score, ranking, and
disposition. Do not attach misleading automatic embedding evidence to a manual decision.

For an automatically selected, fused pair, use its independent lexical-only score and combined
score to apply these rules exactly:

1. `accepted-auto` requires the lexical-only score **and** combined score each to clear the
   configured high-confidence threshold.
2. If the combined score clears the review threshold but the lexical-only score does not clear the
   high-confidence threshold, classify it as `review-required`.
3. If the combined score does not clear the review threshold, classify it as `unmatched`.

Thus an embedding may promote an unmatched lexical candidate to `review-required`, but can never
make a pair `accepted-auto` unless that exact source-target pair independently clears the lexical
high-confidence threshold. Do not use a semantic score from one pair to qualify another pair.
Preserve the existing lexical-only disposition behavior whenever fusion is disabled or unavailable.
A human decision continues to override automated scoring under the existing governed process.

## Tests and verification

Add focused offline matcher tests using injected in-memory contexts; no test may load a cache or
reach a network endpoint. Cover:

- exact weighted fusion arithmetic;
- embedding-driven reranking, including stable canonical-ID ordering for tied combined scores;
- lexical fallback for disabled embeddings and for a source or target with no vector;
- an embedding promotion from unmatched to `review-required`;
- refusal to auto-accept a semantically high-scoring pair whose independent lexical score is below
  the high-confidence threshold;
- auto-accept only when both independent lexical and combined scores clear that threshold;
- manual overrides retaining precedence; and
- complete, exact typed evidence for fused pairs, with no evidence for disabled, missing-vector, or
  manual paths.

Install filesystem-cache, environment, network/model-client, refresh-command, compiler-write,
runtime, and MCP sentinels after fixture setup. Prove matcher entry points exercise none of them.
Include a regression proving disabled mode produces byte-identical mapping results, reasons,
dispositions, and generated ontology artifacts. Keep `embedding.enabled` false in committed
configuration; do not hand-edit `ontology/compiled/` or generated Project Narrative output.

Run the repository's full check (currently `npm run check`, if still provided), focused matcher
regression and boundary tests, and `git diff --check`. Inspect generated artifacts and confirm
`ontology/compiled/` has no diff. Commit locally with a focused message. Do not push.

## Governance

This is a decision-bearing architecture and product implementation. Before merging a target
repository pull request, apply `narrative-required` together with substantive `## Narrative
Context`, `## Narrative Decision`, and `## Narrative Consequences` sections in the same action.
Never hand-edit, hand-merge, or otherwise author generated `Narrative.md`; use a reviewed fragment
and the target repository's generation process. The resulting Narrative-only pull request must not
have `narrative-required`, or it would recursively create another entry.
