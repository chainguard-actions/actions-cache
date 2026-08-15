<!-- markdownlint-disable -->

# Hardening Report: actions--cache/v6.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--cache/v6.1.0** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell commands. In workflow.yml, `${{ runner.os }}` is interpolated directly into shell command arguments on multiple lines (e.g., `run: __tests__/create-cache-files.sh ${{ runner.os }} test-cache` and `run: __tests__/verify-cache-files.sh ${{ runner.os }} ~/test-cache`). Even context values that appear GitHub-controlled flow through YAML template substitution before the shell sees them, enabling injection. Sub-rule (a) violated.

Locations:

- `.github/workflows/workflow.yml:43`
- `.github/workflows/workflow.yml:45`
- `.github/workflows/workflow.yml:68`
- `.github/workflows/workflow.yml:70`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell commands in issue-opened-workflow.yml. The 'add_assignees' step interpolates `${{github.repository}}`, `${{ github.event.issue.number}}`, and `${{steps.oncall.outputs.CURRENT}}` directly into a curl command string. These are attacker-controllable values (issue number from event, step output derived from external API). Sub-rule (a) violated.

Locations:

- `.github/workflows/issue-opened-workflow.yml:22`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell commands in pr-opened-workflow.yml. The 'Request Review' and 'Add Assignee' steps interpolate `${{github.repository}}`, `${{ github.event.pull_request.number}}`, and `${{steps.oncall.outputs.CURRENT}}` directly into curl command strings. This workflow is triggered by pull_request_target, making github.event.pull_request.number and steps outputs attacker-influenced. Sub-rule (a) violated.

Locations:

- `.github/workflows/pr-opened-workflow.yml:22`
- `.github/workflows/pr-opened-workflow.yml:27`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside a run: shell command in licensed.yml. The 'Check cached dependency records' step uses `cd ${{ github.workspace }}` directly in a run: block. Sub-rule (a) violated — any ${{ }} in a run: block is a script-injection risk regardless of context.

Locations:

- `.github/workflows/licensed.yml:40`

### unpinned-uses (severity: high)

Multiple uses: references in check-dist.yml are pinned to a branch or tag instead of a full 40-character commit SHA. Failing reference: `actions/reusable-workflows/.github/workflows/check-dist.yml@main` (branch ref).

Locations:

- `.github/workflows/check-dist.yml:14`

### unpinned-uses (severity: high)

uses: reference in close-inactive-issues.yml is pinned to a tag instead of a full 40-character commit SHA. Failing reference: `actions/stale@v9`.

Locations:

- `.github/workflows/close-inactive-issues.yml:12`

### unpinned-uses (severity: high)

Multiple uses: references in codeql.yml are pinned to tags instead of full 40-character commit SHAs. Failing references: `actions/checkout@v5`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`.

Locations:

- `.github/workflows/codeql.yml:17`
- `.github/workflows/codeql.yml:22`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:43`

### unpinned-uses (severity: high)

Multiple uses: references in licensed.yml are pinned to tags instead of full 40-character commit SHAs. Failing references: `actions/checkout@v5`, `ruby/setup-ruby@v1`.

Locations:

- `.github/workflows/licensed.yml:21`
- `.github/workflows/licensed.yml:27`

### unpinned-uses (severity: high)

Multiple uses: references in publish-immutable-actions.yml are pinned to tags instead of full 40-character commit SHAs. Failing references: `actions/checkout@v5`, `actions/publish-immutable-action@0.0.3`.

Locations:

- `.github/workflows/publish-immutable-actions.yml:13`
- `.github/workflows/publish-immutable-actions.yml:16`

### unpinned-uses (severity: high)

uses: reference in release-new-action-version.yml is pinned to a tag instead of a full 40-character commit SHA. Failing reference: `actions/publish-action@v0.3.0`.

Locations:

- `.github/workflows/release-new-action-version.yml:22`

### unpinned-uses (severity: high)

Multiple uses: references in workflow.yml are pinned to tags instead of full 40-character commit SHAs. Failing references: `actions/checkout@v5` (multiple occurrences), `actions/setup-node@v4`.

Locations:

- `.github/workflows/workflow.yml:25`
- `.github/workflows/workflow.yml:27`
- `.github/workflows/workflow.yml:42`
- `.github/workflows/workflow.yml:57`
- `.github/workflows/workflow.yml:100`
- `.github/workflows/workflow.yml:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all script-injection findings by moving ${{ }} expressions into env: blocks and referencing them as plain shell variables. Fixed all unpinned-uses findings by resolving tags/branches to full 40-character commit SHAs using lookup_action_sha. Files modified: workflow.yml (script-injection for runner.os in 4 places, pinned checkout@v5 in 5 places and setup-node@v4 in 1 place), issue-opened-workflow.yml (script-injection for github.repository, issue.number, steps output), pr-opened-workflow.yml (script-injection for github.repository, pr.number, steps output in 2 steps), licensed.yml (script-injection for github.workspace, pinned checkout@v5 and ruby/setup-ruby@v1), check-dist.yml (pinned reusable-workflows@main), close-inactive-issues.yml (pinned stale@v9), codeql.yml (pinned checkout@v5 and 3 codeql-action refs), publish-immutable-actions.yml (pinned checkout@v5 and publish-immutable-action@0.0.3), release-new-action-version.yml (pinned publish-action@v0.3.0).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in two workflow files by moving `${{ secrets.PAGERDUTY_TOKEN }}` out of the `run:` shell command string and into an `env:` block in the 'Get current oncall' step of both `.github/workflows/issue-opened-workflow.yml` and `.github/workflows/pr-opened-workflow.yml`. The curl command now references the token as the plain environment variable `$PAGERDUTY_TOKEN`, preventing the Actions template engine from substituting the secret value directly into the shell command string.

