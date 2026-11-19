---
name: git-commit
description: >-
  Commit the current changes, choose the message from the branch, and push
  after a successful commit. On main, confirm whether to commit there or
  create a branch first. On a branch named for a GitHub issue, the first
  commit follows the issue intent; later commits on any non-main branch are
  WIP. Use when the user asks to commit, create a commit, or write the commit
  message for the current changes.
---

# Git commit

Commit the current changes, then push that commit. Do not update git config. Do not skip hooks (`--no-verify`). Do not amend unless the user explicitly asks. Do not force-push.

## Gather

Run these in parallel from the repository root:

- `git status`
- `git diff` and `git diff --staged`
- `git log -8 --oneline`
- `git branch --show-current`
- `git rev-parse --verify main`
- `git rev-list --count main..HEAD`

If this directory is not a git repository, stop.

If `HEAD` is detached, stop and ask. Do not commit.

If there is nothing to commit, stop. Do not create an empty commit.

If `main` does not exist, stop and ask which ref is the base. Do not guess, and do not treat another branch as `main`.

Leave secrets unstaged (`.env`, credentials, keys, tokens). If the user asked to commit one, warn and leave it out.

## Which message

Apply the first match.

1. **Current branch is `main`.** Formulate the message from the current changes. Follow **On main**. Do not commit yet.
2. **Not `main`, and `git rev-list --count main..HEAD` is greater than 0.** This is a commit after the first. The subject is `WIP -` plus a short imperative description of the current changes. A `feature/` prefix and a GitHub issue do not change this. Example: `WIP - keep the eject latch closed`.
3. **Associated with a GitHub issue, and the count is 0.** This is the first commit. Formulate the message from the intent of that issue. State the outcome the issue asks for, from its title and body. Keep it true of the diff.
4. **Not `main`, not associated with a GitHub issue, and the count is 0.** This is the first commit. Formulate the message from the current changes.

Do not list files. Follow **Message** for every commit. Do not copy `git log` when it conflicts with those rules.

## Message

1. Use the imperative mood. Write the subject line as a command.
2. Limit the subject line to 50 characters. Keep it concise so it doesn't get truncated in git log --oneline or GitHub UIs.
3. Do not capitalize the subject line. Start it with a lowercase letter. On a WIP commit, `WIP` is all uppercase and the rest of the subject stays lowercase. Write the body in normal sentence case.
4. Do not end the subject line with a period. Save space and treat it like a title.
5. Separate subject from body with a blank line. Git tools rely on this blank line to distinguish the summary from the details.
6. Wrap the body at 72 characters. Keep lines readable in terminal windows without horizontal scrolling.
7. Use the body to explain what and why, not how. The code diff already shows how you made the change; the commit body should explain the context and reasoning. (Note: The imperative rule applies only to the subject line—you can write the body in standard past/present tense.)

## Associated branch

A branch is associated with a GitHub issue only when its name is `<issue-number>-<title-slug>`.

Slug the title in lowercase with spaces turned into hyphens. Issue #73, "Reset the warp core", accepts: `73-reset-the-warp-core`

When the count is 0 and the branch matches `^[0-9]+-.+`, load `gh issue view <number> --json number,title,body`. Build `<number>-<slug>` from that title. The branch is associated only when `git branch --show-current` equals that string. If `gh` cannot load the issue, or the slug differs, the branch is not associated.

`feature/reset-the-warp-core` is not associated. Only the `<issue-number>-<title-slug>` form is.

## On main

Show the change-based message. Ask whether the commit should occur on `main` or a branch should be created. Wait. Do not commit and do not create a branch until the user answers.

If the commit should occur on `main`, commit with that message.

If a branch should be created, formulate a reasonable branch name from the current changes: lowercase, spaces turned into hyphens. Show that name and wait for confirmation. Do not create the branch until the user confirms. If that name already exists, say so and ask for a different name. Do not switch to the existing branch.

Create the confirmed branch, which keeps the uncommitted changes:

```bash
git checkout -b <confirmed-name>
```

The new branch has no commits of its own, so choose the message again with **Which message**. If the user already changed the message, use their text. If they did not, and the new name is an associated issue branch, use the issue intent. Otherwise use the change-based message already shown. Then follow **Commit**.

Use the branch name the user supplies when they change it.

## Commit

The diff that decides the message is the diff that gets committed.

- If anything is already staged, commit the staged changes only. Tell the user which unstaged and untracked paths were left out.
- If nothing is staged, stage the current changes, including untracked files from `git status`, except secrets, and commit those.

```bash
git commit -m "$(cat <<'EOF'
subject line here

Body wrapped at 72 characters. Explain what and why.

EOF
)"
```

If a hook rejects the commit, fix the failure and create a new commit. Do not amend that rejected commit. Do not push until a commit succeeds.

After a successful commit, push the current branch. If it has no upstream, use `git push -u origin HEAD`. Otherwise use `git push`. Do not force-push. If the push fails, report the error and leave the commit local.

Run `git status` afterward and report the branch, the commit subject, and the push result.
