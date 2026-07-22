# Prompt 6 — Delivery, security, documentation, and Narrative

Using the previously supplied repository contract, implement Stage 6: production packaging and
repository governance.

Add:

- a multi-stage Node 22 Alpine Dockerfile;
- production dependency pruning;
- an unprivileged runtime user;
- GitHub Actions using Node 22;
- `npm ci`;
- `npm run check`;
- `npm audit --audit-level=high`;
- a pull-request template;
- complete README usage and MCP connection instructions;
- architecture documentation;
- system-registration and human-review documentation.

Document these deployment boundaries explicitly:

- Production HTTP must sit behind TLS and enterprise authentication.
- Authorization, tenant isolation, rate limiting, and audit logging belong at the gateway.
- The service must not be directly exposed to an untrusted network.
- An external triple store is a future adapter, not a current dependency.
- Model-assisted matching may be added only at build time with recorded model and version evidence.

Adopt the Project Narrative mechanism from `jamiemitchellconsultants/Narrative`:

- Meaningful product, architecture, governance, operational, correction, or experimental pull
  requests use the `narrative-required` label.
- Qualifying pull requests contain Narrative Context, Narrative Decision, and Narrative
  Consequences.
- `Narrative.md` is generated from reviewed fragments and is never edited directly.
- Add validation and post-merge Narrative workflows following the upstream repository contract.

Ensure the documentation explains the main decision:

Keep deployment to one Node process and one container, using generated OWL and SHACL, build-time
inference, and in-process SPARQL. Accept the larger application dependency tree and memory-resident
query data to avoid the cost and complexity of operating a separate triple store.

Run the complete verification suite and build the container if Docker is available. Commit locally
but do not push.
