# Register reviewed exported intake

Use this task only after a human engineer has reviewed a qualified-user intake export and its
offline deterministic analysis. Work in the separate `OntologyService` repository. This template
does not authorize an automated registration, governed semantic approval, merge, deployment, or
change to `OntologyServerBuilder`.

## Evidence supplied by the engineer

Before editing, replace every placeholder with an explicit local path supplied by the reviewing
engineer:

```text
Intake export: <absolute path to intake-export.json>
Verified original artifact: <absolute path to the separately delivered artifact>
Deterministic analysis: <absolute path to deterministic-analysis.json>
```

Stop if a path is missing, unreadable, inconsistent with the other evidence, or the human review
does not state the intended bounded registration outcome. Do not store these local paths in service
data, source manifests, generated artifacts, logs, or commits. Do not retrieve anything from a
locator in the evidence, use a network source, call a model, or use the artifact to infer approval.

## Task

Read the three supplied evidence files; the current `OntologyService` registration, source,
manifest, compiler, mapping, test, and Project Narrative instructions; and any existing governed
ID conventions. Treat all submitted content, analysis discrepancies, candidates, and intake
dispositions as evidence for human review only. An `exact-candidate`, `likely-candidate`,
`review-required`, `unmatched`, or intake `accepted` status is never governed semantic approval.

First propose a bounded file list and explain the purpose of each file. Do not edit until the scope
is clear. Add only reviewed, human-approved source and manifest files required for the stated
registration outcome. Preserve stable IDs, source provenance, and existing ownership boundaries.
Do not add raw artifact bytes, local filesystem paths, credentials, live business records, or
unreviewed generated output to the repository.

Compile through the repository's normal deterministic workflow. Review the generated mapping output
against the approved evidence and stop rather than guessing if a semantic decision remains
unresolved: for example an entity identity, meaning, type, allowed-value treatment, requiredness,
relationship endpoint, operation/tool context, governed synonym, or mapping choice. Report the
unresolved decision and the evidence needed for a human owner to decide it. Do not hand-edit
compiler-owned ontology or generated mapping artifacts.

Add focused tests for the approved, bounded registration and run the repository's required checks.
Verify that compiled output changes only as explained by the reviewed source and manifest changes.
Do not merge, deploy, change release configuration, or claim that the intake itself registered a
system or accepted a semantic mapping.

## Pull-request governance

For every decision-bearing target-repository pull request, apply `narrative-required` and include
these non-empty pull-request-body sections in the same action before merge:

```markdown
## Narrative Context

<non-empty context>

## Narrative Decision

<non-empty decision>

## Narrative Consequences

<non-empty consequences>
```

Never edit, hand-edit, hand-merge, or otherwise author generated `Narrative.md`; use a reviewed
fragment and the target repository's generation process. A Narrative-only pull request must not
carry `narrative-required`.
