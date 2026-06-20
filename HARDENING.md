<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **openfga--action-openfga-test/v0.1.1** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` expressions are directly interpolated inside the `run:` shell block in action.yml, enabling script injection. An attacker-controlled caller can supply values containing shell metacharacters (`;`, `|`, `$(...)`, etc.) that will be executed by the shell before any quoting takes effect. Affected expressions:
- `${{ inputs.fga_server_url }}` used in `if [[ -z "${{ inputs.fga_server_url }}" ]]`, `echo`, and `--api-url`
- `${{ inputs.fga_api_token }}` assigned directly: `fga_token="${{ inputs.fga_api_token }}"`
- `${{ inputs.fga_server_store_id }}` used in `--store-id "${{ inputs.fga_server_store_id }}"`
- `${{ inputs.test_path }}` used unquoted in `find ${{ inputs.test_path }}`
- `${{ inputs.test_files_pattern }}` used in `-name "${{ inputs.test_files_pattern }}"`
Fix: move all inputs into `env:` variables and reference them as quoted shell variables (e.g. `"$FGA_SERVER_URL"`).

Locations:

- `action.yml:44`
- `action.yml:47`
- `action.yml:48`
- `action.yml:51`
- `action.yml:53`
- `action.yml:54`
- `action.yml:57`
- `action.yml:58`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or the upstream repositories are compromised:
- `jaxxstorm/action-install-gh-release@v1.12.0` (line 33)
- `chrisdickinson/setup-yq@v1.0.1` (line 36)
Fix: pin each to a full SHA, e.g. `jaxxstorm/action-install-gh-release@<40-char-sha> # v1.12.0`.

Locations:

- `action.yml:33`
- `action.yml:36`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:52`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_api_token }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:53`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:58`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_store_id }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:59`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_path }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:64`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_files_pattern }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:64`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_path }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:67`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_files_pattern }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:67`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed all findings in action.yml:
1. Pinned `jaxxstorm/action-install-gh-release@v1.12.0` to SHA `25d5e2dd555cd74f1fab9ac1e6ea117acde2c0c4` with tag comment.
2. Pinned `chrisdickinson/setup-yq@v1.0.1` to SHA `3d931309f27270ebbafd53f2daee773a82ea1822` with tag comment.
3. Moved all `${{ inputs.* }}` expressions (fga_server_url, fga_api_token, fga_server_store_id, test_path, test_files_pattern) into an `env:` block on the 'Run OpenFGA CLI' step, and replaced all inline `${{ inputs.* }}` references in the `run:` shell script with the corresponding environment variable names (`$FGA_SERVER_URL`, `$FGA_API_TOKEN`, `$FGA_SERVER_STORE_ID`, `$TEST_PATH`, `$TEST_FILES_PATTERN`), properly double-quoted to prevent shell injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in action.yml:
1. Quoted `${test_file}` and `${test_file_without_model}` in the `yq` command (line 61).
2. Removed the unquoted `${fga_server_opts}` expansion (line 63) — the variable was never set so it was always empty; removing it eliminates the risk without changing behavior.
3. Quoted the inner `${fga_token}` inside the conditional expansion: changed `${fga_token:+--api-token ${fga_token}}` to `${fga_token:+--api-token "${fga_token}"}` (line 66).
Also fixed a pre-existing bug: `test_file_without_model=mktemp` was corrected to `test_file_without_model=$(mktemp)` so the variable holds an actual temp file path rather than the literal string 'mktemp'.

