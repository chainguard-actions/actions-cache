<!-- markdownlint-disable -->

# Hardening Report: actions--cache/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--cache/v4.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings. In issue-opened-workflow.yml, the run: blocks use ${{github.repository}}, ${{ github.event.issue.number}}, and ${{steps.oncall.outputs.CURRENT}} directly in shell commands. In pr-opened-workflow.yml, the run: blocks use ${{github.repository}}, ${{ github.event.pull_request.number}}, and ${{steps.oncall.outputs.CURRENT}} directly in shell commands. In licensed.yml, ${{ github.workspace }} is interpolated directly in a run: block. In workflow.yml, ${{ runner.os }} is interpolated directly in run: shell commands.

Locations:

- `.github/workflows/issue-opened-workflow.yml:12`
- `.github/workflows/issue-opened-workflow.yml:16`
- `.github/workflows/pr-opened-workflow.yml:11`
- `.github/workflows/pr-opened-workflow.yml:15`
- `.github/workflows/pr-opened-workflow.yml:20`
- `.github/workflows/licensed.yml:34`
- `.github/workflows/workflow.yml:27`
- `.github/workflows/workflow.yml:30`
- `.github/workflows/workflow.yml:52`
- `.github/workflows/workflow.yml:57`

### github-env-injection (severity: high)

A run: block writes a value derived from an untrusted/external source to $GITHUB_OUTPUT without sanitization. In issue-opened-workflow.yml, the 'Get current oncall' step writes the result of a curl+jq pipeline (CURRENT=...) to $GITHUB_OUTPUT without applying printf '%s' ... | tr -d '\n\r'. In pr-opened-workflow.yml, the same pattern is used. The CURRENT value is then used unsanitized in subsequent run: steps via ${{steps.oncall.outputs.CURRENT}}.

Locations:

- `.github/workflows/issue-opened-workflow.yml:12`
- `.github/workflows/pr-opened-workflow.yml:11`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of full 40-character SHA commit digests. Failing references: check-dist.yml: actions/reusable-workflows/.github/workflows/check-dist.yml@main (branch). close-inactive-issues.yml: actions/stale@v9 (tag). codeql.yml: actions/checkout@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3 (tags). licensed.yml: actions/checkout@v4, ruby/setup-ruby@v1 (tags). publish-immutable-actions.yml: actions/checkout@v4, actions/publish-immutable-action@0.0.3 (tags). release-new-action-version.yml: actions/publish-action@v0.3.0 (tag). workflow.yml: actions/checkout@v4, actions/setup-node@v4 (tags).

Locations:

- `.github/workflows/check-dist.yml:9`
- `.github/workflows/close-inactive-issues.yml:10`
- `.github/workflows/codeql.yml:18`
- `.github/workflows/codeql.yml:23`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:40`
- `.github/workflows/licensed.yml:18`
- `.github/workflows/licensed.yml:22`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/workflow.yml:21`
- `.github/workflows/workflow.yml:23`

### missing-permissions (severity: medium)

Several workflow files have no top-level permissions: key and no job-level permissions: blocks, meaning they run with the default (potentially broad) token permissions. Affected files: check-dist.yml (delegates to reusable workflow with no permissions set), issue-opened-workflow.yml (no permissions at any level), licensed.yml (no permissions at any level), workflow.yml (no permissions at any level).

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/issue-opened-workflow.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/workflow.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four finding types across 9 workflow files:

1. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks in issue-opened-workflow.yml, pr-opened-workflow.yml, licensed.yml (replaced ${{ github.workspace }} with $GITHUB_WORKSPACE env var), and workflow.yml (moved runner.os to RUNNER_OS env var).

2. github-env-injection: In both issue-opened-workflow.yml and pr-opened-workflow.yml, sanitized the curl+jq output before writing to $GITHUB_OUTPUT using: raw=$(...); safe=$(printf '%s' "$raw" | tr -d '\n\r'); echo "CURRENT=$safe" >> "$GITHUB_OUTPUT".

3. unpinned-uses: Pinned all action references to full 40-character SHA digests:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/stale@v9 → @5bef64f19d7facfb25b37b414482c7164d639639
   - github/codeql-action/init@v3 → @02c5e83432fe5497fd85b873b6c9f16a8578e1d9
   - github/codeql-action/autobuild@v3 → @02c5e83432fe5497fd85b873b6c9f16a8578e1d9
   - github/codeql-action/analyze@v3 → @02c5e83432fe5497fd85b873b6c9f16a8578e1d9
   - ruby/setup-ruby@v1 → @d45b1a4e94b71acab930e56e79c6aa188764e7f9
   - actions/publish-immutable-action@0.0.3 → @4b1aa5c1cde5fedc80d52746c9546cb5560e5f53 (resolved as v0.0.3)
   - actions/publish-action@v0.3.0 → @f784495ce78a41bac4ed7e34a73f0034015764bb
   - actions/reusable-workflows@main → @4735e71081024a944852f4ab9d1495b6dd2de8f2

4. missing-permissions: Added top-level permissions blocks to check-dist.yml (contents: read), issue-opened-workflow.yml (issues: write), licensed.yml (contents: read), and workflow.yml (contents: read).

### Iteration 2

**Fixes applied:** missing-permissions

**Notes:**

Added a top-level `permissions: pull-requests: write` block to `.github/workflows/pr-opened-workflow.yml`. The workflow uses `pull_request_target` and makes API calls to request reviewers and add assignees to PRs, both of which require `pull-requests: write`. No other permissions are needed, so all other permissions default to `none`, satisfying the principle of least privilege.

