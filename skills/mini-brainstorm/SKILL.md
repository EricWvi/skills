---
name: mini-brainstorm
description: "Use when the user has a clear feature idea and needs project-specific implementation planning, codebase research, incremental recommendations, and a running decision log in docs/BRAINSTORM.md."
---

# Mini Brainstorm for Implementation

Use this skill when the user already knows what they want to build, but needs help fitting it cleanly into the current project architecture.

## Goal

Turn a feature idea into a concrete, repo-aware implementation plan through short back-and-forth discussion.

## Workflow

1. Start from the user's feature intent and identify the smallest useful implementation scope.
2. Inspect the codebase and nearby architecture before suggesting changes.
3. Reply with a brief recommendation first, not a long design essay.
4. Continue the conversation iteratively; if the topic deepens or the architecture needs a fuller explanation, switch to a more detailed response.
5. After every round of discussion, update docs/BRAINSTORM.md with the current state of the conversation.
6. When the plan is nearly settled, ask the user for confirmation before treating the outcome as final.
7. After the user confirms, consolidate docs/BRAINSTORM.md into the final artifact for the feature discussion.

## Note Log Rules

Keep docs/BRAINSTORM.md as the single running record of the discussion.

- Record the user's goal in plain English.
- Summarize codebase findings, constraints, and tradeoffs.
- Capture the current recommendation and any open questions.
- Update the document every round, even if the update is small.
- Keep the log concise while the discussion is still exploratory.

## Response Style

- Prefer short, actionable feedback early in the discussion.
- Be specific about repository structure, ownership boundaries, and integration points.
- Only expand into a detailed explanation when the user asks for depth or the design truly needs it.
- Avoid generic brainstorming that is disconnected from the current project.

## Completion Criteria

Treat the discussion as ready to finalize only when the implementation approach, ownership boundaries, and major risks are clear and the user has explicitly agreed.
