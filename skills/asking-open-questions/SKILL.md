---
name: asking-open-questions
description: "Use when writing specs or plans and you need to collect unresolved questions in docs/questions.md before continuing."
---

# Asking Open Questions

Use this skill when a spec or plan cannot move forward cleanly because one or more requirements, decisions, or assumptions are still unclear.

## Workflow

1. While drafting a spec or plan, identify every blocking uncertainty as soon as it appears.
2. Record each unresolved question in `docs/questions.md` using one bullet per question and the exact format below.
3. Keep `A:` empty until the user answers.
4. Continue drafting around the open questions only when the missing detail does not block progress.
5. After the user answers, update the spec or plan, then delete the resolved question and answer entry from `docs/questions.md`.
6. Repeat until no blocking questions remain.

## Question Format

```text
- Q: ...
   A:
- Q: ...
   A:
```

## Rules

- Use `docs/questions.md` as the single place for open questions.
- Keep questions concise, specific, and decision-oriented.
- Prefer one question per bullet instead of combining multiple unknowns.
- Remove resolved entries promptly so the file reflects only active blockers.
- If a question is no longer relevant because the spec or plan changed, delete it instead of leaving stale context behind.

## Completion Check

This workflow is done when the spec or plan is complete and `docs/questions.md` contains no unresolved questions related to the work.
