# For agents working in this repository

Reusable GitHub Actions workflows for every LeafLab-sh repository, specified in the `leaflab` store under `standards/shared-workflows`. [`README.md`](README.md) documents each workflow, its inputs and its gotchas. A change here reaches every caller only when it is released (a semver tag and GitHub release on the merged commit) and callers move their SHA pin. Keep the repository public, and never run code from the pull request a workflow checks.

## Working in LeafLab-sh

This repository follows the LeafLab-sh org's standards. They're specified in the org spec store, [`LeafLab-sh/openspec`](https://github.com/LeafLab-sh/openspec) (store id `leaflab`). This section says where to find them, not what they all say.

### Process

- **Commits** (`standards/commit-messages`): Conventional Commits, a `Refs: LEA-<n>` footer for Linear work, and no `fixup!` commits left on `main`.
- **Pull requests** (`standards/pull-requests`): open as a draft, and mark ready after self-review once CI passes. Fix findings with `fixup!` commits and autosquash before merging. Link Linear with `Fixes LEA-<n>` or `Part of LEA-<n>` in the body. Merge by rebase only.
- **Branches** (`integrations/linear`): `lea/lea-<n>-<slug>`, named after the Linear issue.

### Reviewing specs

This project's own specs are in `openspec/specs/`:

```sh
openspec list --specs
openspec show <spec-id> --type spec
```

The org's specs are in the `leaflab` store, which `openspec/config.yaml` references. Pull your clone of the store first, because OpenSpec never does. `openspec context` shows where the clone is.

```sh
openspec list --specs --store leaflab
openspec show <spec-id> --type spec --store leaflab
```

If `openspec doctor` reports the `leaflab` reference as unresolved, clone and register the store as its README says.

### Contributing to specs

- **This project's behavior:** an OpenSpec change in this repository. Use `/opsx:propose` (or `openspec new change <id>`), pass `openspec validate <id> --strict`, and run `openspec archive <id>` in the pull request that implements it.
- **An org standard, an integration, or how repositories are set up:** an OpenSpec change in `LeafLab-sh/openspec`, by pull request there. Never write the store from this repository.
