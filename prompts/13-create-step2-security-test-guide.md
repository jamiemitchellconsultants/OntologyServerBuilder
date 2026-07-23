# Prompt 13 — Create the Step 2 security-test guide

Using the previously supplied repository contract and the testing recommendation in
`docs/nextsteps.md`, create `docs/step2.md` as a prompt-by-prompt teaching guide for closing the
highest-risk test gaps.

This stage writes the guide only. Do not implement the tests.

## Guide contract

Title the document:

`# Step 2 — Security-critical test gaps: a prompt-by-prompt guide`

Explain that the guide concentrates on:

1. Direct tests for the read-only SPARQL boundary and result cap.
2. Negative-path tests for compiler configuration, manifests, relationships, and manual decisions.

Keep parser-wide coverage, store resolution, and HTTP transport coverage as named follow-up work so
the guide remains reviewable.

Every prompt must include context, goal, threat/failure model, acceptance criteria, verification,
scope boundary, and a “Why this prompt works” teaching note.

Repeat these invariants:

- SPARQL remains local, read-only, and limited to `SELECT` and `ASK`.
- Federation, external datasets, updates, and administrative operations remain rejected.
- Results remain capped at 100 rows.
- Malformed compiler inputs fail closed.
- Tests use local synthetic fixtures and never make network requests.
- Validation and assertions are never weakened to make tests pass.
- Compiled artifacts remain deterministic and compiler-owned.

## Required prompt sequence

Write these prompts in order:

0. **Orient and inventory, no code.** Enumerate every current SPARQL rejection rule, size/result
   limit, compiler validation branch, and existing direct or incidental test.
1. **SPARQL rejection matrix.** Table-test every forbidden keyword, external `FROM` and
   `FROM NAMED` form, non-SELECT/ASK query, length limit, case handling, and word boundaries. Prove
   representative local queries still work.
2. **100-row cap and result contract.** Use more than 100 local triples and verify truncation,
   deterministic bindings, SELECT shape, and ASK shape without relying on network data.
3. **Minimal isolated compiler fixture.** Build a temporary, synthetic fixture seam that does not
   mutate checked-in ontology sources or compiled files.
4. **Configuration and manifest failures.** Cover invalid thresholds, missing files, unsupported
   formats, path violations, malformed manifests, duplicate system/entity IDs, and deterministic
   diagnostics.
5. **Graph integrity and manual-decision failures.** Cover unknown relationship endpoints, unknown
   mapping targets, duplicate/conflicting decisions, invalid dispositions, and fail-closed
   traversal.
6. **Adversarial verification and scope audit.** Run the full suite repeatedly, prove no generated
   diff or network access, review test quality, and produce a requirements-to-evidence table.
7. **Documentation and Narrative classification.** Update only documentation made inaccurate by
   the tests; classify test-only work as mechanical unless it changes shipped behavior or policy.

Finish with an “After you finish” checklist and a short lesson on writing effective security-test
prompts.

Update `docs/nextsteps.md` so Step 2 links to this guide.

## Acceptance criteria

- The guide tests intentional rejection behavior, not private implementation details.
- It tells the learner how to distinguish direct from incidental coverage.
- It provides safe fixture isolation and cleanup expectations.
- No production code, tests, or generated ontology artifacts change in this stage.
- Markdown links resolve and `git diff --check` passes.

Commit locally with a focused documentation message. Do not push.
