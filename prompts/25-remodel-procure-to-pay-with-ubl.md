# Prompt 25 — Remodel procure-to-pay with OASIS UBL alignments

Using the previously supplied repository contract and the accounting core from Prompts 23–24, add
the procure-to-pay semantic layer. Use OASIS UBL 2.4 as the primary document and process alignment
source, while keeping UBL messages separate from economic events and accounting recognition.

Read before editing:

- the migration plan and all current canonical/system relationships;
- the current Procurement, Billing, Brightflag, and related source snapshots;
- mapping review output and mapping instructions;
- official OASIS UBL 2.4 documentation, schemas, business objects, and process descriptions;
- the standards registry, lifecycle rules, architecture, and Narrative guidance.

Add or refine canonical concepts for at least:

- Purchase Requisition where supported;
- Purchase Order and Purchase Order Line;
- Purchase Commitment;
- Goods or Services Receipt;
- Invoice and Invoice Line;
- Credit Note and Debit Note;
- Accounts Payable Claim;
- Payment Terms;
- Tax, Allowance, and Charge;
- Remittance Advice;
- Supplier and Customer accounting roles.

Model the business chain explicitly:

```text
purchase order document
  -> records purchase commitment
  -> fulfilled by receipt/performance event
  -> gives rise to payable claim
  -> evidenced by invoice
  -> recognised by journal entry
  -> eligible for a separately governed settlement process
```

Do not assume:

- one purchase order produces one invoice;
- every invoice has a purchase order;
- an invoice proves receipt;
- an approved invoice has been posted;
- posting means payment approval;
- a payment instruction proves settlement;
- supplier, payee, and beneficiary are always the same party or role;
- tax treatment is universal across jurisdictions.

Use UBL `representation-of` alignments for message/document structures and exact or close semantic
mappings only when justified. Preserve UBL release and component identifiers. Do not copy the full
UBL schema set into the repository unless the owner has approved a versioned local snapshot and its
repository impact. The existing XSD adapter may process a reviewed subset or fixture, but neither
the compiler nor runtime may fetch OASIS resources.

Add governed predicates and cardinalities for order lines, receipts, invoice references, claims,
credits, and remittance. Make many-to-many order/invoice relationships possible. Represent
correction, cancellation, and credit without deleting historical business facts.

Keep the old direct canonical order-to-invoice relationship working during this additive stage,
but mark its future replacement path in lifecycle metadata and documentation. Do not yet remove it.
Do not approve the Brightflag invoice-to-payment mapping instruction.

Add tests covering:

- one invoice referencing multiple orders and one order referenced by multiple invoices;
- non-PO invoices;
- partial receipt and partial invoicing;
- credit notes and corrected invoices;
- supplier versus accounting supplier/payee role distinctions;
- UBL representation alignment not producing class equivalence;
- existing system mappings remaining review-governed;
- deterministic compiled JSON, OWL, and SHACL.

Update the architecture's reference explanation so it can show both the temporary direct edge and
the future evidence-bearing path without claiming that the migration is complete.

## Acceptance criteria

- Procure-to-pay semantics distinguish documents, commitments, events, claims, and recognition.
- UBL provides versioned document alignment without becoming the ontology's inheritance tree.
- Common exception paths are modeled without invented one-to-one cardinalities.
- Existing consumers remain functional pending Prompt 28 migration.
- No mapping instruction becomes actionable.
- `npm run check` and `git diff --check` pass.
- Generated artifact changes are compiler-owned and reviewed.

This is a meaningful product and semantic-model decision. Use `narrative-required` and substantive
Narrative sections if opening a pull request; review the separate Narrative proposal after merge.

Commit locally with a focused procure-to-pay message. Do not push.
