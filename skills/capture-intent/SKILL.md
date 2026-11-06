---
name: capture-intent
description: >-
  Interview the user and write one uncommitted root INTENT.md. Use when the
  user asks to capture or shape intent, wants to explore an idea or defect
  before filing an issue, or asks for INTENT.md. Does not create a GitHub
  issue.
---

# Capture intent

**Scope:** Interview the user and write a single `INTENT.md` at the target repository root. The file is a working draft and must not be committed. This skill does not create a GitHub issue. After the human accepts the draft, `create-feature`, `create-task`, or `create-bug` files that issue later.

## Overwrite

When starting a new `INTENT.md`, check for a root `INTENT.md`. If it exists, ask before overwriting it and wait. If the user declines, stop and leave the file unchanged. If it is absent, or the user agrees to replace it, continue.

When revising a draft already written in this conversation, read root `INTENT.md` first so any edits the user made are preserved, then update it in place without asking to overwrite.

## Interview

Ask one question at a time in chat and wait for the user's answer before asking the next. Let each answer shape the next question until each section below can be filled without inventing details. Ground the questions in the target repository:

- Read files that bear on the desire.
- Use git facts: repository root, status, and recent commits.
- Use `gh` facts — the repo and open issues — when the desire may overlap work already filed.

Gather enough detail so the later issue skill (`create-feature`, `create-task`, or `create-bug`) can fill its `.github/ISSUE_TEMPLATE/` headings from `INTENT.md` without re-interviewing:

- **Feature or task:** user stories or goal, affected processes and components, and observable acceptance criteria.
- **Bug:** steps to reproduce, current vs. expected behavior, and observable acceptance criteria.

Do not use an editor-only question tool. Do not run `gh issue create` or any other command that files, edits, or comments on an issue.

## Write INTENT.md

Write root `INTENT.md` with only these headings, in this order. Every heading is present. Do not add headings.

```markdown
# Problem

# Proposed outcome

# Affected users and systems

# Constraints

# Open questions
```

- **Problem:** What is missing or broken. For a defect, include steps to reproduce and current behavior.
- **Proposed outcome:** User stories or goal, expected behavior, and observable acceptance criteria.
- **Affected users and systems:** Who is affected, and the processes, components, and files involved.
- **Constraints:** Boundaries, non-goals, or technical constraints. When the user has none, say so under this heading.
- **Open questions:** Unresolved questions, or state that none remain.

## Recommend a type

After the file is written, tell the user to review and edit `INTENT.md`, recommend one type from this table, and then stop. Name the later skill: Feature → `create-feature`, Task → `create-task`, Bug → `create-bug`.

| Type | When to use it | What happens next |
| ---- | -------------- | ----------------- |
| Feature | The work must be broken into child tasks. A change that does not need a breakdown is a task. | Child tasks via `create-tasks` after the feature exists. Never `PLAN.md` on the feature. |
| Task | Always one unit of work. Stand-alone, or a child of a feature when the human attaches it. Never a parent. | A leaf task is planned on request. |
| Bug | A defect. Always stand-alone, with no child tasks. A short report uses `create-bug` with no `INTENT.md`. A larger defect may start as `INTENT.md` and then become one bug. | Always one issue, then `PLAN.md`. |

A short defect still gets `INTENT.md` when this skill was asked to write it. The recommendation can note that a future short report may use `create-bug` with no `INTENT.md`.

Do not draft the issue and do not start the next skill.
