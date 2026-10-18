## What is here

Two workflows, and this repository really does hold them.

| file | kind | what it is |
| --- | --- | --- |
| `.github/workflows/gate.yml` | `workflow_call` | **Ecosystem invariants.** The reusable gate: it clones every repo in the ecosystem and runs the invariant checks over all of them at once |
| `.github/workflows/invariants.yml` | `workflow_dispatch` and `schedule` | the caller that runs the gate, weekly at 04:00 on Mondays, or on demand against a ref you name |

An org may draw on exactly two workflows repositories: its own, which is this one, and
[`intisy/workflows`](https://github.com/intisy/workflows) for anything shared across orgs. Nearly
every caller in the tree points at the second, because test, merge, publish and README generation
are not org-specific. What lives here is the one thing that is: a gate that judges THIS ecosystem's
repositories against its own invariants.

## Running the gate

It is a sweep across repositories rather than a check of one, so it does not run on push.

```yaml
jobs:
  invariants:
    uses: forebay/workflows/.github/workflows/gate.yml@main
    with:
      clone_ref: development
    secrets:
      envPAT: ${{ secrets.PAT }}
```

`envPAT` is not optional in practice. The gate reads other repositories and several of them are
private, so the job token cannot see them and the workflow fails its own pre-flight with that
message rather than reporting a misleading pass.

| input | default | what it does |
| --- | --- | --- |
| `tool_repository` | `forebay/npm-dual-publish` | the repo holding the gate tool |
| `tool_ref` | the tool repo's default branch | which ref of the tool to run |
| `clone_ref` | each repo's default branch | the branch to judge every repo at |
| `timeout_minutes` | `45` | raise it if the sweep outgrows it |

`clone_ref` is the input worth knowing. Empty, which is what the schedule passes, judges the
released state. Pointing it at `development` catches a violation before a promotion carries it to
`main`.

## Branches

Two long-lived branches per repository, `main` and `development`, with `main` the default. Work
happens on `development` and is merged into `main`; anything else is a short-lived feature branch
off `development`.

A repository whose content IS the deliverable, which is what this one is, may carry `main` alone
and take changes directly on it. That is why there is no `development` branch here and why its
absence is not a deviation.
