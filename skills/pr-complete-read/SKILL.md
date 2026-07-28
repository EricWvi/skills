---
name: pr-complete-read
description: Read every change in a GitHub PR or Git diff and write a complete, neutral, file-by-file walkthrough to pr-read.md at the repository root. Use when the user asks to fully read, deeply understand, or explain all PR/diff changes without reviewing, judging, summarizing only highlights, or looking for bugs.
---

# PR Complete Read

Produce a durable walkthrough of every changed file. Explain implementation facts and behavior changes without evaluating the PR.

## Hard boundaries

- Read every diff hunk and cover every changed file, including tests, migrations, generated contracts, configuration, and lockfiles.
- Do not review the implementation. Omit praise, criticism, risks, bugs, style comments, alternatives, recommendations, and questions for the author.
- Do not collapse the result into an overall PR summary. Organize the explanation by file.
- Distinguish facts visible in code from necessary inference. Phrase an inference as such and never invent a deletion rationale.
- Write the result to `pr-read.md` in the repository root. Replace an existing `pr-read.md` because it represents the current requested diff.
- Do not modify any project file other than `pr-read.md`.

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
- Explain the behavior before the change and after the change.
- Explain how changed data or control enters and leaves that local area when relevant.
- For an added function, state its purpose, inputs, and output. If it is complex, also describe its internal stages.
- For a deleted function, briefly explain why it is no longer needed based on the diff, replacement call path, PR description, or history. If the reason is not provable, say that the diff only shows which replacement now owns the responsibility.
- Cover changed tests by explaining the scenario and observable behavior they encode, not by narrating assertion syntax.

Do not explain unchanged functions merely because they share the file. Read enough unchanged context to make each hunk understandable.

### Added mode

Use for an actual added file. Also use it for a modified file whose diff replaces or rewrites most of the file, while keeping `M` as its tree status.

Explain it in this order:

1. Give one short paragraph describing the file's responsibility and its place in the surrounding module.
2. Reproduce every struct/class/record/data type defined by the resulting file in full. Add a concise explanatory comment after every field. Preserve important types, generics, visibility, and optionality. For tuple structs or enum payload fields, annotate each component. Do not omit private fields.
3. For each simple function or method, state its inputs and output, including important side effects.
4. For each complex function or method, additionally describe its major internal stages, branches, and collaborators.
5. Explain constants, traits/interfaces, enums, module exports, and tests when they contribute behavior or define a boundary.

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
# PR 逐文件阅读

## Diff 文件树

```text
.
├── path
│   ├── changed.rs (M)
│   └── new.rs (A)
└── old.ts (D)
```

## 推荐阅读顺序

1. `path/to/entry.rs` — 从控制流入口开始
2. `path/to/model.rs` — 接着理解核心数据

## 1. `path/to/entry.rs` (M)

### 文件职责

...

### 改动讲解

#### `function_or_hunk_name`

- 改动前：...
- 改动后：...

## 2. `path/to/new.rs` (A)

### 文件职责

...

### 数据结构

```rust
struct Example {
    id: Id,          // 该对象的稳定标识
    value: String,   // 交给下游处理的值
}
```

### 函数

#### `simple(input) -> Output`

- 输入：...
- 输出：...

#### `complex(input) -> Output`

- 输入：...
- 输出：...
- 内部逻辑：...
````

The first section must be a complete directory tree of diff paths with status annotations. After the tree, include the recommended reading order, then exactly one numbered section per file.

Adapt subsection names to the file type. Do not add an overall verdict, impact assessment, risk list, review findings, or suggested next steps.

## 5. Verify completeness

Before finishing:

1. Compare every path from the canonical diff inventory with the tree in `pr-read.md`.
2. Confirm every path has exactly one numbered file section.
3. Confirm every diff hunk is addressed in its file section.
4. For every Added-mode file, confirm all resulting structures and all functions are covered, with every structure field annotated.
5. Confirm Deleted-mode explanations do not claim an unsupported reason.
6. Search the document for evaluative language and remove review conclusions or implementation judgments.
7. Confirm only `pr-read.md` was changed in the project outside the skill's own creation or maintenance task.

Return a concise completion message with the path to `pr-read.md` and the number of files covered.
