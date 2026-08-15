<!-- markdownlint-disable -->

# Hardening Report: actions--cache/v5.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--cache/v5.0.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full 40-character SHA commits. This exposes the workflow to supply-chain attacks if the referenced tag or branch is moved to a malicious commit. Failing references include: check-dist.yml: `actions/reusable-workflows/.github/workflows/check-dist.yml@main`; close-inactive-issues.yml: `actions/stale@v9`; codeql.yml: `actions/checkout@v5`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`; licensed.yml: `actions/checkout@v5`, `ruby/setup-ruby@v1`; publish-immutable-actions.yml: `actions/checkout@v5`, `actions/publish-immutable-action@0.0.3`; release-new-action-version.yml: `actions/publish-action@v0.3.0`; workflow.yml: `actions/checkout@v5`, `actions/setup-node@v4`.

Locations:

- `.github/workflows/check-dist.yml:16`
- `.github/workflows/close-inactive-issues.yml:13`
- `.github/workflows/codeql.yml:18`
- `.github/workflows/codeql.yml:22`
- `.github/workflows/codeql.yml:32`
- `.github/workflows/codeql.yml:46`
- `.github/workflows/licensed.yml:19`
- `.github/workflows/licensed.yml:24`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/workflow.yml:26`
- `.github/workflows/workflow.yml:29`
- `.github/workflows/workflow.yml:44`
- `.github/workflows/workflow.yml:57`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands, violating rule (a). (1) issue-opened-workflow.yml 'add_assignees' step: `${{github.repository}}`, `${{ github.event.issue.number}}`, and `${{steps.oncall.outputs.CURRENT}}` are interpolated directly into a curl command string. (2) pr-opened-workflow.yml 'Request Review' and 'Add Assignee' steps: `${{github.repository}}`, `${{ github.event.pull_request.number}}`, and `${{steps.oncall.outputs.CURRENT}}` are interpolated directly into curl command strings. (3) workflow.yml 'Generate files' and 'Verify cache files' steps: `${{ runner.os }}` is interpolated directly into shell script invocations (e.g., `run: __tests__/create-cache-files.sh ${{ runner.os }} test-cache`). (4) licensed.yml 'Check cached dependency records' step: `${{ github.workspace }}` is interpolated directly into a `cd` command.

Locations:

- `.github/workflows/issue-opened-workflow.yml:18`
- `.github/workflows/issue-opened-workflow.yml:21`
- `.github/workflows/pr-opened-workflow.yml:15`
- `.github/workflows/pr-opened-workflow.yml:21`
- `.github/workflows/pr-opened-workflow.yml:25`
- `.github/workflows/pr-opened-workflow.yml:29`
- `.github/workflows/workflow.yml:36`
- `.github/workflows/workflow.yml:38`
- `.github/workflows/workflow.yml:55`
- `.github/workflows/workflow.yml:59`
- `.github/workflows/licensed.yml:37`

### github-env-injection (severity: high)

In both issue-opened-workflow.yml and pr-opened-workflow.yml, the 'Get current oncall' step writes the output of a `jq` command (which processes external API response data from PagerDuty) directly to $GITHUB_OUTPUT without sanitization via `printf '%s' ... | tr -d '\n\r'`. An attacker who can influence the PagerDuty API response (e.g., by controlling a user's display name) could inject newlines into $GITHUB_OUTPUT, potentially overwriting subsequent output variables. The pattern is: `echo "CURRENT=$(... | jq -r '.oncalls[].user.name')" >> $GITHUB_OUTPUT`.

Locations:

- `.github/workflows/issue-opened-workflow.yml:17`
- `.github/workflows/pr-opened-workflow.yml:15`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all findings across 8 workflow files:

1. unpinned-uses: Pinned all action references to full SHA commits:
   - check-dist.yml: actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2
   - close-inactive-issues.yml: actions/stale@v9 → @5bef64f19d7facfb25b37b414482c7164d639639
   - codeql.yml: actions/checkout@v5 → @93cb6efe..., github/codeql-action/{init,autobuild,analyze}@v3 → @b7351df7...
   - licensed.yml: actions/checkout@v5 → @93cb6efe..., ruby/setup-ruby@v1 → @003a5c4d...
   - publish-immutable-actions.yml: actions/checkout@v5 → @93cb6efe..., actions/publish-immutable-action@0.0.3 → @4b1aa5c1... (resolved as v0.0.3)
   - release-new-action-version.yml: actions/publish-action@v0.3.0 → @f784495c...
   - workflow.yml: actions/checkout@v5 → @93cb6efe... (multiple), actions/setup-node@v4 → @49933ea5...

2. script-injection: Moved all ${{ }} expressions from run: blocks to env: blocks:
   - workflow.yml: runner.os moved to RUNNER_OS env var for create-cache-files.sh and verify-cache-files.sh calls
   - licensed.yml: github.workspace moved to WORKSPACE env var for cd command
   - issue-opened-workflow.yml: github.repository, github.event.issue.number, steps.oncall.outputs.CURRENT moved to env vars
   - pr-opened-workflow.yml: github.repository, github.event.pull_request.number, steps.oncall.outputs.CURRENT moved to env vars

3. github-env-injection: Both issue-opened-workflow.yml and pr-opened-workflow.yml now sanitize PagerDuty API output before writing to $GITHUB_OUTPUT using `printf '%s' "$raw" | tr -d '\n\r'` to strip newlines that could enable output variable injection.

