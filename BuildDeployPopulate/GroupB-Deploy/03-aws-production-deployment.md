# Prompt B-03 — Deploy the service to AWS for production use

Using the previously supplied repository contract, implement an operator-ready AWS deployment path
for the live service. This stage documents and automates deployment. It must not widen the MCP
surface by a single tool, resource, or argument, must not change ontology facts or generated
artifacts, and must not weaken any existing runtime trust boundary.

This stage depends on the container image and CI from Prompt A-08, the host allow-list correction from
Prompt B-01, and the in-process Entra validation from Prompt A-18. It does not depend on Prompts C-03–C-11.

Read before editing, and adapt every name and path below to the repository's current structure:

- `Dockerfile`, `.env.example`, and the homelab deployment added in Prompt B-02;
- `README.md`, `docs/architecture.md`, and the security and operations documentation;
- `src/server.ts`, the HTTP transport, and the host-validation and authentication tests;
- the CI workflow that builds and publishes the image;
- the repository's agent instructions and Narrative rules.

Two framings govern everything below.

**This is the live deployment.** Prompt B-02 could label an untested control a manual gate and move
on, because a homelab claim is a claim about one operator's house. A claim here is a claim about
production. A control that is not tested is documented as untested, never as supported.

**Infrastructure does not make the service correct.** The runtime is deterministic and read-only, so
horizontal scaling is safe in a way it is not for a service that writes. That makes availability
easy and makes it tempting to present availability as though it were the hard problem here. It is
not. The hard problem is that every replica must be serving the same ontology, and that the token
validation performed in-process is the only thing standing between a caller and the graph.

## Reconciling this stage with the repository contract

The contract forbids introducing an external database, triple store, message broker, or cloud
service. This stage does not relax it. The prohibition governs what the **running process depends
on**, not what hosts the container:

- the process makes no AWS SDK call, reads no cloud metadata service, and fetches nothing at runtime
  except the identity provider's JWKS;
- RDF and SPARQL stay in the Node process, and no managed graph, search, or cache service is
  introduced;
- configuration arrives as container environment values; if any value is genuinely secret, ECS
  injects it into the container and the server reads it exactly as it already does. The server does
  not learn to fetch secrets.

State this reconciliation in the documentation, because the next person to read the contract and the
infrastructure code together will otherwise conclude that one of them is wrong.

## Supported baseline

Choose and state one primary, reproducible baseline:

- ECS on Fargate running the Prompt A-08 image, pulled from ECR by immutable digest, built for
  `linux/amd64`, never by tag;
- an Application Load Balancer terminating TLS with an ACM certificate. The task serves plain HTTP
  on the internal VPC network and is never itself a public listener;
- an **internal** load balancer unless external reachability is an explicit, recorded requirement.
  "The clients are internal" is the common case and should not quietly acquire a public listener;
- Streamable HTTP at `/mcp`, reachable only through the load balancer;
- `MCP_AUTH_MODE=entra`, with issuer, audience, and every required role and scope configured.

Record the tested region, account boundary, Fargate platform version, image digest and source
revision, and the exact load balancer, certificate, and Entra configuration. Use placeholders for
account identifiers, hostnames, and tenant values. Everything is checked-in infrastructure as code;
nothing is created by console click and documented afterwards, because that is what makes an
environment undescribable six months later.

Note in the documentation that Entra ID as the identity provider does not imply an Azure deployment,
and that an Azure equivalent of this guide would be a separate stage with its own tested baseline —
not a second column in this one.

## Replica count and ontology version skew

Multiple tasks are safe: the runtime is read-only, the transport is stateless, and `GET` and
`DELETE` on `/mcp` return 405, so there are no sessions and session affinity is not required. State
that, and state its limit in the same breath.

The hazard is version skew. The compiled ontology ships inside the image, so the image digest **is**
the ontology release. During a rolling deployment two tasks behind one load balancer can serve two
different ontologies, and identical queries return different answers with no error anywhere. The
service already exposes the material fact: `/health` reports the source fingerprint. Use it.

- every task in a service runs the same digest, and the deployment definition makes a mixed-digest
  steady state impossible rather than merely unlikely;
- document the rolling-deployment window as the interval during which answers may differ, name how
  long it lasts in the tested configuration, and state what a client observes;
- alarm on fingerprint divergence across tasks, or state precisely why the chosen deployment
  strategy makes divergence unobservable and prove it;
- if a deployment strategy that avoids the mixed window entirely is chosen, say which and what it
  costs.

Autoscaling that can move the task count is out of scope. If it is added later, the reason the count
is what it is must be written where whoever adds the scaling policy will read it.

## Network boundary

Document the concrete layout and the full request path from an allowed MCP client to `/mcp`,
accounting for DNS, the listener, the target group and its health check, and the task's placement.

- tasks run in private subnets with no public IP;
- the task security group accepts inbound traffic **only** from the load balancer's security group,
  by security group reference rather than by CIDR. A CIDR that happens to cover the load balancer's
  subnet is a weaker statement that admits whatever later occupies that range;
- egress is scoped to named destinations: the identity provider's JWKS endpoint, and the AWS
  endpoints required for image pull and logs. Prefer VPC endpoints where they remove a NAT path
  entirely. A default `0.0.0.0/0` egress rule is a finding, not a baseline;
- the production runtime never fetches source definitions or standards documents. If the task needs
  outbound access to anything beyond JWKS and AWS endpoints, that is evidence of a runtime fetch the
  contract forbids — investigate it rather than widening the rule.

Prove as tests, not as assertions, that the task's port is unreachable from anywhere except the load
balancer, and that the service is unreachable over plain HTTP from outside the VPC.

## Host validation and the target-group health check

Determine by test, not by assumption, whether the host allow-list applies to `/health` as well as to
`/mcp`.

It matters because an ALB target-group health check sends the target's own IP address in the `Host`
header and cannot be given a custom one. If the allow-list covers `/health`, every health check
fails, every task is killed and replaced, and the deployment presents as an application crash loop
with no application error in the logs. Decide the resolution deliberately — an ECS container-level
health check against loopback, an allow-list value that covers the health path, or a documented
change with tests — and prove the chosen resolution before deploying. Do not resolve it by removing
host validation.

Set the health-check grace period from the measured time for the process to start and load the
compiled ontology, not from a default. Set target-group thresholds and timeouts from the same
measurement.

## TLS and forwarded headers

TLS terminates at the load balancer. Configure the trusted proxy hop to be the load balancer
specifically, not a wildcard, and prove that a request arriving with a spoofed forwarded-protocol or
forwarded-host header from an untrusted source does not change the server's view of the connection
or defeat host validation. That test is what distinguishes a configured trust boundary from a
decorative one.

Enforce a modern TLS policy on the listener, redirect or refuse plain HTTP at the listener, and
document certificate renewal and the failure mode if renewal lapses. Ensure the proxy configuration
does not buffer responses in a way that defeats streamed MCP replies, and that its idle timeout
exceeds the longest expected response.

## Identity

Document issuer, audience, required roles and scopes, JWKS cache lifetime, and behavior when the
JWKS endpoint is unreachable at startup and at renewal. Unreachable at startup must fail closed.

Prove on the artifact actually deployed that `MCP_AUTH_MODE` is `entra`, that no static token value
is present anywhere in the task definition, infrastructure code, or image, and that selecting
`static` or `none` in this environment is either impossible or a documented, alarmed
misconfiguration rather than a silent downgrade. Keep `/health` unauthenticated and minimal.

Record explicitly which parts of the organisation's MCP authorization profile, gateway controls, and
production trust policy remain outstanding. A VPC boundary and a load balancer are not an
authorization profile, and the gateway still owns rate limiting, request-size limits, audit logging,
and network policy.

## Secrets

In `entra` mode this service holds no outbound credential. The tenant identifier, audience, issuer,
and required roles and scopes are configuration, not secrets. Do not invent a vault story for values
that are not secret; a task definition full of Secrets Manager references for public identifiers is
noise that hides the one thing that would matter.

If the deployment genuinely carries a secret, then: ECS injects it via `valueFrom` from Secrets
Manager or SSM SecureString, the task role grants access to exactly those ARNs and nothing broader,
no value appears in the task definition, infrastructure code, state files, or logs, and a rotation
procedure and its observable effect on a running service are documented. Infrastructure state files
are a real leak path and are routinely forgotten by scanning aimed at the repository and the image.

## Resources, observability, and operations

- size task memory from the resident set measured after the ontology is loaded and a representative
  query has run, with stated headroom. An out-of-memory kill on a memory-resident graph presents as
  a crash loop with no application error, and the graph grows every time the ontology does. Alarm on
  memory utilisation, not only on task health;
- structured logs to CloudWatch Logs carrying the correlation identifier and no token, claim set, or
  full payload. Document the retention period;
- alarms on task health, restart count, load balancer 5xx rate, memory utilisation, and the rate of
  401 responses. A 401 spike after a certificate, audience, or app-registration change is the
  signature of a misconfiguration that leaves the service healthy and useless, and nothing else
  reports it;
- deployment by immutable digest with rollback, retaining the auditable source revision, and stating
  plainly that a rollback is an ontology rollback: callers will see the previous fingerprint and the
  previous facts;
- a runbook covering deployment, rollback, JWKS unavailability, fingerprint divergence, memory
  exhaustion, and certificate renewal, written for whoever is on call rather than for whoever wrote
  it.

## Automated validation

Add a validation target that:

- validates the infrastructure definitions and rejects unresolved placeholders;
- proves the effective task definition carries every hardening control Prompt A-08 and Prompt B-02
  require — unprivileged user, read-only root filesystem, dropped capabilities, no privileged mode,
  bounded CPU and memory — and declares no persistent volume;
- proves the task definition contains no plaintext secret and no static bearer token, and references
  only the intended ARNs if any;
- proves the task security group admits only the load balancer, by reference, and that egress is
  scoped to named destinations;
- proves the image is referenced by digest and that every task in the service uses the same digest;
- verifies readiness, and that the reported fingerprint matches the compiled artifacts in the
  deployed image;
- verifies through the load balancer that unauthenticated, malformed, expired, wrong-issuer,
  wrong-audience, and insufficient-role or insufficient-scope requests are refused, using locally
  minted test tokens and a local JWKS fixture rather than the live tenant;
- verifies a spoofed forwarded header from an untrusted source changes nothing;
- verifies the registered MCP surface is unchanged from the stage that last defined it;
- tears down only resources carrying the validation run's unique name.

Anything that cannot be validated safely against a non-production account is a labelled manual gate
with an exact expected result and a place to record what was observed. Skipped checks are never
reported as passing — Prompt B-02's rule, and it does not relax because this environment matters more.

## Acceptance criteria

- The service is reachable only over TLS, only through the load balancer, and only by authenticated
  callers holding the configured roles and scopes.
- Every task serves the same image digest, and the ontology version skew window is documented,
  bounded, and observable.
- The task security group admits only the load balancer by reference; egress is scoped to named
  destinations.
- No credential, static token, or secret value in the image, repository, task definition,
  infrastructure code, state files, or logs.
- Host validation and in-process token validation are intact, and the target-group health check
  works without weakening either.
- Task memory is derived from measurement, and exhaustion is alarmed rather than diagnosed later.
- Deployment and rollback use immutable digests, retain auditable source revisions, and state the
  ontology consequence of a rollback.
- Deferred corporate alignment and remaining gateway responsibilities are recorded as open, not
  quietly treated as closed.
- The MCP surface, ontology facts, and generated artifacts are unchanged by this stage.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This is a meaningful architecture and operational decision. If opening a pull request, apply
`narrative-required` and include substantive `## Narrative Context`, `## Narrative Decision`, and
`## Narrative Consequences` sections recording the supported baseline, the reconciliation between
the contract's no-cloud-service rule and a managed hosting substrate, the network boundary and how
it is proven, the TLS termination point and trusted hop, the treatment of ontology version skew, and
which corporate-alignment items remain open. Never hand-edit generated `Narrative.md`.

Commit locally with a focused deployment message. Do not push.
