<!-- markdownlint-disable -->

# Hardening Report: actions--cache/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--cache/v6.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are directly interpolated inside run: shell commands, allowing template substitution before the shell processes the value. In workflow.yml, `${{ runner.os }}` is interpolated directly into shell script arguments on four lines. In issue-opened-workflow.yml, `${{github.repository}}`, `${{ github.event.issue.number}}`, and `${{steps.oncall.outputs.CURRENT}}` are interpolated directly into a curl command. In pr-opened-workflow.yml, the same contexts are interpolated into curl commands in two steps. In licensed.yml, `${{ github.workspace }}` is interpolated directly into a run: block (`cd ${{ github.workspace }}`). All of these should be moved to env: variables and then double-quoted in the shell script.

Locations:

- `.github/workflows/workflow.yml:43`
- `.github/workflows/workflow.yml:46`
- `.github/workflows/workflow.yml:65`
- `.github/workflows/workflow.yml:68`
- `.github/workflows/issue-opened-workflow.yml:17`
- `.github/workflows/issue-opened-workflow.yml:21`
- `.github/workflows/pr-opened-workflow.yml:17`
- `.github/workflows/pr-opened-workflow.yml:21`
- `.github/workflows/pr-opened-workflow.yml:27`
- `.github/workflows/licensed.yml:33`

### github-env-injection (severity: high)

In issue-opened-workflow.yml and pr-opened-workflow.yml, the 'Get current oncall' step writes the output of an external API call (via curl + jq) directly to $GITHUB_OUTPUT without sanitization: `echo "CURRENT=$(curl ... | jq -r '.oncalls[].user.name')" >> $GITHUB_OUTPUT`. The value from the external PagerDuty API response is written to $GITHUB_OUTPUT without applying `printf '%s' ... | tr -d '\n\r'` to strip newlines. A malicious or compromised API response containing newline characters could inject additional key=value pairs into $GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `.github/workflows/issue-opened-workflow.yml:17`
- `.github/workflows/pr-opened-workflow.yml:17`

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files uses a mutable tag or version string instead of a full 40-character commit SHA. This exposes the workflows to supply-chain attacks if any referenced action's tag is moved or compromised. Failing references include: `actions/reusable-workflows/.github/workflows/check-dist.yml@main` (check-dist.yml), `actions/stale@v9` (close-inactive-issues.yml), `actions/checkout@v5` (codeql.yml, licensed.yml, publish-immutable-actions.yml, workflow.yml), `github/codeql-action/init@v3` (codeql.yml), `github/codeql-action/autobuild@v3` (codeql.yml), `github/codeql-action/analyze@v3` (codeql.yml), `ruby/setup-ruby@v1` (licensed.yml), `actions/publish-immutable-action@0.0.3` (publish-immutable-actions.yml), `actions/publish-action@v0.3.0` (release-new-action-version.yml), `actions/setup-node@v4` (workflow.yml). All should be pinned to full SHA digests.

Locations:

- `.github/workflows/check-dist.yml:14`
- `.github/workflows/close-inactive-issues.yml:10`
- `.github/workflows/codeql.yml:14`
- `.github/workflows/codeql.yml:18`
- `.github/workflows/codeql.yml:27`
- `.github/workflows/codeql.yml:38`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/licensed.yml:18`
- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/workflow.yml:26`
- `.github/workflows/workflow.yml:28`
- `.github/workflows/workflow.yml:40`
- `.github/workflows/workflow.yml:55`
- `.github/workflows/workflow.yml:100`
- `.github/workflows/workflow.yml:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three finding types across 7 workflow files:

1. script-injection: Moved all ${{ }} expressions from run: shell commands to env: blocks in workflow.yml (runner.os in 4 steps), issue-opened-workflow.yml (github.repository, github.event.issue.number, steps.oncall.outputs.CURRENT), pr-opened-workflow.yml (same contexts in 2 steps), and licensed.yml (github.workspace).

2. github-env-injection: In issue-opened-workflow.yml and pr-opened-workflow.yml, the PagerDuty API response is now captured in a variable, sanitized with printf '%s' "$raw" | tr -d '\n\r', and then written to $GITHUB_OUTPUT.

3. unpinned-uses: Pinned all 10 action references to full 40-character commit SHAs across check-dist.yml, close-inactive-issues.yml, codeql.yml, licensed.yml, publish-immutable-actions.yml, release-new-action-version.yml, and workflow.yml. actions/checkout@v5->fbc6f39, actions/setup-node@v4->49933ea, actions/stale@v9->5bef64f, github/codeql-action/*@v3->4187e74, ruby/setup-ruby@v1->95ef2b0, actions/publish-immutable-action@0.0.3->4b1aa5c, actions/publish-action@v0.3.0->f784495, actions/reusable-workflows@main->4735e71.

