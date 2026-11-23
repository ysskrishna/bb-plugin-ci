# Changelog

## [Unreleased]

## [1.1.1]

### Fixed

- Prerelease runs: the working-copy `engines.bb` now allows the base version too (`>=0.50.2 || 0.50.2-nightly.N`), because a BB nightly reports itself as its base version. Before, BB refused to start the plugin and every nightly smoke test failed with "status: incompatible".
- The self-test's nightly job runs the default typecheck and smoke test, so it covers this path.

## [1.1.0]

### Added

- `always-test` input: run the checks even when the BB under test is a stable release inside `engines.bb`. A pass on the latest BB, or on the version an open `bb-compat` issue is about, closes that issue; a failure opens or updates "bb X fails: <check>".
- `fail-on-error` input (default on): when off, a failing check is reported in the issue without failing the run.
- The self-test covers both: a forced failure, then a full typecheck and smoke test on the latest BB.

### Changed

- The checks run with `engines.bb` in the working copy set to the BB under test, because BB will not run a plugin outside its range.
- The README recommends an open-ended `engines.bb` and a daily caller that tests the latest stable BB, with the nightly by manual dispatch only.

## [1.0.0]

### Added

- The reusable `bb-compat` workflow: it tests a BB plugin against a BB release outside its `engines.bb`, or a prerelease you ask for, and keeps one `bb-compat` issue with the result.
- A self-test that runs `bb-compat` against Idle Compact on pull requests and pushes to `main` that change a workflow.
- Repository files: README badges and sections, changelog, security policy, code owners, funding, pull request template, and agent instructions.
