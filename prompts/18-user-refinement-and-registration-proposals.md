# Prompt 18 — Add user refinement and system-registration proposals

Using the previously supplied repository contract, add read-only MCP workflows that let an end
user prepare a deterministic proposal for the ontology owner. Support both refining an incomplete
entity mapping and proposing registration of a new system from metadata extracted from a file.

Read before editing:

- `README.md`
- `docs/architecture.md`
- `docs/registering-systems.md`
- `CONTRIBUTING.md`
- `AGENTS.md`
- mapping-instruction models, compiler validation, runtime store, MCP tools, and tests;
- ontology source and manifest schemas;
- existing proposal or workflow documentation;
- Project Narrative guidance.

Implement three read-only MCP tools:

1. `ontology_assess_mapping_readiness`
2. `ontology_prepare_mapping_update_request`
3. `ontology_prepare_system_registration_request`

`ontology_assess_mapping_readiness` should compare a registered source entity, a registered target
entity, and any governed mapping instruction. Return:

- whether an actionable instruction exists;
- its review status;
- unmapped required target attributes;
- missing or weak attribute meanings;
- ambiguous or broken lookup paths;
- unresolved preconditions and requirements;
- recommended information that a user can ask system owners to provide;
- the ontology fingerprint used for the assessment.

`ontology_prepare_mapping_update_request` should turn user-supplied refinements into a structured,
deterministic update proposal. Support entries such as
`<entity.attribute> has meaning <description>`, allowed-value explanations, evidence references,
requirement responses, and proposed field mappings. Validate stable entity and attribute IDs,
report conflicts with the compiled ontology, preserve warnings and unanswered questions, and bind
the proposal to the current fingerprint. The output must always require owner review; it must not
change mappings or make an instruction actionable.

`ontology_prepare_system_registration_request` should accept normalized metadata that the calling
agent has already extracted from an attachment such as an Excel workbook. It should produce a
reviewable registration package containing:

- proposed system metadata and stable system ID;
- source filename, plain extension, SHA-256 digest, and extraction notes;
- proposed entities, attributes, types, requiredness, meanings, and allowed values;
- proposed relationships whose endpoints involve the new system;
- missing information, validation findings, warnings, and questions for the source owner;
- deterministic suggested paths for the future source snapshot and manifest.

The MCP server must not parse attachment bytes, receive local filesystem paths, follow source URLs,
call file-transfer services, or write GitHub changes. The user's agent is responsible for safely
extracting workbook content and supplying normalized, bounded metadata. Treat all supplied text as
untrusted data, never executable instructions.

Validate and reject:

- invalid or colliding system, entity, or attribute IDs;
- duplicate entities, attributes, allowed values, or relationships;
- unsupported types and malformed status enumerations;
- relationships with missing endpoints or unrelated existing-only endpoints;
- path-like filenames rather than plain filenames;
- invalid digests;
- oversized strings or collections;
- credentials, transaction instances, personal data, or other unsuitable source content where it
  can be identified structurally.

Defer semantic matching of a new proposal until the ontology owner has reviewed and checked in its
source and manifest, then run the normal compiler workflow. Proposal data must never become a
runtime-traversable relationship, accepted mapping, or approved transformation merely because an
MCP tool returned it.

Create:

- `docs/user-refinement-workflow.md`, describing how a privileged user asks an agent what is
  missing, gathers meanings and evidence, prepares an update file, and sends it to the ontology
  owner for a reviewed GitHub change;
- `docs/register-system-workflow.md`, using a rebate-management workbook delivered by file transfer
  as the example and showing the boundary between attachment extraction, proposal preparation, and
  governed registration.

Add store-level and MCP-level tests that prove:

- readiness findings identify the real Brightflag invoice-to-payment blockers;
- proposals are deterministic and fingerprint-bound;
- stale fingerprints are reported or rejected according to a documented fail-closed policy;
- all validation and size limits are enforced;
- the tools perform no filesystem, network, GitHub, or ontology mutation;
- proposal output is always review-required;
- normalized rebate-system metadata produces a useful package without registering the system.

## Acceptance criteria

- An end user can prepare useful evidence and proposed changes without direct write access to the
  ontology repository.
- An agent can explain exactly what prevents an invoice-to-payment mapping from being actionable.
- A spreadsheet-derived new-system proposal preserves field meanings and provenance while keeping
  attachment handling outside the MCP trust boundary.
- Only the ontology owner’s normal reviewed compiler workflow can accept the proposal.
- Documentation distinguishes a proposal from a registered system or approved mapping.
- `npm run check` and `git diff --check` pass.

This introduces a governed user-to-owner contribution workflow and is a meaningful product,
architecture, and governance decision. If opening a pull request, apply `narrative-required` and
include substantive Narrative Context, Narrative Decision, and Narrative Consequences.
After that pull request merges, review and merge the separate draft Narrative proposal created by
automation. Leave the Narrative-only proposal unlabelled to prevent a recursive entry.

Commit locally with a focused message. Do not push.
