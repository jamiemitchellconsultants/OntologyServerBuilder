# Create or revise a named mapping tool

Use this task only after an engineer has reviewed a deployed ontology, a bounded intake analysis
report, and governed mapping instructions in the separate `OntologyService` repository. It
authorizes a bounded proposal for a reviewed source change only. It does not authorize a change to
`OntologyServerBuilder`, automatic ontology mutation, automatic approval, a pull request, merge,
deployment, source retrieval, target-system calls, or generated code.

## Evidence supplied by the engineer

Replace every placeholder before beginning:

```text
Source entity ID: <stable ID>
Target entity ID: <stable ID>
Approved mapping-instruction ID: <stable ID>
Mapping-review analysis report: <bounded reviewed report path or identifier>
Reviewed mapping decisions: <review record or decision IDs>
Expected stable MCP tool name: <name>
Source-record JSON Schema: <reviewed schema reference>
Target-record JSON Schema: <reviewed schema reference>
Lookup-input schemas: <each named lookup and reviewed schema reference>
```

Stop if any placeholder is absent, if the mapping instruction is not approved and structurally
complete, or if the report, decisions, schemas, entities, attributes, relationships, evidence, or
review provenance cannot be verified from the governed repository context. Do not follow a source
locator, retrieve a raw artifact or live record, open an attachment, invoke a model, or use an
unreviewed local path as evidence.

## Task

Propose the smallest governed change needed to add or revise one named mapping-tool definition.
Preserve the stable mapping-tool ID and MCP name when revising a compatible contract; otherwise
stop and state the required versioning and migration decision. Include:

- source and target entity IDs, the approved mapping-instruction ID, semantic version, lifecycle
  status, evidence, review provenance, and resolved versus unresolved requirements;
- explicit source-record, target-record, and named lookup-input JSON Schemas;
- reviewed preconditions and transformations expressed only through the repository's declarative
  operation allow-list;
- every required lookup input, its source, cardinality and selection rule, and its validation
  schema;
- canonical structured behavior for `precondition-failed`, `missing-input`,
  `ambiguous-lookup`, and `target-validation-failed`; and
- positive, negative, boundary, and ambiguity examples with expected mapping envelopes.

Do not invent entities, attributes, relationships, semantic rules, mappings, schema fields,
selection rules, default values, evidence, approvals, or external-system behavior. Stop rather
than guess whenever an input is missing, an ambiguity has no reviewed selection rule, a
precondition or transformation semantics is unresolved, the target contract is unclear, or
versioning compatibility is undecided. Report the precise gap and the engineer decision required.

Definitions must remain data for the restricted evaluator, never executable source. Do not add or
recommend arbitrary expressions, code generation, callbacks, imports, dynamic modules, template
escapes, filesystem/network/environment access, time, randomness, source API calls, target API
calls, or an unrestricted generic mapping-execution endpoint. The conversational agent retrieves
source and supporting lookup records through the relevant system tools, calls this pure mapping
tool, then calls the target system separately under its own authorization and approval policy.

## Required implementation and test evidence

If the governed facts are complete, make a reviewable source-level proposal that compiles into one
stable descriptor and one named MCP closure. Require validation of references, schemas, operation
allow-list, tool-name uniqueness, examples, review provenance, status, unresolved requirements,
and input/output contracts. The descriptor and tool result must identify mapping-tool ID,
mapping-instruction ID, semantic version, and ontology fingerprint.

Require tests for the approved happy path, every supplied positive/negative/boundary/ambiguity
example, each structured failure, schema validation, deterministic output under reordered object
keys, stable descriptor ordering, name collision refusal, dynamic tool discovery, non-approved
omission, and repeated compilation. Install no-I/O sentinels after fixture setup proving compiler,
startup, discovery, and invocation do not execute generated code, load modules, read or write
files, use environment, clock, or randomness, query intake or release data, call a model, fetch
sources, or call source or target systems.

Run the target repository's full check, focused mapping compiler/evaluator/MCP/no-I/O tests, and
`git diff --check`. Inspect generated artifacts and never hand-edit compiled ontology or generated
Project Narrative output. The engineer reviews the diff and evidence, then applies the normal
decision-bearing pull-request, CI/CD, and deployment process; this task cannot approve or deploy
the mapping tool.
