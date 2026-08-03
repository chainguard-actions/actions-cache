<!-- markdownlint-disable -->

# Hardening Report: actions--cache/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--cache/v4** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks.

.github/workflows/check-dist.yml: `actions/reusable-workflows/.github/workflows/check-dist.yml@main`
.github/workflows/close-inactive-issues.yml: `actions/stale@v9`
.github/workflows/codeql.yml: `actions/checkout@v4`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`
.github/workflows/licensed.yml: `actions/checkout@v4`, `ruby/setup-ruby@v1`
.github/workflows/publish-immutable-actions.yml: `actions/checkout@v4`, `actions/publish-immutable-action@0.0.3`
.github/workflows/release-new-action-version.yml: `actions/publish-action@v0.3.0`
.github/workflows/workflow.yml: `actions/checkout@v4`, `actions/setup-node@v4`

Locations:

- `.github/workflows/check-dist.yml:9`
- `.github/workflows/close-inactive-issues.yml:10`
- `.github/workflows/codeql.yml:16`
- `.github/workflows/codeql.yml:21`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:43`
- `.github/workflows/licensed.yml:17`
- `.github/workflows/licensed.yml:22`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/publish-immutable-actions.yml:18`
- `.github/workflows/release-new-action-version.yml:21`
- `.github/workflows/workflow.yml:18`
- `.github/workflows/workflow.yml:20`

### script-injection (severity: high)

Multiple run: blocks interpolate ${{ }} expressions directly into shell commands (rule a), allowing an attacker to inject arbitrary shell commands.

1. workflow.yml: `${{ runner.os }}` is interpolated directly in four `run:` steps that pass it as a shell argument to test scripts. Any expression in a run: block is a script-injection risk regardless of context.
   - `run: __tests__/create-cache-files.sh ${{ runner.os }} test-cache` (×2)
   - `run: __tests__/verify-cache-files.sh ${{ runner.os }} test-cache` (×2)

2. licensed.yml: `${{ github.workspace }}` is interpolated directly in a `run:` block:
   - `cd ${{ github.workspace }}`

3. issue-opened-workflow.yml: The `add_assignees` run: block interpolates `${{github.repository}}`, `${{ github.event.issue.number}}`, and `${{steps.oncall.outputs.CURRENT}}` directly into a shell curl command. `github.event.issue.number` and `steps.oncall.outputs.CURRENT` are attacker-influenced values.

4. pr-opened-workflow.yml: The `Request Review` and `Add Assignee` run: blocks interpolate `${{github.repository}}`, `${{ github.event.pull_request.number}}`, and `${{steps.oncall.outputs.CURRENT}}` directly into shell curl commands. This workflow is triggered by `pull_request_target`, making `github.event.pull_request.number` and `steps.oncall.outputs.CURRENT` attacker-controlled.

Locations:

- `.github/workflows/workflow.yml:43`
- `.github/workflows/workflow.yml:46`
- `.github/workflows/workflow.yml:62`
- `.github/workflows/workflow.yml:65`
- `.github/workflows/licensed.yml:38`
- `.github/workflows/issue-opened-workflow.yml:14`
- `.github/workflows/pr-opened-workflow.yml:14`
- `.github/workflows/pr-opened-workflow.yml:18`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be broad), violating the principle of least privilege.

- workflow.yml: no permissions at top level or on any of its 4 jobs (build, test-save, test-restore, test-proxy-save, test-proxy-restore)
- check-dist.yml: no permissions at top level or job level
- issue-opened-workflow.yml: no permissions at top level or job level
- licensed.yml: no permissions at top level or job level
- pr-opened-workflow.yml: no permissions at top level or job level

Locations:

- `.github/workflows/workflow.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/issue-opened-workflow.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/pr-opened-workflow.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding categories across 7 workflow files:

1. unpinned-uses: Pinned all 13 action references to full 40-char SHA hashes with tag comments preserved. Used lookup_action_sha for all SHAs. Note: actions/publish-immutable-action@0.0.3 was resolved as v0.0.3 tag (SHA: 4b1aa5c1cde5fedc80d52746c9546cb5560e5f53).

2. script-injection: Moved all ${{ }} expressions out of run: blocks into step env: blocks. In workflow.yml, runner.os is now passed via RUNNER_OS env var. In licensed.yml, replaced ${{ github.workspace }} with the built-in $GITHUB_WORKSPACE env var. In issue-opened-workflow.yml and pr-opened-workflow.yml, all github context values and step outputs are now in env: blocks, with ONCALL_USER sanitized via tr -d '\n\r' before use in curl commands.

3. missing-permissions: Added top-level permissions blocks to workflow.yml (contents: read), check-dist.yml (contents: read), licensed.yml (contents: read), issue-opened-workflow.yml (issues: write), and pr-opened-workflow.yml (pull-requests: write, issues: write). The close-inactive-issues.yml already had job-level permissions so was not changed.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in both .github/workflows/issue-opened-workflow.yml (line 18) and .github/workflows/pr-opened-workflow.yml (line 18). In both files, the 'Get current oncall' step was rewritten to: (1) capture the curl|jq output into a 'raw' variable, (2) sanitize it with `printf '%s' "$raw" | tr -d '\n\r'` into a 'safe' variable, and (3) write `echo "CURRENT=$safe" >> "$GITHUB_OUTPUT"`. This prevents newline injection from externally-sourced PagerDuty API data into the GITHUB_OUTPUT special environment file.

