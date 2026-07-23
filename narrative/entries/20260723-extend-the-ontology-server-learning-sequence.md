---
date: 2026-07-23
slug: extend-the-ontology-server-learning-sequence
title: "Extend the ontology server learning sequence"
summary: "Extend the guide beyond initial reconstruction and establish Narrative before substantive implementation decisions."
kind: product
status: accepted
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/2; merge commit 8f2ea4c345e81f693a6dba950b1bd62ceef32357"
---

## Context

OntologyServerBuilder ended with an independent reconstruction audit, while OntologyService had
continued to mature through operational corrections, repository-governance improvements,
security-testing plans, an embedding-matcher plan, coding-agent instructions, and a clearer
general-purpose product position. Junior developers following the guide could reproduce the
initial implementation but could not learn how review findings and later product feedback become
bounded, governed follow-up work.

The guide also introduced Project Narrative late in the sequence, after many of the architecture
and governance decisions that Narrative is intended to preserve. Keeping the shorter sequence would
have reduced the amount of material, but it would have omitted the maintenance and decision-making
work that turns a successful prototype into a sustainable repository.

## Decision

Extend the staged learning sequence with prompts covering the implemented OntologyService
milestones: the container host allow-list correction; reuse, security, and contribution policies;
an evidence-based `nextsteps.md`; prompt-driven Step 2 security testing and Step 4 embedding guides;
canonical coding-agent instructions; and domain-neutral product positioning.

Add a dedicated Project Narrative bootstrap immediately after the scaffold, before heterogeneous
source ingestion and the remaining substantive implementation stages. Require the learner to
publish and merge that bootstrap before later decision-bearing pull requests. Continue using
finance as the first reference domain while keeping the service contract and teaching language
general purpose.

## Consequences

Learners now see both initial construction and the follow-up cycle of audit, prioritisation,
correction, governance, and roadmap design. The sequence is longer and requires more deliberate
checkpoints, but each new stage remains independently reviewable and includes explicit scope,
acceptance criteria, and verification.

Narrative becomes available early enough to capture later decisions through its normal two-PR
workflow. The guide must keep prompt numbering, cross-references, upstream Narrative instructions,
and the distinction between platform contracts and finance reference data current as the
repositories evolve.
