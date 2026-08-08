# Prompt 37 — Add embedding text and cache primitives

Implement only Prompt 1, "Shared embedding text and cache primitives", from the current
`docs/step4.md` in the separate `OntologyService` repository. Read that guide when this prompt
runs, together with `AGENTS.md`, its Project Narrative rules, the current lexical matcher, its
normalization helper or call sites, relevant entity types, and matcher tests. Adapt names and paths
to the repository as it exists; do not assume the guide's examples are its current implementation.

This stage creates the pure, offline primitives that later embedding stages consume. It adds no
configuration, refresh command, HTTP client, credential read, scoring change, compiler integration,
runtime integration, MCP tool, or CLI command.

## Shared text and similarity contract

- Reuse the matcher's shared lexical normalization. If it is private, extract and export it without
  duplicating tokenization or normalization logic; both the existing matcher and the new embedding
  text function must use the same behavior.
- Add one exported canonical entity-to-embedding-text function. Its normalized entity name,
  description, and attribute names must have an explicit deterministic layout and attribute names
  sorted by stable ID or normalized name before serialization. Equivalent entity input ordering must
  yield byte-identical text.
- Hash the exact UTF-8 bytes of that canonical text with SHA-256. The content hash must change when
  the canonical text changes.
- Add a pure cosine-similarity primitive for finite, fixed-dimension numeric vectors. It must reject
  dimension mismatch and non-finite components. Define and test zero-vector handling explicitly;
  zero vectors must be rejected anywhere a cache entry is accepted. Include the guide's unit-score
  normalization helper as an exported `normalizeToUnitScore(cosine)` function mapping cosine
  `[-1, 1]` to `[0, 1]`.

## Cache contract

Implement typed read, validation, identity, and write primitives for the current Prompt 1 cache
shape. Preserve its schema version and exact model identity fields (ID, version, and dimension).

- Validate the whole cache before use: supported schema version; non-empty model ID/version;
  positive fixed integer dimension; finite, correctly dimensioned, non-zero vectors; and entries
  whose key equals the SHA-256 hash of their stored canonical text.
- Reject duplicate entity IDs and duplicate content hashes before caching. Do not silently retain
  one of two conflicting inputs.
- Treat model ID, version, and dimension as exact cache identity. Provide an explicit identity
  check and never merge, relabel, or otherwise reuse entries across a mismatch.
- Serialize deterministically: sort entity IDs and cache entry keys; round every vector component to
  six decimal places before serialization; and use stable numeric/JSON serialization so equivalent
  valid cache data produces byte-identical output.
- Write a cache atomically through a sibling temporary file: write it, call `fsync`, close it, then
  rename it over the destination, in that order. This is the crash-safety boundary: never rename an
  unflushed or open temporary file. On any write, `fsync`, close, or rename failure, remove only the
  temporary file and leave the previous destination cache intact.

## Boundaries

- These primitives must remain offline. Do not import a network client or read environment tokens,
  API keys, endpoints, or credentials.
- Do not load a cache from matching, alter lexical scores, change matcher dispositions, modify
  compiler inputs or generated artifacts, or enable embeddings. With embeddings disabled, matcher
  behavior and generated ontology output must remain unchanged.
- Do not hand-edit `ontology/compiled/` or generated Project Narrative output.

## Tests and verification

Add focused offline tests using the repository's existing test framework. Cover:

- known-vector cosine results, dimension mismatch, non-finite values, and zero-vector rejection;
- `normalizeToUnitScore(cosine)` results for `-1`, `0`, and `1`;
- normalization equivalence between the existing matcher and canonical embedding text;
- stable text ordering and SHA-256 content-hash changes;
- malformed cache entries, duplicate entity IDs, duplicate content hashes, invalid model identity,
  vector dimension mismatch, and non-finite or zero vector rejection;
- exact cache-identity mismatch; deterministic sorted output and six-decimal numeric rounding; and
  failed write, `fsync`, close, and rename operations, each proving temporary-file cleanup,
  preservation of the previous cache, and that rename follows a successful `fsync` and close;
- existing matcher regression tests, proving unchanged lexical matching with embeddings disabled.

Keep all tests offline; no test may use a live endpoint, environment credential, compiler mutation,
or generated-artifact edit. Run the repository's full check (currently `npm run check`, if still
provided), `git diff --check`, and the matcher regression tests. Confirm that
`ontology/compiled/` has no diff.

Commit locally with a focused message. Do not push.

## Governance

Before changes, classify this implementation decision under the target repository's Project
Narrative policy. If it is decision-bearing and a pull request is opened, apply
`narrative-required` together with substantive `## Narrative Context`, `## Narrative Decision`,
and `## Narrative Consequences` sections before merge. Never hand-edit compiled `Narrative.md`;
follow the target repository's fragment and generation rules instead.
