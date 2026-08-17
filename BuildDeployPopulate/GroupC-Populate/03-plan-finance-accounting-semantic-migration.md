# Prompt C-03 — Plan the finance and accounting semantic migration

Using the previously supplied repository contract, inspect the current OntologyService and create a
reviewable migration plan from its thin finance reference model to a standards-aligned finance and
accounting semantic layer.

This is an orientation and decision-recording stage. Do not change ontology sources, mappings,
axioms, compiled artifacts, runtime behavior, or MCP contracts in this prompt.

The target organisation is a UK-based group with international operations. Its semantic core must
work across legal entities and jurisdictions. Reporting requirements belong in explicit profiles;
they must not redefine universal operational concepts.

Read before editing:

- `README.md`, `AGENTS.md`, `CONTRIBUTING.md`, and `Narrative.md`;
- `docs/architecture.md`, `docs/registering-systems.md`, `docs/owl-profile.md`, and relevant
  refinement workflows;
- every governed ontology input and system manifest;
- generated mapping review output, without editing it;
- compiler models, validation, deterministic sorting, runtime traversal, and related tests;
- official primary-source documentation for FIBO, ISO/IEC 15944-4, XBRL and XBRL Global Ledger,
  OASIS UBL, ISO 20022, the IFRS Accounting Taxonomy, the FRC UK Taxonomy Suite, and HMRC taxonomy
  acceptance.

Use current official sources at execution time. Record access dates and exact releases. Pin FIBO
for this migration to the Q1 2026 production release, tag `master_2026Q1`, published 20 April 2026.
Resolve and record the tag's full commit SHA plus hashes for any selected local modules; never
follow the moving `master` branch. If a later production release exists when this prompt is run,
report it for a separate governed upgrade decision but do not silently replace the pinned Q1 2026
baseline.

As a July 2026 starting point, also verify rather than blindly assume:

- FIBO Q1 2026 includes high-level chart-of-accounts and expanded ledger-account concepts relevant
  to this migration;
- the 2025 IFRS Accounting Taxonomy remains current for 2026 reporting;
- the FRC 2026 Taxonomy Suite contains UK IFRS, FRS 101, FRS 102, UKSEF, Irish, and Charities entry
  points;
- HMRC determines which taxonomy releases it accepts for a filing period;
- UBL 2.4 is the current OASIS Standard;
- ISO/IEC 15944-4:2015 remains published but is under systematic review;
- ISO 20022 repository and message-set releases may change independently.

Do not infer the accounting basis solely from the company being based in the UK. UK parent and
subsidiary entities may use UK-adopted IFRS, FRS 101, FRS 102, or another permitted basis. Overseas
entities may have local statutory profiles while group consolidation uses a separate group policy.
Record accounting-basis selection as an explicit governed decision.

Create `docs/finance-accounting-semantic-migration.md` containing:

1. An inventory of the current canonical entities, attributes, subclass axioms, relationships,
   mappings, mapping instructions, documentation examples, tests, and public IDs affected.
2. A target layered model:
   - foundations and parties;
   - REA-inspired accounting economics;
   - ledger and journal;
   - procure-to-pay and order-to-cash;
   - treasury, cash, payment, and settlement;
   - external and statutory reporting profiles.
3. A standards responsibility matrix explaining where each standard is authoritative, useful only
   as an alignment source, or intentionally out of scope.
4. An explicit distinction between:
   - an economic event;
   - a commitment;
   - a claim or obligation;
   - a commercial/accounting document;
   - a ledger recognition or posting;
   - an instruction/message;
   - an executed transaction and settlement;
   - a reportable accounting fact.
5. A compatibility inventory for `canonical.financial-document`, `canonical.purchase-order`,
   `canonical.invoice`, `canonical.payment-instruction`, the two direct canonical relationships,
   and the Brightflag invoice-to-payment instruction.
6. A proposed stable-ID policy. Preserve useful public IDs where their essential meaning remains
   valid; do not preserve an ID if doing so would silently change its identity.
7. A reporting-profile strategy for:
   - group accounting policy;
   - UK statutory entities;
   - UK tax and Companies House tagging;
   - overseas statutory entities;
   - consolidation and intercompany elimination;
   - effective dates, comparatives, and taxonomy upgrades.
8. A source-acquisition and licensing table. Distinguish freely reusable machine-readable
   standards from copyrighted or access-controlled specifications. Record FIBO's pinned
   `master_2026Q1` release, full resolved commit SHA, MIT licence, selected modules, and snapshot
   hashes. Do not copy ISO text or redistribute taxonomy packages without confirmed rights.
9. A staged execution plan corresponding to Prompts C-04–C-11, including rollback points and the
   expected generated-artifact impact of each stage.
10. Open decisions requiring an accountant, tax owner, treasury owner, or ontology owner.

The plan must recommend selective alignment rather than wholesale ontology imports. It must retain
the repository boundaries:

- reviewed, local, version-pinned source material only;
- no build-time or runtime URL fetching;
- no runtime reasoning beyond the supported bounded profile;
- no external triple store;
- no unreviewed mapping becoming traversable;
- no transaction records, bank details, personal data, or credentials in fixtures.

Add documentation checks or link checks only if the repository already has a suitable local
mechanism. Do not add a new documentation framework for this stage.

## Acceptance criteria

- The existing semantic debt and every affected stable ID are enumerated.
- The target architecture clearly separates documents, economic reality, accounting recognition,
  payment execution, and reporting.
- UK and international reporting profiles are explicit and effective-dated.
- The document does not incorrectly declare the group to use IFRS, UK-adopted IFRS, or FRS 102
  without a governed owner decision.
- Standards versions, provenance, licensing, and conformance limitations are visible; FIBO is
  pinned to `master_2026Q1` rather than a moving branch.
- No ontology or generated artifact changes.
- `npm run check` and `git diff --check` pass.

This plan records a meaningful product, architecture, and governance direction. If opening a pull
request, apply `narrative-required` and include substantive Narrative Context, Narrative Decision,
and Narrative Consequences. After merge, review the separate Narrative proposal without applying
`narrative-required` to that Narrative-only pull request.

Commit locally with a focused documentation message. Do not push.
