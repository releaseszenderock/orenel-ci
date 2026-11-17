# orenel-ci

Public CI runner for the private `zenderock/orenel-platform` monorepo. Workflows here check out the platform at a given ref, run its checks and report a commit status (`orenel-ci/<name>`) back on the platform commit, so pull requests there show the result.

**Logs are public.** Workflows only print check results. Never echo platform source, `.env` files, tokens or customer data, and never add a step that uploads the platform checkout as an artifact. Known metadata that does appear: the checked-out commit SHA and subject line, and, if the collector fails to start, its last log lines (fixture data only).

Third-party actions are pinned to commit SHAs because the checkout step receives a token for the private repository. The `orenel-ci/<name>` status is advisory (it is not a required check on the platform repository): before merging, confirm the status was posted for the PR's current head SHA, and give the token an expiry date.

## Setup

1. Create a fine-grained personal access token restricted to `zenderock/orenel-platform` with:
   - Contents: read
   - Commit statuses: read and write
2. Add it to this repository as the Actions secret `ORENEL_PLATFORM_TOKEN`.

## Workflows

| Workflow | Trigger | Checks | Status context |
| --- | --- | --- | --- |
| `platform-infra.yml` | manual (`workflow_dispatch`, input `ref`) | `infra/scripts/verify.sh`: compose rendering, required-secret guards, no published ports, `otelcol-contrib validate` | `orenel-ci/infra` |
| `platform-go.yml` | manual (`workflow_dispatch`, input `ref`) | gofmt, go vet, generated code up to date (`go generate`, `sqlc diff`), `go test -race` with integration tests required, api and worker image builds | `orenel-ci/go` |
| `platform-web.yml` | manual (`workflow_dispatch`, input `ref`) | frozen install, root scripts `lint`, `typecheck`, `test`, OpenAPI lint and client regeneration (`api:lint`, `api:generate` if defined), `build`, Playwright `test:e2e`, web image build | `orenel-ci/web` |

Run it from the Actions tab, or with the GitHub CLI:

```bash
for wf in platform-infra platform-go platform-web; do
  gh workflow run "$wf.yml" -R releaseszenderock/orenel-ci -f ref=<branch-or-sha>
done
```

`workflow_dispatch` only works once the workflow file is on this repository's default branch.
