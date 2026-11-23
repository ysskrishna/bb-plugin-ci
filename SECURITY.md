# Security policy

## Supported versions

Security fixes target the current `v1` tag and `main`. Older commits and
unmerged branches are not maintained separately.

## Report a vulnerability

Use [GitHub's private vulnerability reporting](https://github.com/ysskrishna/bb-plugin-ci/security/advisories/new).
Include the affected tag or commit, reproduction steps, and likely impact.
Do not post tokens, private repository contents, or exploit details in public issues.

The maintainer will review the report and coordinate a fix and disclosure. There
is no guaranteed response time or bug bounty program.

## Workflow access

`bb-compat` runs in the calling repository with the permissions the caller
grants. The job that installs and runs the plugin's code has `contents: read`
and does not keep the checkout token. A separate job with `issues: write` reads,
creates, edits and closes issues labelled `bb-compat` in the calling repository;
it runs no plugin code. Inputs reach shell steps through environment variables,
not expression interpolation. The workflow downloads packages from the npm
registry and, when asked, apt packages and `@openai/codex`. It uses no secrets.
