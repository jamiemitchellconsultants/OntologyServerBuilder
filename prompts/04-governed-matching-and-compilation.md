# Prompt 4 — Governed semantic matching and compilation

Using the previously supplied repository contract, implement Stage 4: semantic matching, review
governance, and deterministic compilation.

Build an explainable deterministic matcher between source entities and canonical concepts.

Use:

- normalized name similarity;
- description-token overlap;
- attribute-name overlap;
- finance synonym expansion, including PO/purchase order, AP/accounts payable, bill/invoice,
  remittance/payment, disbursement/payment, and supplier/vendor.

Use configurable thresholds in `ontology/config.json`:

- `highConfidenceThreshold`: `0.85`
- `reviewThreshold`: `0.55`

Use this suggested scoring:

- name: 65%;
- description: 20%;
- attributes: 15%;
- exact normalized names receive at least 0.95.

Each mapping candidate must include:

- source entity;
- proposed canonical entity when applicable;
- confidence;
- disposition;
- component-level reasons.

Apply these rules:

- Confidence at or above the high threshold becomes `accepted-auto`.
- Confidence at or above the review threshold becomes `review-required`.
- Lower confidence becomes `unmatched`.
- Review-required and unmatched mappings must not enter the runtime graph.
- Human decisions in `ontology/manual-mappings.json` override suggestions.
- Accepted human decisions become `accepted-human`.
- Rejected and confirmed-unmatched decisions remain excluded.

Build a compiler that writes deterministic:

- `ontology/compiled/ontology.json`
- `ontology/compiled/mapping-review.json`

Include a stable fingerprint derived only from governed inputs. Add a check command that fails if
checked-in compiled artifacts are stale.

Materialize accepted mappings as graph relationships and combine them with governed canonical
relationships.

Prove with tests that:

- Procurement PurchaseOrder connects to Billing Invoice.
- Billing Invoice connects to Treasury PaymentInstruction.
- A review-required mapping cannot be traversed.
- Manual acceptance activates a mapping.
- Repeated compilation produces byte-identical output.

Run `npm run check` and commit the stage locally.
