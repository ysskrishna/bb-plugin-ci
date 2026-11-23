# Changelog

## [Unreleased]

### Added

- The reusable `bb-compat` workflow: it tests a BB plugin against a BB release outside its `engines.bb`, or a prerelease you ask for, and keeps one `bb-compat` issue with the result.
- A self-test that runs `bb-compat` against Idle Compact on pull requests and pushes to `main` that change a workflow.
- Repository files: README badges and sections, changelog, security policy, code owners, funding, pull request template, and agent instructions.
