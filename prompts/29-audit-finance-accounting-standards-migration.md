# Prompt 29 — Independently audit the finance and accounting standards migration

Use this prompt in a fresh coding-agent task or, preferably, with a different coding agent after
Prompts 21–28 have been completed.

Audit the standards-aligned finance and accounting migration against the repository contract,
approved migration plan, governed inputs, generated artifacts, runtime behavior, and primary
standards sources. Begin read-only. Implement fixes only for concrete findings within this scope;
do not redesign the ontology or add another standard.

Read:

- all repository and agent instructions;
- the complete migration plan and standards/profile documentation;
- standards registry, alignments, lifecycle metadata, canonical sources, relationships, axioms,
  mappings, and mapping instructions;
- compiler, generated JSON/OWL/SHACL, runtime store, MCP tools, SPARQL restrictions, and tests;
- every Project Narrative entry relevant to the migration;
- official primary sources for each active standards release.

Audit these areas.

## 1. Semantic integrity

Prove that:

- party and role are distinct;
- document, commitment, event, claim, obligation, posting, instruction, transaction, settlement,
  and reporting fact are distinct;
- supplier, payee, beneficiary, debtor, and creditor are roles rather than permanently disjoint
  organisation types;
- payment instruction never implies settlement;
- invoice never implies receipt, posting, payment approval, or settlement;
- account never becomes equivalent to a reporting concept;
- shared superclasses cannot create misleading business paths.

## 2. Standards fidelity and restraint

For every alignment:

- verify the publisher, exact release, official URI, concept identifier, relation strength,
  effective date, licence status, and rationale;
- downgrade unjustified exact matches;
- ensure `representation-of` does not generate OWL class equivalence;
- ensure `informed-by` does not claim formal compliance;
- ensure no copied definition exceeds authorised use;
- ensure no FIBO, UBL, ISO, XBRL, IFRS, or FRC import is followed over the network.

Flag, rather than invent, any standard concept that cannot be verified.

## 3. UK and international reporting governance

Prove that:

- no accounting basis is inferred from a UK address or group membership;
- group, UK statutory, UK tax/filing, and overseas statutory profiles are separately selectable;
- profiles and mappings are effective-dated;
- historical periods retain their original taxonomy release;
- local taxonomy extensions remain distinguishable;
- unapproved account-to-reporting mappings cannot create facts;
- documentation does not present the service as accounting, tax, audit, or legal advice.

## 4. Migration and compatibility

Trace every original canonical ID, relationship, test, documentation example, mapping, and
instruction. Verify that:

- preserved IDs retain identity;
- deprecated IDs and predicates have rationale and replacements;
- deprecated shortcuts are excluded from new traversal;
- review-required mappings did not become accepted;
- no unexplained mapping-review disposition changed;
- runtime responses remain bounded and deterministic.

## 5. Security and operational boundaries

Prove with tests or code inspection that:

- compiler/runtime never fetch standards or source URLs;
- fixtures contain no transaction records, personal data, bank coordinates, credentials, or live
  company records;
- no external triple store, reasoner, model call, or filing service was introduced;
- SPARQL remains local `SELECT`/`ASK` with existing limits;
- HTTP authentication and host validation remain intact;
- generated artifacts are compiler-owned.

## 6. Executable verification

Run:

- focused ontology/compiler/runtime tests;
- `npm run ontology:compile`;
- `npm run check`;
- `git diff --check`;
- a clean regeneration comparison;
- repository-wide searches for stale `FinancialDocument`, direct order-to-invoice/payment
  shortcuts, unversioned standard names, and overclaims such as “compliant” or “certified”.

If changes are necessary, add regression tests for each repaired finding and regenerate artifacts.
Do not weaken validation or delete historical provenance to make the audit pass.

Create `docs/finance-accounting-standards-audit.md` recording:

- scope and date;
- commits or working-tree state inspected;
- evidence and commands;
- findings by severity;
- fixes made;
- accepted residual risks;
- owner decisions still outstanding;
- a standards-release upgrade checklist.

## Acceptance criteria

- Every active alignment is traceable to a verified, versioned source.
- The model passes the semantic-integrity assertions.
- UK/international profile selection is explicit and historically reproducible.
- Original IDs and relationships have a complete migration disposition.
- Trust boundaries and runtime guardrails are unchanged.
- `npm run check` and `git diff --check` pass.
- Generated artifacts have no unexplained diff.

Classify the audit and any fixes under the repository's Narrative rules. A finding that changes
product, architecture, governance, or previously shipped meaning requires `narrative-required` and
substantive Narrative sections. A purely evidentiary audit with no meaningful decision may be
mechanical. Never edit generated `Narrative.md` directly.

Commit locally with a focused audit message. Do not push.
