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
8. [Delivery, security, documentation, and
   governance](prompts/07-delivery-security-and-governance.md)
9. [Independent reconstruction audit](prompts/08-independent-reconstruction-audit.md)
10. [Correct the container host allow-list](prompts/09-container-host-allow-list.md)
11. [Add reuse, security, and contribution
    policies](prompts/10-reuse-security-and-contribution-policies.md)
12. [Create the post-build review and next-steps document](prompts/11-create-nextsteps-review.md)
13. [Create the Step 4 embedding-matcher guide](prompts/12-create-step4-embedding-guide.md)
14. [Add canonical coding-agent instructions](prompts/13-canonical-agent-instructions.md)
15. [Create the Step 2 security-test guide](prompts/14-create-step2-security-test-guide.md)
16. [Position the service as general purpose](prompts/15-general-purpose-positioning.md)
17. [Register a real invoice system](prompts/16-register-real-invoice-system.md)
18. [Add governed entity-mapping instructions](prompts/17-governed-mapping-instructions.md)
19. [Add user refinement and system-registration
    proposals](prompts/18-user-refinement-and-registration-proposals.md)
20. [Audit proposal safety without rejecting unusual
    systems](prompts/19-audit-proposal-assurance.md)
21. [Validate access tokens in-process](prompts/20-in-process-access-token-validation.md)
22. [Plan the finance and accounting semantic
    migration](prompts/21-plan-finance-accounting-semantic-migration.md)
23. [Add governed standards registries and semantic
    alignments](prompts/22-add-governed-standards-alignments.md)
24. [Add FIBO-aligned foundations and a REA accounting
    core](prompts/23-add-foundational-and-rea-accounting-model.md)
25. [Add ledger and journal semantics informed by XBRL
    Global Ledger](prompts/24-add-ledger-and-journal-semantics.md)
26. [Remodel procure-to-pay with OASIS UBL
    alignments](prompts/25-remodel-procure-to-pay-with-ubl.md)
27. [Remodel treasury, payments, and settlement with ISO 20022
    alignments](prompts/26-remodel-treasury-and-payments-with-iso-20022.md)
28. [Add UK and international accounting-reporting
    profiles](prompts/27-add-uk-and-international-reporting-profiles.md)
29. [Migrate the existing finance reference
    model](prompts/28-migrate-existing-finance-reference-model.md)
30. [Independently audit the finance and accounting standards
    migration](prompts/29-audit-finance-accounting-standards-migration.md)
31. [Deploy one instance to a homelab or local
    network](prompts/30-homelab-local-network-deployment.md)
32. [Deploy the service to AWS for production use](prompts/31-aws-production-deployment.md)
33. [Accept Keycloak access tokens and answer the MCP OAuth
    challenge](prompts/32-keycloak-mcp-oauth-access-tokens.md)
34. ~~[Make the home-lab deployment actually run `keycloak`
    mode](prompts/32a-deploy-keycloak-mode-on-the-home-lab.md)~~ — **withdrawn**; it assumed
    this repository owned an ingress that belongs to the LocalAI deployment
35. [Tell a key-retrieval failure apart from an invalid
    token](prompts/32b-distinguish-jwks-retrieval-failure.md)
36. [Plan qualified-user intake](prompts/33-plan-qualified-user-intake.md)
37. [Add durable intake](prompts/34-add-durable-intake-and-capabilities.md)
38. [Add qualified-user intake submissions](prompts/35-add-qualified-intake-submissions.md)
39. [Add the engineer intake workbench](prompts/36-add-engineer-intake-workbench.md)

The sequence deliberately separates architectural boundaries. Each stage requires executable
evidence before the next begins. The reconstruction audit closes the initial build; the remaining
prompts teach how to turn review findings and product feedback into governed follow-up work.
Prompts 21–29 form a second, compatibility-sensitive sequence for replacing the thin finance
reference model with a standards-aligned finance and accounting semantic layer. Run them in order:
the first prompt inventories and plans without changing ontology facts, the middle prompts add
governed layers alongside existing behavior, Prompt 28 performs the migration, and Prompt 29 audits
the result independently. The reporting stages support a UK-based international group but require
an explicit accounting-basis decision for each legal entity and consolidation context.

Prompts 30 and 31 are the deployment stages. They depend on the container image from Prompt 7, the
host allow-list correction from Prompt 9, and the in-process token validation from Prompt 20 — not
on the finance sequence, so a repository that stopped after Prompt 20 deploys the same way as one
that completed Prompt 29. Run Prompt 30 first: it establishes the deployment vocabulary on a host
where a mistake is recoverable, and Prompt 31 refers back to its hardening controls. Each states one
tested baseline and labels anything it could not verify as a manual gate rather than as support.

Prompt 32 supersedes Prompt 30's rule that `MCP_AUTH_MODE=static` is the only acceptable home-lab
mode. It adds a Keycloak mode and, more to the point, the OAuth protected-resource metadata and 401
challenge that let a conversational agent obtain a token for itself — without which a home-lab
deployment is reachable only by clients that can be handed a token in a config file. Production
remains Entra, unchanged. Run it after Prompt 30; it does not affect Prompt 31.

Prompt 2 is a governance bootstrap. Commit it locally first, then explicitly publish and merge that
installation before opening decision-bearing pull requests from later prompts. The maintenance
workflow must exist on the default branch before it can capture subsequent decisions.

> **Remember the second pull request.** A decision-bearing implementation is not finished when its
> labelled pull request merges. Project Narrative automation then opens a separate draft proposal
> containing the new fragment and regenerated `Narrative.md`. Review and merge that proposal
> separately. Do not add `narrative-required` to the Narrative-only pull request, or it would create
> a recursive entry.

> **Built Project Narrative yourself?** If you completed
> [NarrativeBuilder](https://github.com/jamiemitchellconsultants/NarrativeBuilder), Prompt 2 lets
> you use the compatible Narrative repository you built instead of the reference
> `jamiemitchellconsultants/Narrative` repository. Later prompts continue using whichever Narrative
> implementation you installed in Prompt 2.
