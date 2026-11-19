# Contributing

Thank you for contributing to **my-skills**. This document covers how to prepare changes and how to ask an agent to publish a release.

## Skills layout

Each skill lives in its own folder under `skills/`:

```
skills/
  my-skill/
    SKILL.md
```

The `name` field in `SKILL.md` (first `name:` line in the front matter or header) **must match the folder name** exactly. The release workflow enforces this.

Before opening a pull request, validate the plugin yourself from the repository root:

```bash
npx -y @anthropic-ai/claude-code@2.1.300 plugin validate --strict .
```

Run that command directly. `npx -y` downloads `@anthropic-ai/claude-code` at `2.1.300` and runs `plugin validate --strict` against the current directory (`.`). The [Release workflow](.github/workflows/release.yml) runs this same command in the **Validate plugin and marketplace** step when a version tag is pushed. Fix any errors before you open the pull request.

## Releasing a version

Tell your agent to release the skills. It follows [release-plugin](.cursor/skills/release-plugin/SKILL.md).

What to expect:

- Be on `main`. This release runs on `main`, and the agent stops when the current branch is not `main`.
- The agent asks whether this release is major, minor, or patch.
- The agent asks you to verify the release before the final push.

You need permission to push tags. After that push, the [Release workflow](.github/workflows/release.yml) validates the repo, builds a zip of `skills/`, and creates a GitHub Release with that zip attached.
