---
date: 2026-07-31
slug: add-homelab-and-aws-deployment-prompts
title: "Add homelab and AWS deployment prompts"
summary: "Add two prompts, homelab first, and derive their specifics from what the built OntologyService actually exposes rather than from the source sequence."
kind: product
status: accepted
sequence: 2026-07-31T04:59:04.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/18; merge commit 66095a9f47cb2248bfc660a9fd692254adb90e87"
---

## Context

The sequence taught a learner to build, govern, secure, and migrate the ontology service, and then
stopped. Nothing told them how the thing they built is actually run, so the deployment boundaries
stated in Prompt 7 and Prompt 20 — TLS at a fronting proxy, gateway responsibilities, the
distinction between host validation and authentication — remained assertions in documentation that
no stage ever exercised.

BrightFlagProxyMCPBuilder had already solved the same problem for a different service, with a
homelab stage and an AWS production stage. Its structure transfers: one stated tested baseline,
hardening controls expressed as validation assertions rather than prose, an explicit network and TLS
boundary, an automated validation target, and an honest manual-gate list for anything that could not
be verified. Its *content* does not transfer, because that service pays invoices and this one is a
deterministic read-only graph server. A direct copy would have taught a payment-shaped set of
concerns to a service that has none.

## Decision

Add two prompts, homelab first, and derive their specifics from what the built OntologyService
actually exposes rather than from the source sequence.

Where the two services differ, the prompts differ:

- **No persistent volume.** The runtime is read-only and ships its compiled artifacts in the image.
  A volume almost always means the ontology is being mounted from the host, which defeats the
  guarantee that the deployed artifacts are the reviewed, committed, image-pinned ones.
- **Health checks.** The image carries `node` and no HTTP client, and `GET /mcp` returns 405 by
  design, so `/health` is the only usable probe target. Installing `curl` to make the probe
  convenient changes the artifact Prompt 7 pruned and Prompt 8 audited.
- **`ALLOWED_HOSTS` behind a proxy.** A proxy that rewrites `Host` produces a uniform rejection that
  reads as an authentication fault and is not one. The load-balancer form is worse: an ALB
  target-group health check sends the target IP as `Host` and cannot be given a custom one, so if
  host validation covers `/health`, every task is killed and replaced and it presents as an
  application crash loop with no application error in the logs.
- **Version skew replaces the source sequence's concurrency dependency.** Replicas are safe here —
  stateless transport, read-only runtime, no sessions — but the image digest *is* the ontology
  release, so a rolling deployment can leave two tasks answering identical queries differently. The
  service already reports `sourceFingerprint` from `/health`, so Prompt 31 requires that signal be
  used and alarmed rather than inventing a new mechanism.
- **Contract reconciliation.** The repository contract forbids introducing a cloud service. Prompt
  31 states that the prohibition governs what the running process depends on, not what hosts the
  container: no AWS SDK call, no metadata service, ECS injects and the server does not fetch. It
  also states that in `entra` mode the service holds no outbound credential, so a task definition
  full of Secrets Manager references for public identifiers is noise that hides anything real.

Rejected: a single combined deployment prompt, which would have collapsed a recoverable environment
and a production one into one baseline; and an Azure variant alongside the AWS one, since Entra ID
as the identity provider does not imply an Azure deployment and a second column is not a tested
baseline. Also rejected: adding a `plans/` directory to mirror the source repository, which has no
precedent here.

## Consequences

The sequence now ends with a learner able to run what they built, and the deployment boundaries
asserted in Prompts 7 and 20 acquire stages that test them. Both prompts state one baseline and
label everything they could not verify as a manual gate with an exact expected result, so a reader
can tell support from aspiration.

The cost is two more prompts to maintain against a moving target: Docker Desktop, ECS, and Entra all
change independently of this repository, and the prompts name concrete behaviours of each. The
version-skew treatment in Prompt 31 depends on `/health` continuing to report a source fingerprint;
if that response changes, the alarm the prompt requires becomes unimplementable and the prompt is
wrong rather than merely stale.

Deliberately left open: autoscaling policy, an Azure or other-cloud equivalent, and the parts of an
organisation's MCP authorization profile that a VPC and a load balancer do not constitute — Prompt
31 requires those be recorded as outstanding rather than treated as closed.
