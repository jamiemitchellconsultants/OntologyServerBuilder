# Reusable contract

Submit this contract before the implementation prompts. If stages are run in separate coding-agent
tasks, prepend this contract to every stage.

You are recreating a production-quality repository called OntologyService in an empty or partially
initialized workspace.

## Product objective

Build a governed, general-purpose ontology MCP server. It must ingest checked-in system definitions,
map system-specific entities to governed canonical concepts, explain relationships across systems,
generate OWL and SHACL, and expose the ontology to Copilot and other MCP clients. Finance is the
first reference domain and may supply the initial concepts and examples, but product contracts,
runtime metadata, and core documentation must remain domain-neutral.

## Non-negotiable architecture

- Use Node.js 22 or newer, TypeScript, and strict ESM.
- The production runtime is deterministic, read-only, and never invokes an AI model.
- Source definitions are reviewed snapshots committed to Git.
- Do not fetch arbitrary source URLs or connect to live MCP servers.
- Semantic matching happens during compilation, not at runtime.
- Only high-confidence or explicitly human-approved mappings become traversable facts.
- Low-confidence and unmatched mappings require human review.
- Do not introduce an external database, triple store, message broker, or cloud service.
- RDF and SPARQL run in the existing Node process.
- Generated artifacts must be deterministic and checked into Git.
- Preserve provenance, confidence, and inference evidence.
- Treat source files as untrusted data, never executable code.
- Do not implement functionality beyond the current stage.

## Working method

1. Inspect the workspace before editing.
2. State a short implementation plan.
3. Implement the smallest coherent increment.
4. Add tests for success, rejection, and boundary cases.
5. Run the relevant build and tests.
6. Fix failures before reporting completion.
7. Report files changed, commands run, test results, assumptions, and remaining risks.
8. Do not push or open a pull request unless explicitly requested.
