<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openfga--action-openfga-test/v0.1.2** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Run OpenFGA CLI' step in action.yml directly interpolates multiple `inputs.*` expressions inside a `run:` shell script (sub-rule a). Before the shell ever executes, GitHub Actions substitutes these values verbatim into the script text, allowing an attacker who controls the calling workflow's inputs to inject arbitrary shell commands.

Offending lines:
- `if [[ -z "${{ inputs.fga_server_url }}" ]];` — inputs.fga_server_url interpolated into a conditional
- `echo "...against OpenFGA server ${{ inputs.fga_server_url }}"` — inputs.fga_server_url in echo
- `fga_token="${{ inputs.fga_api_token }}"` — inputs.fga_api_token assigned to shell variable
- `--api-url "${{ inputs.fga_server_url }}"` — inputs.fga_server_url as CLI argument
- `--store-id "${{ inputs.fga_server_store_id }}"` — inputs.fga_server_store_id as CLI argument
- `find ${{ inputs.test_path }} -name "${{ inputs.test_files_pattern }}"` — both inputs.test_path and inputs.test_files_pattern in find command
- `echo "No FGA test file found for path '${{ inputs.test_path }}' and pattern '${{ inputs.test_files_pattern }}'"` — both inputs in echo

Fix: Move all inputs into `env:` variables and reference them as quoted shell variables (e.g., `"$FGA_SERVER_URL"`) — never interpolate `${{ ... }}` directly inside a `run:` block.

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:52`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:56`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_api_token }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:57`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:62`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_store_id }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:63`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_path }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:68`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_files_pattern }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:68`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_path }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:71`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_files_pattern }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:71`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all 10 script injection findings in action.yml by moving all ${{ inputs.* }} expressions from the run: shell block into an env: block on the 'Run OpenFGA CLI' step. The five inputs (fga_server_url, fga_api_token, fga_server_store_id, test_path, test_files_pattern) are now exposed as environment variables (FGA_SERVER_URL, FGA_API_TOKEN, FGA_SERVER_STORE_ID, TEST_PATH, TEST_FILES_PATTERN) and referenced as quoted shell variables throughout the script. The remaining ${{ inputs.* }} references in the file are in env: and with: blocks, which are not shell injection vectors.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in the 'Run OpenFGA CLI' step of action.yml:
1. Quoted `${test_file}` and `${test_file_without_model}` in the yq command: `yq 'del(.model_file, .model)' "${test_file}" > "${test_file_without_model}"`.
2. Replaced the unquoted `${fga_server_opts}` expansion and the unquoted `${fga_token}` inside the parameter expansion with a bash array approach: `fga_args=(--api-url "$FGA_SERVER_URL" --store-id "$FGA_SERVER_STORE_ID")` and `[ -n "${fga_token}" ] && fga_args+=(--api-token "${fga_token}")`; then `fga model test "${fga_args[@]}" --tests "${test_file_without_model}"`. This keeps each argument token separate while ensuring all values are properly double-quoted.
3. Also fixed a pre-existing bug: `test_file_without_model=mktemp` → `test_file_without_model=$(mktemp)` (missing command substitution).

