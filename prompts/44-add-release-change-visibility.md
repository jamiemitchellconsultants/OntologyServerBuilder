# Prompt 44 — Add deployment release-change visibility

Using the qualified-user intake submission path from Prompt 35, add deterministic current-release
change visibility in the separate `OntologyService` repository. Do not change
`OntologyServerBuilder` in this stage. A qualified user can inspect the delta in the deployment
they are using and submit a further correction proposal, but this stage does not make a proposal
part of the ontology or expose pending intake information.

## Read before editing

Read Prompts 33–35 and 43, the approved qualified-user intake plan, `AGENTS.md`, Project Narrative
rules, compiled ontology and fingerprint code, CI/CD deployment configuration, MCP registration and
resource conventions, capability authorization, the two intake submission handlers, canonical JSON
and digest helpers, and their tests. Also read the Builder template at
`prompts/templates/apply-ontology-change-proposal.md`, which this stage adds. Adapt names and paths
to the target repository as it exists. Do not assume a later prompt has added named mapping tools,
the accounts-payable example, a release-history service, or a semantic-query surface.

## Deterministic release-manifest inputs and classification

Add a build or CI command that accepts exactly these explicit inputs:

- the explicitly identified, previously deployed compiled ontology artifact;
- the candidate compiled ontology artifact; and
- an explicit release metadata file containing a release ID, deployment timestamp, and the
  provenance required to verify the previous artifact.

Do not infer the previous artifact from a working tree, a Git revision, a network lookup, an intake
record, an unpinned mutable path, or the candidate artifact. Require and validate the previous
artifact's provenance in the release metadata file. CI must fail closed before generating a manifest
if either the previous artifact or its provenance is absent, unreadable, malformed, or does not
verify according to the repository's existing provenance conventions. A first-deployment exception,
if the repository needs one, must be an explicit reviewed baseline artifact and provenance, not an
implicit empty prior release.

Write a canonical `release-manifest.json` from only the three verified inputs. Its byte order,
collection ordering, identifiers, and digest behavior must be deterministic. Repeating generation
with byte-identical inputs must produce byte-identical output. The release ID and deployment
timestamp are release metadata, not values inferred at runtime; validate their format and reject
ambiguous or duplicate release metadata.

Compare stable IDs and classify added, changed, deprecated, and removed records for each of these
governed classes:

- systems;
- entities;
- attributes;
- relationships;
- semantic mappings;
- mapping definitions; and
- named mapping tools, if present in the candidate artifact.

For a stable ID present in both artifacts, determine `changed` from the canonical representation
that defines that class's deployed semantics. Do not classify a formatting-only difference as a
semantic change. Preserve the applicable governed provenance for every result. Define and document
the deterministic sort order and tie-breakers, including ordering across classes and stable IDs.
Classify compatibility impact using explicit reviewed rules that distinguish at least compatible,
potentially-breaking, and breaking effects. A removed item, a deprecation, and an altered schema or
mapping contract must not be silently treated as equivalent. If a class or field cannot be assessed
from the compiled artifact, report that limitation as a deterministic review flag rather than
guessing compatibility.

## Fingerprint and delivery-plane boundary

The release manifest is delivery metadata; it must not contribute to the compiled ontology
fingerprint, change the fingerprint algorithm, or make `ontology:check` consider an otherwise
unchanged compiled ontology stale. Demonstrate this in tests. The manifest must not contain raw
source artifacts, pending proposal payloads, intake IDs, submission subjects, receipts, lifecycle
events, engineer dispositions, local paths, credentials, access tokens, live records, payment data,
or personal data.

Load and serve only the current deployment's bounded manifest. Do not promise release history,
diff arbitrary artifacts at runtime, retrieve an earlier artifact, query the intake store, mutate
the manifest, write a file, execute a model, fetch a source, or construct a network client. A
release manifest is a reviewed CI/CD output associated with deployment, not an alternative ontology
source and not a route around governed compilation and review.

## Qualified-user read surface and feedback loop

Register exactly one read-only MCP tool named `ontology_get_release_changes` and one resource named
`ontology://release/current`. Both require `ontology:read` after authentication has supplied an
already validated `AuthorisedPrincipal`; tool discovery or a valid token alone is insufficient.
Use existing authorization and generated-artifact loading conventions. Give the tool read-only,
idempotent annotations consistent with the repository's MCP conventions.

Define one shared deterministic projection and serializer for the two surfaces. The resource takes
no input. The tool may accept only a documented bounded filter or limit if the target repository
needs one; validate every input and keep the tool's default response byte-equivalent to the
resource's record array and metadata. In all cases, bound output, sort records by the documented
stable order, include clear `total`, `returned`, and `truncated` metadata, and return enough
release, fingerprint, class, change-kind, compatibility-impact, and governed-provenance evidence
for a qualified user to prepare a correction. Do not return a full compiled artifact or an
unbounded manifest dump.

Document the pull-based workflow: an authorized conversational agent reads
`ontology://release/current` or calls `ontology_get_release_changes`; it identifies the deployed
stable references and current fingerprint; it prepares a correction; and it submits it through the
existing `ontology_submit_change_proposal` tool. The change-read surface never changes deployed
behavior, accepts a proposal, applies a correction, queues an automatic workflow, rebases a stale
proposal, or calls an external system.

When a proposal's base fingerprint differs from the current deployment fingerprint, preserve the
existing `stale-base` warning in the intake payload. The warning is review evidence only: it must
not mutate the deployed manifest, alter runtime behavior, hide the submission, automatically
rebase it, or promote it. The service must continue to prevent qualified users from listing,
retrieving, querying, updating, deleting, exporting, or disposing of pending intake submissions.

## Tests and boundaries

Add focused, deterministic tests covering:

- every governed class and each added, changed, deprecated, and removed change class, including
  compatibility-impact classification and governed provenance;
- stable sorting, documented tie-breakers, canonical serialization, and repeated byte-identical
  manifest generation from equivalent explicit inputs;
- missing, malformed, mismatched, or unprovenanced baseline failure before manifest generation,
  including refusal to infer a baseline from Git, the working tree, or the network;
- manifest fingerprint non-participation: changing valid release metadata or the manifest does not
  change the compiled ontology fingerprint or make the compiled ontology stale;
- `ontology:read` enforcement on both the exact MCP tool and resource, refusal before projection
  for a validated principal without that capability, and preservation of existing authorization
  behavior;
- one shared bounded projection: exact default tool/resource equivalence, stable selection,
  `{ total, returned, truncated }` metadata, invalid-bound rejection, and no unbounded artifact
  dump;
- manifest non-disclosure of intake records, identifiers, receipts, subjects, lifecycle events,
  dispositions, raw artifacts, local paths, credentials, or business records; and
- sentinels installed after fixtures proving generation, CI checks, server startup and requests,
  MCP registration and calls neither query nor mutate `IntakeStore`, fetch a source, follow a
  locator, read an attachment, write an artifact, call a model, resolve an endpoint, read a
  credential, or create a network side effect.

Keep tests offline and deterministic. Do not hand-edit `ontology/compiled/` or generated Project
Narrative output.

## Acceptance criteria and verification

- CI produces a deterministic `release-manifest.json` solely from verified explicit previous and
  candidate compiled artifacts plus explicit release metadata, and it fails closed without a
  baseline and provenance.
- The manifest classifies stable-ID changes, compatibility impact, and governed provenance across
  every specified class without changing the ontology fingerprint.
- `ontology_get_release_changes` and `ontology://release/current` expose the same bounded,
  authorized current-release projection; they neither promise history nor disclose intake content.
- An authorized qualified user can use the current delta and fingerprint to submit a further
  review-required correction through the existing submission tool; it cannot affect deployment
  behavior automatically.
- Generation, delivery, and MCP access are read-only with respect to the intake plane and require
  no network, model, source retrieval, or runtime artifact generation.

Run the repository's full check (currently `npm run check`, if still provided), focused manifest,
fingerprint, CI/provenance, authorization, MCP/resource, bounded-output, intake-non-disclosure,
and no-I/O boundary tests, plus `git diff --check`. Inspect generated-artifact changes and explain
every legitimate change. Confirm `ontology/compiled/` has no unreviewed manual edit. Commit locally
with the focused message `Add release visibility`. Do not push.

## Governance

This is a decision-bearing product, architecture, governance, and operational implementation.
Before merging a target-repository pull request, apply `narrative-required` together with
substantive `## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences`
sections in the same action, before merge. Never hand-edit, hand-merge, or otherwise author
generated `Narrative.md`; use a reviewed fragment and the target repository's generation process.
The resulting Narrative-only pull request must not have `narrative-required`, or it would
recursively create another entry.
