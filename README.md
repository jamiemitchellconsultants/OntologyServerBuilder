# OntologyServerBuilder

A staged prompt sequence for building and maturing a governed, general-purpose ontology MCP server
with a coding agent. Finance is the first reference domain, not a restriction on the service.

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
9. [Correct the container host allow-list](prompts/08-container-host-allow-list.md)
10. [Add reuse, security, and contribution policies](prompts/09-reuse-security-and-contribution-policies.md)
11. [Create the post-build review and next-steps document](prompts/10-create-nextsteps-review.md)
12. [Create the Step 4 embedding-matcher guide](prompts/11-create-step4-embedding-guide.md)
13. [Add canonical coding-agent instructions](prompts/12-canonical-agent-instructions.md)
14. [Create the Step 2 security-test guide](prompts/13-create-step2-security-test-guide.md)
15. [Position the service as general purpose](prompts/14-general-purpose-positioning.md)

The sequence deliberately separates architectural boundaries. Each stage requires executable
evidence before the next begins. The reconstruction audit closes the initial build; the remaining
prompts teach how to turn review findings and product feedback into governed follow-up work.
