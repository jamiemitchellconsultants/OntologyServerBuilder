# Qualified-user ontology intake implementation plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development
> (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use
> checkbox (`- [ ]`) syntax for tracking.

**Goal:** Extend OntologyServerBuilder with Prompts 33–47 and reusable operational templates that
teach a coding agent to build qualified-user intake, governed analysis, release visibility, and
named pure mapping tools in OntologyService.

**Architecture:** Add documentation only to OntologyServerBuilder. Each numbered prompt specifies
one independently reviewable OntologyService stage, preserves the separation between the mutable
intake plane and immutable delivery plane, and updates the README sequence in the same commit. Four
reusable templates guide later engineer work without editing either repository automatically.

**Tech Stack:** Markdown, the existing staged-prompt contract, TypeScript/Node.js 22 terminology for
the target OntologyService, MCP, SQLite, JSON Schema, deterministic JSON compilation, cached
embeddings, and coding-agent JSON analysis.

## Global constraints

- Read and follow `CLAUDE.md`, `README.md`, `START-HERE.md`, and the `Narrative.md` index before
  editing.
- Markdown wraps at 100 columns and uses absolute dates.
- A prompt specifies work in the separate OntologyService repository and cannot assume a later
  prompt has run.
- Every prompt names its dependencies, exclusions, executable checks, acceptance criteria,
  Narrative classification, local commit boundary, and `Do not push` instruction.
- Add each prompt to `README.md`'s ordered sequence in the same commit that creates the prompt.
- Never weaken an earlier acceptance check or edit a generated OntologyService artifact by hand.
- Never edit `Narrative.md`; this decision-bearing Builder change will use the pull-request label
  and body workflow rather than a hand-authored fragment.
- The built runtime never retrieves interface sources, opens user attachments, or calls a model.
- Proposal data never becomes a compiled ontology fact, traversable relationship, accepted mapping,
  or executable mapping tool without repository review, CI/CD, and deployment.
- The initial SQLite intake adapter is single-instance. Multi-instance production intake remains
  unsupported; intake must stay disabled in such deployments.
- Prompt 33 follows the active Prompt 32b. The withdrawn Prompt 32a remains in the README history,
  so Prompts 33–47 occupy ordered-list positions 36–50.
- Run `git diff --check` after every task and keep `.superpowers/` files out of commits.

## File structure

### Numbered prompts

- `prompts/33-plan-qualified-user-intake.md` — read-only architecture and migration plan.
- `prompts/34-add-durable-intake-and-capabilities.md` — authorization and SQLite intake
  foundation.
- `prompts/35-add-qualified-intake-submissions.md` — registration and ontology-change submissions.
- `prompts/36-add-engineer-intake-workbench.md` — review operations and deterministic analysis.
- `prompts/37-add-embedding-text-and-cache-primitives.md` — Step 4 guide stage 1.
- `prompts/38-add-embedding-configuration-and-evidence.md` — Step 4 guide stage 2.
- `prompts/39-add-embedding-refresh-command.md` — Step 4 guide stage 3.
- `prompts/40-fuse-embeddings-into-matching.md` — Step 4 guide stage 4.
- `prompts/41-integrate-embedding-cache-with-compilation.md` — Step 4 guide stage 5.
- `prompts/42-complete-embedding-matcher-delivery.md` — combines Step 4 guide stages 6 and 7.
- `prompts/43-add-llm-intake-analysis.md` — bounded coding-agent analysis and consolidation.
- `prompts/44-add-release-change-visibility.md` — deterministic release delta and user feedback.
- `prompts/45-compile-named-mapping-tools.md` — approved definitions become pure named MCP tools.
- `prompts/46-add-accounts-payable-mapping-example.md` — reviewed invoice-to-payment example.
- `prompts/47-audit-qualified-user-workflow.md` — independent end-to-end audit.

### Reusable operational templates

- `prompts/templates/register-exported-intake.md` — turn a reviewed export into a bounded
  OntologyService registration task.
- `prompts/templates/resolve-intake-mapping-review.md` — produce and review the advisory semantic
  analysis package.
- `prompts/templates/apply-ontology-change-proposal.md` — apply accepted deployed-ontology
  feedback.
- `prompts/templates/create-named-mapping-tool.md` — author one governed mapping definition and
  its examples.

### Sequence documentation

- `README.md` — links every new prompt and explains the Prompt 33–47 dependency chain, the
  single-instance intake limitation, the six Builder embedding prompts covering seven guide
  stages, and the operational templates.

---

### Task 1: Add Prompt 33, the read-only architecture plan

**Files:**

- Create: `prompts/33-plan-qualified-user-intake.md`
- Modify: `README.md`
- Modify: `docs/superpowers/specs/2026-08-07-qualified-user-ontology-intake-design.md`

**Interfaces:**

- Consumes: behavior produced by Prompts 3, 4, 6, 12, 17–20, 30–32b.
- Produces: `docs/qualified-user-intake-and-mapping-tools.md` in OntologyService, containing the
  inventory and migration plan consumed conceptually by Prompt 34.

- [ ] **Step 1: Draft the Prompt 33 orientation and exclusions**

Use `apply_patch` to create the prompt with this exact responsibility:

```text
# Prompt 33 — Plan qualified-user intake and governed mapping-tool delivery

Inspect the current built service and write a reviewable plan. Do not change runtime behavior,
ontology inputs, compiled artifacts, authentication behavior, deployment configuration, or MCP
contracts in this stage.
```

Require the agent to read the current architecture, proposal workflows, authentication, deployment,
matcher, compiler, mapping instructions, MCP surface, tests, and Narrative rules.

Correct the approved Builder design specification so that the new sequence follows active Prompt
32b rather than Prompt 32. Do not alter any other approved design decision.

- [ ] **Step 2: Specify the plan document's required decisions**

Require `docs/qualified-user-intake-and-mapping-tools.md` to contain:

```text
1. Current-state inventory and gaps.
2. Two-plane architecture and dependency diagram.
3. Capability matrix for read, propose, and intake review.
4. Immutable submission and append-only event schemas.
5. Single-instance SQLite baseline and disabled-by-default migration.
6. Engineer export and offline analysis flow.
7. Deterministic, embedding, and LLM evidence boundaries.
8. Release-manifest contract.
9. Named mapping-tool source, compiler, runtime, and failure contracts.
10. Prompt 34–47 file-impact plan, rollback points, and open decisions.
```

Require explicit reconciliation with Prompt 30's no-volume deployment and Prompt 31's multi-instance
AWS deployment: a later intake volume is justified only for a single instance, and intake remains
disabled where a shared durable adapter is unavailable.

- [ ] **Step 3: Add acceptance and governance requirements**

Require no ontology or generated-artifact diff, `npm run check`, `git diff --check`, and a focused
local documentation commit. Classify the target-service plan as a meaningful product,
architecture, governance, and operational decision requiring the complete Narrative PR workflow.

- [ ] **Step 4: Add Prompt 33 to the README sequence**

Append this ordered entry immediately after active Prompt 32b:

```markdown
36. [Plan qualified-user intake](prompts/33-plan-qualified-user-intake.md)
```

- [ ] **Step 5: Verify and commit Task 1**

Run:

```bash
awk 'length($0) > 100 { print FNR ":" length($0) ":" $0 }' \
  README.md prompts/33-plan-qualified-user-intake.md
rg -n '^## Acceptance criteria$|narrative-required|Commit locally|Do not push' \
  prompts/33-plan-qualified-user-intake.md
git diff --check
```

Expected: no `awk` or `git diff --check` output; `rg` finds all four required prompt controls.

Commit:

```bash
git add README.md prompts/33-plan-qualified-user-intake.md \
  docs/superpowers/specs/2026-08-07-qualified-user-ontology-intake-design.md
git commit -m "Add qualified-user intake planning prompt"
```

### Task 2: Add Prompt 34, capability authorization and durable intake

**Files:**

- Create: `prompts/34-add-durable-intake-and-capabilities.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: the provider-specific validated principal from Prompt 20/32 and the plan from Prompt 33.
- Produces: `Capability`, `AuthorisedPrincipal`, `IntakeStore`, `SubmissionReceipt`,
  `SubmissionEvent`, and disabled/SQLite intake configuration for Prompt 35.

- [ ] **Step 1: Specify capability mapping**

Require provider-neutral capabilities named exactly `ontology:read`, `ontology:propose`, and
`ontology:intake:review`. Require handlers to check capabilities after token validation. Preserve
existing authentication modes, but refuse to enable intake in `none` or `static` because they do not
provide attributable qualified-user identity. Require explicit claim-to-capability configuration
for Keycloak and Entra and fail closed when intake is enabled without it.

- [ ] **Step 2: Specify the storage contract and SQLite adapter**

Require a focused storage module with these conceptual operations:

```ts
interface IntakeStore {
  submit(input: NewSubmission): Promise<SubmissionReceipt>;
  list(query: IntakeListQuery): Promise<IntakeSummary[]>;
  export(id: string): Promise<IntakeExport>;
  appendEvent(id: string, event: NewSubmissionEvent): Promise<SubmissionEvent>;
}
```

Require canonical JSON payloads, SHA-256 payload digests, immutable submission rows, append-only
events, atomic receipt creation, and uniqueness on authenticated subject plus idempotency key.
Specify `INTAKE_MODE=disabled|sqlite`, default `disabled`, and `INTAKE_SQLITE_PATH` as required for
SQLite mode.

- [ ] **Step 3: Specify deployment and recovery boundaries**

Require a persistent intake volume only when SQLite intake is enabled. Compiled ontology artifacts
remain inside the immutable image. State that SQLite supports exactly one intake-enabled service
instance. Require startup failure when SQLite is combined with a declared multi-instance mode.
Update homelab guidance with backup, restore, file ownership, encryption, database rotation, and
capacity monitoring. Document that Prompt 31's multi-instance AWS baseline keeps intake disabled.

- [ ] **Step 4: Specify tests and acceptance**

Require tests for claim mapping, missing capabilities, disabled mode, SQLite initialization,
atomicity, idempotent identical replay, conflicting replay, immutable payloads, append-only events,
restart persistence, corrupt database handling, and prohibition of SQLite intake in a
multi-instance configuration. Require no MCP tool addition in this stage.

- [ ] **Step 5: Update README, verify, and commit**

Add:

```markdown
37. [Add durable intake](prompts/34-add-durable-intake-and-capabilities.md)
```

Run the Task 1 `awk` and `git diff --check` commands against this prompt and README, plus:

```bash
rg -n 'ontology:read|ontology:propose|ontology:intake:review|INTAKE_MODE|single-instance' \
  prompts/34-add-durable-intake-and-capabilities.md
```

Expected: all five boundaries are present and formatting checks are silent.

Commit:

```bash
git add README.md prompts/34-add-durable-intake-and-capabilities.md
git commit -m "Add durable intake foundation prompt"
```

### Task 3: Add Prompt 35, qualified-user submission tools

**Files:**

- Create: `prompts/35-add-qualified-intake-submissions.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: `IntakeStore`, compiled source fingerprint, existing proposal validators, and
  `ontology:propose` from Prompt 34.
- Produces: `ontology_submit_system_registration`, `ontology_submit_change_proposal`, and their
  canonical submission/receipt schemas for Prompt 36.

- [ ] **Step 1: Specify the system-registration submission tool**

Require the tool to accept the normalized output of the existing preparation workflow plus source
format, media type, byte size, SHA-256 digest, inert source locator, extractor provenance, ontology
fingerprint, and idempotency key. Accept OpenAPI JSON/YAML, exported MCP catalog JSON, and WSDL.
Never accept raw bytes, credentials, records, payment data, personal data, local paths, or a URL to
fetch.

- [ ] **Step 2: Specify the ontology-change submission tool**

Require stable deployed references, base fingerprint, change kind, evidence, expected workflow
effect, assumptions, gaps, and idempotency key. Validate references against the compiled ontology.
Preserve a stale submission with `stale-base` warning; never promote it automatically.

- [ ] **Step 3: Set exact bounds and receipt semantics**

State these limits in the prompt:

```text
canonical payload: 2 MiB
normalized filename: 255 Unicode scalar values
entities: 500
attributes per entity: 1,000
operations/tools: 1,000
relationships: 1,000
allowed values per attribute: 1,000
individual free-text field: 16,000 Unicode scalar values
idempotency key: 8–128 characters from [A-Za-z0-9._:-]
SHA-256: 64 lower-case hexadecimal characters
```

Require a receipt with opaque ID, payload digest, received timestamp, and `received`. Do not add a
qualified-user list, get, update, delete, status, or disposition operation.

- [ ] **Step 4: Specify MCP annotations and tests**

Mark both tools state-changing, non-destructive, and idempotent. Require store-level and in-memory
MCP tests for authorization, every bound, duplicate keys, digest mismatch, Unicode normalization,
stale fingerprints, unfamiliar safe metadata, prohibited structural content, queue failure, and
proof that compiled ontology bytes are unchanged.

- [ ] **Step 5: Update README, verify, and commit**

Add:

```markdown
38. [Add qualified-user intake submissions](prompts/35-add-qualified-intake-submissions.md)
```

Verify the line limit, diff whitespace, both tool names, exact limits, `ontology:propose`, and the
absence of any proposal retrieval tool. Commit:

```bash
git add README.md prompts/35-add-qualified-intake-submissions.md
git commit -m "Add qualified intake submission prompt"
```

### Task 4: Add Prompt 36 and the registration template

**Files:**

- Create: `prompts/36-add-engineer-intake-workbench.md`
- Create: `prompts/templates/register-exported-intake.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: stored submissions from Prompt 35 and existing offline adapters/matcher.
- Produces: engineer list/export/disposition operations, canonical `intake-export.json`,
  `deterministic-analysis.json`, and a registration task template used after human review.

- [ ] **Step 1: Specify engineer-only operations**

Require `ontology_intake_list`, `ontology_intake_export`, and `ontology_intake_disposition`, all
gated by `ontology:intake:review`. List returns bounded summaries. Export returns canonical payload,
events, and digests and may append an `exported` audit event. Disposition appends only `accepted`,
`rejected`, or `superseded`, with actor, reason, and timestamp. Add a CLI wrapper that can write an
explicitly named local export file.

- [ ] **Step 2: Specify offline artifact verification and re-parsing**

Require the engineer to pass a local artifact path to the CLI. The path never enters the intake
record. Verify filename, format, byte size, and digest before parsing. Reuse existing adapters with
network disabled. Compare parser output with submitted normalized metadata and report omissions,
additions, type differences, requiredness differences, and provenance conflicts.

- [ ] **Step 3: Specify deterministic intake matching**

Run name, description, attribute, type, allowed-value, operation/tool context, relationship, and
governed-synonym comparisons. Emit only `exact-candidate`, `likely-candidate`, `review-required`, or
`unmatched`; never emit a governed accepted disposition. Sort every report array by stable ID.

- [ ] **Step 4: Write the reusable registration template**

The template must require an engineer to supply explicit paths to the export, verified artifact,
and analysis report. It instructs an OntologyService coding agent to inspect the evidence, propose a
bounded file list, add reviewed source/manifest files, compile, review mapping output, and stop on
unresolved semantic decisions. It forbids treating advisory candidates as approval.

- [ ] **Step 5: Specify tests, update README, verify, and commit**

Require authorization, pagination bounds, canonical export, append-only disposition, digest
mismatch, no-network parsing, parser discrepancy, stable match sorting, reordered-input identity,
and non-mutation tests. Add:

```markdown
39. [Add the engineer intake workbench](prompts/36-add-engineer-intake-workbench.md)
```

Commit:

```bash
git add README.md prompts/36-add-engineer-intake-workbench.md \
  prompts/templates/register-exported-intake.md
git commit -m "Add engineer intake workbench prompt"
```

### Task 5: Add Prompt 37, embedding text and cache primitives

**Files:**

- Create: `prompts/37-add-embedding-text-and-cache-primitives.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: `docs/step4.md` Prompt 1 and normalization behavior from the existing matcher.
- Produces: shared embedding text, content hashing, cosine similarity, cache types, validation, and
  atomic cache-writing primitives for Prompt 38.

- [ ] **Step 1: Translate Step 4 Prompt 1 into a numbered implementation stage**

Require the coding agent to implement only the shared offline primitives from the guide. It must
reuse the matcher's normalization logic, define one canonical entity-to-embedding-text function,
hash UTF-8 text with SHA-256, validate finite fixed-dimension vectors, calculate cosine similarity,
and write caches atomically without adding an HTTP client or CLI command.

- [ ] **Step 2: Preserve deterministic and no-network boundaries**

Require sorted entity IDs, rounded numeric serialization, duplicate/content-hash rejection, and no
network import or environment-token read. Embedding code must not affect matching while disabled.

- [ ] **Step 3: Specify exact tests**

Require known-vector cosine tests, zero-vector rejection, normalization equivalence, content-hash
changes, stable text ordering, malformed cache entries, duplicate IDs, dimension mismatch, numeric
rounding, atomic-write failure cleanup, and existing matcher regression tests.

- [ ] **Step 4: Update README, verify, and commit**

Add:

```markdown
40. [Add embedding text and cache primitives](prompts/37-add-embedding-text-and-cache-primitives.md)
```

Verify the prompt names `docs/step4.md`, limits scope to guide Prompt 1, and requires no matcher or
compiler integration. Commit the prompt and README with message:

```text
Add embedding primitives implementation prompt
```

### Task 6: Add Prompt 38, embedding configuration and evidence

**Files:**

- Create: `prompts/38-add-embedding-configuration-and-evidence.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: Prompt 37 primitives and `docs/step4.md` Prompt 2.
- Produces: validated embedding configuration and typed candidate/compilation evidence for Prompt
  39, without changing scoring.

- [ ] **Step 1: Specify configuration types and defaults**

Require `embedding.enabled=false`, model ID/version/dimension, cache path, lexical/embedding weights
that sum to 1, and a maximum semantic-candidate count. Reject unknown keys, non-finite values,
invalid dimensions, missing model identity, and invalid weights.

- [ ] **Step 2: Specify evidence contracts**

Require optional evidence containing lexical score, embedding score, combined score, model
ID/version/dimension, cache digest, and source/target content hashes. Define compilation-level model
and cache provenance without loading a cache yet.

- [ ] **Step 3: Require compatibility tests**

Test defaults, every invalid field, serialization, evidence round-trip, and byte-identical compiled
artifacts with embeddings disabled. No network, cache load, refresh command, scoring, or MCP change
is allowed.

- [ ] **Step 4: Update README, verify, and commit**

Add:

```markdown
41. [Add embedding evidence](prompts/38-add-embedding-configuration-and-evidence.md)
```

Verify and commit with message `Add embedding configuration implementation prompt`.

### Task 7: Add Prompt 39, explicit embedding refresh

**Files:**

- Create: `prompts/39-add-embedding-refresh-command.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: Prompt 37 cache primitives, Prompt 38 configuration, and `docs/step4.md` Prompt 3.
- Produces: an explicit maintainer-only cache refresh command and committed cache for Prompt 40.

- [ ] **Step 1: Specify the only network-enabled embedding path**

Require `npm run ontology:embed` as the sole path allowed to call the configured embedding endpoint.
Compilation, tests, runtime, MCP, and ordinary checks must never call it. Inject the embedding
client for tests. Require explicit endpoint, credential environment-variable name, timeout, and
batch size; never write the credential value to files or logs.

- [ ] **Step 2: Specify content-addressed refresh behavior**

Reuse cached vectors whose content hash and model tuple match, request only missing entries,
validate the complete cache before atomic replacement, and preserve the old cache on failure. Sort
requests and cache entries by stable entity ID.

- [ ] **Step 3: Specify tests and documentation**

Cover no-op refresh, partial refresh, model change, malformed response, wrong dimension, timeout,
redacted errors, stable batching, atomic failure, and sentinels proving every non-refresh path has
no network access.

- [ ] **Step 4: Update README, verify, and commit**

Add:

```markdown
42. [Add the explicit embedding refresh command](prompts/39-add-embedding-refresh-command.md)
```

Verify and commit with message `Add embedding refresh implementation prompt`.

### Task 8: Add Prompt 40, governed embedding fusion

**Files:**

- Create: `prompts/40-fuse-embeddings-into-matching.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: validated configuration/cache from Prompts 38–39 and `docs/step4.md` Prompt 4.
- Produces: combined ranking and evidence for Prompt 41.

- [ ] **Step 1: Specify fusion behavior**

Require combined score `lexicalWeight * lexicalScore + embeddingWeight * embeddingScore` when both
vectors exist. Preserve lexical behavior when disabled or a vector is absent. Rank by combined
score, then stable canonical ID for ties.

- [ ] **Step 2: Specify governance constraints**

An embedding may move an unmatched pair to `review-required`, never to `accepted-auto`. A pair may
remain `accepted-auto` only when its independent lexical score already clears the high-confidence
threshold. Manual mappings retain precedence.

- [ ] **Step 3: Specify tests**

Cover fusion arithmetic, embedding-driven reranking, stable ties, missing vectors, manual overrides,
review promotion, prohibition of semantic auto-acceptance, and byte-identical behavior while
disabled.

- [ ] **Step 4: Update README, verify, and commit**

Add:

```markdown
43. [Fuse embeddings into governed matching](prompts/40-fuse-embeddings-into-matching.md)
```

Verify and commit with message `Add embedding fusion implementation prompt`.

### Task 9: Add Prompt 41, compiler integration

**Files:**

- Create: `prompts/41-integrate-embedding-cache-with-compilation.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: Prompt 40 matcher context and `docs/step4.md` Prompt 5.
- Produces: compiler cache loading, fingerprint participation, stale checks, and generated evidence
  for Prompt 42 and the intake workbench.

- [ ] **Step 1: Specify cache loading and validation**

When embeddings are disabled, do not require or read a cache. When enabled, require the committed
cache; validate model tuple, dimension, complete entity coverage, content hashes, numeric values,
and cache digest before matching.

- [ ] **Step 2: Specify deterministic compilation impact**

Include embedding configuration and cache digest in governed compiler inputs and source fingerprint.
Project compilation-level and per-candidate evidence into generated JSON. Do not claim procedural
embedding evidence as OWL inference.

- [ ] **Step 3: Specify stale and no-I/O tests**

Cover missing cache, stale text hash, wrong model, incomplete coverage, corrupted digest, reordered
cache entries, byte-identical repeated compile, stale-artifact check, and network sentinels around
compile/check/runtime paths.

- [ ] **Step 4: Update README, verify, and commit**

Add:

```markdown
44. [Integrate the embedding cache](prompts/41-integrate-embedding-cache-with-compilation.md)
```

Verify and commit with message `Add embedding compiler integration prompt`.

### Task 10: Add Prompt 42, embedding delivery and acceptance

**Files:**

- Create: `prompts/42-complete-embedding-matcher-delivery.md`
- Modify: `README.md`
- Modify: `docs/superpowers/specs/2026-08-07-qualified-user-ontology-intake-design.md`

**Interfaces:**

- Consumes: Prompts 37–41 and `docs/step4.md` Prompts 6 and 7.
- Produces: complete embedding documentation, runtime evidence visibility, and an accepted Step 4
  implementation baseline for Prompt 43.

- [ ] **Step 1: Specify MCP evidence visibility without model execution**

Expose the same stored embedding evidence through both existing mapping-review responses. Give the
tool an optional limit from 1 to 100 with default 100. The resource returns the same first 100
records in compiled stable-ID order. Both responses include total, returned, and truncated metadata.
Do not add a tool that creates embeddings or accepts free-form embedding queries. Preserve
read-only, idempotent annotations.

- [ ] **Step 2: Specify operator and governance documentation**

Document explicit refresh, cache review/commit, model upgrades, costs, credential handling, disabled
default, deterministic replay, and the rule that semantic-only confidence always requires review.

- [ ] **Step 3: Specify final acceptance matrix**

Require the full guide matrix: disabled compatibility, enabled ranking, no-network compile/runtime,
cache freshness, fingerprint change, MCP evidence, deterministic artifacts, manual precedence, and
no embedding-only automatic acceptance.

- [ ] **Step 4: Update README, verify, and commit**

Add:

```markdown
45. [Complete embedding matcher delivery](prompts/42-complete-embedding-matcher-delivery.md)
```

Verify and commit with message `Add embedding delivery implementation prompt`.

### Task 11: Add Prompt 43 and semantic-analysis template

**Files:**

- Create: `prompts/43-add-llm-intake-analysis.md`
- Create: `prompts/templates/resolve-intake-mapping-review.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: deterministic analysis from Prompt 36 and embedding evidence from Prompt 42.
- Produces: `llm-analysis-request.json`, validated `llm-analysis-response.json`, and consolidated
  `intake-analysis.json` for engineer review.

- [ ] **Step 1: Specify bounded request generation**

Require a CLI command that selects no more than 200 candidates and emits no more than 2 MiB of
canonical JSON. Include normalized source/target evidence, deterministic scores, embedding scores,
provenance, conflicts, and explicit questions. Treat source descriptions as quoted inert data.

- [ ] **Step 2: Specify response schema and provenance**

Require model ID/version, prompt-template version, request digest, analysis timestamp, candidate
recommendations, explanations, gaps, disagreements, and questions. Allowed recommendation values
are `likely-candidate`, `review-required`, and `unmatched`; the response cannot assert accepted
governance state.

- [ ] **Step 3: Specify validation and consolidation**

Validate the response offline. Merge it without deleting or rewriting deterministic/embedding
evidence. Invalid, mismatched, or absent output sets `semanticAnalysisStatus=incomplete` and retains
earlier results. Sort all merged arrays canonically.

- [ ] **Step 4: Write the reusable semantic-review template**

Require the coding agent to read only the request and named governed context, return JSON matching
the schema, quote evidence, distinguish inference from source fact, enumerate gaps, and perform no
repository edits, network calls, or acceptance decisions.

- [ ] **Step 5: Specify tests, update README, verify, and commit**

Cover request bounds, canonical generation, prompt-injection-shaped text, schema violations, wrong
digest, unsupported status, merge preservation, incomplete status, and absence of a live-model CI
dependency. Add:

```markdown
46. [Add bounded LLM intake analysis](prompts/43-add-llm-intake-analysis.md)
```

Commit the prompt, template, and README with message `Add LLM intake analysis prompt`.

### Task 12: Add Prompt 44 and ontology-change template

**Files:**

- Create: `prompts/44-add-release-change-visibility.md`
- Create: `prompts/templates/apply-ontology-change-proposal.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: two compiled ontology artifacts, explicit release metadata, and the submission tool from
  Prompt 35.
- Produces: deterministic `release-manifest.json`, `ontology_get_release_changes`,
  `ontology://release/current`, and an engineer application template.

- [ ] **Step 1: Specify release comparison inputs and output**

Require an explicit previous compiled artifact, candidate compiled artifact, and release metadata
file containing release ID and deployment timestamp. Compare stable IDs and classify added,
changed, deprecated, removed, and compatibility-impacting systems, entities, attributes,
relationships, semantic mappings, mapping definitions, and named mapping tools.

- [ ] **Step 2: Preserve fingerprint and deployment boundaries**

The release manifest is deterministic from the three inputs but does not contribute to the ontology
fingerprint. CI must fail if a previous artifact or its provenance is missing. Runtime serves only
the current bounded delta; it does not promise release history or expose intake records.

- [ ] **Step 3: Specify qualified-user visibility and feedback**

Gate the read-only tool/resource with `ontology:read`. Document how an agent reads the current delta
and submits a correction through `ontology_submit_change_proposal`. A stale fingerprint warning is
preserved and cannot change deployed behavior.

- [ ] **Step 4: Write the change-application template**

Require exact proposal path, current fingerprint comparison, stable-reference validation, evidence
review, bounded destination files, compilation, generated-artifact review, tests, and Narrative
classification. Forbid direct editing of compiled files or copying proposal status into governed
status.

- [ ] **Step 5: Specify tests, update README, verify, and commit**

Cover every change class, stable sorting, repeated byte identity, missing baseline, provenance,
fingerprint non-participation, authorization, bounded MCP output, and pending-intake non-disclosure.
Add:

```markdown
47. [Add deployment release-change visibility](prompts/44-add-release-change-visibility.md)
```

Commit with message `Add release visibility implementation prompt`.

### Task 13: Add Prompt 45 and named-mapping template

**Files:**

- Create: `prompts/45-compile-named-mapping-tools.md`
- Create: `prompts/templates/create-named-mapping-tool.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: approved mapping instructions from Prompt 17 and compiled entity/attribute schemas.
- Produces: governed `ontology/mapping-tools.json` definitions, compiled tool descriptors, a pure
  evaluator, and one MCP registration per approved definition.

- [ ] **Step 1: Specify the governed mapping-tool source**

Each entry must contain stable ID, stable MCP name, approved mapping-instruction ID, source/target
entity IDs, semantic version, input/output JSON Schemas, lookup-input schemas, failure contract,
status, evidence, review provenance, and positive/negative/boundary/ambiguity examples.

- [ ] **Step 2: Specify compilation and registration**

Validate all references, schema keywords, required fields, operation allow-list, tool-name
collisions, unresolved requirements, and examples. Compile approved complete entries into sorted
descriptors. At startup, register one named closure per descriptor backed by the restricted
evaluator. Never generate or execute arbitrary source code.

- [ ] **Step 3: Specify pure execution**

Input contains a source record and all supporting lookup records. Output contains mapping ID,
version, ontology fingerprint, and either target record or one of `precondition-failed`,
`missing-input`, `ambiguous-lookup`, or `target-validation-failed`. Forbid filesystem, network,
environment, clock, randomness, and source/target API calls.

- [ ] **Step 4: Write the reusable mapping-tool template**

Require source/target IDs, analysis report, reviewed decisions, expected tool name, schemas,
preconditions, transformations, lookup inputs, failure behavior, and examples. Instruct the coding
agent to stop when semantics or selection rules remain unresolved.

- [ ] **Step 5: Specify tests, update README, verify, and commit**

Cover every validation rule, non-approved omission, name collision, dynamic tool discovery, schema
validation, stable results under reordered object keys, each structured failure, no-I/O sentinels,
clock/random sentinels, and repeated compilation. Add:

```markdown
48. [Compile named mapping tools](prompts/45-compile-named-mapping-tools.md)
```

Commit with message `Add named mapping tool implementation prompt`.

### Task 14: Add Prompt 46, the accounts-payable example

**Files:**

- Create: `prompts/46-add-accounts-payable-mapping-example.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: Brightflag entities from Prompt 16, mapping instructions from Prompt 17, semantic
  analysis from Prompt 43, and named-tool compiler from Prompt 45.
- Produces: a synthetic accounts-payable OpenAPI system and approved
  `map_brightflag_invoice_to_accounts_payable_payment_instruction` tool.

- [ ] **Step 1: Specify the synthetic accounts-payable source**

Require a checked-in OpenAPI snapshot with payment-instruction request schema, idempotency key,
invoice reference, beneficiary reference, amount, currency, requested execution date if required,
and validation rules. Use synthetic identifiers and no bank data, personal data, credentials, or
live endpoint. Separate business entities from transport/error wrappers.

- [ ] **Step 2: Resolve every prior mapping blocker explicitly**

The source must define payable invoice statuses, required target fields, identifier format,
currency/amount constraints, idempotency behavior, and beneficiary-selection behavior. Supporting
lookup input must identify exactly one eligible pay site; zero or multiple eligible results produce
a structured failure. Do not approve the mapping if the fixture or governed evidence leaves one of
these rules unresolved.

- [ ] **Step 3: Specify the named mapping and examples**

Require the exact MCP name from the interface block. Map source invoice ID deterministically to the
target idempotency/reference fields, approved gross total to amount, currency code to currency, and
the reviewed pay-site reference to beneficiary. Include payable, non-payable, missing-pay-site,
ambiguous-pay-site, invalid-currency, and reordered-key examples.

- [ ] **Step 4: Specify the synthetic user workflow test**

Use an in-memory invoice MCP fixture to return an invoice and supporting records, invoke the named
mapping tool, validate the target payload, and stop before any accounts-payable HTTP call. Assert
that a target invocation is a separate user-agent responsibility.

- [ ] **Step 5: Update README, verify, and commit**

Add:

```markdown
49. [Add the accounts-payable mapping example](prompts/46-add-accounts-payable-mapping-example.md)
```

Verify that the prompt contains the exact tool name, every resolved blocker, no live payment, full
acceptance/Narrative language, and formatting checks. Commit with message
`Add accounts-payable mapping example prompt`.

### Task 15: Add Prompt 47, final audit, and sequence guidance

**Files:**

- Create: `prompts/47-audit-qualified-user-workflow.md`
- Modify: `README.md`

**Interfaces:**

- Consumes: all behavior from Prompts 33–46 and the four operational templates.
- Produces: `docs/qualified-user-intake-audit.md` in OntologyService and the final Builder sequence
  documentation.

- [ ] **Step 1: Specify an audit-only task**

Require an independent evidence-led audit. It may add an audit document and test-only harnesses but
must not repair runtime code, ontology inputs, mappings, generated artifacts, deployment
configuration, or acceptance checks. Failed controls remain reported failures with exact commands
and evidence.

- [ ] **Step 2: Specify the end-to-end audit matrix**

Audit:

```text
qualified submission and unauthorized rejection
receipt durability and user non-retrieval
engineer export, digest verification, and offline re-parse
deterministic report and reordered-input identity
embedding provenance and no semantic auto-acceptance
bounded LLM report, prompt-injection-shaped text, and incomplete-model behavior
repository promotion boundary and generated-artifact ownership
release manifest visibility and pending-intake non-disclosure
ontology-change proposal staleness and non-mutation
named mapping-tool purity, determinism, schema validation, and structured failures
synthetic invoice retrieval -> mapping -> target payload, with no target call
single-instance SQLite enforcement and intake-disabled multi-instance deployment
backup/restore and append-only audit evidence
```

- [ ] **Step 3: Specify audit report structure**

Require scope, environment, commit, evidence table, pass/fail/manual-gate status, threat findings,
limitations, unresolved risks, and prioritized recommendations. A manual gate names the exact
command and expected result. No skipped check may be reported as passing.

- [ ] **Step 4: Complete README sequence guidance**

Add:

```markdown
50. [Independently audit the qualified-user workflow](prompts/47-audit-qualified-user-workflow.md)
```

After the ordered list, add prose stating:

```text
Prompts 33–47 follow active Prompt 32b and form one ordered qualified-user contribution sequence.
Prompt 33 plans without changing behavior. Prompts 34–36 add the isolated single-instance intake
plane and workbench.
Prompts 37–41 execute the first five build-time embedding stages from the existing Step 4 guide.
Prompt 42 deliberately combines the guide's Prompt 6 assurance and Prompt 7 documentation stages.
Prompt 43 adds advisory coding-agent analysis, Prompt 44 adds deployed change visibility, Prompt 45
adds named pure mapping tools, Prompt 46 proves the invoice-to-payment example, and Prompt 47 audits
the whole boundary independently. Intake remains disabled in multi-instance deployments until a
later governed storage adapter is added.
```

Add an `Operational prompt templates` subsection linking all four templates and state their stage
prerequisites.

- [ ] **Step 5: Verify the complete Builder change**

Run:

```bash
set -euo pipefail
test "$(find prompts -maxdepth 1 -name '*.md' | wc -l | tr -d ' ')" -eq 50
for number in $(seq 33 47); do
  test "$(find prompts -maxdepth 1 -name "${number}-*.md" | wc -l | tr -d ' ')" -eq 1
done
test "$(find prompts/templates -maxdepth 1 -name '*.md' | wc -l | tr -d ' ')" -eq 4
awk 'length($0) > 100 { print FNR ":" length($0) ":" FILENAME }' \
  README.md prompts/{33..47}-*.md prompts/templates/*.md
for file in prompts/{33..47}-*.md; do
  rg -q '^## Acceptance criteria( and verification)?$' "$file"
  rg -q 'npm run check' "$file"
  rg -q 'git diff --check' "$file"
  rg -q 'narrative-required' "$file"
  rg -q 'Commit locally' "$file"
  rg -q 'Do not push' "$file"
done
git diff --check
git status --short
```

Expected: all count and `rg` assertions succeed, the line-length and diff checks print nothing, and
status lists only Prompt 47 and README as tracked Task 15 changes. Call out any deliberately
untracked local planning or visual-companion files separately.

- [ ] **Step 6: Review decision-bearing PR requirements and commit Task 15**

Inspect the complete diff. Confirm no `Narrative.md` or fragment change exists. Prepare a future PR
body with substantive `## Narrative Context`, `## Narrative Decision`, and
`## Narrative Consequences`; the `narrative-required` label and completed body must be applied
together before merge.

Commit:

```bash
git add README.md prompts/47-audit-qualified-user-workflow.md
git commit -m "Add qualified-user workflow audit prompt"
```

Do not push or open a pull request unless the user explicitly requests it.
