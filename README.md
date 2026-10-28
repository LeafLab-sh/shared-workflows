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

## `dependabot-auto-merge.yml`

When Dependabot opens a pull request, this turns on auto-merge (rebase), so the PR merges once the required checks pass. Use it only where `main` has required status checks.

```yaml
name: Dependabot auto-merge

on:
  pull_request:
    types: [opened, reopened]

permissions:
  contents: write
  pull-requests: write

jobs:
  dependabot:
    uses: LeafLab-sh/shared-workflows/.github/workflows/dependabot-auto-merge.yml@<sha> # vX.Y.Z
    with:
      approve: true
      app-client-id: ${{ vars.LEAFLAB_AUTOMATION_CLIENT_ID }}
      # update-types: version-update:semver-patch,version-update:semver-minor  # default
    secrets:
      app-private-key: ${{ secrets.LEAFLAB_AUTOMATION_PRIVATE_KEY }}
```

| Input | Default | Meaning |
|---|---|---|
| `update-types` | `version-update:semver-patch,version-update:semver-minor` | Update types to auto-merge, as `dependabot/fetch-metadata` reports them. Anything else waits for a person. |
| `approve` | `false` | Also approve the PR. Needs "Allow GitHub Actions to create and approve pull requests" when the token is `GITHUB_TOKEN`. |
| `app-client-id` | `""` | Client ID of the automation GitHub App. It isn't secret, so pass the org variable. |

| Secret | Meaning |
|---|---|
| `app-private-key` | Private key (PEM) of the automation GitHub App. When it and `app-client-id` are both set, the App does the approving and enabling. Without them, `GITHUB_TOKEN` does, with a warning. |

## Gotchas

- **Pass the automation App** wherever something has to run after a Dependabot merge (release-please on `push`). GitHub starts no workflow for anything done with a workflow's `GITHUB_TOKEN`, so a PR merged that way never triggers them.
- **The App's private key must be a Dependabot secret** (Settings → Secrets → Dependabot), not only an Actions secret. Runs started by Dependabot only receive Dependabot secrets.
- **It must be the private key (PEM)**, generated in the App's settings. The App's OAuth client secret can't mint installation tokens.
- This repository must stay **public**. Public repositories can't call reusable workflows that live in a private repository.
- Rules here apply to every repository. A repository's local commitlint config (for git hooks) should stay at least as strict, or a commit can pass locally and fail in CI.
