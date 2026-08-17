# Prompt C-08 — Remodel treasury, payments, and settlement with ISO 20022 alignments

Using the previously supplied repository contract and the finance/accounting layers from Prompts C-05–C-07, add treasury and payment semantics aligned selectively to the ISO 20022 business model and
message catalogue.

Read before editing:

- the migration plan, current Treasury and ERP source snapshots, canonical payment instruction,
  payable relationships, and mapping instruction;
- the standards registry and semantic alignment behavior;
- current official ISO 20022 methodology, repository, business areas, and relevant Payments and
  Cash Management message definitions;
- architecture, security, sensitive-data restrictions, and Narrative guidance.

Pin exact ISO 20022 repository or message-definition releases used as evidence. Message versions
change independently, so never align only to an unversioned label such as `pain.001`. Record the
complete message definition identifier where a message alignment is intended.

Add or refine canonical concepts for at least:

- Payment Instruction;
- Payment Transaction;
- Payment Batch or Group;
- Payer/Debtor role;
- Payee/Creditor role;
- Ultimate payer/payee where needed;
- Beneficiary role;
- Cash Account and Servicing Institution;
- Remittance Information;
- Payment Status;
- Clearing;
- Settlement;
- Cash Entry and Bank Statement;
- Payment Return, Reversal, Rejection, and Cancellation.

Make the lifecycle explicit:

```text
approved payable claim
  -> settlement obligation
  -> payment instruction
  -> one or more payment transactions
  -> clearing/settlement event
  -> cash entry and reconciliation evidence
```

An instruction authorises or requests processing; it does not prove execution. A status report,
bank statement entry, and settlement event are different evidence. Preserve partial payment,
netting, batch payments, fees, foreign exchange, returns, and reversals without overwriting the
original instruction or claim.

Keep account identity separate from sensitive account coordinates. Ontology fixtures may describe
that a field is an IBAN, domestic account identifier, or BIC, but must not contain real account
numbers, names, transaction references, or bank messages. Do not add validation that requires
secret or personal data.

Use `representation-of` for XML message structures. Use exact/close mappings only for verified
business concepts in a pinned release. Do not download ISO packages during build/runtime or claim
ISO 20022 conformance without a schema/conformance implementation.

Add governed relationships for:

- instruction containing transactions;
- transaction intended to settle one or more claims;
- settlement satisfying a claim in full or part;
- cash entry providing evidence for reconciliation;
- return or reversal referring to the original transaction;
- parties playing contextual debtor, creditor, payer, payee, and beneficiary roles.

Do not migrate the Brightflag instruction yet, but add the semantic path it will require in Prompt C-10. Preserve every unresolved payable-status, beneficiary-resolution, and target-system blocker.

Add tests for:

- one instruction with multiple transactions;
- multiple invoices/claims settled by one transaction and partial settlement of one claim;
- rejected instruction versus returned settled payment;
- reversal retaining the original event;
- payer/payee/beneficiary distinctions;
- no inference from instruction to settlement;
- no real bank identifiers or external network calls;
- deterministic RDF/SHACL and existing runtime compatibility.

## Acceptance criteria

- Payment instruction, transaction, clearing, settlement, cash entry, and reconciliation are
  distinct concepts.
- ISO 20022 alignments are release-specific and do not overclaim conformance.
- Multi-claim, partial, returned, and reversed payments are representable.
- Sensitive banking and transaction data remain outside fixtures and runtime ontology facts.
- Existing review-required mapping blockers are preserved.
- `npm run check` and `git diff --check` pass.

This changes the treasury semantic model and requires the normal decision-bearing Narrative
workflow if published. Review the generated Narrative-only proposal separately after merge.

Commit locally with a focused treasury-model message. Do not push.
