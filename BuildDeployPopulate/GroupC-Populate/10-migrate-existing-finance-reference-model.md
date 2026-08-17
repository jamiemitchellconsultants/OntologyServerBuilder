# Prompt C-10 — Migrate the existing finance reference model

Using the previously supplied repository contract and the standards-aligned layers implemented in
Prompts C-04–C-09, migrate the original four-concept finance reference model and its examples onto the
new semantics.

This is the compatibility-sensitive stage. Inspect every reference before editing and preserve
unrelated user changes.

Read before editing:

- the approved migration inventory from Prompt C-03;
- every governed ontology source, manifest, relationship, axiom, alignment, mapping decision, and
  mapping instruction;
- every test and document reference to the original canonical IDs and direct relationships;
- compiled artifacts for comparison only;
- runtime search, traversal, SPARQL, MCP response, fingerprint, and lifecycle behavior;
- Project Narrative requirements.

Preserve these stable IDs unless the Prompt C-03 review explicitly found an identity conflict:

- `canonical.purchase-order`;
- `canonical.invoice`;
- `canonical.payment-instruction`.

Refine their classifications and relationships:

- Purchase Order is a commercial document recording a purchase commitment.
- Invoice is an accounting/commercial document evidencing a claim; it is not itself the payable
  claim, receipt, economic event, or journal posting.
- Payment Instruction is an instruction/message authorising or requesting payment processing; it
  is not itself a payment transaction or settlement.

Remove the misleading subclass axioms that make all three merely subclasses of
`canonical.financial-document`. Either deprecate `canonical.financial-document` with a documented
replacement model or retain it only as a deliberately broad document category if that meaning
remains useful. Do not use it to connect unrelated concepts through reasoning.

Replace the direct canonical shortcuts:

```text
purchase order --is billed by--> invoice
invoice --is settled via--> payment instruction
```

with evidence-bearing paths through commitment, performance/receipt, claim, recognition,
instruction, transaction, and settlement. Handle migration according to the approved compatibility
policy:

- deprecate old predicates with replacement-path metadata when consumers may still reference
  them;
- exclude deprecated shortcuts from new relationship traversal;
- preserve historical provenance where needed;
- do not silently change an old predicate's meaning under the same ID.

Re-evaluate system-to-canonical mapping candidates against the richer model. Do not auto-accept a
different semantic target solely because the new label scores highly. Existing accepted mappings
may remain only when their meanings still match; otherwise move them through governed review.

Update the Brightflag invoice-to-payment instruction record:

- preserve its `review-required` status;
- preserve payable-status, beneficiary-resolution, and unregistered-target blockers;
- make explicit that an invoice cannot by itself prove an approved payable claim, payment
  eligibility, or settlement;
- reference the new semantic path and required evidence;
- do not promote it to approved;
- retain its stable instruction identity only if source and target identities are unchanged.

Update examples, architecture diagrams, refinement workflows, SPARQL examples, and tests so they
describe the richer path. Avoid implying that every hop represents a live transaction or that the
runtime performs accounting.

Regenerate all compiler-owned artifacts using `npm run ontology:compile`; never hand-edit them.
Review the mapping queue and explain every mapping disposition change.

Add migration tests proving:

- preserved public IDs still resolve;
- deprecated shortcuts are not selected for new business-path traversal;
- the new order-to-settlement explanation exposes each semantic layer and provenance;
- no shared superclass creates a false shortcut;
- review-required mappings remain non-traversable;
- historical lifecycle metadata remains queryable where designed;
- SPARQL and MCP examples use valid current predicates;
- fingerprints and generated output are deterministic.

## Acceptance criteria

- The original thin model has been migrated rather than duplicated indefinitely.
- Stable IDs are preserved where their identities remain valid.
- Documents, commitments, claims, postings, instructions, transactions, and settlements are no
  longer conflated.
- Deprecated relationships cannot support new runtime claims.
- The Brightflag instruction remains safely review-required with all blockers.
- `npm run check` and `git diff --check` pass.
- Working tree changes belong only to this migration.

This is a meaningful correction and semantic architecture change. If opening a pull request, apply
`narrative-required` and explain the old model's limitation, the compatibility choice, alternatives,
and consequences in all three required Narrative sections. Review the separate Narrative-only
proposal after merge without labelling it.

Commit locally with a focused migration message. Do not push.
