# Changelog

## [Unreleased]

## [1.0.0]

### Added

- The reusable `bb-compat` workflow: it tests a BB plugin against the BB nightly or a stable release outside its `engines.bb`, and keeps one `bb-compat` issue while something needs attention.
- A self-test that runs `bb-compat` against Idle Compact (BB nightly, bb 0.50.0 with the full checks, and the latest) on pull requests and pushes to `main` that change a workflow.
- A passing prerelease opens no issue and closes an open prerelease failure. An issue about a stable release closes once `engines.bb` includes it. A new result reopens the most recent closed issue instead of opening another, and comments appear only when the outcome changes.
- The README recommends an open-ended `engines.bb` and a daily caller that tests the BB nightly.
- Repository files: README badges and sections, changelog, security policy, code owners, funding, pull request template, and agent instructions.
