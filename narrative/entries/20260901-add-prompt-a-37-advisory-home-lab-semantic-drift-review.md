---
date: 2026-09-01
slug: add-prompt-a-37-advisory-home-lab-semantic-drift-review
title: "Add Prompt A-37: advisory home-lab semantic-drift review"
summary: "Added Prompt A-37 as an advisory-only review rather than a blocking check: it runs a model on the operator's own home-lab gateway (private network, not a third-party API), a finding never turns the job red, and the job skips cleanly in…"
kind: product
status: accepted
sequence: 2026-09-01T10:09:39.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/45; merge commit bded580831c631dfcfc0bd74b6304a9d3c23f2a2"
---

## Context

Earlier stages give the compiler and CI deterministic, model-free checks against ontology drift
(term reuse, compiled-artifact fingerprinting). Those checks catch structural drift but not prose
that silently redefines or contradicts a canonical concept in documentation, guides, or generated
docs — something only a model reading for meaning can flag.

## Decision

Added Prompt A-37 as an advisory-only review rather than a blocking check: it runs a model on the
operator's own home-lab gateway (private network, not a third-party API), a finding never turns
the job red, and the job skips cleanly in any built repository with no home lab configured. This
is deliberately the weakest kind of dependency on a model this project takes on, and it inherits
Prompt A-31's rule that text arriving from outside the repository is inert quoted data, not
instructions. It does not attempt a deterministic term check over prose and says so, rather than
presenting the advisory review as a control it is not.

## Consequences

This is the first stage in which the project's CI talks to a model at all. Built repositories
without a home-lab gateway get no coverage from this stage, by design — it is additive advisory
signal for operators who have one, not a required gate. The prompt can run any time after the
compiled ontology is available and depends on the Keycloak, intake-submission, and engineer-
workbench stages (Prompts A-19, A-23, and A-24) for the home-lab and intake context it reads.
