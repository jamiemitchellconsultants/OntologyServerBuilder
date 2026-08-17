# Prompt C-06 — Add ledger and journal semantics informed by XBRL Global Ledger

Using the previously supplied repository contract and the accounting foundation from Prompt C-05,
add a governed operational accounting and ledger layer. Use XBRL Global Ledger as a semantic and
interchange reference, not as the service's native storage format.

Read before editing:

- the migration plan, accounting core, standards registry, and alignment rules;
- canonical sources, relationships, mapping instructions, SHACL generation, and tests;
- official XBRL specifications and the published status of XBRL Global Ledger modules;
- documentation of the chosen XBRL GL release and its licence;
- Project Narrative requirements.

Add canonical concepts for at least:

- Chart of Accounts;
- Account;
- Ledger and Subledger;
- Journal;
- Journal Entry;
- Posting or Journal Line;
- Accounting Period;
- Accounting Date;
- Accounting Entity;
- Business Unit;
- Cost Centre;
- Debit and Credit Direction;
- Account Balance;
- Source Document Reference;
- Exchange Rate and Currency Context;
- Reconciliation;
- Consolidation Adjustment and Intercompany Elimination.

Keep these distinctions explicit:

- an account is not a reporting taxonomy concept;
- a journal entry recognises or adjusts an accounting effect; it is not the source economic event;
- a posting is part of an entry and affects an account;
- a source document reference is evidence, not the transaction itself;
- an operational balance and a reported fact may use different dimensions, policy, currency, and
  effective period.

Model multi-entity and multi-currency operations. Support transaction currency, functional
currency, and presentation currency as contextual roles rather than three unrelated strings.
Exchange-rate type, source, and effective time must be governed evidence; do not infer or fetch a
rate.

Represent debit/credit direction explicitly and avoid assuming that debit always means an increase.
Its effect depends on the account classification. Add only those balance constraints that can be
validated from a complete entry and declared currency context.

Add SHACL or deterministic compiler validation for structural rules such as:

- an entry contains postings;
- a posting identifies an account, amount, currency context, and debit/credit direction;
- a complete balanced entry balances under its declared balancing scope;
- required period/entity references resolve;
- a consolidation adjustment identifies its consolidation context;
- an intercompany elimination retains both sides and provenance.

Do not reject incomplete source-system schemas merely because they cannot provide a balanced
instance. Shapes describe governed structure; this runtime stores ontology metadata, not ledger
transactions. Test with synthetic schema metadata and tiny fictitious examples, never real entries.

Add XBRL GL alignments only after verifying exact concepts in the pinned release. Mark broader
canonical abstractions as `close-match` or `informed-by`. Do not claim that the service validates
XBRL GL instances unless an actual conformant parser and conformance suite are implemented, which
is outside this prompt.

Add tests proving:

- deterministic validation and RDF/SHACL projection;
- debit/credit direction is not treated as positive/negative arithmetic;
- account and reporting concept remain distinct;
- functional and presentation currency are not conflated;
- intercompany and consolidation concepts retain legal-entity and period context;
- no live rates, ledgers, or external taxonomy URLs are accessed;
- existing ontology behavior remains compatible.

Update architecture and registration guidance with examples of mapping an ERP journal schema and
chart of accounts without importing transaction data.

## Acceptance criteria

- The service can describe ledger structure and accounting recognition across systems.
- XBRL GL is a pinned alignment source, not an unqualified conformance claim.
- Multi-entity, multi-currency, consolidation, and reconciliation contexts are represented.
- No real financial records or rates enter the repository.
- `npm run check` and `git diff --check` pass.
- Generated artifacts are current and explainable.

This is a meaningful product and ontology decision. Follow the `narrative-required` workflow for a
pull request and review the later Narrative-only proposal separately.

Commit locally with a focused ledger-model message. Do not push.
