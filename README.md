# orenel-ci

Public CI runner for the private `zenderock/orenel-platform` monorepo. Workflows here check out the platform at a given ref, run its checks and report a commit status (`orenel-ci/<name>`) back on the platform commit, so pull requests there show the result.

**Logs are public.** Workflows only print check results. Never echo platform source, `.env` files, tokens or customer data, and never add a step that uploads the platform checkout as an artifact.

## Setup

1. Create a fine-grained personal access token restricted to `zenderock/orenel-platform` with:
   - Contents: read
   - Commit statuses: read and write
2. Add it to this repository as the Actions secret `ORENEL_PLATFORM_TOKEN`.

## Workflows

| Workflow | Trigger | Checks | Status context |
| --- | --- | --- | --- |
| `platform-infra.yml` | manual (`workflow_dispatch`, input `ref`) | `infra/scripts/verify.sh`: compose rendering, required-secret guards, no published ports, `otelcol-contrib validate` | `orenel-ci/infra` |

Run it from the Actions tab, or with the GitHub CLI:

```bash
gh workflow run platform-infra.yml -R releaseszenderock/orenel-ci -f ref=<branch-or-sha>
```

`workflow_dispatch` only works once the workflow file is on this repository's default branch.
