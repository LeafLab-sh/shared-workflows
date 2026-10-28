# shared-workflows

Reusable GitHub Actions workflows for LeafLab-sh repositories.

## `commits.yml`

Checks every commit in a pull request:

- **Conventional Commits**, enforced by commitlint (`@commitlint/config-conventional` 21.2.4, with `body-leading-blank` and `footer-leading-blank` as errors). release-please release commits and Dependabot commits are ignored.
- **No `fixup!` / `squash!` / `amend!` commits.** Autosquash before merging:
  `GIT_SEQUENCE_EDITOR=: git rebase -i --autosquash origin/main && git push --force-with-lease`

Add `.github/workflows/commits.yml` to a repository:

```yaml
name: Commits

on:
  pull_request:
  merge_group:

permissions:
  contents: read

jobs:
  commits:
    uses: LeafLab-sh/shared-workflows/.github/workflows/commits.yml@<sha> # vX.Y.Z
```

Pin by SHA. In repositories whose `dependabot.yml` includes the `github-actions` ecosystem, Dependabot keeps the pin current.

The required status check is named **`commits / validate`**. Outside pull requests (`merge_group`, `push`), it passes without checking anything, so it can be required in a repository with a merge queue.

## Gotchas

- This repository must stay **public**. Public repositories can't call reusable workflows that live in a private repository.
- Rules here apply to every repository. A repository's local commitlint config (for git hooks) should stay at least as strict, or a commit can pass locally and fail in CI.
