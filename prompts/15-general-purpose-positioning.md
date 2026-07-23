# Prompt 15 — Position OntologyService as general purpose

Using the previously supplied repository contract, audit and revise the repository so the service
is clearly general purpose while finance remains its first reference domain.

Read before editing:

- `README.md`
- `docs/architecture.md`
- `docs/registering-systems.md`
- `docs/nextsteps.md`
- `CONTRIBUTING.md`
- `AGENTS.md`
- `package.json`
- `.vscode/mcp.json`
- MCP, CLI, server, matcher, and tests;
- ontology source manifests and generated artifacts;
- `Narrative.md` and its source fragments.

Separate two layers:

1. **Product identity and contracts** must be domain-neutral.
2. **Reference data and examples** may remain finance-specific and should be labelled accordingly.

Revise:

- repository title and opening description;
- architecture objective and component labels;
- contributor and agent guidance;
- system-registration examples, including at least one non-finance example;
- roadmap language;
- MCP server name, tool descriptions, resource titles, and schema help;
- CLI usage, server logs, package binary names, Docker tags/hostnames, and VS Code MCP name;
- generated mapping-relationship wording.

Use `ontology-service` for the MCP/package/CLI identity. Keep finance fixtures, canonical concepts,
and relationship examples where they provide useful first-consumer coverage. Do not rename data
files or stable ontology IDs merely to hide the reference domain.

Never hand-edit generated ontology artifacts. Change their source wording, run
`npm run ontology:compile`, and review the generated diff. Do not rewrite historical generated
Narrative entries; they record the decisions accepted at the time.

Add tests that prove:

- the MCP server advertises the domain-neutral identity;
- public tool/resource descriptions do not present finance as a platform restriction;
- existing finance reference relationship queries still work.

## Acceptance criteria

- A new reader understands within the README opening that the service supports any governed domain.
- Finance is consistently described as the first reference domain.
- Runtime and package metadata agree with the documentation.
- Generated diffs are limited to explained wording changes.
- `npm run check`, `git diff --check`, and the final terminology audit pass.

This is a meaningful product-positioning decision. If opening a pull request, apply
`narrative-required` and include substantive Narrative Context, Narrative Decision, and Narrative
Consequences.

Commit locally with a focused message. Do not push.
