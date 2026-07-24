<!-- markdownlint-disable -->

# Hardening Report: actions--cache/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--cache/v4.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks.

- check-dist.yml: `actions/reusable-workflows/.github/workflows/check-dist.yml@main` (branch ref)
- close-inactive-issues.yml: `actions/stale@v9`
- codeql.yml: `actions/checkout@v4`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`
- licensed.yml: `actions/checkout@v4`, `ruby/setup-ruby@v1`
- publish-immutable-actions.yml: `actions/checkout@v4`, `actions/publish-immutable-action@0.0.3`
- release-new-action-version.yml: `actions/publish-action@v0.3.0`
- workflow.yml: `actions/checkout@v4`, `actions/setup-node@v4`

Locations:

- `.github/workflows/check-dist.yml:13`
- `.github/workflows/close-inactive-issues.yml:10`
- `.github/workflows/codeql.yml:17`
- `.github/workflows/codeql.yml:22`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:43`
- `.github/workflows/licensed.yml:16`
- `.github/workflows/licensed.yml:20`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/publish-immutable-actions.yml:19`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/workflow.yml:22`
- `.github/workflows/workflow.yml:24`

### script-injection (severity: high)

Multiple workflow run: blocks directly interpolate GitHub Actions expressions (${{ ... }}) into shell commands, violating rule (a). This allows template substitution to inject arbitrary shell metacharacters before the shell parses the command.

- workflow.yml: `${{ runner.os }}` is interpolated directly into shell run: commands passed as arguments to test scripts (e.g., `run: __tests__/create-cache-files.sh ${{ runner.os }} test-cache`).
- licensed.yml: `${{ github.workspace }}` is interpolated directly into a `cd` command in a run: block.
- issue-opened-workflow.yml: `${{github.repository}}`, `${{ github.event.issue.number}}`, and `${{steps.oncall.outputs.CURRENT}}` are all interpolated directly into a curl command in a run: block.
- pr-opened-workflow.yml: `${{github.repository}}`, `${{ github.event.pull_request.number}}`, and `${{steps.oncall.outputs.CURRENT}}` are interpolated directly into curl commands in run: blocks. This workflow is triggered by `pull_request_target`, making `github.event.pull_request.*` attacker-controlled.

Locations:

- `.github/workflows/workflow.yml:30`
- `.github/workflows/workflow.yml:33`
- `.github/workflows/workflow.yml:55`
- `.github/workflows/workflow.yml:58`
- `.github/workflows/licensed.yml:33`
- `.github/workflows/issue-opened-workflow.yml:15`
- `.github/workflows/pr-opened-workflow.yml:15`
- `.github/workflows/pr-opened-workflow.yml:18`
- `.github/workflows/pr-opened-workflow.yml:21`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be broad), violating the principle of least privilege.

- workflow.yml: No top-level or job-level permissions defined across all jobs (build, test-save, test-restore, test-proxy-save, test-proxy-restore).
- check-dist.yml: No top-level or job-level permissions defined.
- licensed.yml: No top-level or job-level permissions defined.
- issue-opened-workflow.yml: No top-level or job-level permissions defined.
- pr-opened-workflow.yml: No top-level or job-level permissions defined (especially concerning as it is triggered by `pull_request_target`).

Locations:

- `.github/workflows/workflow.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/issue-opened-workflow.yml:1`
- `.github/workflows/pr-opened-workflow.yml:1`

### github-env-injection (severity: high)

In issue-opened-workflow.yml and pr-opened-workflow.yml, the 'Get current oncall' step writes the output of a curl+jq pipeline directly to $GITHUB_OUTPUT without sanitization (`echo "CURRENT=$(curl ... | jq -r '.oncalls[].user.name')" >> $GITHUB_OUTPUT`). The value from the external API response may contain newlines or special characters that could inject additional key=value pairs into the GITHUB_OUTPUT file, which are then consumed by subsequent steps. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `.github/workflows/issue-opened-workflow.yml:11`
- `.github/workflows/pr-opened-workflow.yml:11`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions, github-env-injection

**Notes:**

Fixed all findings across 8 workflow files:

**unpinned-uses**: Pinned all action references to full 40-char SHAs:
- check-dist.yml: actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2
- close-inactive-issues.yml: actions/stale@v9 → @5bef64f19d7facfb25b37b414482c7164d639639
- codeql.yml: actions/checkout@v4 → @11d596..., github/codeql-action/{init,autobuild,analyze}@v3 → @4187e74...
- licensed.yml: actions/checkout@v4 → @11d596..., ruby/setup-ruby@v1 → @95ef2b0...
- publish-immutable-actions.yml: actions/checkout@v4 → @11d596..., actions/publish-immutable-action@0.0.3 → @4b1aa5c... (resolved as v0.0.3)
- release-new-action-version.yml: actions/publish-action@v0.3.0 → @f784495...
- workflow.yml: actions/checkout@v4 → @11d596..., actions/setup-node@v4 → @49933ea...

**script-injection**: Moved all ${{ }} expressions out of run: blocks into env: blocks:
- workflow.yml: ${{ runner.os }} moved to RUNNER_OS env var in test-save/test-restore jobs
- licensed.yml: ${{ github.workspace }} moved to WORKSPACE env var
- issue-opened-workflow.yml: All github context expressions moved to env vars
- pr-opened-workflow.yml: All github context expressions moved to env vars

**missing-permissions**: Added top-level permissions blocks:
- workflow.yml: permissions: contents: read
- check-dist.yml: permissions: contents: read
- licensed.yml: permissions: contents: read
- issue-opened-workflow.yml: permissions: issues: write
- pr-opened-workflow.yml: permissions: pull-requests: write

**github-env-injection**: Sanitized GITHUB_OUTPUT writes in issue-opened-workflow.yml and pr-opened-workflow.yml using printf '%s' | tr -d '\n\r' before writing the oncall value.

