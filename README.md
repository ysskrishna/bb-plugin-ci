# BB Plugin CI

[![CI](https://github.com/ysskrishna/bb-plugin-ci/actions/workflows/self-test.yml/badge.svg)](https://github.com/ysskrishna/bb-plugin-ci/actions/workflows/self-test.yml)
[![GitHub release](https://img.shields.io/github/v/release/ysskrishna/bb-plugin-ci?label=release)](https://github.com/ysskrishna/bb-plugin-ci/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue)](LICENSE)
[![Author site](https://img.shields.io/badge/author-ysskrishna.space-informational)](https://ysskrishna.space)

Shared GitHub Actions workflows for [BB](https://getbb.app) plugins.

BB already refuses to install or update a plugin whose `engines.bb` excludes the
running BB. What it cannot do is warn the plugin author before a new BB release
leaves the plugin behind, or before an update to BB breaks a plugin whose range
already allows it. The `bb-compat` workflow does that: the recommended caller
runs the plugin's checks every day against the latest stable BB.

## Features

| Workflow | What it does |
| --- | --- |
| **bb-compat** | Tests a plugin against the latest stable BB (or a release or prerelease you ask for) and keeps one `bb-compat` issue while something needs attention. |

How `bb-compat` works:

1. It resolves `bb-app@latest` (or the `bb-version` input) and reads `engines.bb`
   from the plugin's `package.json`.
2. If that BB is a stable release inside `engines.bb` and `always-test` is off,
   it tests nothing and ends green. When that BB is the latest, it also closes
   any open `bb-compat` issue.
3. Otherwise it runs `npm ci`, installs that `bb-app` plus the
   `@get-bb/plugin-sdk` version the release ships (read from its bundle), sets
   `engines.bb` in the working copy to that version, plus its base version for
   a prerelease (BB will not run a plugin outside it), and runs the plugin's checks, each with only `PATH` and `HOME`
   set. A failing check fails the run unless `fail-on-error` is off.
4. It keeps one open issue labelled `bb-compat`, with a link to the run:
   - Inside `engines.bb` (with `always-test`): a pass on the latest BB, or on
     the version the open issue is about, closes it; a failure opens or
     updates "bb X fails: <check>".
   - Outside `engines.bb`: "bb X passes: widen engines.bb to include it" or
     "bb X fails: <check>".
   - A prerelease: "bb X passes the checks (prerelease)" or "bb X fails:
     <check>". A prerelease result never replaces an open issue about a
     stable release.

Once you widen `engines.bb` and release, the next run on the latest BB closes
the issue. Pull requests from forks and Dependabot run the checks but skip the
issue, because their token is read-only.

## Installation

Use an open-ended range such as `"engines": { "bb": ">=0.50.1" }`, so new BB
releases do not lock users out, and let the daily run against the latest stable
BB tell you when one breaks the plugin. Then fix forward. Add
`.github/workflows/bb-compat.yml` to the plugin:

```yaml
name: BB compat

on:
  schedule:
    - cron: "23 6 * * *"
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
      always-test: true
      codex-version: 0.169.0
      apt-packages: lsof procps
```

Scheduled and pull request runs test the latest stable BB. The `pull_request`
trigger only runs on a PR that changes the caller file itself. To try a BB
before it is stable, dispatch by hand with `bb-version: nightly` or an exact
version; scheduling the nightly is not recommended, because it breaks often.

### Inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `bb-version` | empty (latest) | bb-app version or dist-tag to test. |
| `always-test` | `false` | Test a stable release even when it is inside `engines.bb`. The recommended caller sets it. |
| `fail-on-error` | `true` | Fail the run when a check fails. When off, only the issue records it. |
| `checks` | `npm run typecheck` and `npm run e2e:smoke` | One command per line. |
| `node-version` | `22` | Node.js version. |
| `codex-version` | empty | Installs `@openai/codex` globally, for scratch e2e fixtures that need it. |
| `apt-packages` | empty | Space-separated apt packages, for example `lsof procps`. |
| `setup` | empty | Extra shell commands after `npm ci`, for example a Playwright browser install. |
| `repository` | calling repository | Plugin repository to check out. Used by this repo's self-test. |

Outputs: `tested` (`true` when the checks ran), `action` (`created`, `updated`,
`closed` or `none`; empty when the issue step is skipped) and `issue` (the issue
number acted on).

### Permissions

The job that runs plugin code gets `contents: read` only and does not keep the
checkout token. A separate job with `issues: write` writes the issue, so the
caller grants `contents: read` and `issues: write`.

## See it in action

`.github/workflows/self-test.yml` runs on pull requests and pushes to `main` that
change a workflow. It runs `bb-compat` against [Idle Compact](https://github.com/ysskrishna/bb-idle-compact)
four times: the BB nightly with the full checks (opens a prerelease issue in
this repository), the latest BB without `always-test` (tests nothing, closes
it), the latest BB with `always-test` and a check that always fails, with
`fail-on-error` off (opens "fails"), and the latest BB with `always-test` and
the full typecheck and smoke test (passes, closes it). A last job fails unless
all four behaved. See
the [self-test runs](https://github.com/ysskrishna/bb-plugin-ci/actions/workflows/self-test.yml)
and the [bb-compat issues](https://github.com/ysskrishna/bb-plugin-ci/issues?q=label%3Abb-compat) they produced.

## Versions

Callers pin the major tag `v1`. Moving `v1` to a newer commit is a deliberate
release step: merge the change, run the self-test on `main`
(`gh workflow run self-test.yml --ref main`) and wait for it to pass, then move
the tag with `git tag -f v1 <commit> && git push -f origin v1` and say so in the
PR or release notes. A breaking change to inputs or behavior gets a new `v2`
tag instead. [AGENTS.md](AGENTS.md#releases) has the full release steps.

## Development

<details>
<summary><b>Change and check a workflow</b></summary>

```sh
git clone https://github.com/ysskrishna/bb-plugin-ci.git
cd bb-plugin-ci
```

There is nothing to build or install. Open a pull request: the self-test runs
the changed workflow against Idle Compact, and the pull request shows the
result. To try a change from a plugin before a release, point its caller at
your branch (`@your-branch`) instead of `@v1`.

The self-test writes issues in this repository, which a pull request from a
fork cannot do, so its last job fails there. The maintainer re-runs such a
change from a branch in this repository.

</details>

## Project links

- [Releases](https://github.com/ysskrishna/bb-plugin-ci/releases) and [issues](https://github.com/ysskrishna/bb-plugin-ci/issues)
- [Security policy](SECURITY.md)
- Written for [Idle Compact](https://github.com/ysskrishna/bb-idle-compact), [Profiles](https://github.com/ysskrishna/bb-profiles) and [Thread Inbox](https://github.com/ysskrishna/bb-thread-inbox)

## Built and maintained by

BB Plugin CI is built and maintained by:

| | Name | GitHub | Role |
| --- | --- | --- | --- |
| <img src="https://github.com/ysskrishna.png?size=120" width="60" height="60" alt="Y. Siva Sai Krishna"> | **Y. Siva Sai Krishna** | [@ysskrishna](https://github.com/ysskrishna) | Creator |

## Changelog

See [CHANGELOG](CHANGELOG.md) for release history and unreleased changes.

## Support

If you find this project helpful:

- ⭐ Star the repository
- 🐛 [Report issues](https://github.com/ysskrishna/bb-plugin-ci/issues)
- 🔀 Submit pull requests
- 💝 [Sponsor on GitHub](https://github.com/sponsors/ysskrishna)

## License

MIT © [Y. Siva Sai Krishna](https://github.com/ysskrishna) - see [LICENSE](LICENSE) for details.

---

<p align="left">
  <a href="https://github.com/ysskrishna">Author's GitHub</a> •
  <a href="https://linkedin.com/in/ysskrishna">Author's LinkedIn</a> •
  <a href="https://ysskrishna.space">Author's site</a> •
  <a href="https://github.com/ysskrishna/bb-plugin-ci/releases">Releases</a> •
  <a href="https://github.com/ysskrishna/bb-plugin-ci/issues">Report Issues</a>
</p>
