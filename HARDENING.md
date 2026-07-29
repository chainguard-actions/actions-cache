<!-- markdownlint-disable -->

# Hardening Report: actions--cache/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--cache/v5.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag-based or branch-based refs instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised.

Failing references:
- check-dist.yml: `uses: actions/reusable-workflows/.github/workflows/check-dist.yml@main`
- close-inactive-issues.yml: `uses: actions/stale@v9`
- codeql.yml: `uses: actions/checkout@v5`, `uses: github/codeql-action/init@v3`, `uses: github/codeql-action/autobuild@v3`, `uses: github/codeql-action/analyze@v3`
- licensed.yml: `uses: actions/checkout@v5`, `uses: ruby/setup-ruby@v1`
- publish-immutable-actions.yml: `uses: actions/checkout@v5`, `uses: actions/publish-immutable-action@0.0.3`
- release-new-action-version.yml: `uses: actions/publish-action@v0.3.0`
- workflow.yml: `uses: actions/checkout@v5` (×4), `uses: actions/setup-node@v4` (×2)

Locations:

- `.github/workflows/check-dist.yml:14`
- `.github/workflows/close-inactive-issues.yml:12`
- `.github/workflows/codeql.yml:17`
- `.github/workflows/codeql.yml:22`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:44`
- `.github/workflows/licensed.yml:21`
- `.github/workflows/licensed.yml:26`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:25`
- `.github/workflows/workflow.yml:25`
- `.github/workflows/workflow.yml:27`
- `.github/workflows/workflow.yml:43`
- `.github/workflows/workflow.yml:60`
- `.github/workflows/workflow.yml:113`
- `.github/workflows/workflow.yml:163`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands, allowing template substitution before the shell parses the string. This enables script injection if any of the values contain shell metacharacters.

(a) workflow.yml — `${{ runner.os }}` is interpolated directly into shell command arguments:
  - `run: __tests__/create-cache-files.sh ${{ runner.os }} test-cache`
  - `run: __tests__/verify-cache-files.sh ${{ runner.os }} test-cache`
  These should use the `$RUNNER_OS` environment variable instead.

(a) licensed.yml — `${{ github.workspace }}` is interpolated directly into a run: block:
  - `cd ${{ github.workspace }}`
  Should use `$GITHUB_WORKSPACE` instead.

(a) issue-opened-workflow.yml — `${{ github.repository }}`, `${{ github.event.issue.number }}`, and `${{ steps.oncall.outputs.CURRENT }}` are interpolated directly into curl command strings in run: blocks. `steps.oncall.outputs.CURRENT` is attacker-influenced (derived from external API data written to GITHUB_OUTPUT) and is especially dangerous.

(a) pr-opened-workflow.yml — Same pattern: `${{ github.repository }}`, `${{ github.event.pull_request.number }}`, and `${{ steps.oncall.outputs.CURRENT }}` are interpolated directly into curl -d request body strings. This workflow is triggered by `pull_request_target`, making it particularly high-risk as it runs with write permissions on attacker-controlled PR data.

Locations:

- `.github/workflows/workflow.yml:47`
- `.github/workflows/workflow.yml:50`
- `.github/workflows/workflow.yml:72`
- `.github/workflows/workflow.yml:75`
- `.github/workflows/licensed.yml:38`
- `.github/workflows/issue-opened-workflow.yml:20`
- `.github/workflows/issue-opened-workflow.yml:22`
- `.github/workflows/pr-opened-workflow.yml:20`
- `.github/workflows/pr-opened-workflow.yml:22`
- `.github/workflows/pr-opened-workflow.yml:26`

### github-env-injection (severity: high)

In both issue-opened-workflow.yml and pr-opened-workflow.yml, the 'Get current oncall' step writes the value of `steps.oncall.outputs.CURRENT` (the name of the on-call person, fetched from the PagerDuty API) to $GITHUB_OUTPUT without sanitization:

  `echo "CURRENT=$(... | jq -r '.oncalls[].user.name')" >> $GITHUB_OUTPUT`

The value is derived from an external API response and is not sanitized with `printf '%s' ... | tr -d '\n\r'` before being written to $GITHUB_OUTPUT. A malicious value containing newlines could inject additional key=value pairs into the GITHUB_OUTPUT file, potentially overwriting other outputs. The unsanitized output is then also directly interpolated into subsequent run: blocks (see script-injection finding).

Locations:

- `.github/workflows/issue-opened-workflow.yml:17`
- `.github/workflows/pr-opened-workflow.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding categories across 9 workflow files:

1. unpinned-uses: Pinned all 17 unpinned action references to full 40-char SHA digests with original tag as comment. Used lookup_action_sha for each: actions/reusable-workflows@main→9e7901ee, actions/stale@v9→5bef64f1, actions/checkout@v5→fbc6f399, github/codeql-action/*@v3→4187e74d, ruby/setup-ruby@v1→95ef2b04, actions/publish-immutable-action@0.0.3→4b1aa5c1 (resolved via v0.0.3 tag), actions/publish-action@v0.3.0→f784495c, actions/setup-node@v4→49933ea5.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks. In workflow.yml, replaced ${{ runner.os }} with $RUNNER_OS. In licensed.yml, replaced ${{ github.workspace }} with $GITHUB_WORKSPACE. In issue-opened-workflow.yml and pr-opened-workflow.yml, moved github.repository, issue/PR number, and steps.oncall.outputs.CURRENT into env: blocks and referenced as plain env vars.

3. github-env-injection: In both issue-opened-workflow.yml and pr-opened-workflow.yml, the PagerDuty API response is now captured into a variable, sanitized with `printf '%s' "$raw" | tr -d '\n\r'`, and then written to $GITHUB_OUTPUT, preventing newline injection attacks.

