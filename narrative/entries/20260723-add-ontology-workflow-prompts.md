---
date: 2026-07-23
slug: add-ontology-workflow-prompts
title: "Add ontology workflow prompts"
summary: "Add three bounded prompts after the existing general-purpose positioning stage."
kind: product
status: accepted
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/8; merge commit 79e74201b0a3ffe93c098806bf348f2ebb40ce1f"
---

## Context

OntologyService has gained governed Brightflag registration, mapping-instruction, and user-proposal workflows that are not yet represented in the junior-developer training sequence. Learners also need a clearer reminder that a labelled decision-bearing implementation PR is followed by a second, separately reviewed Narrative proposal PR.

## Decision

Add three bounded prompts after the existing general-purpose positioning stage. Each prompt preserves the offline compiler and read-only runtime trust boundaries, specifies rejection tests and acceptance criteria, and requires the normal Project Narrative classification. Add an explicit sequence-level reminder that the automated Narrative-only proposal is reviewed and merged separately without the `narrative-required` label.

## Consequences

Learners can now reproduce the real-system registration, governed transformation, and user-to-owner refinement capabilities in deliberate stages. The extra stages lengthen the course and must be maintained alongside OntologyService contracts, but they avoid collapsing source onboarding, executable mapping guidance, and untrusted proposal handling into one oversized prompt. The optional adversarial audit stage remains intentionally shelved.
