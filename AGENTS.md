# Working in BB Plugin CI

## Scope

This repository holds reusable GitHub Actions workflows for BB plugins. Today
that is `bb-compat`: it warns a plugin author when a BB release breaks the
plugin's checks. The recommended caller tests the latest stable BB daily. BB itself owns
install and update compatibility (`bb plugin outdated`, `bb plugin update`); do
not duplicate those here. Plugin specific steps belong in the plugin's caller
workflow inputs, not in this repository.

## Workflow rules

- `.github/workflows/bb-compat.yml` is called as `@v1` by every plugin. Its
  inputs, outputs, issue label and issue titles are a public interface. Adding
  an optional input is compatible. Renaming or removing one, or changing the
  label, needs a new major tag (`v2`).
- Least privilege: the job that runs plugin code keeps `contents: read` and
  `persist-credentials: false`. Only the report job gets `issues: write`, and it
  runs no plugin code.
- Pass inputs and step outputs to shell through `env:`, never by `${{ }}`
  interpolation inside `run:`.
- Keep the README inputs table, outputs line and caller example in step with
  the workflow.

## Checks and changes

- Keep changes focused. Commit verified slices on a task branch. Follow the PR
  template; do not merge without the user's request. No AI authors or
  attribution trailers.
- There is no package or build. The check is the Self-test workflow, which runs
  on every pull request that changes a workflow and on `main`. Link its run in
  the PR. For a change that affects a specific plugin, also run that plugin's
  caller against your branch.
- The self-test opens and closes real `bb-compat` issues in this repository.
  Leave them; they are the evidence.
- Keep credentials, machine configuration and private notes out of this public
  repository.

## Releases

- Callers pin the major tag `v1`. The first release is `v1.0.0`, and `v1`
  starts on the same commit.
- To release: move the `## [Unreleased]` notes into a `## [1.Y.Z]` section
  through a PR. After it merges, run the self-test on `main`
  (`gh workflow run self-test.yml --ref main`; a changelog-only merge does not
  start it) and wait for it to pass. Then tag
  that commit with `git tag -a v1.Y.Z -m 'Release v1.Y.Z'`, push the tag, and
  publish a GitHub Release from that changelog section. Then move `v1` to the
  same commit (`git tag -f v1 <commit> && git push -f origin v1`) and say so in
  the release notes.
- Never move `v1` silently, and never retag an existing `v1.Y.Z`. A breaking
  change gets `v2`.
