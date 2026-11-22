---
name: merge-pull-request
description: >-
  Merge a GitHub pull request by rebase or squash, never with a merge commit.
  Squash when any commit subject is WIP, and use the first commit's message.
  Remove root PLAN.md and REVIEW.md before merging when they exist.
  Then switch to main, pull, and delete the pull request's local and remote
  branches. Use when the user asks to merge a pull request.
---

# Merge pull request

Merge one open pull request on GitHub. Never create a merge commit. Do not pass `--merge`. Do not use `git merge`. Do not update git config. Do not skip hooks. Do not force-push, except the force-push in **Remove plan and review**.

The pull request is the one the user names. When they do not name one, use the pull request for the current branch.

## Gather

Run these in parallel from the repository root:

- `git status`
- `git branch --show-current`
- `git rev-parse --verify main`
- `gh pr view <number-or-branch> --json number,state,url,baseRefName,headRefName`

If this directory is not a git repository, stop.

If `main` does not exist, stop and ask. Do not guess another branch.

If the working tree is dirty, stop. Do not merge and do not stash. Root `PLAN.md` and root `REVIEW.md` are the exception. Untracked, modified, staged, or deleted copies of those two files do not count. Any other path does.

If the pull request is not open, stop and report its state.

If `headRefName` is `main`, stop. Do not delete `main`.

Fetch the pull request base and head, then list that branch's commits from oldest to newest:

```bash
git fetch origin <baseRefName> <headRefName>
git log --reverse --format='%H%n%an <%ae>%n%s%n%b%n---' origin/<baseRefName>..origin/<headRefName>
```

The first commit is the oldest one in that list. A WIP subject is a subject line that starts with `WIP - `.

## Strategy

Apply the first match. The strategy is for this merge only. Choose it before **Remove plan and review**. Do not choose it again.

1. **One commit.** Rebase.
2. **More than one commit, and no subject starts with `WIP - `.** Rebase.
3. **More than one commit, and at least one subject starts with `WIP - `.** Squash.

A WIP mention in a commit body does not count. One WIP commit is a rebase.

## Remove plan and review

Do this after the strategy is chosen and before the merge.

A file is committed when `git cat-file -e origin/<headRefName>:<file>` succeeds for root `PLAN.md` or root `REVIEW.md`.

Enter this section only when either file is committed on `origin/<headRefName>`, or the current branch is `<headRefName>` and either file exists in the worktree. Otherwise skip it.

When either file is committed, check out `<headRefName>` if the current branch is different. If that local branch does not exist, `git checkout --track origin/<headRefName>`. If checkout fails, stop. Do not merge. `HEAD` must then equal `origin/<headRefName>`. If it does not, stop. Do not reset and do not merge.

Remove each of those files:

- When the current branch is `<headRefName>`, the file exists, and it is not committed, delete it with `rm -f -- <file>`. Do not commit that deletion.
- When it is committed, remove it with `git rm -f -- <file>`, including a committed file already missing from the worktree. Pass only committed paths.

If neither file was committed, do not commit. Continue to the merge.

If either file was committed, commit that removal and push `<headRefName>` before the merge. One commit covers every committed path. Use the strategy already chosen.

For a rebase, amend the last commit and force-push. Keep its message. Do not use `--reset-author`.

```bash
git commit --amend --no-edit
git push --force-with-lease origin HEAD
```

For a squash, create a new commit and push it. Do not amend. Do not force-push.

```bash
git commit -m "$(cat <<'EOF'
remove plan and review files

Drop these notes so they do not land on main.
EOF
)"
git push origin HEAD
```

If the commit, amend, or push fails, stop and report the error. Do not merge.

## Merge

Rebase:

```bash
gh pr merge <number> --rebase
```

Squash uses the message from the first commit. `--subject` is that commit's subject line. `--body` is the rest of that message, keeping its blank lines. Do not use a later commit's message, the pull request title, or a new summary.

A co-author is another author in `origin/<baseRefName>..origin/<headRefName>`, or a `Co-authored-by:` trailer on any of those commits. Add each one to the end of the body as `Co-authored-by: Name <email>` when that trailer is not already in the first commit message. Do not add the first commit's author again. Put a blank line before the trailers when the body does not already end with one.

```bash
gh pr merge <number> --squash --subject "<first subject>" --body "$(cat <<'EOF'
<body of the first commit>

Co-authored-by: Name <email>
EOF
)"
```

Omit the trailer block when there is nothing to add. If rebase or squash fails, stop and report the error. Do not retry with `--merge` or `--admin`.

## After the merge

Switch to `main`, pull, and remove the pull request branch locally and on the remote:

```bash
git checkout main
git pull
git branch -D <headRefName>
git push origin --delete <headRefName>
```

`git branch -D` is required because a GitHub rebase or squash does not make the old branch an ancestor of `main`. Delete only `<headRefName>`. If the remote branch is already gone, report that and continue.

Report the pull request URL, whether it was rebased or squashed, and that `main` is updated and both branch copies are gone. When `PLAN.md` or `REVIEW.md` was removed, say so.
