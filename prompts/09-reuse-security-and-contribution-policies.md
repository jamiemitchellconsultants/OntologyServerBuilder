# Prompt 9 — Add reuse, security, and contribution policies

Using the previously supplied repository contract, close the repository-policy gaps discovered by
the reconstruction audit.

Read before editing:

- `README.md`
- `docs/architecture.md`
- `docs/registering-systems.md`
- `package.json`
- `.github/pull_request_template.md`
- the Project Narrative documentation and workflows

Add the following repository-level policies:

1. An MIT `LICENSE` file and matching `package.json` license metadata.
2. `SECURITY.md` covering:
   - private vulnerability reporting;
   - the supported-version policy;
   - compiler/runtime separation;
   - reviewed source snapshots;
   - governed traversal;
   - constrained SPARQL;
   - host-header validation;
   - the unprivileged container;
   - gateway-owned TLS, authentication, authorization, rate limiting, audit logging, and tenant
     isolation;
   - a ban on credentials, live responses, personal data, and production transaction data in
     fixtures.
3. `CONTRIBUTING.md` covering:
   - Node.js prerequisites and development commands;
   - the compile → review → commit ontology workflow;
   - generated-artifact ownership;
   - sensitive-data restrictions;
   - pull-request quality expectations;
   - the `narrative-required` process;
   - the rule that `Narrative.md` is generated and never edited directly.
4. README links to the license, contribution guide, and security policy.

Keep the documents consistent with the implemented code. Do not promise authentication, tenant
isolation, live source ingestion, full OWL compliance, or a supported deployment mode that does not
exist.

## Acceptance criteria

- Reuse terms are explicit and package metadata agrees.
- Vulnerability reports have a private route.
- Contributors have one accurate governed workflow.
- Security boundaries agree across README, architecture, security policy, and runtime behavior.
- `npm run check` and `git diff --check` pass.

This is a meaningful governance decision. If opening a pull request, apply `narrative-required` and
write substantive Narrative Context, Narrative Decision, and Narrative Consequences.

Commit locally with a focused documentation/governance message. Do not push.
