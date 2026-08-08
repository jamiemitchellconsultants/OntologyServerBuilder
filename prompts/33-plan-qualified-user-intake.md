# Prompt 33 — Plan qualified-user intake and governed mapping-tool delivery

Inspect the current built service and write a reviewable plan. Do not change runtime behavior,
ontology inputs, compiled artifacts, authentication behavior, deployment configuration, or MCP
contracts in this stage.

This stage follows Prompt 32b. It consumes the behavior produced by Prompts 3, 4, 6, 12, 17–20,
and 30–32b. It produces the architecture and migration plan that Prompts 34–47 will implement.

## Read before planning

Read the current architecture, proposal workflows, authentication, deployment, matcher, compiler,
mapping instructions, MCP surface, tests, and Project Narrative rules. In particular, establish
what the service actually does today and distinguish it from the intended qualified-user workflow.
Read the approved Builder design specification for this sequence before deciding the plan's
boundaries.

## Plan document

Create `docs/qualified-user-intake-and-mapping-tools.md`. It is the reviewable source of the
implementation sequence, not a runtime contract and not a generated artifact. It must make the
following decisions explicit:

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

The plan must reconcile Prompt 30's no-volume deployment with Prompt 31's multi-instance AWS
deployment. A later intake volume is justified only for a single instance, and intake remains
disabled where a shared durable adapter is unavailable. Do not imply that SQLite is a
multi-instance store or that deployment may enable intake without its required durable storage.

The plan must preserve the current compiler-owned generated-artifact boundary, the read-only
delivery plane, and the rule that qualified-user submissions remain review-required evidence until
engineers promote reviewed repository changes through the normal pull-request, CI/CD, and
deployment path. It must identify the relevant existing source, test, deployment, and documentation
locations for every later prompt, while leaving the implementation decisions that need new evidence
plainly open.

## Scope exclusions

Do not add an intake store, authorization capability, MCP tool, CLI command, mapping definition,
release manifest, migration, database, volume, or deployment setting. Do not edit ontology inputs
or compiled artifacts. Do not alter authentication behavior, existing MCP contracts, or generated
output. This stage is documentation and architectural planning only.

## Acceptance criteria

- `docs/qualified-user-intake-and-mapping-tools.md` contains all ten required planning decisions
  and explicitly reconciles the Prompt 30 and Prompt 31 deployment constraints.
- The plan is grounded in the current service rather than assuming future files or behavior exist.
- No ontology or generated-artifact diff.
- `npm run check` and `git diff --check` pass.

This target-service plan is a meaningful product, architecture, governance, and operational
decision. If opening a pull request, apply `narrative-required` and include substantive
`## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences` sections before
merge. Never hand-edit generated `Narrative.md`.

Commit locally with a focused local documentation message. Do not push.
