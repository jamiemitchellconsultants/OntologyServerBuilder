---
date: 2026-08-08
slug: move-qualified-user-intake-storage-to-s3-add-mcp-server-registration-tem
title: "Move qualified-user intake storage to S3; add MCP-server registration template and skill"
summary: "Replace the `IntakeStore` adapter's storage technology from SQLite to S3-compatible object storage."
kind: product
status: accepted
sequence: 2026-08-08T19:55:46.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/33; merge commit 22994c4f747526b4d2b0420ac46fe888faba78ce"
---

## Context

The approved 2026-08-07 design spec picked SQLite as a deliberate single-instance intake baseline,
accepting that Prompt 31's multi-instance AWS deployment would keep intake disabled until a later
governed storage adapter arrived. Prompts 33-47 had not yet been played against `OntologyService`,
so they remained editable specifications rather than shipped behavior. Revisiting the target
deployment shape surfaced that S3-compatible object storage is easier to deploy against the intended
AWS production architecture, and that the home-lab environment already runs a LocalStack S3-compatible
endpoint, giving both environments the same code path instead of a SQLite file on one and a database
service on the other.

Separately, the existing four operational templates are all engineer-facing and assume repository
write access. Walking through the qualified-user side of the workflow (a user's agent connecting to
a supplier's newly announced MCP server) showed there was no equivalent packaged instruction for the
user's own submission step, which today has to be reconstructed by hand from Prompts 18 and 35 each
time.

## Decision

Replace the `IntakeStore` adapter's storage technology from SQLite to S3-compatible object storage.
Model each submission as one object written once with a create-only conditional request (payload +
initial `received` event together, so the write is atomic without a multi-object transaction);
enforce subject+idempotency-key uniqueness with a separate lookup object under the same create-only
precondition; and record every later lifecycle event as its own append-only object under a
submission-scoped, sortable key prefix, giving `list()` a native pagination cursor. Because
create-only conditional writes are a server-side atomic guarantee rather than a client-side lock,
the single-instance restriction is removed outright rather than carried forward: a multi-instance
deployment, including Prompt 31's AWS baseline, may enable intake once its bucket and access scope
are provisioned and verified. `INTAKE_S3_ENDPOINT`/`INTAKE_S3_FORCE_PATH_STYLE` let the same adapter
target LocalStack at home and real AWS S3 in production.

Add `register-supplier-mcp-server.md` as a fifth template, but keep it deliberately narrow: it only
covers MCP-server registration, not OpenAPI, WSDL, or GraphQL. Qualified users are expected to
recognize an MCP server from everyday AI-agent use but not to distinguish interface-description
formats; those other formats continue through direct engineer/supplier onboarding, where the
engineer is likely to wrap them in an MCP server anyway before a qualified user would touch them.
This template also differs in kind from the other four: it runs entirely in the qualified user's own
MCP-capable client and never touches the `OntologyService` repository, so it was added as a
separately labeled entry in both the README and the design spec's template list rather than folded
into the existing engineer-facing group.

Add Prompt 48 to convert that template into a Claude Skill rather than leaving it as a
copy-paste-only instruction. It only depends on Prompt 35's exact tool names, so it is numbered
last purely to avoid renumbering the already-audited 33-47 sequence, not because it depends on
Prompts 36-47. The skill is placed at `skills/register-supplier-mcp-server/SKILL.md` in
`OntologyService`, not under `.claude/skills/`, because the latter only auto-loads for an engineer
with that repository open in Claude Code — the wrong audience for a qualified user who has no
reason to have the repository checked out at all.

## Consequences

Every future coding-agent session that plays Prompts 34-36 will build the intake store against S3
rather than SQLite, and homelab/production deployment guidance now covers bucket IAM scope,
versioning, and a LocalStack-compatible endpoint override instead of file-based backup and restore.
Multi-instance AWS deployments gain the option to enable intake, which is a materially larger blast
radius than the original single-instance design and will need its own verification once Prompt 34 is
actually implemented and audited under Prompt 47. The qualified-user-facing template adds a fifth
reusable prompt with a distinct governance surface (no repository access, no pull request results
from it) alongside the four engineer-facing ones. Prompt 48 adds a sixth prompt-sequence stage and a
new top-level `skills/` distribution surface in `OntologyService` that did not exist before; whoever
plays Prompt 48 also has to document, in `docs/registering-systems.md` or its equivalent, how a
qualified user installs the skill, since installing it grants no access by itself.

`docs/superpowers/plans/2026-08-07-qualified-user-ontology-intake.md` is deliberately left
unchanged: it is tracked as a historical record of the reasoning that produced the already-merged,
SQLite-based Prompts 33-47, and rewriting it would misrepresent that history. This PR's narrative
entry is the record of the correction; the old plan stays as evidence of what was originally decided
and why it needed revisiting.
