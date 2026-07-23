# Prompt 8 — Independent reconstruction audit

Use this prompt as a fresh task or, ideally, with a different coding agent.

Audit the completed OntologyService reconstruction against the intended product and architecture.
Do not add new features unless required to correct a defect.

Verify independently:

1. Every supported source format has a parser and test fixture.
2. Input paths remain local and untrusted input is bounded.
3. IDs, sorting, fingerprints, JSON, OWL, and SHACL are deterministic.
4. Low-confidence mappings cannot enter relationship traversal.
5. Human decisions are persisted and override suggestions correctly.
6. Every supported OWL rule has positive and negative tests.
7. Inference evidence contains rules, premises, and confidence.
8. Superclass inference cannot create misleading business paths.
9. SPARQL cannot perform updates, federation, or external loading.
10. MCP works over stdio and Streamable HTTP.
11. Runtime code invokes no AI model or external ontology service.
12. CI, Docker, documentation, and Narrative governance agree with implementation.

Run:

- `npm ci`
- `npm run check`
- `npm audit --audit-level=high`
- `git diff --check`
- a Docker build if available

Report findings by severity with exact file and line references. Fix confirmed defects, rerun
verification, and provide a final requirements-to-evidence table. Do not push or merge.
