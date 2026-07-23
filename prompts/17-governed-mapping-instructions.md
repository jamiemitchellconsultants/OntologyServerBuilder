# Prompt 17 — Add governed entity-mapping instructions

Using the previously supplied repository contract, add reviewed, attribute-level mapping
instructions that tell an agent how to construct one registered entity from another. Use a
Brightflag invoice to payment-instruction transformation as the worked example.

Read before editing:

- `README.md`
- `docs/architecture.md`
- `docs/registering-systems.md`
- `CONTRIBUTING.md`
- `AGENTS.md`
- ontology models, compiler, generated artifacts, manual mappings, and relationship traversal;
- MCP tools, resources, schemas, store implementation, and tests;
- the Brightflag source and manifest added in Prompt 16.

Introduce a governed source file such as `ontology/mapping-instructions.json`. Define and validate:

- stable instruction IDs;
- source and target entity IDs;
- status: `approved`, `review-required`, or `deprecated`;
- preconditions using a small declarative operator set such as `equals`, `not-equals`, `in`, and
  `exists`;
- target-field transformations using a small declarative operation set such as `copy`, `constant`,
  `template`, `coalesce`, and governed `lookup`;
- failure behavior such as `error`, `omit`, or `require-human-input`;
- unresolved requirements, evidence, provenance, and review notes.

These instructions are data, never executable code. Do not allow arbitrary expressions, scripts,
callbacks, network calls, external datasets, or template features that can escape the declared
operation set.

Make mapping instructions part of deterministic compilation and the compiled ontology
fingerprint. Validate that:

- every referenced entity and attribute exists;
- a lookup traverses governed, accepted relationships only;
- lookup steps are contiguous and have an unambiguous result attribute;
- required target attributes are mapped;
- an `approved` instruction has no unresolved requirements;
- `review-required` and `deprecated` instructions are never actionable.

Project reviewed instructions into the compiled JSON representation and, where appropriate, into
the OWL output without pretending procedural transformations are OWL inference rules. Never
hand-edit compiled artifacts.

Add an MCP tool named `ontology_get_mapping_instructions` and, if consistent with the existing
runtime design, a read-only mapping-instructions resource. The tool should accept stable source and
target entity IDs and return:

- instruction status and whether it is actionable;
- preconditions;
- ordered target-field transformations;
- governed lookup paths;
- failure behavior;
- unresolved requirements, evidence, and ontology fingerprint.

Use the Brightflag example to model at least:

- a payment-instruction identifier derived deterministically from the source invoice;
- beneficiary resolution through vendor to vendor office to pay site;
- payment amount copied from the approved gross total;
- currency copied from the invoice currency code.

Keep the example `review-required` until the target Accounts Payable payment-instruction entity is
registered and human reviewers have resolved, at minimum:

- which invoice statuses are payable;
- how to choose among multiple offices or pay sites;
- which target attributes are required;
- any target-system formatting, validation, and idempotency rules.

Do not invent a target entity merely to approve the example.

Add tests that prove:

- an approved, complete instruction is returned as actionable;
- review-required and deprecated instructions are returned as non-actionable;
- missing entities or attributes, broken lookup paths, unsupported operators, duplicate target
  mappings, and unresolved required fields fail compilation;
- approved instructions with unresolved requirements are rejected;
- the MCP tool is read-only, bounded, deterministic, and exposes the compiled fingerprint;
- repeated compilation produces identical output.

## Acceptance criteria

- Mapping instructions are reviewed ontology inputs, not runtime inference or generated advice.
- An agent can retrieve precise field-level instructions without receiving permission to mutate
  the ontology.
- Actionability is fail-closed and depends on review status plus structural completeness.
- The Brightflag example preserves its genuine blockers instead of claiming premature readiness.
- Generated JSON and OWL changes are explained and compiler-produced.
- `npm run check` and `git diff --check` pass.

This adds a governed transformation contract and is a meaningful product and governance decision.
If opening a pull request, apply `narrative-required` and include substantive Narrative Context,
Narrative Decision, and Narrative Consequences.
After that pull request merges, review and merge the separate draft Narrative proposal created by
automation. Leave the Narrative-only proposal unlabelled to prevent a recursive entry.

Commit locally with a focused message. Do not push.
