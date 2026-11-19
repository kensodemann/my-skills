---
name: release-plugin
description: >-
  Publish a version of the my-skills repository from main. Ask whether the
  release is major, minor, or patch, then bump both plugin manifests, update
  the changelog, validate, commit, and tag. Ask the human to verify before
  the final push. Use when the user asks to release the skills, publish, or
  tag a version of my-skills.
---

# Release plugin

Release the skills in this repository when the user asks. This skill lives in `.cursor/skills/` and is not one of the skills under `skills/`.

Do not update git config. Do not skip hooks. Do not amend. Do not force-push `main`. Do not create a branch.

## Process

The release is automated when a semver Git tag is pushed. The [Release workflow](../../../.github/workflows/release.yml) validates the repo, builds a zip of `skills/`, and creates a GitHub Release with notes and the zip attached. GitHub Actions creates that release (`contents: write`). The human needs permission to push tags.

The changes for this release must already be on `main`. If they are still on another branch, tell the human and stop. Do not merge.

Do not edit version files, edit `CHANGELOG.md`, validate, commit, tag, or push until the checks below have passed and the human has answered which release this is.

### This repository

Confirm this is the my-skills repository:

- `git remote get-url origin` refers to `kensodemann/my-skills`
- `name` in `.claude-plugin/plugin.json` and `.cursor-plugin/plugin.json` is `my-skills`

If either check fails, say this is not the my-skills repository and stop.

### On main

Run `git branch --show-current`.

The branch must be `main`. This is one of the few processes that are run directly on `main`. If it is not, inform the human which branch is checked out and stop. Do not check out `main`.

### Which release

Ask the human if this is a major, minor, or patch release. Wait for the answer. Do not infer it from the diff.

Read `version` from both plugin files. Those values must already match. Call that value `A.B.C`.

| Answer | New version |
| ------ | ----------- |
| major  | `(A+1).0.0` |
| minor  | `A.(B+1).0` |
| patch  | `A.B.(C+1)` |

If the answer is not one of those three, ask again.

### 1. Bump the plugin version

Set `version` in `.claude-plugin/plugin.json` and `.cursor-plugin/plugin.json` to the new semver without a `v` prefix (for example `1.2.0`). Both files stay identical.

The Git tag pushed later must be `v` plus that exact string (for example tag `v1.2.0` for version `1.2.0`). If they differ, the release job fails.

### 2. Update the changelog

Add a `## [x.y.z]` section to `CHANGELOG.md` for the same version, directly under the title and above the previous `## [` section. No `v` prefix. Fill it with the changes since the previous version. Substitute the new version for `x.y.z`:

```markdown
## [x.y.z]

- Short description of what changed.
```

Take the list from commits since tag `vA.B.C` (`git log vA.B.C..HEAD --oneline`) and from release changes still uncommitted. Write short bullets. Leave the previous sections as they are.

The workflow copies everything under that heading until the next `## [` heading into the GitHub Release notes. If the section is missing or empty, the release fails. If the section would be empty, stop and tell the human. Do not tag an empty release.

### 3. Confirm skills and plugin validation

Ensure every `skills/*/SKILL.md` has a `name` that matches its parent folder name.

From the repository root, run:

```bash
npx -y @anthropic-ai/claude-code@2.1.300 plugin validate --strict .
```

`npx -y` downloads the pinned Claude Code CLI and validates `.`. CI runs this exact command in the **Validate plugin and marketplace** step. Fix any errors and run the command again. Do not tag until it passes.

### 4. Commit and tag

Commit the version and changelog updates, and any other uncommitted changes that belong in this release. The subject is `release x.y.z` with the new version substituted, no `v` prefix, and no period.

Leave unrelated paths and secrets (`.env`, credentials, keys, tokens) unstaged. If a path is not clearly part of the release, name it and stop. Do not commit it.

```bash
git add .claude-plugin/plugin.json .cursor-plugin/plugin.json CHANGELOG.md
git commit -m "$(cat <<'EOF'
release x.y.z

EOF
)"
```

Stage any other release paths before that commit. Substitute the new version in the subject.

Create a tag matching `v*.*.*` (three numeric components). A lightweight or annotated tag is valid. Substitute the new version:

```bash
git tag vX.Y.Z
```

Confirm `git status` is clean and `git rev-parse vX.Y.Z` is `HEAD`.

Stop and ask the human to verify this release before the final push. Show the new version, the changelog bullets, that validation passed, and the tag. Do not push.

If the human wants changes, apply them, commit again, delete the local tag (`git tag -d vX.Y.Z`), and create it on the new commit. Then stop and ask them to verify again.

### 5. Push main

Resume only when the human has verified the release and asks to continue. Further review comments are more changes under step 4, not permission to push.

```bash
git push origin main
```

### 6. Push the tag

```bash
git push origin vX.Y.Z
```

Only pushing the tag triggers the Release workflow. Pushing `main` alone does not publish a release.

### 7. What CI does

On tag push `v*.*.*`, the workflow:

1. Checks that `v$(jq -r .version .claude-plugin/plugin.json)` equals the tag name.
2. Runs `npx -y @anthropic-ai/claude-code@2.1.300 plugin validate --strict .` from the repository root (**Validate plugin and marketplace**).
3. Verifies skill folder names match each skill’s `name` field.
4. Extracts release notes from `CHANGELOG.md` for that version.
5. Creates `my-skills-<tag>.zip` containing the `skills/` directory.
6. Publishes a GitHub Release titled with the tag, with the changelog body and the zip as a release asset.

Tell the human the tag push started that workflow. Report the run when `gh` can see it. Do not create the GitHub Release yourself. Do not describe the release as published until that workflow succeeds.

If the workflow fails, fix the reported issue and add a commit. Move or recreate the tag on the corrected commit, then push that tag again. Do not force-push `main`.
