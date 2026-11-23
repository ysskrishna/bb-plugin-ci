# bb-plugin-ci

Shared GitHub Actions workflows for [BB](https://www.npmjs.com/package/bb-app) plugins.

## bb-compat

BB already refuses to install or update a plugin whose `engines.bb` excludes the
running BB. What it cannot do is warn the plugin author before a new BB release
leaves the plugin behind. `bb-compat` does that:

1. It resolves `bb-app@latest` (or the `bb-version` input) and reads `engines.bb`
   from the plugin's `package.json`.
2. If that BB is a stable release inside `engines.bb`, it tests nothing and ends
   green. When that BB is the latest, it also closes any open `bb-compat` issue.
3. Otherwise (a stable release outside `engines.bb`, or a prerelease you asked
   for) it runs `npm ci`, installs that `bb-app` plus the `@get-bb/plugin-sdk`
   version the release ships (read from its bundle), and runs the plugin's
   checks, each with only `PATH` and `HOME` set.
4. It keeps one open issue labelled `bb-compat`: "bb X passes: widen engines.bb
   to include it", "bb X passes the checks (prerelease)", or "bb X fails:
   <check>", with a link to the run. A failing check also fails the run.

Once you widen `engines.bb` and release, the next run on the latest BB closes
the issue. Pull requests from forks and Dependabot run the checks but skip the
issue, because their token is read-only.

### Use it

Add `.github/workflows/bb-compat.yml` to the plugin:

```yaml
name: BB compat

on:
  schedule:
    - cron: "23 6 * * 1"
  workflow_dispatch:
    inputs:
      bb-version:
        description: bb-app version or dist-tag to test (empty means latest)
        default: ""
  pull_request:
    paths:
      - .github/workflows/bb-compat.yml

permissions:
  contents: read
  issues: write

jobs:
  compat:
    uses: ysskrishna/bb-plugin-ci/.github/workflows/bb-compat.yml@v1
    with:
      bb-version: ${{ inputs.bb-version }}
      codex-version: 0.169.0
      apt-packages: lsof procps
```

The `pull_request` trigger only runs the workflow on a PR that changes the
caller file itself. To try a newer BB before it is the latest, dispatch with
`bb-version: nightly` or an exact version.

### Inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `bb-version` | `latest` | bb-app version or dist-tag to test. |
| `checks` | `npm run typecheck` and `npm run e2e:smoke` | One command per line. |
| `node-version` | `22` | Node.js version. |
| `codex-version` | empty | Installs `@openai/codex` globally, for scratch e2e fixtures that need it. |
| `apt-packages` | empty | Space-separated apt packages, for example `lsof procps`. |
| `setup` | empty | Extra shell commands after `npm ci`, for example a Playwright browser install. |
| `repository` | calling repository | Plugin repository to check out. Used by this repo's self-test. |

Outputs: `tested` (`true` when the checks ran), `action` (`created`, `updated`,
`closed` or `none`) and `issue` (the issue number acted on).

### Permissions

The job that runs plugin code gets `contents: read` only and does not keep the
checkout token. A separate job with `issues: write` writes the issue, so the
caller grants `contents: read` and `issues: write`.

## Versions

Callers pin the major tag `v1`. Moving `v1` to a newer commit is a deliberate
release step: merge the change, check the self-test run on `main`, then move
the tag with `git tag -f v1 <commit> && git push -f origin v1` and say so in the
PR or release notes. A breaking change to inputs or behavior gets a new `v2`
tag instead.

## Self-test

`.github/workflows/self-test.yml` runs on pull requests that change a workflow.
It runs `bb-compat` against [Idle Compact](https://github.com/ysskrishna/bb-idle-compact)
with the BB nightly, which opens a `bb-compat` issue in this repository, then
with the latest BB, which closes it. A last job fails unless both happened.

## License

[MIT](LICENSE)
