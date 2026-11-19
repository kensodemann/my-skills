---
name: create-pull-request
description: >-
  Create a GitHub pull request whose title and body summarize the branch
  change. When the branch name is an issue number plus the issue title slug,
  associate the pull request with that issue. Use when the user asks to open,
  create, or submit a pull request.
---

# Create pull request

Open a pull request for the current branch against `main`. The title and body describe the whole change on the branch. Do not commit. Do not update git config. Do not force-push. Do not use `gh pr create --fill`.

## Gather

Run these in parallel from the repository root:

- `git status`
- `git branch --show-current`
- `git rev-parse --verify main`
- `git log --oneline main..HEAD`
- `git diff main...HEAD`
- `gh pr view --json url,state`

If this directory is not a git repository, stop.

If `HEAD` is detached, stop and ask. Do not open a pull request.

If `main` does not exist, stop and ask which branch is the base. Do not guess.

If the current branch is `main`, stop. A pull request needs a different branch.

If the working tree is dirty, stop. Name the unstaged and untracked paths. Do not commit them. **git-commit** commits.

If `main...HEAD` has no changes, stop. There is nothing to open.

If `gh pr view` returns an open pull request, report its URL and stop. Do not create a second one.

## Title and body

Write them from the resulting diff against `main`, not from the latest commit and not from intermediate WIP commits.

The title is one imperative command that summarizes the change. Capitalize it. Do not end it with a period.

The body explains what and why. Do not list files. Use this shape:

```markdown
## Summary
- <what changed and why, from the whole diff>

## Test plan
- [ ] <how to verify this change>
```

## Associated branch

A branch is associated with a GitHub issue only when its name is `<issue-number>-<title-slug>`.

Slug the title in lowercase with spaces turned into hyphens. Issue #73, "Reset the warp core", accepts: `73-reset-the-warp-core`

When the branch matches `^[0-9]+-.+`, load `gh issue view <number> --json number,title,body`. Build `<number>-<slug>` from that title. The branch is associated only when `git branch --show-current` equals that string. If `gh` cannot load the issue, or the slug differs, the branch is not associated.

`feature/reset-the-warp-core` is not associated. Only the `<issue-number>-<title-slug>` form is.

When the branch is associated, end the body with `Fixes #<number>`. That links the pull request to the issue. Merging the pull request closes the issue. Do not add that line for any other issue.

## Open

Push the branch if it has no upstream or it is ahead of its upstream. If it has no upstream, use `git push -u origin HEAD`. Otherwise use `git push`. Do not force-push.

Then create the pull request:

```bash
gh pr create --base main --head <branch> --title "<title>" --body "$(cat <<'EOF'
## Summary
- What changed and why.

## Test plan
- [ ] How to verify this change.

Fixes #73
EOF
)"
```

Omit the `Fixes` line when the branch is not associated with an issue.

Report the pull request URL. If the push or `gh pr create` fails, report the error and do not retry with a force-push.
