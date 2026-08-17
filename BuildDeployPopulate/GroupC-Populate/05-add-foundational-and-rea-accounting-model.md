# Prompt C-05 — Add FIBO-aligned foundations and a REA accounting core

Using the previously supplied repository contract and the governed standards mechanism from Prompt C-04, add the foundational and accounting-economic concepts needed by a finance and accounting
department. Add them alongside the current canonical entities; do not migrate or remove the old
relationships yet.

Read before editing:

- the migration plan and standards registry/alignment implementation;
- the current canonical source, relationships, OWL axioms, compiled output, matcher, and tests;
- official FIBO production documentation and the OMG Commons modules used by that release;
- the official abstract and legally available material for ISO/IEC 15944-4;
- the repository's supported OWL profile and Narrative rules.

Use FIBO selectively for shared financial-business meaning and identifiers. Do not import the FIBO
master ontology or its transitive import closure. ISO/IEC 15944-4 is a conceptual influence: unless
the owner supplies licensed machine-readable material, record `informed-by` provenance and do not
reproduce protected definitions or claim formal compliance.

Add canonical concepts, with precise descriptions and only necessary attributes, for at least:

- Party;
- Person and Organisation where required to distinguish roles safely;
- Legal Entity;
- Party Role;
- Identifier and Identifier Scheme;
- Currency and Monetary Amount;
- Economic Agent;
- Economic Resource;
- Economic Event;
- Commitment;
- Economic Claim;
- Obligation;
- Economic Agreement.

Model roles separately from parties. A supplier, customer, debtor, creditor, beneficiary, payer,
and payee are contextual roles; they are not necessarily disjoint kinds of legal entity.

Model monetary amount as an amount plus currency and applicable context. Do not make currency a
free-text semantic identity merely because source systems carry a string code. Align currency codes
to ISO 4217 by governed identifier scheme without copying the ISO list unless redistribution has
been approved.

Add governed relationship predicates sufficient to express:

- a party playing a role;
- an agent participating in an event;
- an event affecting a resource;
- a commitment involving parties and promising a future event;
- a claim against an obligated party;
- an obligation satisfying a claim;
- reciprocal or dual economic events;
- a document evidencing, recording, or authorising something without becoming that thing.

Choose cardinalities conservatively. Do not assert disjointness, inverse functionality, uniqueness,
or universal restrictions that the current source evidence cannot support. Keep legal ownership,
economic control, custody, and operational responsibility distinct.

Add reviewed FIBO/Commons alignments only where the exact external term and release have been
verified. Prefer `close-match` where the canonical concept is deliberately simpler. Use
`informed-by` for the REA/OeBTO pattern unless a stronger mapping is justified and reviewable.

Do not:

- alter `canonical.purchase-order`, `canonical.invoice`, or
  `canonical.payment-instruction` semantics yet;
- classify all new concepts under `FinancialDocument`;
- add jurisdiction-specific reporting concepts;
- add transaction instances or real people/companies;
- activate new mappings based only on label similarity.

Add SHACL shapes for structural constraints supported by the canonical model, including monetary
amount/currency pairing and valid internal references. Do not encode accounting-policy judgments as
universal SHACL constraints.

Add tests proving:

- roles and parties remain distinct;
- a document can evidence a claim without becoming a claim;
- a commitment, event, claim, and obligation are not inferred equivalent;
- selective external alignments do not pull external imports;
- reasoning terminates deterministically under the supported OWL subset;
- existing system mappings and relationship explanations remain unchanged in this additive stage.

Update architecture documentation with a small example showing commitment, performance, claim,
recognition, and settlement as separate layers.

## Acceptance criteria

- The ontology has a usable accounting-economic foundation without importing all of FIBO.
- REA-inspired distinctions are explicit and evidence-bearing.
- Party roles, legal entities, documents, and economic events are not conflated.
- The change is additive and does not break the current reference path.
- `npm run check` and `git diff --check` pass.
- Generated artifacts are compiler-produced and reviewed.

This is a meaningful semantic architecture decision. If opening a pull request, apply
`narrative-required`, complete the three Narrative sections, and later review the separate
Narrative-only proposal without labelling it.

Commit locally with a focused ontology-foundation message. Do not push.
