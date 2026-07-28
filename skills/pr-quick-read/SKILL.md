---
name: pr-quick-read
description: Quickly understand a GitHub PR or Git diff at high level and build lasting familiarity with the codebase. Use when user asks to review, understand, summarize, or explain a PR/diff or wants to get familiar with the project through PRs. Focus on intent, architecture context, related modules, and mental model building. Skip low-level syntax noise especially in Rust.
---

# PR Quick Understand + Codebase Familiarity

## Goal

Two outcomes in one review:

1. **Immediate**: User grasps the valuable information of this PR/diff in under 1 minute.
2. **Lasting**: User leaves the review with a clearer mental model of the surrounding codebase (modules, data flow, ownership, key abstractions).

Never do traditional bug-hunting or style nits. Treat the PR/diff as a guided tour of the relevant part of the system.

## Core principles (strict)

1. **Comprehension + Orientation first**  
   Explain *what* changed and *where it sits* in the larger system. Only surface high-level risks (architectural, behavioral, data, compatibility).

2. **Ignore language-level noise**  
   Especially in Rust: never discuss `.map()`, `.map_err()`, `.and_then()`, `?`, `unwrap`/`expect` (unless system-level error semantics change), iterator chains, simple match arms, etc. Treat them as pure implementation detail.

3. **Build the mental model**  
   Always connect the PR/diff to:
   - which modules / crates / packages own the changed concepts
   - upstream callers and downstream consumers
   - key types / traits / interfaces that act as boundaries
   - how data or control flows through the area

4. **Stay concise but educational**  
   Prefer short bullets. Every sentence should either explain the PR or teach the user something durable about the codebase.

## Required workflow

1. Gather context with `gh` (preferred):
   ```bash
   gh pr view <number> --json title,body,author,baseRefName,headRefName,files,additions,deletions
   gh pr diff <number>
   ```
   If needed, also explore related files/modules with `gh` or local tools to understand ownership and neighbors.

2. Read title + description first, then look at the file list and module structure before reading individual diffs.

3. Produce output **strictly** in the format below.

## Output format (mandatory)

Write in the user's language. Use this structure:

```markdown
## 一句话总结
<这个 PR 解决了什么问题 / 带来了什么价值>

## 核心变更
- **<模块或概念>**: <做了什么 + 为什么重要>
- ...

## 它在代码库中的位置（帮你建立全局感）
- **所属模块 / 边界**: <这个变更属于哪个子系统、哪个 crate/package、主要责任是什么>
- **上游依赖它的地方**: <谁会调用 / 依赖这里的结果>
- **下游被它影响的地方**: <它改变了谁的行为或数据>
- **关键抽象 / 类型 / 接口**: <必须记住的几个核心类型或 trait（只列真正重要的）>

## 影响面
- 行为变化：...
- API / 数据 / 状态：...
- 兼容性 / 迁移：...
- 性能或资源（仅明显时）：...

## 关键决策点
- <设计选择或权衡>
- ...

## 需要你亲自确认的点
- <最多 3 条高阶判断题。没有则写「无」>

## 建议下一步（帮你更熟代码库）
1. 先看：<最值得立即打开的 1-2 个文件 + 原因>
2. 相关扩展阅读：<同模块或相邻模块中值得顺手扫一眼的文件 / 入口>
3. 记忆锚点：<用一句话记住这个区域的核心职责，方便以后回忆>
```

## What to explicitly avoid

- Traditional review comments (“consider adding a test”, “this could be more idiomatic”).
- Listing or criticizing Rust syntax patterns.
- Expanding every changed function body.
- Inventing problems. Clean focused PRs get short, positive orientation.

## Deeper mode

When user says “深入解释 XXX”、“重点看这个文件” or “帮我更熟这个模块”:
- Zoom into the requested part.
- Still stay above pure syntax.
- Explicitly map data/control flow and ownership.
- Point out 1-2 neighboring files that complete the mental model.

## Tools

Prefer `gh` for GitHub data.  
When exploring the codebase for context (module ownership, callers, related types), use available local tools or `gh` as needed.  
If no network/`gh`, work from the provided diff + any local files the user supplies.
