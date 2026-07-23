# Prompt 13 — Add canonical coding-agent instructions

Using the previously supplied repository contract, make the repository’s development and governance
rules discoverable to major coding agents without duplicating the policy.

Read all existing contributor, security, architecture, ontology-governance, and Project Narrative
documentation before editing.

Create root-level `AGENTS.md` as the canonical agent instruction file. It must cover:

- the product objective and finance-as-reference-domain position;
- Node.js, TypeScript, build, test, and full-gate commands;
- the governed ontology change workflow;
- the rule against hand-editing compiled artifacts;
- stable IDs, provenance, confidence, dispositions, inference rules, and premises;
- runtime and SPARQL trust boundaries;
- source-snapshot and sensitive-data rules;
- test and final-verification expectations;
- Project Narrative classification and exact required PR sections;
- Git/review discipline and preservation of unrelated changes.

Create minimal pointer files for:

- `CLAUDE.md`
- `GEMINI.md`
- `.github/copilot-instructions.md`

Each pointer should direct the agent to read and follow `AGENTS.md`; do not copy the full policy into
multiple files.

Project Narrative rules must explicitly say:

- meaningful decision-bearing PRs require `narrative-required`;
- the PR body contains `## Narrative Context`, `## Narrative Decision`, and
  `## Narrative Consequences`;
- mechanical PRs omit the label and sections;
- `Narrative.md` is generated and never edited directly;
- narrative-only proposal/repair PRs do not carry `narrative-required`, preventing recursion.

## Acceptance criteria

- One canonical file owns the complete policy.
- Agent-specific files are short pointers and cannot drift independently.
- Instructions agree with README, contribution, security, architecture, and Narrative workflows.
- `npm run check` and `git diff --check` pass.

Classify this as a meaningful governance/product decision. Commit locally, but do not push.
