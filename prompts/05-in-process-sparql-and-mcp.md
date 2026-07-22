# Prompt 5 — In-process graph queries and MCP

Using the previously supplied repository contract, implement Stage 5: runtime graph access,
constrained SPARQL, and MCP delivery.

Create an in-memory ontology store that:

- resolves entities by stable ID or unambiguous name;
- searches IDs, names, descriptions, and attributes;
- lists systems and mapping reviews;
- explains governed cross-system relationships using bounded breadth-first traversal;
- returns every hop with predicate, direction, cardinality, confidence, and provenance.

Add Comunica over an in-memory RDF/JS store. Load Comunica lazily on the first query.

## SPARQL policy

- Accept `SELECT` and `ASK` only.
- Limit queries to 5,000 characters.
- Limit results to 100 rows.
- Forbid `SERVICE`, `FROM` or external datasets, `LOAD`, updates, and administrative operations.
- Provide no network source to the query engine.

Implement an MCP server exposing these tools:

- `ontology_list_systems`
- `ontology_search_entities`
- `ontology_describe_entity`
- `ontology_explain_relationship`
- `ontology_list_mapping_reviews`
- `ontology_supported_sources`
- `ontology_query_sparql`

Expose these MCP resources:

- `ontology://catalog`
- `ontology://mapping-review`
- `ontology://owl`
- `ontology://shacl`

Support:

- local stdio transport;
- stateless Streamable HTTP at `/mcp`;
- `/health` endpoint;
- `HOST` and `PORT` configuration;
- host-header protection;
- refusal to bind publicly without explicit allowed hosts.

All tools must be marked read-only and idempotent where appropriate. Return structured results and
clear error responses.

Add in-memory MCP protocol tests, SPARQL `SELECT` and `ASK` tests, forbidden-query tests,
ambiguous-name tests, and end-to-end relationship explanation tests.

Add `.vscode/mcp.json` for local Copilot usage.

Run `npm run check` and commit the stage locally.
