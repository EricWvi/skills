---
name: pr-complete-read
description: Read every change in a GitHub PR or Git diff and write a complete, neutral, file-by-file walkthrough to pr-read.md at the repository root. Use when the user asks to fully read, deeply understand, or explain all PR/diff changes without reviewing, judging, summarizing only highlights, or looking for bugs.
---

# PR Complete Read

Produce a durable walkthrough of every changed file. Explain implementation facts and behavior changes without evaluating the PR.

Write at the logic level a reviewer needs to understand the change. Prefer responsibilities, control flow, data flow, state transitions, contracts, side effects, and observable behavior over source-level mechanics.

## Hard boundaries

- Read every diff hunk and cover every changed file, including tests, migrations, generated contracts, configuration, and lockfiles.
- Inspect every hunk, but do not force every syntactic edit into the document. Fold edits with the same purpose into one logical explanation.
- Do not review the implementation. Omit praise, criticism, risks, bugs, style comments, alternatives, recommendations, and questions for the author.
- Do not collapse the result into an overall PR summary. Organize the explanation by file.
- Distinguish facts visible in code from necessary inference. Phrase an inference as such and never invent a deletion rationale.
- Write the result to `pr-read.md` in the repository root. Replace an existing `pr-read.md` because it represents the current requested diff.
- Do not modify any project file other than `pr-read.md`.
- Omit source mechanics that do not change behavior or clarify architecture. Do not narrate imports, module declarations, re-exports, visibility keywords, local variable renames, formatting, ownership moves, borrows, clones, temporary values, destructuring, or equivalent language syntax unless they materially change lifetime, concurrency, API boundaries, state ownership, or observable behavior.
- Do not create headings for import-only or declaration-only edits. Incorporate a newly connected component into the relevant behavioral explanation instead.

For example, describe a session-creation change as "session creation now schedules a delayed title refresh that reads the current title from the external service and persists it." Omit that a working-directory value was cloned, clients were copied into a closure, or helper functions were imported unless one of those details changes the behavior or ownership boundary.

## 1. Resolve the diff

Accept a PR number/URL, commit range, branch range, or already supplied diff.

For a GitHub PR, read metadata and the complete diff before individual files:

```bash
gh pr view <pr> --json title,body,author,baseRefName,headRefName,files,additions,deletions
gh pr diff <pr>
```

For a Git range, use the range exactly as supplied:

```bash
git diff --name-status <range>
git diff --stat <range>
git diff --find-renames <range>
```

Use `rg`, `git show`, and surrounding source files to understand types, callers, callees, and pre-change behavior. If commits are locally available, prefer reading both file snapshots with `git show <before>:<path>` and `git show <after>:<path>`. Do not check out the PR merely to read it.

Record one canonical inventory containing every path and its actual Git status:

- `M`: modified
- `A`: added
- `D`: deleted
- Normalize a rename into the old path marked `D` and the new path marked `A`, so the output tree uses only `M`, `A`, and `D`. Explain the destination with Added mode and use the Deleted section to point to the move without implying feature removal.

## 2. Classify how each file must be explained

Use the actual Git status in the tree. Separately choose the explanation mode below.

### Modified mode

Use when the meaningful hunks are local changes inside an otherwise stable file.

For every diff hunk:

- Identify the enclosing function, method, type, constant, test, or configuration section.
- Determine the hunk's logical purpose and group it with other hunks that implement the same behavior.
- Explain behavior before and after when both sides matter to understanding the change. For purely additive behavior, describe the new flow directly instead of manufacturing a repetitive before/after pair.
- Explain how changed data or control enters and leaves that local area when relevant.
- For an added function, state its purpose, role in the flow, meaningful inputs and outputs, and important side effects. If it is complex, also describe its internal stages.
- For a deleted function, briefly explain why it is no longer needed based on the diff, replacement call path, PR description, or history. If the reason is not provable, say that the diff only shows which replacement now owns the responsibility.
- Cover changed tests by explaining the scenario and observable behavior they encode, not by narrating assertion syntax.
- Skip prose about syntax-only edits after confirming they have no independent behavioral meaning.
- Reproduce the resulting implementation of an important changed function or method directly in a fenced code block. Include a reasonably sized function in full. For a very large function, preserve its signature, every changed line, and enough surrounding control flow to make the change understandable; replace only unrelated regions with a language-appropriate omission comment such as `// ... unchanged validation ...`. Never use an omission that hides changed behavior or makes branches, error handling, or data flow ambiguous.

Do not explain unchanged functions merely because they share the file. Read enough unchanged context to make each hunk understandable.

### Added mode

Use for an actual added file. Also use it for a modified file whose diff replaces or rewrites most of the file, while keeping `M` as its tree status.

Explain it in this order:

1. Give one short paragraph describing the file's responsibility and its place in the surrounding module.
2. Reproduce every struct/class/record/data type defined by the resulting file in full. Add a concise explanatory comment after every field. Preserve important types, generics, visibility, and optionality. For tuple structs or enum payload fields, annotate each component. Do not omit private or incidental fields. After the declaration, briefly explain the type's semantic role and any important relationships or invariants.
3. Group functions and methods by responsibility or call flow. For simple helpers, one collective explanation is enough when their individual mechanics do not add understanding.
4. For complex or central functions, reproduce the implementation using the same full-or-elided rule from Modified mode, then describe their major stages, branches, collaborators, outputs, and side effects.
5. Explain constants, traits/interfaces, enums, exports, and tests only when they contribute behavior or define a boundary. Do not narrate boilerplate declarations.

If the file contains no structures or functions, explicitly say so and explain the relevant declarations or generated data instead.

### Deleted mode

Briefly describe the deleted file's former responsibility and why it was removed. Ground the reason in its replacement, moved destination, removed feature, PR description, or visible caller changes. If the reason is not explicit, state only what ownership or call path replaced it.

### Mechanical files

Still give these files their own sections:

- For migrations, explain schema state before and after, including up/down behavior.
- For generated contracts, explain the public shape that changed and where it is consumed; do not enumerate generator boilerplate.
- For lockfiles, state which dependency entries changed and which manifest change caused them when visible.
- For snapshots or fixtures, explain what expected behavior changed.
- For binary files, record that the diff cannot be explained textually and describe only metadata or usage that can be verified from the repository.

## 3. Choose the reading order

Order sections to help a reader follow the change's control and data flow, rather than blindly using alphabetical order. A useful default is:

1. Entry points and orchestration
2. Core domain logic and key abstractions
3. Persistence and external integrations
4. Public contracts and adapters
5. Callers or UI consumers
6. Tests, fixtures, generated files, migrations, and configuration near the implementation they validate

Adjust this order when another dependency direction makes the change easier to understand. Every inventoried file must appear exactly once regardless of order.

## 4. Write `pr-read.md`

Write in the user's language. Use this structure:

````markdown
# PR File-by-File Walkthrough

## Diff File Tree

```text
.
├── path
│   ├── changed.rs (M)
│   └── new.rs (A)
└── old.ts (D)
```

## Recommended Reading Order

1. `path/to/entry.rs`
2. `path/to/model.rs`

## 1. `path/to/entry.rs` (M)

### File Responsibility

...

### Behavioral Changes

#### `function_or_hunk_name`

```rust
fn function_or_hunk_name(...) {
    // ... unchanged setup ...
    changed_behavior();
    // ... unchanged cleanup ...
}
```

- Previous behavior: ...
- New behavior: ...

## 2. `path/to/new.rs` (A)

### File Responsibility

...

### Data Model

```rust
struct Example {
    id: Id,          // Stable identifier for this object
    value: String,   // Value passed to downstream processing
}
```

`Example` represents ... Its identity is ..., while ... controls ...

### Main Flow

...

### Supporting Operations

These helpers collectively ...
````

The first section must be a complete directory tree of diff paths with status annotations. After the tree, include the recommended reading order, then exactly one numbered section per file.

In the recommended reading order, list only file paths. Do not append a rationale or description to any item.

Adapt subsection names to the file type. Do not add an overall verdict, impact assessment, risk list, review findings, or suggested next steps.

## 5. Verify completeness

Before finishing:

1. Compare every path from the canonical diff inventory with the tree in `pr-read.md`.
2. Confirm every path has exactly one numbered file section.
3. Confirm every diff hunk was inspected and either contributes to a logical explanation or is intentionally omitted as syntax-only or mechanical detail.
4. For every Added-mode file, confirm every resulting structure is reproduced in full and every field is annotated, while its responsibilities, semantic data model, central flows, externally relevant behavior, and important side effects are also covered.
5. Confirm Deleted-mode explanations do not claim an unsupported reason.
6. Search the document for evaluative language and remove review conclusions or implementation judgments.
7. Search for low-level narration about imports, module declarations, clones, moves, borrows, local temporaries, and similar syntax; remove it unless it explains a material behavior or ownership boundary.
8. Confirm the recommended reading order contains paths only, with no per-file descriptions.
9. Confirm important changed functions are reproduced in full when reasonably sized; for elided functions, confirm every changed line and the necessary control-flow context remain visible.
10. Confirm only `pr-read.md` was changed in the project outside the skill's own creation or maintenance task.

Return a concise completion message with the path to `pr-read.md` and the number of files covered.
