---
date: 2026-08-17
slug: split-prompts-into-build-deploy-populate-groups
title: "Split prompts into Build/Deploy/Populate groups"
summary: "Added a parallel, reorganized view (`BuildDeployPopulate/`) rather than renumbering or moving `prompts/` itself, because `prompts/` is the canonical, execution-ordered sequence that `README.md` and every prompt's cross-references depend…"
kind: product
status: accepted
sequence: 2026-08-17T11:40:17.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/37; merge commit 1870274140102a5f28a2c34357ff00648750781f"
---

## Context

The 53-prompt sequence in `prompts/` is ordered strictly by execution order, which conflates
three different concerns: building the generic ontology-service engine, deploying a built
instance, and populating the ontology with finance-domain content. Understanding "which
prompts are infrastructure vs. domain content" required manually reading all 53 files, since
the numbering alone doesn't signal it.

## Decision

Added a parallel, reorganized view (`BuildDeployPopulate/`) rather than renumbering or moving
`prompts/` itself, because `prompts/` is the canonical, execution-ordered sequence that
`README.md` and every prompt's cross-references depend on, and reordering it would violate the
repository rule that stages are submitted in order, one per agent task. The new directory is
purely a categorized reference copy.

Classification followed two axes: whether a prompt adds generic mechanism vs. finance-domain
ontology content (Build vs. Populate), and, within mechanism, whether a prompt is about running
a built instance somewhere (Deploy) vs. the mechanism itself (Build). Borderline calls: prompt
44 (release-change visibility) went to Deploy despite being implemented as an intake-plane
feature, because its purpose is entirely about deployment-release visibility; prompts 32/32b
(Keycloak OAuth mode, JWKS-failure distinction) stayed in Build rather than Deploy because they
add a capability the service has everywhere, independent of where it's hosted — only prompt 32a,
which wires that mode into the homelab deployment specifically, is Deploy.

## Consequences

`BuildDeployPopulate/` will drift from `prompts/` if new prompts are added there without a
matching update here; keeping the two in sync is a manual step, not an enforced invariant. The
classification and internal renumbering are a snapshot as of this PR — prompt 49, which landed
on `main` after this reorganization was first drafted, was included as `A-35` to keep the copy
current at merge time, but no automated check keeps future additions in sync.
