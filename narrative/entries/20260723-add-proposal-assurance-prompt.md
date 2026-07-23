---
date: 2026-07-23
slug: add-proposal-assurance-prompt
title: "Add proposal assurance prompt"
summary: "Add a separate Prompt 19 that hardens plain filenames, exercises accepted, accepted-with-gaps, and rejected cases through both store and MCP boundaries, proves deterministic and read-only no-I/O behavior, and documents rejection versus…"
kind: product
status: accepted
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/10; merge commit ef776acf7531cd8512442f6091ed7266087a6838"
---

## Context

The new proposal workflows need assurance against structurally unsafe input, but real enterprise systems often contain non-English labels, supplier-specific datatypes, unfamiliar status conventions, incomplete meanings, and evidence that resembles code or instructions. The earlier Prompt 18 wording mixed structural safety with broader rejection and stale/size policies that OntologyService deliberately did not adopt because they could block legitimate systems or add user friction.

## Decision

Add a separate Prompt 19 that hardens plain filenames, exercises accepted, accepted-with-gaps, and rejected cases through both store and MCP boundaries, proves deterministic and read-only no-I/O behavior, and documents rejection versus preservation. Explicitly keep broad content filters, stale-fingerprint enforcement, and new restrictive payload limits outside this stage. Update the conflicting Prompt 18 bullets so unfamiliar types and incomplete semantics remain review gaps.

## Consequences

Learners receive a concrete security-testing stage that mirrors OntologyService’s implemented assurance policy. They learn to reject incoherent structure while preserving unusual business metadata as inert, review-required evidence. This makes the sequence more precise and adds another maintained prompt, while avoiding a general-purpose content filter and unnecessary high-friction workflows. Future payload or stale-context controls remain separate decisions rather than being introduced implicitly by this audit.
