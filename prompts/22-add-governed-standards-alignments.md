# Prompt 22 — Add governed standards registries and semantic alignments

Using the previously supplied repository contract and the approved migration plan from Prompt 21,
add a deterministic mechanism for recording external standards and reviewed semantic alignments.
Do not yet remodel the finance concepts.

Read before editing:

- the Prompt 21 migration document and its approved open decisions;
- all ontology input formats, model types, compiler validation, RDF projection, SHACL generation,
  runtime store, MCP descriptions, and tests;
- `AGENTS.md`, architecture, registration, security, and Narrative guidance;
- SKOS mapping properties, OWL equivalence semantics, and PROV-O provenance from official W3C
  specifications.

Implement governed, checked-in inputs with names adapted to the repository, such as:

- `ontology/standards.json`, containing standards and exact releases;
- `ontology/external-alignments.json`, containing entity/property alignments.

Each standards registry entry must support:

- stable internal standard ID;
- title, publisher, official source URI, namespace where applicable, and exact release;
- release/effective dates and optional superseded release;
- scope and intended use;
- local snapshot path and SHA-256 only when a reviewed, redistributable snapshot exists;
- licence or terms status, including `approved`, `restricted`, `reference-only`, or
  `review-required`;
- lifecycle status;
- provenance and reviewer rationale.

Each alignment must support:

- a registered internal entity, attribute, relationship predicate, or reporting concept;
- the registered standard release;
- an external IRI or stable concept identifier;
- exactly one governed relation:
  - `exact-match`;
  - `close-match`;
  - `broad-match`;
  - `narrow-match`;
  - `representation-of`;
  - `informed-by`;
- review status, rationale, evidence, and provenance;
- optional effective and expiry dates.

Use SKOS predicates for exact, close, broad, and narrow mappings in RDF. Define a project predicate
for `representation-of` because a document or message representation is not automatically an OWL
equivalent class. Treat `informed-by` as provenance, not logical equivalence.

Validate and reject:

- unknown standards, internal subjects, or relation types;
- duplicate alignment identities;
- malformed or relative external IRIs where an IRI is required;
- `exact-match` with review-required evidence;
- active alignments to expired standards without an explicit historical effective period;
- local snapshot paths outside governed ontology inputs;
- missing digests for declared snapshots;
- attempts to turn `reference-only` or restricted content into imported ontology data.

Do not:

- follow external OWL imports;
- fetch standards during compilation or runtime;
- add `owl:equivalentClass` merely because two labels look similar;
- copy protected definitions into fixtures;
- claim standards conformance from an alignment;
- make an alignment traversable as a business relationship unless separately governed as one.

Project the registry and alignments deterministically into compiled JSON and OWL. Include them in
entity descriptions where useful, but preserve bounded response sizes. Add a read-only MCP query
only if it is the smallest coherent way to inspect standards metadata; otherwise extend the
existing entity-description response.

Add lifecycle metadata usable by later prompts for canonical entities and relationships:

- `active`;
- `deprecated`, with rationale and optional replacement;
- `retired`, excluded from new matching and traversal but retained where required for historical
  interpretation.

Do not deprecate current finance entities in this stage.

Seed only well-supported registry entries. If a specification or taxonomy package has not been
supplied and its reuse rights reviewed, record it as `reference-only` or `review-required`; do not
download or invent a local snapshot. Tests may use small synthetic standards and alignments rather
than copied standard content.

Add tests proving:

- deterministic ordering and fingerprints;
- valid alignment RDF projection;
- exact versus close match remain distinguishable;
- representation links do not create subclass or business-path inference;
- invalid subjects, releases, IRIs, statuses, and snapshot metadata fail closed;
- compilation and runtime perform no external fetch;
- lifecycle metadata cannot silently remove historical provenance;
- checked-in compiled artifacts are current.

Update architecture and registration documentation with the owner workflow for acquiring,
licensing, hashing, reviewing, upgrading, and retiring a standard release.

## Acceptance criteria

- External standards are versioned governed inputs, not informal links in descriptions.
- Alignments preserve semantic strength and cannot accidentally assert OWL equivalence.
- No standards package is fetched or redistributed without an approved snapshot.
- Registry and alignment changes affect the ontology fingerprint deterministically.
- Runtime remains read-only and offline.
- `npm run check` and `git diff --check` pass.
- Generated artifacts contain only compiler-produced, explained changes.

This changes ontology governance and the public semantic model. If opening a pull request, use the
repository's `narrative-required` workflow with all three substantive Narrative sections. Review
the separate Narrative-only proposal after merge and leave it unlabelled.

Commit locally with a focused standards-governance message. Do not push.
