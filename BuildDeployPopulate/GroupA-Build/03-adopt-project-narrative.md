# Prompt A-03 — Adopt Project Narrative

Using the previously supplied repository contract, implement Stage 2: install Project Narrative
before the remaining product and architecture work begins.

## Choose the Narrative repository

The reference implementation is `jamiemitchellconsultants/Narrative`.

> **Using a Narrative you built yourself?** If you completed
> [NarrativeBuilder](https://github.com/jamiemitchellconsultants/NarrativeBuilder), you may select
> the compatible Narrative repository you built instead. Substitute its `OWNER/REPOSITORY` for
> `jamiemitchellconsultants/Narrative` throughout this prompt. Do not substitute the
> `NarrativeBuilder` repository itself.

Read the current consumer setup contract in the selected Narrative repository before editing. Do not
reconstruct the integration from memory. Confirm that its action and CLI support the contract
required below, and record the repository and ref or commit you reviewed. If no compatible
learner-built repository is available, use the reference implementation.

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
- invoke the selected Narrative repository with the repository `GITHUB_TOKEN`;
- require the exact `narrative-required` label;
- let the upstream action create a separate draft proposal rather than write accepted history
  directly to the default branch.

The validation workflow must:

- run only when configuration, fragments, the preamble, or `Narrative.md` changes;
- use read-only repository permissions;
- run the upstream action in `check` mode.

Choose deliberately between tracking the selected Narrative action at `@main` and pinning a reviewed
commit SHA. `@main` receives processor updates immediately; a SHA gives tighter supply-chain control
but requires deliberate upgrades. Use the same selected repository consistently and document the
choice.

## Add the authoring contract

The pull-request template must explain:

- meaningful product, architecture, governance, operational, correction, and experimental decisions
  require `narrative-required`;
- mechanical changes remain unlabelled;
- the three required headings are exactly:
  - `## Narrative Context`
  - `## Narrative Decision`
  - `## Narrative Consequences`
- unneeded Narrative sections are removed from mechanical pull requests.

Document that:

- fragments under `narrative/entries/` are authoritative;
- `Narrative.md` is a deterministic projection and is never authored, hand-edited, or hand-merged —
  including when resolving a merge conflict, where two branches each adding an entry collide only on
  the projection while the fragments merge cleanly, so the resolution is to discard both sides of
  the projection and recompile rather than reconcile the markers;
- the label and the three body sections are applied in the same action, never label-first: a
  labelled pull request whose body lacks the sections fails the maintenance run, and if it merges in
  that window the failure is permanent because the action reads the body from the merge event
  payload;
- the maintenance workflow fires on the merge event only, so labelling a merged pull request does
  nothing and a missed entry must be hand-written as a fragment;
- creating a pull request with a supplied body replaces the repository template wholesale, which is
  the most common way an entry is silently lost;
- an accepted entry is never rewritten to read as though a later framing had been present from the
  start — a reversal is a new entry of kind `correction` citing the original by slug;
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

## Pre-merge narrative gate

The scaffolded validation workflow checks fragment validity and compiled freshness. Add a second job
to it, triggered so that it re-evaluates whenever the label or the body changes:

```yaml
on:
  pull_request:
    types: [opened, edited, reopened, synchronize, labeled, unlabeled]
```

**When the pull request carries `narrative-required`**, the job fails unless the body contains all
three non-empty `## Narrative …` sections.

This mirrors the post-merge gate before merge, and it exists because of a specific, observed
failure: the label can be applied while the body still lacks the sections, and if the pull request
merges in that window the maintenance run fails permanently. The action reads the body from the
merge event payload, so re-running it reads the same incomplete text and the entry must be
hand-written instead. A pre-merge check turns that into a red result on an open pull request, which
is fixable.

The job must read the body from an environment variable rather than interpolating it into a shell
command — pull-request prose is untrusted input.

It deliberately does not check for a *missing* label; only a human can classify a change. It catches
the case where classification was declared and the evidence was not supplied.

## Repository settings checklist

Do not silently assume GitHub settings can be changed from the worktree. Report the manual or
repository-owner actions required before merge:

1. Allow the consumer repository to use the selected public action or pin.
2. Set Actions workflow permissions to read/write.
3. Enable “Allow GitHub Actions to create and approve pull requests.”
4. Create the exact `narrative-required` label with an explanatory description.

If the target repository does not yet exist on GitHub, stop after the local commit and tell the
learner that the installation must be published and merged before Prompt A-04.

## Acceptance criteria

- The configuration, preamble, fragment, and generated `Narrative.md` validate and compile
  deterministically.
- The validation workflow fails a `narrative-required` pull request whose body is missing any of the
  three sections, and reads the body from an environment variable rather than a shell interpolation.
- The authoring contract states the never-hand-merge rule, the merge-event-only limitation, the
  supplied-body caveat, and the never-rewrite-an-accepted-entry rule.
- Validation is local, model-free, and network-free after the action/CLI is available.
- Maintenance permissions are no broader than documented.
- A labelled PR missing any required section fails visibly rather than receiving invented text.
- An unmerged or unlabelled PR produces no narrative proposal.
- The PR template and workflows use the exact same label and section names.
- `git diff --check` passes.

Commit this bootstrap locally with a focused governance message. Do not push automatically. Pause
the learning sequence until the learner explicitly publishes and merges the installation to the
default branch.
