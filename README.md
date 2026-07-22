# OntologyServerBuilder

A staged prompt sequence for building a governed, general-purpose finance ontology MCP server with
a coding agent.

## How to use the prompts

Use the same coding-agent task throughout when possible. Submit the reusable contract first, then
submit each implementation stage only after the previous stage has passed its acceptance checks.

1. [Reusable contract](prompts/00-reusable-contract.md)
2. [Project scaffold and domain model](prompts/01-scaffold-and-domain-model.md)
3. [Heterogeneous source ingestion](prompts/02-heterogeneous-source-ingestion.md)
4. [Governed semantic matching and compilation](prompts/03-governed-matching-and-compilation.md)
5. [OWL, SHACL, and build-time reasoning](prompts/04-owl-shacl-and-reasoning.md)
6. [In-process graph queries and MCP](prompts/05-in-process-sparql-and-mcp.md)
7. [Delivery, security, documentation, and Narrative](prompts/06-delivery-security-and-governance.md)
8. [Independent reconstruction audit](prompts/07-independent-reconstruction-audit.md)

The sequence deliberately separates architectural boundaries. Each stage requires executable
evidence before the next begins, while the final audit is designed for a fresh agent or context.
