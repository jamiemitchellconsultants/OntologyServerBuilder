# Prompt 16 — Register a real invoice system

Using the previously supplied repository contract, register a real invoice system from a public
OpenAPI description. Use Brightflag as the worked example and preserve enough governed detail for
a later agent to construct a payment instruction from a Brightflag invoice.

Read before editing:

- `README.md`
- `docs/architecture.md`
- `docs/registering-systems.md`
- `CONTRIBUTING.md`
- `AGENTS.md`
- the ontology source parsers, manifests, compiler, matcher, and tests;
- existing ontology sources, manual mappings, generated review output, and compiled artifacts;
- the public Brightflag OpenAPI description at
  `https://app.brightflag.com/v3/api-docs/external`.

Treat the remote OpenAPI document as onboarding input only. Save a reviewed, deterministic source
snapshot or a curated normalized extract under `ontology/sources/`; compilation and runtime must
never fetch it. Record the source URL, retrieval date, source version when available, and the
curation decisions. Do not store credentials, live API responses, personal data, invoice instances,
or payment details.

Do not indiscriminately turn every schema into a governed business entity. Distinguish:

- business entities and their durable attributes;
- transport request and response wrappers;
- pagination, validation, and error schemas;
- identity-provisioning or SCIM schemas;
- deprecated or compatibility-only API shapes.

Register the business entities required to understand invoice approval and payment preparation.
The worked example should consider at least:

- invoice and invoice payment status;
- accounts-payable batch;
- vendor, vendor office, and pay site;
- trading entity;
- purchase order and purchase-order line item;
- matter;
- tax rate.

Preserve stable source attribute IDs, types, descriptions, requiredness, allowed values, and
provenance. Pay particular attention to fields needed for a future transformation, including:

- invoice ID, group ID, invoice number, status, approved gross total, and currency;
- vendor, matter, purchase-order, and trading-entity references;
- the identifiers and governed relationships joining vendor to office and office to pay site;
- the source descriptions of any bank-routing or beneficiary-identification schema, without
  including transaction data or asserting unsupported payment semantics.

Add explicit relationship declarations only where the source material supports them. Keep
uncertain semantic matches in review output; do not promote them merely to make the example
traversable. Preserve confidence, provenance, disposition, inference rule, and inference premises.

Compile the ontology and review:

- entity and attribute extraction;
- candidate canonical mappings;
- relationships and relationship paths;
- generated JSON, OWL, SHACL, and mapping-review artifacts;
- deterministic ordering and source fingerprints.

Add tests that prove:

- the checked-in snapshot compiles without network access;
- transport-only schemas are not exposed as governed business entities;
- the key invoice and beneficiary-resolution attributes retain their descriptions and provenance;
- declared relationship endpoints exist and unsupported links remain non-traversable;
- repeated compilation produces identical artifacts;
- malformed or incomplete OpenAPI-derived source data is rejected clearly.

Document the distinction between retrieving a specification during registration and serving the
compiled ontology at runtime.

## Acceptance criteria

- Brightflag is represented as a governed system using stable entity IDs such as
  `brightflag.<entity-slug>`.
- The ontology preserves the invoice, amount, currency, status, and vendor-to-pay-site information
  needed to design a later payment-instruction mapping.
- No live Brightflag endpoint is called by compilation, tests, or runtime.
- No credentials, transaction instances, or personal data are checked in.
- Generated files were produced by the compiler rather than edited by hand.
- `npm run check` and `git diff --check` pass.

This registration establishes governed product data and may include semantic decisions. If opening
a pull request, classify the change under the repository's Project Narrative rules and apply
`narrative-required` with substantive Narrative sections when the change is decision-bearing.
After that pull request merges, review and merge the separate draft Narrative proposal created by
automation. Leave the Narrative-only proposal unlabelled to prevent a recursive entry.

Commit locally with a focused message. Do not push.
