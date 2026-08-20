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

### Prompt authoring note for AI agents

`BuildDeployPopulate/` is now the canonical prompt tree. The former `prompts/` directory is legacy
material; do not create new prompts there. When asked to create a prompt, place it in the appropriate
subdirectory:

- `BuildDeployPopulate/GroupA-Build/` for build and service foundations;
- `BuildDeployPopulate/GroupB-Deploy/` for deployment and release operations; or
- `BuildDeployPopulate/GroupC-Populate/` for governed ontology population and domain content.

Follow the existing numbering and filename convention in that group, and update the relevant
sequence index or documentation when adding a prompt.

1. [Reusable contract](BuildDeployPopulate/GroupA-Build/01-reusable-contract.md)
2. [Project scaffold and domain
   model](BuildDeployPopulate/GroupA-Build/02-scaffold-and-domain-model.md) — Prompt
   A-02
3. [Adopt Project
   Narrative](BuildDeployPopulate/GroupA-Build/03-adopt-project-narrative.md) — Prompt
   A-03
4. [Heterogeneous source
   ingestion](BuildDeployPopulate/GroupA-Build/04-heterogeneous-source-ingestion.md) —
   Prompt A-04
5. [Governed semantic matching and
   compilation](BuildDeployPopulate/GroupA-Build/05-governed-matching-and-compilation.md)
   — Prompt A-05
6. [OWL, SHACL, and build-time
   reasoning](BuildDeployPopulate/GroupA-Build/06-owl-shacl-and-reasoning.md) — Prompt
   A-06
7. [In-process graph queries and
   MCP](BuildDeployPopulate/GroupA-Build/07-in-process-sparql-and-mcp.md) — Prompt A-07
8. [Delivery, security, documentation, and
   governance](BuildDeployPopulate/GroupA-Build/08-delivery-security-and-governance.md)
   — Prompt A-08
9. [Independent reconstruction
   audit](BuildDeployPopulate/GroupA-Build/09-independent-reconstruction-audit.md) —
   Prompt A-09
10. [Correct the container host
    allow-list](BuildDeployPopulate/GroupB-Deploy/01-container-host-allow-list.md) —
    Prompt B-01
11. [Add reuse, security, and contribution
    policies](BuildDeployPopulate/GroupA-Build/10-reuse-security-and-contribution-policies.md)
    — Prompt A-10
12. [Create the post-build review and next-steps
    document](BuildDeployPopulate/GroupA-Build/11-create-nextsteps-review.md) — Prompt
    A-11
13. [Create the Step 4 embedding-matcher
    guide](BuildDeployPopulate/GroupA-Build/12-create-step4-embedding-guide.md) — Prompt
    A-12
14. [Add canonical coding-agent
    instructions](BuildDeployPopulate/GroupA-Build/13-canonical-agent-instructions.md) —
    Prompt A-13
15. [Create the Step 2 security-test
    guide](BuildDeployPopulate/GroupA-Build/14-create-step2-security-test-guide.md) —
    Prompt A-14
16. [Position the service as general
    purpose](BuildDeployPopulate/GroupA-Build/15-general-purpose-positioning.md) —
    Prompt A-15
17. [Register a real invoice
    system](BuildDeployPopulate/GroupC-Populate/01-register-real-invoice-system.md) —
    Prompt C-01
18. [Add governed entity-mapping
    instructions](BuildDeployPopulate/GroupC-Populate/02-governed-mapping-instructions.md)
    — Prompt C-02
19. [Add user refinement and system-registration
    proposals](BuildDeployPopulate/GroupA-Build/16-user-refinement-and-registration-proposals.md)
    — Prompt A-16
20. [Audit proposal safety without rejecting unusual
    systems](BuildDeployPopulate/GroupA-Build/17-audit-proposal-assurance.md) — Prompt
    A-17
21. [Validate access tokens
    in-process](BuildDeployPopulate/GroupA-Build/18-in-process-access-token-validation.md)
    — Prompt A-18
22. [Plan the finance and accounting semantic
    migration](BuildDeployPopulate/GroupC-Populate/03-plan-finance-accounting-semantic-migration.md)
    — Prompt C-03
23. [Add governed standards registries and semantic
    alignments](BuildDeployPopulate/GroupC-Populate/04-add-governed-standards-alignments.md)
    — Prompt C-04
24. [Add FIBO-aligned foundations and a REA accounting
    core](BuildDeployPopulate/GroupC-Populate/05-add-foundational-and-rea-accounting-model.md)
    — Prompt C-05
25. [Add ledger and journal semantics informed by XBRL Global
    Ledger](BuildDeployPopulate/GroupC-Populate/06-add-ledger-and-journal-semantics.md)
    — Prompt C-06
26. [Remodel procure-to-pay with OASIS UBL
    alignments](BuildDeployPopulate/GroupC-Populate/07-remodel-procure-to-pay-with-ubl.md)
    — Prompt C-07
27. [Remodel treasury, payments, and settlement with ISO 20022
    alignments](BuildDeployPopulate/GroupC-Populate/08-remodel-treasury-and-payments-with-iso-20022.md)
    — Prompt C-08
28. [Add UK and international accounting-reporting
    profiles](BuildDeployPopulate/GroupC-Populate/09-add-uk-and-international-reporting-profiles.md)
    — Prompt C-09
29. [Migrate the existing finance reference
    model](BuildDeployPopulate/GroupC-Populate/10-migrate-existing-finance-reference-model.md)
    — Prompt C-10
30. [Independently audit the finance and accounting standards
    migration](BuildDeployPopulate/GroupC-Populate/11-audit-finance-accounting-standards-migration.md)
    — Prompt C-11
31. [Deploy one instance to a homelab or local
    network](BuildDeployPopulate/GroupB-Deploy/02-homelab-local-network-deployment.md) —
    Prompt B-02
32. [Deploy the service to AWS for production
    use](BuildDeployPopulate/GroupB-Deploy/03-aws-production-deployment.md) — Prompt
    B-03
33. [Accept Keycloak access tokens and answer the MCP OAuth
    challenge](BuildDeployPopulate/GroupA-Build/19-keycloak-mcp-oauth-access-tokens.md)
    — Prompt A-19
34. ~~[Make the home-lab deployment actually run `keycloak`
    mode](BuildDeployPopulate/GroupB-Deploy/04-deploy-keycloak-mode-on-the-home-lab.md)~~
    — **withdrawn**; it assumed this repository owned an ingress that belongs to the
    LocalAI deployment — Prompt B-04
35. [Tell a key-retrieval failure apart from an invalid
    token](BuildDeployPopulate/GroupA-Build/20-distinguish-jwks-retrieval-failure.md) —
    Prompt A-20
36. [Plan qualified-user
    intake](BuildDeployPopulate/GroupA-Build/21-plan-qualified-user-intake.md) — Prompt
    A-21
37. [Add durable
    intake](BuildDeployPopulate/GroupA-Build/22-add-durable-intake-and-capabilities.md)
    — Prompt A-22
38. [Add qualified-user intake
    submissions](BuildDeployPopulate/GroupA-Build/23-add-qualified-intake-submissions.md)
    — Prompt A-23
39. [Add the engineer intake
    workbench](BuildDeployPopulate/GroupA-Build/24-add-engineer-intake-workbench.md) —
    Prompt A-24
40. [Add embedding text and cache
    primitives](BuildDeployPopulate/GroupA-Build/25-add-embedding-text-and-cache-primitives.md)
    — Prompt A-25
41. [Add embedding
    evidence](BuildDeployPopulate/GroupA-Build/26-add-embedding-configuration-and-evidence.md)
    — Prompt A-26
42. [Add the explicit embedding refresh
    command](BuildDeployPopulate/GroupA-Build/27-add-embedding-refresh-command.md) —
    Prompt A-27
43. [Fuse embeddings into governed
    matching](BuildDeployPopulate/GroupA-Build/28-fuse-embeddings-into-matching.md) —
    Prompt A-28
44. [Integrate the embedding
    cache](BuildDeployPopulate/GroupA-Build/29-integrate-embedding-cache-with-compilation.md)
    — Prompt A-29
45. [Complete embedding matcher
    delivery](BuildDeployPopulate/GroupA-Build/30-complete-embedding-matcher-delivery.md)
    — Prompt A-30
46. [Add bounded LLM intake
    analysis](BuildDeployPopulate/GroupA-Build/31-add-llm-intake-analysis.md) — Prompt
    A-31
47. [Add deployment release-change
    visibility](BuildDeployPopulate/GroupB-Deploy/05-add-release-change-visibility.md) —
    Prompt B-05
48. [Compile named mapping
    tools](BuildDeployPopulate/GroupA-Build/32-compile-named-mapping-tools.md) — Prompt
    A-32
49. [Add the accounts-payable mapping
    example](BuildDeployPopulate/GroupC-Populate/12-add-accounts-payable-mapping-example.md)
    — Prompt C-12
50. [Independently audit the qualified-user
    workflow](BuildDeployPopulate/GroupA-Build/33-audit-qualified-user-workflow.md) —
    Prompt A-33
51. [Package the qualified-user MCP-registration
    skill](BuildDeployPopulate/GroupA-Build/34-package-qualified-user-registration-skill.md)
    — Prompt A-34
52. [Add ephemeral supplier MCP discovery and
    registration](BuildDeployPopulate/GroupA-Build/35-add-ephemeral-supplier-mcp-registration.md)
    — Prompt A-35
53. [Add engineer-workbench embedding
    analysis](BuildDeployPopulate/GroupA-Build/36-add-workbench-embedding-analysis.md) —
    Prompt A-36 (BuildDeployPopulate only; no `prompts/` counterpart)

Each entry links to its file in `BuildDeployPopulate/` and names the canonical `A-NN` / `B-NN` /
`C-NN` identifier that file's own heading carries. The discussion below cites prompts by the number
originally assigned in the legacy `prompts/` tree, which still matches the number in the file's own
title there; use the list above to find that same prompt's current location and identifier in
`BuildDeployPopulate/`. Where a paragraph instead cites a numbered step inside `docs/step4.md` (for
example "the guide's Prompt 6 assurance and Prompt 7 documentation stages"), that number belongs to
that guide's own internal sequence and is unrelated to this repository's prompt numbering.

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

Prompts 33–47 follow active Prompt 32b and form one ordered qualified-user contribution sequence.
Prompt 33 plans without changing behavior. Prompts 34–36 add the isolated S3-backed intake plane
and workbench. Prompts 37–41 execute the first five build-time embedding stages from the
existing Step 4 guide. Prompt 42 deliberately combines the guide's Prompt 6 assurance and Prompt 7
documentation stages. Prompt 43 adds advisory coding-agent analysis, Prompt 44 adds deployed
change visibility, Prompt 45 adds named pure mapping tools, Prompt 46 proves the
invoice-to-payment example, and Prompt 47 audits the whole boundary independently. Intake stays
disabled by default in every deployment and is enabled only once its S3 bucket, access scope, and
monitoring are deliberately provisioned, including under Prompt 31's multi-instance AWS baseline.

Prompt 48 only depends on Prompt 35's submission-tool names and could run as early as that stage
completes. It is numbered after the Prompt 47 audit purely to avoid renumbering the
already-audited 33–47 sequence, not because it depends on any of Prompts 36–47. It packages the
qualified-user side of registration — reachable today only by manually supplying the
`register-supplier-mcp-server.md` template — as an installable Claude Skill distributed from
`OntologyService`'s own repository, so a qualified user's own client can trigger it directly.

Prompt 49 adds a separate direct-registration path and leaves Prompt 48 unchanged. It lets an
attributable qualified user ask `OntologyService` to connect to one supplied HTTPS MCP endpoint,
complete client-credentials or authorization-code authentication through an ephemeral browser
handoff, capture bounded discovery metadata, and submit immutable review-required evidence. The
stage deliberately introduces outbound access only inside that disabled-by-default intake workflow;
the compiler and delivery plane retain their no-network boundary. Prompt 49 depends on Prompts 20,
32, 35, and 36, but not on Prompts 37–48. Earlier prompt files remain unchanged; Prompt 49 itself
names each previously applied constraint that its direct-registration path supersedes.

### Operational prompt templates

Use these only after the prerequisite stage has completed. Four are engineer-maintained operator
inputs: when an engineer starts the corresponding coding-agent task in `OntologyService`, supply the
full template contents alongside that task, after a human engineer has supplied the reviewed
evidence the template names. Do not copy a template into `OntologyService` or expect a Builder path
to exist there.

- [Register a supplier's MCP server](prompts/templates/register-supplier-mcp-server.md) — for the
  qualified user's own MCP-capable client, immediately after connecting to a supplier's server, once
  Prompt 35's submission tools exist. Unlike the four below, this one runs entirely in the user's
  own session and never touches the `OntologyService` repository.
- [Register reviewed exported intake](prompts/templates/register-exported-intake.md) — after
  Prompt 36's export and deterministic analysis.
- [Resolve an intake mapping review](prompts/templates/resolve-intake-mapping-review.md) — after
  Prompt 43 emits a bounded LLM analysis request.
- [Apply a reviewed ontology-change proposal](prompts/templates/apply-ontology-change-proposal.md)
  — after Prompts 35, 36, and 44 provide a reviewed export and deployed release context.
- [Create or revise a named mapping tool](prompts/templates/create-named-mapping-tool.md) — after
  Prompts 36, 43, 44, and 45 provide reviewed analysis, deployed context, and mapping-tool
  contracts.

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
