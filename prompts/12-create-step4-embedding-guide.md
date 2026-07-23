# Prompt 12 — Create the Step 4 embedding-matcher guide

Using the previously supplied repository contract and the embedding-matcher recommendation in
`docs/nextsteps.md`, create `docs/step4.md` as a prompt-by-prompt teaching guide for a junior
developer working with a coding agent.

This stage writes the guide only. Do not implement embeddings.

## Guide contract

Title the document:

`# Step 4 — Embedding Matcher: a prompt-by-prompt guide`

Explain how to submit one prompt at a time, review each diff, run `npm run check`, and commit only a
passing increment. Each implementation prompt must contain:

- exact files or components to inspect;
- one bounded goal;
- non-negotiable constraints;
- concrete acceptance criteria;
- verification commands;
- an explicit scope boundary;
- a short “Why this prompt works” teaching note.

Repeat these invariants prominently:

- Embeddings are refreshed before compilation only.
- Compilation, CI, and the MCP runtime never call a model or the network.
- Compilation remains byte-for-byte deterministic.
- Embeddings may improve proposals but must not independently auto-accept a pair that fails the
  lexical auto-accept threshold.
- Model ID, version, dimension, and content hashes are exact cache identity.
- Caches from different identities are never merged or relabelled.
- Generated artifacts are compiler-owned.
- Secrets come from environment variables and are never committed.

## Required prompt sequence

Write these prompts in order:

0. **Orient, no code.** Inspect matcher, compiler, model, config, and architecture; explain current
   scoring and determinism.
1. **Pure embedding/cache primitives.** Deterministic entity text, content hash, cosine similarity,
   unit-score normalization, strict cache schema validation, six-decimal vectors, stable ordering,
   and atomic writes that preserve the prior cache on failure.
2. **Configuration, types, and evidence.** Add disabled-by-default configuration and explicit
   mapping evidence without changing existing compiled output.
3. **Refresh command, the only networked step.** Use an approved enterprise endpoint, environment
   credentials, bounded batches/timeouts/retries, exact response validation, identity isolation,
   and atomic cache replacement. Tests must use a local fake server.
4. **Matcher fusion, disabled by default.** Define lexical/embedding fusion and prove embeddings
   cannot promote a pair to `accepted-auto` unless its lexical score independently qualifies.
5. **Offline compiler integration and exact fingerprinting.** Load only a complete, matching cache;
   record evidence; include exact governed embedding inputs in deterministic fingerprints; never
   refresh implicitly.
6. **Adversarial determinism and boundary tests.** Prove offline compilation, stale/missing cache
   failure, metadata isolation, stable output, secret safety, and runtime network independence.
7. **Documentation and Narrative classification.** Update architecture, registration, security,
   usage, and governance docs without overstating model trust.

Finish with:

- an “After you finish” checklist;
- a requirements-to-evidence audit;
- the full verification commands;
- a short lesson explaining why staged prompts outperform one large feature request.

Update `docs/nextsteps.md` so Step 4 links to this guide.

## Acceptance criteria

- The guide can be followed without hidden context.
- Every network action is isolated to the explicit refresh command.
- Failure modes and negative tests are at least as detailed as the happy path.
- No production code or generated ontology artifact changes.
- Markdown links resolve and `git diff --check` passes.

Commit locally with a focused documentation message. Do not push.
