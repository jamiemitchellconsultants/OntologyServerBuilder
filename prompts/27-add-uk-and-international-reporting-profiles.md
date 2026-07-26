# Prompt 27 — Add UK and international accounting-reporting profiles

Using the previously supplied repository contract and the operational accounting model from
Prompts 23–26, add governed, effective-dated reporting profiles for a UK-based group with
international operations.

The universal semantic core must remain independent of a reporting regime. A reporting profile
states how a legal entity or consolidation context reports for a period; it does not change the
identity of invoices, payments, accounts, or economic events.

Read before editing:

- the Prompt 21 reporting strategy and unresolved owner decisions;
- standards registry, alignments, accounting entity, period, currency, ledger, and consolidation
  concepts;
- official current IFRS Foundation taxonomy material;
- official FRC Taxonomy Suite documentation for UK IFRS, FRS 101, FRS 102, and UKSEF;
- current HMRC accepted-taxonomy and Companies House filing guidance;
- XBRL specifications for concepts, facts, contexts, units, dimensions, calculations, and taxonomy
  packages;
- any applicable group accounting-policy decision supplied by the owner;
- Narrative and licensing guidance.

As a July 2026 baseline, verify that:

- the IFRS Accounting Taxonomy 2025 remains current for 2026 reporting;
- the FRC 2026 Taxonomy Suite is the current UK suite;
- HMRC accepts particular taxonomy versions according to filing periods;
- UK entity accounting bases are not interchangeable.

Create a governed reporting-profile model supporting:

- profile ID and title;
- reporting framework and exact taxonomy/entry-point release;
- jurisdiction and collector;
- accounting basis;
- legal entity or consolidation scope;
- effective period and comparative treatment;
- functional and presentation currencies;
- lifecycle status;
- local extensions;
- provenance, approval, and owner;
- supersession without rewriting historical periods.

Add canonical reporting concepts sufficient to distinguish:

- Reporting Concept;
- Reportable Fact;
- Reporting Context;
- Period;
- Unit;
- Dimension and Member;
- Presentation and Calculation Relationship;
- Disclosure;
- Accounting Policy;
- Consolidation Scope;
- Intercompany Elimination;
- Taxonomy Extension.

Do not load full IFRS or FRC taxonomy packages unless reviewed local packages and redistribution
rights have been supplied. The implementation must still be useful with reference-only registry
entries and a small synthetic test taxonomy. If packages are supplied, ingest them only through a
bounded, deterministic owner-side adapter, pin their hashes, and never fetch imports.

Support at least these inactive-until-selected profile templates:

- group IFRS reporting;
- UK statutory UK IFRS;
- UK FRS 101;
- UK FRS 102;
- overseas local statutory reporting.

Do not activate a template for a legal entity merely because it is incorporated in the UK. Require
an explicit governed selection with an effective period. Do not encode tax or accounting advice in
the ontology.

Model mappings from operational accounts to reporting concepts as separate governed, effective-
dated mapping records with:

- source account or account class;
- target taxonomy concept;
- legal entity/consolidation scope;
- dimensions and sign/conversion policy;
- evidence and owner;
- lifecycle and review status.

Unapproved mappings must not produce reportable facts. The runtime must not calculate accounts,
prepare filings, perform currency conversion, or determine compliance.

Add tests proving:

- two legal entities can use different statutory profiles for the same period;
- group and local profiles coexist without merging concept identities;
- expired taxonomy mappings remain available for historical periods but not new ones;
- unselected profile templates are non-actionable;
- local extension concepts do not masquerade as base-taxonomy concepts;
- operational account-to-reporting mappings require approval and effective dates;
- no reporting package is fetched;
- deterministic compilation and bounded runtime output.

Update documentation with a UK parent/overseas subsidiary example and a clear statement that the
ontology assists semantic interoperability but does not replace accountant, auditor, tax, or
filing-software judgment.

## Acceptance criteria

- UK IFRS, FRS 101, FRS 102, group IFRS, and overseas local profiles can be represented without
  conflation.
- No accounting basis is selected implicitly.
- Taxonomy versions and mappings are effective-dated and historically reproducible.
- Reporting mappings remain governed and non-actionable until approved.
- `npm run check` and `git diff --check` pass.
- Generated changes are deterministic and explained.

This is a meaningful reporting-governance decision. Use the full `narrative-required` workflow for
a pull request and review the later Narrative-only proposal separately.

Commit locally with a focused reporting-profile message. Do not push.
