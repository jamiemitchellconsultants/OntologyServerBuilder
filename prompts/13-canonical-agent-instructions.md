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

Create minimal pointer files for every tier-one agent:

- `CLAUDE.md`
- `GEMINI.md`
- `.github/copilot-instructions.md`
- `.cursor/rules/agent-instructions.mdc`, with `alwaysApply: true` front matter
- `.windsurf/rules/agent-instructions.md`, with `trigger: always_on` front matter
- `.clinerules/agent-instructions.md`

Each pointer should direct the agent to read and follow `AGENTS.md`; do not copy the full policy
into multiple files. An agent whose tool has no pointer file sees no project instructions at all, so
a missing pointer is not a cosmetic omission. Equally, do not list a pointer location the repository
does not actually contain — a pointer to an absent directory teaches a future reader that the set is
maintained when it is not.

Project Narrative rules must explicitly say:

- meaningful decision-bearing PRs require `narrative-required`;
- the PR body contains these headings, each non-empty:
  - `## Narrative Context`
  - `## Narrative Decision`
  - `## Narrative Consequences`
- the label and those sections are applied in the same action, never label-first — a labelled PR
  whose body lacks the sections fails the maintenance run, and if it merges in that window the
  failure is permanent, because the action reads the body from the merge event payload and a re-run
  reads the same incomplete text;
- the maintenance workflow fires on the merge event only, so labelling a merged PR does nothing and
  a missed entry must be hand-written as a fragment;
- creating a PR with a supplied body replaces the repository template wholesale, which is the most
  common way an entry is silently lost;
- mechanical PRs omit the label and sections;
- `Narrative.md` is generated and is never authored, hand-edited, or hand-merged; the only
  hand-written narrative file is a fragment. When two branches each add an entry the fragments merge
  cleanly and only the projection collides, so the resolution is to discard both sides of the
  projection and recompile rather than reconcile the markers;
- an accepted entry is never rewritten to read as though a later framing had been present from the
  start — a reversal is a new entry of kind `correction` citing the original by slug;
- narrative-only proposal/repair PRs do not carry `narrative-required`, preventing recursion.

The same rule against hand-editing generated output governs both compiled ontology artifacts and the
compiled narrative. State it once as a principle and apply it to both, rather than as two unrelated
prohibitions.

## Acceptance criteria

- One canonical file owns the complete policy.
- Agent-specific files are short pointers and cannot drift independently, and one exists for each of
  Claude, Gemini, Copilot, Cursor, Windsurf, and Cline.
- No pointer cites an instruction-file location the repository does not contain.
- The Narrative rules state the merge-event-only limitation, the supplied-body caveat, the
  never-hand-merge rule, and the never-rewrite-an-accepted-entry rule.
- Instructions agree with README, contribution, security, architecture, and Narrative workflows.
- `npm run check` and `git diff --check` pass.

Classify this as a meaningful governance/product decision. Commit locally, but do not push.
