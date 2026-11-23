# BB Plugin CI

[![CI](https://github.com/ysskrishna/bb-plugin-ci/actions/workflows/self-test.yml/badge.svg)](https://github.com/ysskrishna/bb-plugin-ci/actions/workflows/self-test.yml)
[![GitHub release](https://img.shields.io/github/v/release/ysskrishna/bb-plugin-ci?label=release)](https://github.com/ysskrishna/bb-plugin-ci/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue)](LICENSE)
[![Author site](https://img.shields.io/badge/author-ysskrishna.space-informational)](https://ysskrishna.space)

Shared GitHub Actions workflows for [BB](https://getbb.app) plugins.

BB already refuses to install or update a plugin whose `engines.bb` excludes the
running BB. What it cannot do is warn the plugin author before a new BB release
leaves the plugin behind. The `bb-compat` workflow does that every week, from a
short caller workflow in each plugin.

## Features

| Workflow | What it does |
| --- | --- |
| **bb-compat** | Tests a plugin against a BB release outside its `engines.bb` (or a prerelease you ask for) and keeps one `bb-compat` issue with the result. |

How `bb-compat` works:

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
   <check>", with a link to the run. A failing check also fails the run. A
   prerelease result never replaces an open issue about a stable release.

Once you widen `engines.bb` and release, the next run on the latest BB closes
the issue. Pull requests from forks and Dependabot run the checks but skip the
issue, because their token is read-only.

## Installation

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
`closed` or `none`; empty when the issue step is skipped) and `issue` (the issue
number acted on).

### Permissions

The job that runs plugin code gets `contents: read` only and does not keep the
checkout token. A separate job with `issues: write` writes the issue, so the
caller grants `contents: read` and `issues: write`.

## See it in action

`.github/workflows/self-test.yml` runs on pull requests and pushes to `main` that
change a workflow. It runs `bb-compat` against [Idle Compact](https://github.com/ysskrishna/bb-idle-compact)
with the BB nightly, which opens a `bb-compat` issue in this repository, then
with the latest BB, which closes it. A last job fails unless both happened. See
the [self-test runs](https://github.com/ysskrishna/bb-plugin-ci/actions/workflows/self-test.yml)
and the [bb-compat issues](https://github.com/ysskrishna/bb-plugin-ci/issues?q=label%3Abb-compat) they produced.

## Versions

Callers pin the major tag `v1`. Moving `v1` to a newer commit is a deliberate
release step: merge the change, check the self-test run on `main`, then move
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

</details>

## Project links

- [Releases](https://github.com/ysskrishna/bb-plugin-ci/releases) and [issues](https://github.com/ysskrishna/bb-plugin-ci/issues)
- [Security policy](SECURITY.md)
- Plugins that use it: [Idle Compact](https://github.com/ysskrishna/bb-idle-compact), [Profiles](https://github.com/ysskrishna/bb-profiles) and [Thread Inbox](https://github.com/ysskrishna/bb-thread-inbox)

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
