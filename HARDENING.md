<!-- markdownlint-disable -->

# Hardening Report: actions--cache/v5.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--cache/v5.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 17 `uses:` references across all workflow files are pinned to mutable tags, version strings, or branch names rather than immutable 40-character SHA commit hashes. This exposes the action to supply-chain attacks where a tag is silently moved to a different (potentially malicious) commit. Affected references include: `actions/checkout@v5`, `actions/setup-node@v4`, `actions/stale@v9`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`, `ruby/setup-ruby@v1`, `actions/publish-immutable-action@0.0.3`, `actions/publish-action@v0.3.0`, and `actions/reusable-workflows/.github/workflows/check-dist.yml@main`.

Locations:

- `.github/workflows/check-dist.yml:14`
- `.github/workflows/close-inactive-issues.yml:13`
- `.github/workflows/codeql.yml:14`
- `.github/workflows/codeql.yml:19`
- `.github/workflows/codeql.yml:28`
- `.github/workflows/codeql.yml:38`
- `.github/workflows/licensed.yml:16`
- `.github/workflows/licensed.yml:20`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/workflow.yml:20`
- `.github/workflows/workflow.yml:22`
- `.github/workflows/workflow.yml:35`
- `.github/workflows/workflow.yml:51`
- `.github/workflows/workflow.yml:100`
- `.github/workflows/workflow.yml:163`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell command strings, violating sub-rule (a). Before the shell ever sees the command, GitHub Actions performs YAML template substitution, allowing an attacker-controlled value to inject shell metacharacters.

(1) `.github/workflows/workflow.yml` — `${{ runner.os }}` is interpolated directly into shell commands: `run: __tests__/create-cache-files.sh ${{ runner.os }} test-cache` and `run: __tests__/verify-cache-files.sh ${{ runner.os }} test-cache`. Although `runner.os` is GitHub-controlled, any `${{ ... }}` inside a `run:` block is a script-injection finding per the check rules.

(2) `.github/workflows/issue-opened-workflow.yml` — The `add_assignees` step interpolates `${{github.repository}}`, `${{ github.event.issue.number}}`, and `${{steps.oncall.outputs.CURRENT}}` directly into a `curl` shell command.

(3) `.github/workflows/pr-opened-workflow.yml` — The `Request Review` and `Add Assignee` steps interpolate `${{github.repository}}`, `${{ github.event.pull_request.number}}`, and `${{steps.oncall.outputs.CURRENT}}` directly into `curl` shell commands. This workflow uses `pull_request_target`, making it especially dangerous as it runs with write permissions against attacker-supplied PR data.

(4) `.github/workflows/licensed.yml` — `cd ${{ github.workspace }}` interpolates a GitHub context value directly into a shell command.

Locations:

- `.github/workflows/workflow.yml:30`
- `.github/workflows/workflow.yml:33`
- `.github/workflows/workflow.yml:55`
- `.github/workflows/workflow.yml:58`
- `.github/workflows/issue-opened-workflow.yml:21`
- `.github/workflows/pr-opened-workflow.yml:21`
- `.github/workflows/pr-opened-workflow.yml:25`
- `.github/workflows/licensed.yml:38`

### github-env-injection (severity: high)

Two workflow files write a value derived from `${{ steps.oncall.outputs.CURRENT }}` — a step output that is populated from an external API response — to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in the PagerDuty API response could allow an attacker to inject arbitrary key=value pairs into the GitHub output environment.

(1) `.github/workflows/issue-opened-workflow.yml` — The `Get current oncall` step writes: `echo "CURRENT=$(curl ... | jq -r '.oncalls[].user.name')" >> $GITHUB_OUTPUT`. The value from `jq` is unsanitized.

(2) `.github/workflows/pr-opened-workflow.yml` — Identical pattern in the `Get current oncall` step: `echo "CURRENT=$(curl ... | jq -r '.oncalls[].user.name')" >> $GITHUB_OUTPUT`. The value from `jq` is unsanitized.

Locations:

- `.github/workflows/issue-opened-workflow.yml:16`
- `.github/workflows/pr-opened-workflow.yml:16`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 17 unpinned `uses:` references across 8 workflow files by resolving each to its full 40-character SHA commit hash (preserving the original tag as a comment). Fixed script-injection in workflow.yml (4 runner.os references), licensed.yml (github.workspace), issue-opened-workflow.yml, and pr-opened-workflow.yml by moving all ${{ }} expressions into step `env:` blocks. Fixed github-env-injection in issue-opened-workflow.yml and pr-opened-workflow.yml by sanitizing the PagerDuty API response with `printf '%s' "$raw" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. Note: actions/publish-immutable-action@0.0.3 was resolved using the v0.0.3 tag (with 'v' prefix) which resolved to SHA 4b1aa5c1cde5fedc80d52746c9546cb5560e5f53.

