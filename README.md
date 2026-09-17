# workflows

The reusable GitHub Actions workflows for the forebay ecosystem now live in
[`intisy/workflows`](https://github.com/intisy/workflows), the cross-org shared repository. This
repository holds none of its own.

## Branching and release standard

Every consuming repo follows a two-stage, tag-gated promotion flow:

| You push | What happens |
| --- | --- |
| any branch (`feature/*`, `experimental`) | tests run (repo CI); nothing is merged |
| a non-`v` tag (e.g. on `feature/*`) | the tagged commit is merged into `experimental` |
| a `v*` tag (cut from `experimental`) | the tagged commit is merged into `main` and the repo's own build/publish runs |

Lifecycle: work on `feature/x`, push a non-`v` tag to land it on `experimental`, push a `vX.Y.Z` tag
to release to `main`. Plain branch pushes never merge.

## Consuming the shared merge workflow

Add two thin caller workflows to a repo under `.github/workflows/`:

```yaml
# promote-experimental.yml
name: Promote to Experimental

on:
  push:
    tags: ['*', '!v*']
permissions:
  contents: write
jobs:
  promote:
    uses: intisy/workflows/.github/workflows/merge.yml@main
    with:
      source: ${{ github.ref_name }}
      target: experimental
```

```yaml
# promote-main.yml
name: Promote to Main

on:
  push:
    tags: ['v*']
permissions:
  contents: write
jobs:
  promote:
    uses: intisy/workflows/.github/workflows/merge.yml@main
    with:
      source: experimental
      target: main
      sync_back: true
```

The merge workflow uses the built-in `GITHUB_TOKEN` (org branches are unprotected), so no `PAT` is
required. Each repo's publish/README workflow should trigger on `tags: ['v*']` (not `['*']`) so
non-`v` feature tags do not cut a release.
