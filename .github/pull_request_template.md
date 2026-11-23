## Why

<!-- What was wrong or missing, and who it affects. Link the issue: "Fixes #123". -->

## What changed

<!-- The changes a reviewer needs to know, in a few bullets. -->

## How I verified

<!--
- The Self-test run on this pull request (link it).
- Any plugin you pointed at this branch (`@your-branch`), with the run link.
- Inputs, outputs or issue text you checked by hand.
-->

## Merge Danger

**Door:** <!-- two-way: a revert fully undoes it (wording, docs, a new optional input).
one-way: a revert can't fully undo it (renamed or removed inputs or outputs,
issue title or label changes that orphan open issues, a moved v1 tag). -->

**Blast Radius:** <!-- small: docs or the self-test only. medium: one input or code path.
large: every plugin calling @v1 once v1 moves. Say why. -->

## Checklist

- [ ] Self-test passes on this pull request, or this pull request changes no workflow
- [ ] README inputs, outputs and caller example match the workflow
- [ ] CHANGELOG describes user-visible changes under `## [Unreleased]`
- [ ] Moving `v1` is planned as a separate release step, not part of this PR
- [ ] No AI tool, model or agent credited as an author or co-author
