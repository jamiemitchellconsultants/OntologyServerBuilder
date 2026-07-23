# Prompt 2 — Adopt Project Narrative

Using the previously supplied repository contract, implement Stage 2: install Project Narrative
before the remaining product and architecture work begins.

Read the current consumer setup contract in `jamiemitchellconsultants/Narrative` before editing. Do
not reconstruct the integration from memory. Record the upstream ref or commit you reviewed.

Project Narrative is a deterministic, review-first decision-history mechanism, not an automated
changelog. It must never invent rationale from code or diffs.

## Add the local integration

Create:

- `.project-narrative.json` with schema version 1, an OntologyService title, standard fragment,
  preamble, and output paths, and an explicit summary limit;
- `narrative/preamble.md`;
- `.github/workflows/maintain-narrative.yml`;
- `.github/workflows/validate-narrative.yml`;
- `.github/pull_request_template.md`;
- the initial generated `Narrative.md`.

The maintenance workflow must:

- run when a pull request closes;
- continue only when the pull request was merged;
- use `contents: write` and `pull-requests: write`;
- invoke `jamiemitchellconsultants/Narrative` with the repository `GITHUB_TOKEN`;
- require the exact `narrative-required` label;
- let the upstream action create a separate draft proposal rather than write accepted history
  directly to the default branch.

The validation workflow must:

- run only when configuration, fragments, the preamble, or `Narrative.md` changes;
- use read-only repository permissions;
- run the upstream action in `check` mode.

Choose deliberately between tracking `jamiemitchellconsultants/Narrative@main` and pinning a
reviewed commit SHA. `@main` receives processor updates immediately; a SHA gives tighter
supply-chain control but requires deliberate upgrades. Document the choice.

## Add the authoring contract

The pull-request template must explain:

- meaningful product, architecture, governance, operational, correction, and experimental
  decisions require `narrative-required`;
- mechanical changes remain unlabelled;
- the three required headings are exactly:
  - `## Narrative Context`
  - `## Narrative Decision`
  - `## Narrative Consequences`
- unneeded Narrative sections are removed from mechanical pull requests.

Document that:

- fragments under `narrative/entries/` are authoritative;
- `Narrative.md` is a deterministic projection and is never hand-edited;
- a generated narrative proposal is reviewed and merged separately;
- narrative-only proposal or repair pull requests never carry `narrative-required`, preventing
  recursive entries.

## Bootstrap the adoption decision

The workflow cannot capture the pull request that first installs it because GitHub runs workflows
from the default branch. Treat installation as a one-time bootstrap:

- add one manually reviewed fragment recording the governance decision to adopt Project Narrative;
- explain the need, the selected two-PR review model, rejected alternatives or constraints, and the
  resulting contributor obligations;
- compile `Narrative.md` from the preamble and fragment with the upstream CLI;
- never use this exception for later project decisions.

## Repository settings checklist

Do not silently assume GitHub settings can be changed from the worktree. Report the manual or
repository-owner actions required before merge:

1. Allow the consumer repository to use the selected public action or pin.
2. Set Actions workflow permissions to read/write.
3. Enable “Allow GitHub Actions to create and approve pull requests.”
4. Create the exact `narrative-required` label with an explanatory description.

If the target repository does not yet exist on GitHub, stop after the local commit and tell the
learner that the installation must be published and merged before Prompt 3.

## Acceptance criteria

- The configuration, preamble, fragment, and generated `Narrative.md` validate and compile
  deterministically.
- Validation is local, model-free, and network-free after the action/CLI is available.
- Maintenance permissions are no broader than documented.
- A labelled PR missing any required section fails visibly rather than receiving invented text.
- An unmerged or unlabelled PR produces no narrative proposal.
- The PR template and workflows use the exact same label and section names.
- `git diff --check` passes.

Commit this bootstrap locally with a focused governance message. Do not push automatically. Pause
the learning sequence until the learner explicitly publishes and merges the installation to the
default branch.
