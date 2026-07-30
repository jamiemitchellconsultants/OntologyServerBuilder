---
date: 2026-07-30
slug: add-binding-agent-instructions-harden-prompts-2-and-13
title: "Add binding agent instructions; harden Prompts 2 and 13"
summary: "Prompt 13 keeps `AGENTS.md` canonical and now specifies all six pointers, with the reason stated: an agent whose tool has no pointer sees no instructions, so a missing pointer is not cosmetic — and equally, a pointer must not cite a…"
kind: product
status: accepted
sequence: 2026-07-30T05:27:46.000Z
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/16; merge commit 32c8f72a14241b07a8a0da5f8c1c63f035781f72"
---

## Context

Prompt 13 already required a canonical `AGENTS.md` with pointer files, which is the right design and the reason `OntologyService` has exactly `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` and the Copilot file. It specified **three** pointers, so Cursor, Windsurf and Cline had none — an agent running under any of those saw no project instructions at all, including the ontology change workflow and the rule against hand-editing compiled artifacts. Those three pointers were just added to `OntologyService` directly, so the prompt had drifted from the artifact it specifies and rebuilding would reintroduce the gap.

Prompt 13 is also **Prompt 13**. Stages 3 through 12 are decision-bearing and would each be executed by an agent with no instruction file, because the stage that creates one arrives near the end. Prompt 2 installs the narrative mechanism and, until now, required nothing that would make it discoverable.

Four rules were missing from what both prompts teach, each corresponding to an actual failure in a sibling repository this week:

- A pull request was labelled `narrative-required` before its body carried the three sections and merged inside that window. The maintenance run failed **permanently**: the action reads the body from the merge event payload, so a re-run reads the same incomplete text.
- Four consecutive pull requests merged with no entry, because `gh pr create --body ...` replaces the repository template wholesale and the template was where the label rule lived.
- An automation-proposed entry conflicted with a hand-written one on the compiled `Narrative.md`. "Never edit the compiled file" did not obviously cover "resolve a conflict in it", and hand-reconciling the markers would have produced an index the next compile silently discards.
- An accepted entry's reasoning was contradicted by a later decision, where editing it in place would have erased the evidence that the framing ever needed correcting.

`OntologyService` already prevents the first of these with a pre-merge `require-narrative-sections` job. Neither prompt taught it.

## Decision

Prompt 13 keeps `AGENTS.md` canonical and now specifies all six pointers, with the reason stated: an agent whose tool has no pointer sees no instructions, so a missing pointer is not cosmetic — and equally, a pointer must not cite a location the repository does not contain, or the set reads as maintained when it is not. Its narrative rules gain the four failures above.

It also states that the prohibition on hand-editing generated output governs compiled ontology artifacts and the compiled narrative as **one principle applied twice**, rather than two unrelated rules. These prompts already treat determinism as the thing that makes generated output disposable; saying so once makes both cases harder to erode.

Prompt 2 gains the same rules in its authoring contract, plus the pre-merge gate, because the mechanism must be discoverable from the first decision-bearing stage rather than from Stage 13. Prompt 13 remains the owner of the canonical-file *policy*; Prompt 2 requires the contract be recorded, and `CLAUDE.md` §1 in this repository names Prompt 13 as the owner so the two do not drift again.

The gate deliberately does not check for a *missing* label — only a human can classify a change — and must read the body from an environment variable, since pull-request prose is untrusted input.

For this repository itself, `CLAUDE.md` is the source of truth, with §1 recording that finance is the first reference domain rather than a restriction on the service, and that a prompt hard-coding finance into the service's own contracts is a defect.

## Consequences

Servers built from this sequence get instructions all six tier-one agents can read, from Stage 2 rather than Stage 13, and the label-without-sections failure is caught while a pull request is still open.

The prompt and its artifact are aligned again. Rebuilding `OntologyService` from the sequence now produces the pointer set it actually has.

Prompt 2 and Prompt 13 now overlap on narrative content, which is a real cost: two places can drift. Mitigated by Prompt 13 owning the canonical-file policy explicitly, and by this repository's `CLAUDE.md` naming that ownership — but it remains something to check when either changes.

Not addressed: an unlabelled decision-bearing pull request still produces silence. Catching that requires judging whether a change is decision-bearing, which is a human call.

`narrative check` passes. No ontology contract or compiled artifact is touched.
