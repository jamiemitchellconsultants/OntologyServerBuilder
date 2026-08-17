# Prompt A-27 — Add the explicit embedding refresh command

Implement only Prompt 3, "The refresh command (the only networked step)", from the current
`docs/step4.md` in the separate `OntologyService` repository. Prompts A-25 and A-26 already provide
offline embedding text, cache primitives, configuration, and evidence contracts. Read those
prompts, the current guide, `AGENTS.md`, Project Narrative rules, `package.json`, `src/cli.ts`,
`src/compiler.ts`, the registered-entity loading path, `src/embeddings.ts`, configuration
validation, and relevant CLI, compiler, matcher, runtime, and MCP tests before editing. Adapt
names and paths to the repository as it exists. Do not edit `OntologyServerBuilder` in this stage.

This stage adds the maintainer-operated command that refreshes the committed cache used by a later
prompt. It adds no cache-consuming compilation, embedding scoring or fusion, matcher behavior,
generated-artifact change, runtime behavior, MCP tool, or automatic refresh.

## One explicit network boundary

Add the `ontology:embed` npm script, which invokes an `embed` CLI subcommand. Only
`npm run ontology:embed` may construct the concrete HTTP embedding client or call the configured
embedding endpoint. It is a deliberate maintainer action; no other CLI subcommand may delegate to
it or refresh implicitly.

The command must require explicit, validated refresh settings: the endpoint, the name of the
environment variable containing the credential, a positive bounded timeout, and a positive bounded
batch size. Keep these settings separate from the model identity in Prompt A-26: model ID, version,
dimension, and `embedding.cachePath` remain authoritative configuration; the credential value is
read only at command execution through its supplied environment-variable name. `embedding.cachePath`
is the authoritative path for every cache read and atomic replacement in this command. Endpoint,
credential name, timeout, and batch size may be explicit command options or validated non-secret
refresh configuration, according to existing repository conventions. Do not supply a hidden
provider default, infer a credential name, or accept an empty setting.

Use an `EmbeddingClient` interface and inject it into refresh orchestration tests. Keep the
enterprise HTTP implementation in a module reachable only from the `embed` dispatch path; use a
lazy import or equivalent boundary so ordinary CLI, compiler, test, runtime, and MCP paths neither
import nor instantiate it. The client must apply the configured timeout and send batches in the
defined order. It may read only the named credential environment variable and any separately named
provider deployment variable strictly required by the selected provider. Never persist, return,
snapshot, or log a credential value. Redact the endpoint and credential value from errors, logs,
test failures, and generated artifacts; errors may identify the safe credential-variable name.

## Deterministic, content-addressed refresh

First extract or reuse one shared offline function that loads exactly the entities used by matching;
the compiler and refresh command must use it rather than duplicate manifest parsing. For every such
entity, calculate Prompt A-25's canonical embedding text and content hash. Sort entity work by stable
entity ID, with the content hash as a tie-breaker. Deduplicate identical hashes before requests,
while retaining deterministic entity-to-hash association for diagnostics.

The default refresh reads the existing cache only through Prompt A-25's validated cache primitives.
It may reuse a vector only when the cache model ID, version, and dimension exactly match the
validated configured model and its content hash matches current canonical text. Request exactly the
unique missing texts. Preserve the stable entity-ID ordering when forming requests and batches; each
batch must have a deterministic size and order. If the cache format serializes a content-hash map,
continue to use its deterministic hash-key writer from Prompt A-25, but sort all refresh cache-entry
inputs by stable entity ID before materializing that map.

When a cache exists with different model identity, fail closed before any endpoint call with an
actionable instruction to run `npm run ontology:embed -- --full`. `--full` creates the replacement
cache from zero and never mixes, relabels, deletes in place, or reuses vectors from another model.
No cache means all unique current texts are missing. Do not add a fallback that refreshes during
compile, check, server startup, request handling, MCP registration, or test setup.

Treat provider responses as untrusted. For every requested batch require exactly one vector per
requested text in the original order, no missing or extra responses, finite numeric components, and
the configured dimension. Round, assemble, and fully validate the *complete* next cache with the
Prompt A-25 primitives before attempting any replacement. Write it only through the fsync-safe atomic
writer. On HTTP, timeout, response, validation, write, fsync, close, or rename failure, leave the
previous committed cache unchanged and clean up only the temporary sibling file. Never publish a
partial cache.

## Tests and documentation

Add focused tests with a fake injected `EmbeddingClient`; no test may reach a real endpoint. Cover:

- no-op refresh with zero client calls; partial refresh requesting only missing content hashes; a
  model change requiring explicit `--full`; and full refresh requesting every unique text;
- request and batch ordering by stable entity ID, deterministic batch boundaries, content-addressed
  reuse, duplicate-text deduplication, and byte-identical cache output for equivalent reordered
  inputs;
- malformed, missing, extra, non-finite, wrong-dimension, and timed-out provider responses; safe
  errors that expose neither endpoint nor credential value; and required-setting validation;
- complete-cache validation before replacement plus old-cache preservation for every client,
  validation, write, fsync, close, and rename failure; and
- sentinels installed after fixture setup proving compile, `ontology:check`, ordinary CLI commands,
  tests, runtime startup and requests, MCP registration and calls, and matcher entry points neither
  import, instantiate, nor call the network client or endpoint. Keep these sentinels independent of
  documentation and make a network attempt fail the test immediately.

Document the maintainer procedure in the repository's operational documentation: settings are
explicit; credential values are environment-only; default refresh reuses matching cached vectors;
`--full` is required for an intentional model-identity change; failures retain the old cache; and
the refreshed cache is reviewed and committed as a deliberate input to a later enabled-mode change.
Do not document manual cache edits or turning embeddings on here.

## Acceptance criteria and verification

- `npm run ontology:embed` is the only network-enabled embedding route, requires explicit safe
  refresh settings, supports a deliberate `--full`, and is testable through injected clients.
- Refresh is content-addressed, model-identity exact, stably ordered and batched, fully validated,
  and atomically replaces a cache only after a complete successful refresh.
- Every non-refresh compiler, check, test, ordinary CLI, runtime, MCP, and matcher path remains
  offline. This stage does not consume the cache or change lexical results, compiled artifacts, or
  generated output.
- Secrets are neither committed nor emitted, and every specified failure preserves the old cache.

Run the repository's full check (currently `npm run check`, if still provided), the focused refresh
and boundary tests, and `git diff --check`. Inspect generated artifacts and confirm
`ontology/compiled/` has no diff. Commit locally with the focused message
`Add explicit embedding refresh command`. Do not push.

## Governance

This is a decision-bearing operational and architecture implementation. Do not merge its target
repository pull request without applying `narrative-required` and substantive `## Narrative
Context`, `## Narrative Decision`, and `## Narrative Consequences` sections together before merge.
Do not apply the label first and add the sections later. `gh pr create --body` replaces the pull
request template, so any use of it must carry all three sections rather than bypassing the template.

Never hand-edit, hand-merge, or otherwise author generated `Narrative.md`; use a reviewed fragment
and the target repository's generation process. The resulting Narrative-only pull request must not
have `narrative-required`, or it would recursively create another entry.
