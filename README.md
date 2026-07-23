# OntologyServerBuilder

A staged prompt sequence for building and maturing a governed, general-purpose ontology MCP server
with a coding agent. Finance is the first reference domain, not a restriction on the service.

## New to GitHub or coding agents?

Start with [Start here: set up your repository and coding agent](START-HERE.md). It walks through
creating `MyOntologyServer`, cloning it locally, choosing one coding agent, connecting that agent to
the correct repository, and understanding the difference between local and cloud work.

## How to use the prompts

Use the same coding-agent task throughout when possible. Submit the reusable contract first, then
submit each implementation stage only after the previous stage has passed its acceptance checks.

1. [Reusable contract](prompts/00-reusable-contract.md)
2. [Project scaffold and domain model](prompts/01-scaffold-and-domain-model.md)
3. [Adopt Project Narrative](prompts/02-adopt-project-narrative.md)
4. [Heterogeneous source ingestion](prompts/03-heterogeneous-source-ingestion.md)
5. [Governed semantic matching and compilation](prompts/04-governed-matching-and-compilation.md)
6. [OWL, SHACL, and build-time reasoning](prompts/05-owl-shacl-and-reasoning.md)
7. [In-process graph queries and MCP](prompts/06-in-process-sparql-and-mcp.md)
8. [Delivery, security, documentation, and governance](prompts/07-delivery-security-and-governance.md)
9. [Independent reconstruction audit](prompts/08-independent-reconstruction-audit.md)
10. [Correct the container host allow-list](prompts/09-container-host-allow-list.md)
11. [Add reuse, security, and contribution policies](prompts/10-reuse-security-and-contribution-policies.md)
12. [Create the post-build review and next-steps document](prompts/11-create-nextsteps-review.md)
13. [Create the Step 4 embedding-matcher guide](prompts/12-create-step4-embedding-guide.md)
14. [Add canonical coding-agent instructions](prompts/13-canonical-agent-instructions.md)
15. [Create the Step 2 security-test guide](prompts/14-create-step2-security-test-guide.md)
16. [Position the service as general purpose](prompts/15-general-purpose-positioning.md)

The sequence deliberately separates architectural boundaries. Each stage requires executable
evidence before the next begins. The reconstruction audit closes the initial build; the remaining
prompts teach how to turn review findings and product feedback into governed follow-up work.

Prompt 2 is a governance bootstrap. Commit it locally first, then explicitly publish and merge that
installation before opening decision-bearing pull requests from later prompts. The maintenance
workflow must exist on the default branch before it can capture subsequent decisions.

> **Built Project Narrative yourself?** If you completed
> [NarrativeBuilder](https://github.com/jamiemitchellconsultants/NarrativeBuilder), Prompt 2 lets
> you use the compatible Narrative repository you built instead of the reference
> `jamiemitchellconsultants/Narrative` repository. Later prompts continue using whichever Narrative
> implementation you installed in Prompt 2.
