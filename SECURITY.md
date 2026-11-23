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
grants and the job's `github.token`; it reads no other secrets.

- The check job installs and runs the plugin's code with `contents: read` and
  does not keep the checkout token. `npm ci` lifecycle scripts and the `setup`
  input run with the job's full environment; each `checks` command runs with
  only `PATH` and `HOME`. `setup` and `checks` are shell commands by design, so
  callers must treat them as code.
- On failure it uploads `e2e-artifacts/` as a run artifact, which anyone who can
  read the repository's Actions runs can download.
- A separate report job with `issues: write` runs no plugin code. It creates or
  updates the `bb-compat` label, and creates, edits, comments on and closes
  issues with that label.
- Inputs reach shell steps through environment variables, not expression
  interpolation.
- It downloads packages from the npm registry and, when asked, apt packages and
  `@openai/codex`.
