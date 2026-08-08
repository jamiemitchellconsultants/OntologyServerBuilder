# Prompt 46 — Add the accounts-payable mapping example

Using the Brightflag concepts and governed source example from Prompt 16, mapping instructions
from Prompt 17, the bounded intake analysis from Prompt 43, release visibility from Prompt 44,
and named mapping tools from Prompt 45, add one synthetic, reviewed accounts-payable example in
the separate `OntologyService` repository. Do not change `OntologyServerBuilder` in this stage.

This is a deterministic fixture and assurance example. It demonstrates how a qualified user can
register an invoice-management MCP catalog and an accounts-payable OpenAPI definition, how an
engineer uses governed evidence to add ontology and mapping-tool inputs, and how a user agent can
prepare a payment instruction. It must not add a live payment integration, production customer or
bank data, a route around intake or review, or a finance-specific rule to the general-purpose
service contract.

## Read before editing

Read Prompts 16, 17, 33–45, the approved qualified-user intake plan, `AGENTS.md`, Project
Narrative rules, the registered Brightflag source and compiled artifacts, governed mapping
instructions, intake submission and workbench code, release-manifest code, mapping-tool compiler
and evaluator, MCP authorization conventions, canonical JSON/schema helpers, and their tests.
When an engineer invokes registration, mapping-review, ontology-change, or named-tool workflows,
they must supply the applicable reviewed operator-template contents alongside that task. Those are
separate operator inputs, not target-repository artifacts, and this stage must neither read nor
create them. Adapt paths and identifiers to the target repository as it exists. Do not assume a
production accounts-payable system, a live endpoint, or that an intake proposal is an approved
ontology source. Prompt 16 supplies governed Brightflag OpenAPI source concepts only; it neither
supplies nor implies a Brightflag MCP export, vendor authenticity, or live discovery.

## Synthetic source fixtures and qualified-user registration

Create two complete, checked-in offline fixtures with explicit paths, canonical bytes, and recorded
lower-case SHA-256 digests: `fixtures/qualified-user-intake/synthetic-invoice-mcp-catalog.json`
and `fixtures/qualified-user-intake/synthetic-accounts-payable-openapi.json`. Repeated fixture
generation or canonicalization must produce byte-identical content and the recorded digest. They
have distinct synthetic provenance and source kinds (`mcp` and `openapi` respectively); neither
may claim Brightflag vendor authenticity or use a live locator.

The synthetic MCP catalog must be sufficient both for a system-registration proposal and the later
in-memory “pay invoice 1234” workflow. Define a synthetic server identity and version; an
explicit tool name for retrieving one invoice; bounded input and output JSON Schemas; an `Invoice`
entity with stable identifiers, invoice ID, payable status, approved gross total, currency, vendor
and pay-site-reference fields; and bounded example metadata needed by Prompt 33's registration
submission contract. Use a source kind of `mcp`, its own extractor/provenance, and the catalog's
own digest.
It may reuse governed field concepts and stable references from Prompt 16 solely for this
demonstration; it is a new synthetic artifact, not an exported Brightflag catalog.

The synthetic OpenAPI fixture has source kind `openapi`, its own provenance and digest, and uses
synthetic system, entity, operation, and identifier values. It must define a payment-instruction
request business schema, distinct from transport, authentication, and error wrappers, with at
least:

- invoice reference;
- idempotency key;
- beneficiary/pay-site reference;
- amount;
- ISO currency code; and
- requested execution date if the synthetic target requires one.

Define target requiredness, identifier format, validation rules, amount and currency constraints,
and idempotency behavior in the fixture and governed evidence. Require bounded synthetic values for
every payment-instruction business field, including invoice reference, idempotency key,
beneficiary/pay-site reference, amount, currency, and any required execution date. Do not include
a real endpoint, production payment data, credentials, access tokens, real or production
bank/payment-rail details, personal data, secrets, or live business records. Compilation, tests,
and runtime must not fetch either fixture, discover a catalog, or call an accounts-payable endpoint.

Provide deterministic, in-memory synthetic registration-flow tests for both fixtures. A qualified
user agent, using its own authorized access, reads the checked-in synthetic MCP catalog, extracts
bounded normalized metadata, and submits the registration using
`ontology_submit_system_registration`. The same agent reads the checked-in synthetic
accounts-payable OpenAPI fixture, extracts normalized metadata, and submits a separate
registration. Test that both fixtures exist, validate, retain distinct source kinds, provenance,
and digests, and that each submission is capability-gated by `ontology:propose`, returns only a
receipt to its submitter, and remains isolated intake evidence rather than an ontology fact.
Neither flow uploads raw artifact bytes, asks the server to retrieve a locator, accesses a live
system, or lets the qualified user list, export, or disposition pending intake.

## Engineer analysis, governed source change, and deployment boundary

Model the engineer workflow explicitly and entirely with the two checked-in synthetic artifacts.
An engineer with `ontology:intake:review` exports the two submissions, supplies those local
fixture paths explicitly, verifies their filename, size, format, source kind, provenance, and
digest, and runs the deterministic, embedding, and bounded LLM-advisory workbench paths. Keep
parser discrepancies, candidate evidence, semantic gaps, and disagreements visible; no matching
pass, coding agent, or intake disposition may accept ontology facts or a mapping automatically.

Require the engineer to use separately supplied registration and review operator instructions to
make a normal, reviewable ontology-source change. The change must add only facts supported by the
synthetic source and reviewed evidence, including an accounts-payable payment-instruction entity
and its required business attributes. Do not make pending intake content visible through the
delivery plane, and do not hand-edit generated ontology artifacts.

Before a mapping tool can be approved or activated, create, review, and approve a deterministic,
governed source-to-canonical normalization or equivalence mapping from the synthetic MCP Invoice
entity and its input schema to Prompt 16's governed Brightflag Invoice stable ID. Record both source
and target stable IDs, schema validation, reviewed field correspondence, provenance, lifecycle, and
failure contract in governed source. This adapter is not inferred from structural similarity, source
labels, or a matching score. Its output must be that exact Brightflag Invoice entity and schema,
which the later named mapping tool declares as its governed source. Stop if the exact Brightflag
Invoice stable ID, its source correspondence, schema validation, or approval is missing or
ambiguous. Another canonical source requires a separately reviewed mapping instruction and a
distinct governed mapping-tool identity and MCP name before activation.

Document and test the separate activation path: an engineer lead reviews the decision-bearing pull
request; normal CI compiles and validates it; the pull request merges; and CI/CD deploys the
compiled artifact and release manifest. Only then can `ontology_get_release_changes` or
`ontology://release/current`, authorized by `ontology:read`, show the new deployed systems and
semantic changes to a qualified user. A receipt, an engineer export, an analysis report, an
accepted intake disposition, or an unmerged pull request is not deployment and cannot expose a
new runtime entity or tool. Before any runtime example invokes the named mapping tool, assert that
the current deployed release delta classifies that tool as `added` or `changed`, as appropriate for
the compared deployment; an absent delta or another change kind blocks activation.

## Resolve mapping blockers before approving the mapping

Resolve every blocker retained by Prompt 17 with explicit synthetic source facts and reviewed,
governed evidence. The approved mapping must state:

- payable Brightflag invoice statuses and the canonical failure for every non-payable status;
- every required accounts-payable target field and its schema/format constraints;
- deterministic invoice-reference and idempotency-key derivation from the source invoice ID;
- exact currency-code and amount constraints, including approved gross-total source semantics;
- a reviewed beneficiary-selection rule; and
- the required supporting lookup input that identifies exactly one eligible pay site.

The lookup contract must state its cardinality and selection criterion. Zero eligible pay sites
returns `missing-input`; multiple eligible pay sites returns `ambiguous-lookup`; neither condition
may pick a default beneficiary. Invalid currency or target validation produces the declared
canonical failure. Do not approve the mapping if an identifier rule, payable status, target
requiredness, amount/currency rule, pay-site selection behavior, fixture evidence, or governed
review evidence is unresolved. Report the exact gap instead.

## Reviewed named mapping tool and deterministic examples

After all blockers are resolved and normal ontology changes have been reviewed and deployed, use
separately supplied named-mapping-tool operator instructions to add exactly this approved MCP tool:

`map_brightflag_invoice_to_accounts_payable_payment_instruction`

Its definition must reference the approved Brightflag-invoice-to-accounts-payable-payment-
instruction mapping instruction, the canonical governed Invoice entity and schema produced by the
approved synthetic-source adapter, specifically Prompt 16's Brightflag Invoice stable ID, target
entity ID, reviewed schemas, stable mapping-tool ID, ontology fingerprint/source context, semantic
version, provenance, and review record. The tool's governed source entity must exactly equal that
Brightflag Invoice stable ID and the canonical entity passed to it, never the raw synthetic MCP
entity. Keep the definition governed data for Prompt 45's restricted evaluator; do not add
executable expressions, callbacks, imports, templates, generated code, generic execution, or
dynamic source discovery.

The tool accepts one canonical governed Invoice and named supporting lookup records supplied by the
caller. It maps the invoice ID deterministically to the target invoice reference and idempotency
key, approved gross total to amount, validated invoice currency code to currency, and the exactly-
one reviewed pay-site reference to beneficiary. It returns Prompt 45's deterministic provenance
envelope and either a schema-valid target record or the appropriate structured failure. It is pure,
offline, deterministic, idempotent, and side-effect free: it does not retrieve invoice data, query
for pay sites, call an API, read mutable intake/release state, or access filesystem, network,
environment, clock, or randomness.

Add canonical synthetic examples and focused tests for:

- a payable invoice with exactly one eligible pay site;
- a non-payable invoice;
- a missing eligible pay site;
- multiple eligible pay sites;
- invalid currency;
- target validation failure where applicable; and
- semantically equivalent inputs with reordered object keys.

Also test synthetic-source-to-canonical adapter success, source-schema mismatch, canonical-schema
mismatch, ambiguous correspondence, and missing approved adapter. Each failure must stop before
mapping-tool invocation and before any accounts-payable call. Assert that the adapter target, the
mapping instruction source, the mapping-tool governed source, and the canonical record passed to
the tool all use Prompt 16's exact Brightflag Invoice stable ID.

Assert the exact deterministic output or structured failure for each. Prove no side effect or I/O
can occur during compilation, tool discovery, or invocation.

## End-to-end synthetic user workflow

Add an in-memory, offline assurance test for the instruction “pay invoice 1234”. The qualified
user agent must first authenticate with `ontology:read` and retain its own separate authorization
to the synthetic invoice MCP fixture and target accounts-payable API. Assert this exact runtime
order: fetch synthetic invoice **1234** and supporting pay-site records through the invoice-system
fixture; validate the raw synthetic source record against the catalog's bounded source schema;
invoke the approved pure deterministic source-to-canonical Invoice adapter; validate the resulting
canonical Invoice against its governed schema; then invoke
`map_brightflag_invoice_to_accounts_payable_payment_instruction` with that canonical record and
the lookup records. Validate the returned synthetic payment-instruction payload against the
accounts-payable request schema. Do not infer structural equivalence at any point.

The test stops there. It must assert that an accounts-payable HTTP invocation is a separate
user-agent responsibility, not an effect of mapping. If a future user-agent flow submits the
payload, it must obtain any required user approval, present the mapping provenance and target
details required by policy, supply the deterministic idempotency key, confirm the external result,
and record an audit event through the target-integration boundary. Retries and duplicate requests
must rely on the target's documented idempotency behavior; the mapping tool neither sends,
confirms, retries, nor audits an external payment. Do not perform a real or synthetic target HTTP
call in this workflow test.

## Acceptance criteria and verification

- Checked-in synthetic MCP-catalog and accounts-payable OpenAPI fixtures both validate, have
  canonical deterministic bytes and distinct source kinds, provenance, and digests. The catalog's
  bounded get-invoice schema and metadata support the invoice **1234** orchestration fixture.
- The synthetic accounts-payable OpenAPI snapshot separates business payment-instruction data from
  transport/error wrappers and supplies every fact needed to review the mapping.
- Qualified-user registration stays in the isolated intake plane; engineer analysis, review, CI/CD,
  and deployment remain the only route to delivery-plane ontology facts and mapping tools.
- The exact named mapping tool is approved only after every payable-status, target-contract,
  identifier, amount/currency, idempotency, beneficiary-selection, and source-to-canonical entity
  mapping blocker is resolved; its current deployed delta is `added` or `changed` before use.
- The adapter target, mapping-instruction source, mapping-tool governed source, and canonical input
  record all use Prompt 16's exact Brightflag Invoice stable ID. The pure mapping tool receives that
  canonical entity, deterministically maps invoice **1234** to a schema-valid target payload or a
  canonical structured failure, uses caller-provided lookup records, and has no I/O or side effect.
- The end-to-end fixture ends before an accounts-payable request; authorization, confirmation,
  idempotency, retries, and audit are explicit external user-agent/integration responsibilities.

Run the repository's full check (currently `npm run check`, if still provided), focused adapter,
intake authorization/isolation, workbench, compiler, mapping evaluator, MCP registration,
release-visibility, schema-validation, reordered-input determinism, and no-I/O tests, plus
`git diff --check`. Inspect generated artifacts and explain every legitimate compiler-produced
change. Confirm `ontology/compiled/` has no unreviewed manual edit. Commit locally with the
focused message `Add accounts-payable mapping example`. Do not push.

## Governance

This is a decision-bearing product, architecture, governance, and operational implementation.
Before merging a target-repository pull request, apply `narrative-required` together with
substantive `## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences`
sections in the same action, before merge. Never hand-edit, hand-merge, or otherwise author
generated `Narrative.md`; use a reviewed fragment and the target repository's generation process.
The resulting Narrative-only pull request must not have `narrative-required`, or it would
recursively create another entry.
