# Prompt 19 — Audit proposal safety without rejecting unusual systems

Using the previously supplied repository contract, harden and test the proposal workflows from
Prompt 18 while preserving their ability to represent unfamiliar but legitimate systems.

This is a focused assurance stage. Do not redesign the proposal tools, register another business
system, approve a mapping, add broad content blacklists, introduce stale-fingerprint enforcement,
or impose new restrictive payload limits.

Read before editing:

- `README.md`
- `docs/architecture.md`
- `docs/registering-systems.md`
- `docs/register-system-workflow.md`
- `docs/user-refinement-workflow.md`
- `CONTRIBUTING.md`
- `AGENTS.md`
- proposal input and output models;
- runtime store and MCP tool schemas;
- existing store-level and in-memory MCP tests;
- Project Narrative guidance.

Implement four assurance measures.

## 1. Canonicalize plain source filenames

Strengthen the source-filename boundary used by
`ontology_prepare_system_registration_request`:

- trim surrounding whitespace;
- normalize the filename to Unicode NFC;
- reject `/`, `\`, empty names, `.` and `..`;
- reject control characters, line/paragraph separators, and bidirectional
  embedding/override/isolate controls that can disguise the displayed filename;
- validate the declared source-format extension after normalization;
- use the normalized filename consistently in the proposal and suggested repository paths.

Do not restrict ordinary Unicode letters or require ASCII filenames. Do not reject punctuation
merely because it is unfamiliar when it cannot act as a path component or deceptive control.

## 2. Add a table-driven input matrix at both boundaries

Exercise the same policy directly against the store and through an in-memory MCP client. Structure
the cases as data so future contributors can add examples without copying test setup.

Tests must distinguish three outcomes:

1. **Accept** structurally safe information.
2. **Accept with review gaps or warnings** incomplete or unfamiliar business semantics.
3. **Reject** structurally incoherent or unsafe requests.

Include accepted or preserved cases for:

- non-English system names, entity names, descriptions, and filenames;
- supplier-specific datatypes;
- unfamiliar identifier and status formats;
- missing meanings or evidence that can be reported as gaps;
- XML, formulas, code fragments, or instruction-like wording in descriptions, transformations, or
  evidence.

Instruction-like text is untrusted inert data. Preserve it accurately in the review-required
proposal; never execute it or treat it as authority.

Include rejection cases for:

- forward- and backslash path components;
- control characters or bidirectional display controls in filenames;
- invalid or out-of-scope entity and attribute references;
- stable-ID collisions after normalization;
- duplicate relationship identities;
- malformed digests;
- relationships with missing endpoints or no proposed-system endpoint.

Do not turn an unfamiliar datatype, language, naming convention, or missing meaning into a
rejection merely because it requires owner interpretation.

## 3. Prove deterministic, read-only, no-I/O behavior

Add tests that:

- generate each proposal twice from semantically equivalent inputs supplied in different array
  orders and compare stable serialized output;
- serialize the compiled ontology before and after proposal generation and prove it is unchanged;
- install test sentinels on filesystem reads/writes and common network entry points, including
  `fetch`, HTTP/HTTPS, sockets, and TLS;
- call both proposal workflows while the sentinels are active and prove none were reached;
- perform the MCP cases through linked in-memory transports, never a live endpoint.

Compile or load the governed fixture before activating I/O sentinels. The assertion concerns
proposal generation, not the repository compiler's legitimate reading of checked-in source
snapshots.

Do not weaken the test by checking only the returned `proposedStatus`. The ontology state and
side-effect boundaries are separate invariants.

## 4. Document rejection versus preservation

Update the architecture and both proposal workflow documents to say explicitly:

- structural invalidity fails closed;
- unfamiliar but safe business metadata remains inert proposal data;
- supplier-specific datatypes and incomplete semantics become gaps, warnings, or questions;
- missing evidence does not prevent preparation of a review-required proposal;
- code- or instruction-like text is not executed;
- only ontology-owner review can translate preserved information into governed repository files.

Document the filename normalization and rejection rules. Avoid language claiming the service can
detect every credential, personal record, or malicious statement by inspecting arbitrary text.
The caller and ontology owner remain responsible for excluding prohibited record-level data.

Add or update tests that prove:

- Unicode filenames are normalized deterministically and ordinary Unicode remains accepted;
- path-bearing, control-character, and deceptive bidirectional filenames are rejected;
- unusual datatypes and instruction-like evidence are preserved with review-required status;
- equivalent reordered inputs produce byte-equivalent output;
- the compiled ontology is unchanged;
- filesystem and network sentinels receive no calls;
- store and MCP boundaries agree on accepted and rejected cases.

## Acceptance criteria

- The audit improves structural safety without introducing a general-purpose content filter.
- Genuine but unusual system definitions remain representable.
- Rejection, warning, and preservation behavior is explicit in code, tests, and documentation.
- Proposal generation has executable evidence for determinism, non-mutation, and no I/O.
- No generated ontology artifact changes unless an explained governed source change requires them.
- `npm run check` and `git diff --check` pass.

This establishes a meaningful security and governance policy. If opening a pull request, apply
`narrative-required` and include substantive Narrative Context, Narrative Decision, and Narrative
Consequences.
After that pull request merges, review and merge the separate draft Narrative proposal created by
automation. Leave the Narrative-only proposal unlabelled to prevent a recursive entry.

Commit locally with a focused message. Do not push.
