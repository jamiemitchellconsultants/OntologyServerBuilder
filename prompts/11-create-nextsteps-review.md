# Prompt 11 — Create the post-build review and next-steps document

Using the previously supplied repository contract, perform an evidence-based review of the current
OntologyService repository and create `docs/nextsteps.md`.

This stage is documentation and analysis. Do not implement roadmap features while writing the
review.

## Inspect first

Read:

- `README.md`
- `docs/architecture.md`
- `docs/registering-systems.md`
- `docs/owl-profile.md`
- `SECURITY.md`
- `CONTRIBUTING.md`
- `package.json`
- all source and test filenames;
- ontology configuration, source manifests, manual mappings, and compiled review output;
- Docker and CI configuration;
- recent Git history.

Run:

- `npm run check`
- `git diff --check`
- `git status --short`

Use command output and repository evidence for every concrete claim. Do not copy stale counts,
commit IDs, branch state, open-issue state, or test totals from another repository.

## Required document structure

Create `docs/nextsteps.md` with:

1. A dated review title and a short progress/status line.
2. A health snapshot describing what is genuinely working.
3. A prioritized governance backlog for unresolved manual mappings and gaps in the finance
   reference model. Make clear that these are reference-data gaps, not limitations on supported
   domains.
4. Security-critical test gaps, especially direct SPARQL guardrail tests, the 100-row cap, compiler
   rejection paths, store ambiguity, parsers, and HTTP boundaries.
5. The container host allow-list correction, marked completed only if Prompt 9 is present in Git
   history and verified in the current files.
6. The build-time embedding matcher as the largest product step, with its trust-boundary and
   determinism constraints.
7. Reuse/security/contribution policy work, marked completed only if Prompt 10 is present and the
   files agree.
8. Documented first-release limits that should remain scheduled rather than presented as defects.
9. A concise recommended sequence separating quick governance/test work from larger product bets.

Use strikethrough and a completion marker for completed work, but never mark a step complete merely
because a prompt exists. Include commit identifiers only when verified from local history.

Reserve Step 2 for the security-test guide and Step 4 for the embedding-matcher guide. Later prompts
will add `docs/step2.md` and `docs/step4.md` and update this review with links.

## Quality bar

- Separate facts, inferences, recommendations, and completed work.
- Explain why each recommendation matters and name the smallest useful next action.
- Keep finance examples explicitly framed as the first reference domain.
- Do not edit generated `Narrative.md`.
- Do not modify production code.
- `git diff --check` must pass.

Commit the review locally with a focused documentation message. Do not push.
