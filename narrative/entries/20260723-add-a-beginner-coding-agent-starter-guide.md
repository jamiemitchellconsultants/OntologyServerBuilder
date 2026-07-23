---
date: 2026-07-23
slug: add-a-beginner-coding-agent-starter-guide
title: "Add a beginner coding-agent starter guide"
summary: "Add a standalone beginner guide before the numbered implementation prompts."
kind: product
status: accepted
evidence: "https://github.com/jamiemitchellconsultants/OntologyServerBuilder/pull/5; merge commit c7e84523f162aea7bebc0cf2f0b32dd9bcd9de08"
---

## Context

The learning sequence previously began by assuming that a learner already had a GitHub repository and knew how to connect a coding agent to it. Junior developers who are unfamiliar with AI coding tools could therefore be blocked before reaching the first teaching prompt, or could accidentally open the prompt repository instead of the repository they are meant to build. The onboarding material also needed to distinguish local agents from cloud agents without making editor extensions another prerequisite.

## Decision

Add a standalone beginner guide before the numbered implementation prompts. The guide starts by creating a private `MyOntologyServer` repository, recommends cloning it with GitHub Desktop and using one local desktop agent first, and provides concrete setup routes for Codex and Claude Code. GitHub Copilot's cloud agent is included as the principal cloud alternative. A read-only verification prompt confirms that the selected agent is attached to the correct repository before implementation begins. VS Code integrations remain an optional appendix for learners who already use the editor.

## Consequences

New learners gain a complete path from an empty GitHub account context to the first implementation prompt, while the numbered ontology-building sequence stays focused on engineering concepts. Recommending one local route first makes branches, working-tree changes, commits, and pushes visible, but learners can still choose a cloud workflow. Product interfaces and plan availability can change, so the guide links to official provider documentation and will require occasional maintenance. Covering multiple providers adds some documentation breadth, but avoids presenting a single commercial agent as mandatory.
