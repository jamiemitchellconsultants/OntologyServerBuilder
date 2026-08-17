# Prompt B-02 — Deploy one instance to a homelab or local network

Using the previously supplied repository contract, implement an operator-ready deployment path for
**one** service instance on a trusted home-lab or local-network host. This stage documents and
automates deployment. It must not widen the MCP surface by a single tool, resource, or argument,
must not change ontology facts or generated artifacts, and must not weaken any existing runtime
trust boundary.

This stage depends on the container image and CI from Prompt A-08, the host allow-list correction from
Prompt B-01, and the in-process authentication from Prompt A-18. It does not depend on Prompts C-03–C-11: a
repository that has not run the finance and accounting migration deploys exactly the same way.

Read before editing, and adapt every name and path below to the repository's current structure
rather than assuming these are the names it uses:

- `Dockerfile`, `.env.example`, and any existing deployment configuration;
- `README.md`, `docs/architecture.md`, and the security and operations documentation;
- `src/server.ts`, the HTTP transport, and the host-validation and authentication tests;
- `package.json` scripts, particularly the build, check, and ontology compile/check targets;
- the repository's agent instructions and Narrative rules.

The Windows 11 / Docker Desktop / WSL 2 / PowerShell 7 baseline defined below is the **deployment
target**, not the project's CI or general test environment. Do not change Prompt A-08's workflow, add a
Windows CI job, or make any test outside this stage depend on a Windows host. Prompt A-08's CI keeps
building and testing on the runner it already uses, producing the `linux/amd64` image this stage
deploys. Only checks that are inherently Windows-specific — Defender Firewall rules, NTFS ACLs,
certificate-store trust, Task Scheduler, Docker Desktop's container mode — are Windows-only, and
those are the manual gates named under "Automated validation".

## Supported baseline

Choose and state one primary, reproducible baseline:

- a Windows 11 host with Docker Desktop, its WSL 2 Linux-container backend, and the bundled Docker
  Compose plugin;
- one server container using the image produced by Prompt A-08, pinned by immutable digest;
- exactly one instance. The runtime is read-only and stateless, so this is a simplicity choice
  rather than a correctness constraint — say so, and do not present the single instance as a safety
  property it is not;
- Streamable HTTP at `/mcp`, reachable from explicitly allowed LAN clients only;
- `MCP_AUTH_MODE=static`, which this stage treats as the only acceptable homelab mode.

Use PowerShell 7 for host-side commands and label every command as running in non-elevated
PowerShell, elevated PowerShell, WSL, or the container. Equivalent Windows Server, Hyper-V, Podman,
or NAS steps may be noted but never claimed supported unless tested. Record the tested Windows
edition and build, CPU architecture, WSL version, Docker Desktop and engine versions, Compose
version, container mode, and client environment. Use placeholders for hostnames, addresses, and
credentials. Preflight must detect and fail when Docker is in Windows-container mode.

State Docker Desktop's licensing, interactive-logon, automatic-start, update, and availability
assumptions explicitly. Do not describe a container restart policy as proof that the stack starts
after a Windows reboot. Provide and test a least-privilege Task Scheduler startup procedure if
unattended reboot recovery is claimed; otherwise document operator sign-in and Docker Desktop
startup as a manual availability gate.

## Deployment artifacts

Add a `deploy/homelab/` directory containing a production-quality example Compose file and
checked-in PowerShell preflight, validate, start, stop, upgrade, and rollback scripts. Set strict
mode, stop on errors, reject unresolved placeholders, and change no machine-wide policy. Pin the
image by immutable digest or require the operator to supply one; never deploy a tag. Configure:

- the image's unprivileged user, a read-only root filesystem, dropped Linux capabilities,
  `no-new-privileges`, and a small writable temporary filesystem;
- bounded CPU and memory, with the memory limit derived from the resident set size measured after
  the ontology is loaded, plus stated headroom — not from a round number;
- a restart policy, log rotation, and a documented log location or driver that captures no secrets;
- a health check against `/health`;
- an explicit port binding, to a dedicated LAN address rather than every interface;
- every required configuration value, with fail-closed placeholder detection.

**No persistent volume.** The runtime is read-only, holds its graph in memory, and ships its
compiled artifacts inside the image. A volume in this deployment almost always means the compiled
ontology is being mounted from the host, which breaks the guarantee that the deployed artifacts are
the reviewed, committed, image-pinned ones. If the implementation genuinely requires writable state,
say what state and why, rather than adding a volume quietly.

Do not mount the container engine socket. Do not use host networking, privileged mode, embedded
credentials, a plaintext bearer token in the Compose model, or an automatically downloaded
deployment script. Keep deployment files outside OneDrive and other synchronized folders, quote
paths containing spaces, and document Docker Desktop file-sharing behavior for every bind mount.

Four constraints sit inside the list above, and none of them announces itself while you follow it.

**An unset Compose variable is not an error.** It resolves to an empty string, and a port mapping
whose host part is empty publishes on every interface — the exact exposure this stage exists to
prevent, produced by following it, with no warning at any point. Decide how the model fails closed
on a missing value and prove it by rendering the model with nothing supplied. A model that is only
ever rendered with every value present cannot show you the difference.

**The image contains `node` and no HTTP client.** Node's global `fetch` is the health check;
installing `curl` or `wget` into the delivered image to make the probe convenient is a change to the
artifact Prompt A-08 pruned and Prompt A-09 audited. Note also that `GET /mcp` returns 405 by design, so
`/health` is the only endpoint a probe can usefully call.

**Host validation sees whatever `Host` header the proxy sends.** A reverse proxy that rewrites
`Host` to the backend service name produces a uniform rejection that looks like an authentication
fault and is not one. Either preserve the client's `Host` or add the value the proxy actually sends
to `ALLOWED_HOSTS`, and prove which by test rather than by reading the proxy's defaults.

**`MCP_AUTH_MODE=none` will look reasonable on a LAN.** It is not permitted here. The allow-list is
DNS-rebinding protection, not authentication, and a trusted network is not a caller identity.

## Network boundary and TLS

Document a concrete network layout and the full request path from an allowed MCP client to `/mcp`,
accounting for the Windows host, Docker Desktop's WSL 2 virtual network, published container ports,
and the reverse-proxy container.

A reverse proxy on the same host performs TLS termination. The server container speaks plain HTTP on
an internal Compose network; only the proxy binds the LAN-facing TLS port; the server's port is
never a LAN listener. Bind only the TLS entry point to the intended Windows LAN address. A trusted
private overlay providing authenticated encryption may be noted as an alternative for a reader with
different constraints, but is not the documented, tested path.

Give idempotent PowerShell commands using Windows Defender Firewall to create narrowly named inbound
rules for the selected TCP port, `LocalAddress`, `RemoteAddress` clients or VLAN, profile, and
program or service where meaningful, with default-deny behavior. Include equally exact inspection
and removal commands that affect only those rules.

Include a minimal pinned proxy configuration covering: correct forwarding for Streamable HTTP,
including response buffering disabled so a streamed MCP response is not held until completion;
request and idle timeouts; body-size limits; the `Host` handling decided above; and a test proving
the unencrypted backend port is unreachable from another LAN machine. Do not publish the service to
the public internet, configure router port forwarding, or describe a self-signed certificate as
trusted without installing its CA certificate on each client.

Use Windows certificate-store terminology and PowerShell certificate inspection commands. If a
private CA is used, install only its public root certificate into the appropriate Trusted Root store
on each authorized client, never the CA private key. Explain hostname and SAN matching, prefer a
stable local DNS name over a raw address, and never disable certificate validation in example
clients.

## Authentication and the shared token

Document how an operator:

1. generates a bearer token of at least the enforced minimum length from a cryptographic source on
   an administrator-controlled machine;
2. supplies it to the container through a file or secret reference rather than a Compose literal, a
   command line, or an environment value typed into a shell;
3. protects that file with explicit NTFS ACLs for only the deploying Windows account and required
   administrators, set and verified with idempotent `Get-Acl` and `icacls` commands that never print
   the contents;
4. configures the LAN MCP client to send `Authorization: Bearer <token>`;
5. proves that a missing, malformed, and wrong token are refused before any MCP handling occurs, and
   that `/health` remains reachable without one;
6. rotates the token, and knows what breaks while rotation is in progress.

Explain the effective access seen inside the Linux container and do not claim that a container UID
owns the host file. Do not rely on Unix mode bits on a Windows bind mount as the host-side access
control boundary.

State plainly what a single shared token is and is not: it authenticates the token, not a caller. It
carries no per-caller identity, no roles or scopes, no revocation short of rotating every client,
and no audit attribution. That is the reason it is confined to a trusted host, and the reason the
migration to `entra` mode is a real migration rather than a configuration change. Preserve Prompt A-18's fail-closed HTTP startup, and never represent a LAN boundary, reverse proxy, VPN, or private CA
as a substitute for that migration.

## Operator procedure

Write a copy-and-pasteable guide covering:

1. Windows, WSL 2, virtualization, Docker Desktop, Linux-container, PowerShell, and architecture
   prerequisites;
2. creating deployment directories outside synchronized folders and protecting them with NTFS ACLs;
3. obtaining and verifying the image digest and its source revision;
4. preparing configuration, the token file, certificates, and permissions;
5. running preflight checks before the first start;
6. starting the stack and waiting for readiness;
7. testing TLS, network isolation, host validation, authentication, and the exact MCP surface with
   one read-only request;
8. recording the `sourceFingerprint` reported by `/health` and confirming it matches the compiled
   artifacts in the deployed image;
9. connecting one LAN MCP client, using placeholders where client-specific syntax is unknown;
10. collecting redacted diagnostics;
11. upgrading by digest and rolling back, noting that an image change is also an ontology change;
12. stopping and removing the deployment, naming which state and secrets remain on the host.

Every command must state where it runs. Use elevation only for narrowly scoped prerequisites,
certificate-store changes, firewall rules, ACL ownership where required, and Task Scheduler
registration. Commands must be safe to paste after placeholder substitution, avoid destructive
wildcards, fail on unset values, and preserve PowerShell's execution policy rather than weakening
it.

## Automated validation

Add a validation target that:

- validates the Compose model and rejects unresolved placeholders, including the empty-value case;
- proves the effective container settings include every hardening control above and declare no
  persistent volume;
- starts the stack with a fixture token and verifies readiness and the reported fingerprint;
- verifies HTTPS access and that `/mcp` accepts an authenticated POST from an allowed client;
- verifies that missing, malformed, and wrong bearer tokens are refused, and that `GET` and `DELETE`
  on `/mcp` still return 405;
- verifies that a request with a disallowed `Host` value is refused and that `/health` remains
  unauthenticated and exposes nothing beyond its documented fields;
- verifies direct access to the unencrypted backend port and a simulated disallowed client are
  refused;
- inspects Windows port bindings, firewall rules, certificate hostname and trust, Docker's
  Linux-container mode, and the documented reboot-start behavior;
- verifies the registered MCP surface is unchanged from the stage that last defined it;
- tears down only resources carrying the run's unique project name.

Anything that cannot be automated safely — Defender Firewall, NTFS ACLs, certificate-store trust,
Windows port-binding inspection, container-mode detection, reboot and Docker Desktop restart
recovery, Task Scheduler registration, and the second-machine refusal test — is labelled a manual
gate with the exact command, the exact expected result, and a place to record what was observed. Do
not report a skipped check as passing.

## Acceptance criteria

- A learner can deploy one instance from a clean supported Windows host using only the checked-in
  guide, example files, and a locally supplied token.
- The service is encrypted and reachable only by explicitly allowed local-network clients.
- No bearer token appears in the image, repository, Compose model, logs, PowerShell history, or
  generated documentation.
- HTTP startup still fails closed on missing or invalid authentication configuration, and host
  validation is still enforced.
- The deployment declares no persistent volume, or names the state it requires and why.
- Reboot and Docker Desktop restart recovery is honestly proven or explicitly qualified.
- Upgrade and rollback use immutable digests, retain auditable source revisions, and state that the
  ontology version moves with the image.
- The MCP surface, ontology facts, and generated artifacts are unchanged by this stage.
- `npm run check` and `git diff --check` pass, and generated ontology artifacts have no unexplained
  diff.

This is a meaningful operational decision. If opening a pull request, apply `narrative-required` and
include substantive `## Narrative Context`, `## Narrative Decision`, and `## Narrative Consequences`
sections recording the supported baseline, the network boundary, the shared-token limitation and why
it is confined to a trusted host, the absence of persistent state, and the deliberately unsupported
variants. Never hand-edit generated `Narrative.md`.

Commit locally with a focused deployment message. Do not push.
