<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **openfga--action-openfga-test/v0.1.2** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Run OpenFGA CLI' step in action.yml directly interpolates multiple ${{ inputs.* }} expressions inside the run: shell script (sub-rule a). This allows an attacker who controls the calling workflow's inputs to inject arbitrary shell commands. Affected expressions: ${{ inputs.fga_server_url }} (lines 48, 51, 55), ${{ inputs.fga_api_token }} (line 52), ${{ inputs.fga_server_store_id }} (line 56), ${{ inputs.test_path }} (lines 59, 62), ${{ inputs.test_files_pattern }} (lines 59, 62). All of these should be moved to env: variables and then referenced as properly double-quoted shell variables in the run: block.

Locations:

- `action.yml:48`
- `action.yml:51`
- `action.yml:52`
- `action.yml:55`
- `action.yml:56`
- `action.yml:59`
- `action.yml:62`

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

Fixed all script injection findings in action.yml by moving all ${{ inputs.* }} expressions from the run: block of the 'Run OpenFGA CLI' step into an env: block. The five affected inputs (fga_server_url, fga_api_token, fga_server_store_id, test_path, test_files_pattern) are now set as environment variables (FGA_SERVER_URL, FGA_API_TOKEN, FGA_SERVER_STORE_ID, TEST_PATH, TEST_FILES_PATTERN) and referenced with proper double-quoting in the shell script. This eliminates all shell injection vectors identified in the findings.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection issues in the 'Run OpenFGA CLI' step of action.yml:
1. Quoted ${test_file} and ${test_file_without_model} in the yq command (line 61) to prevent word-splitting on paths derived from inputs.test_path.
2. Replaced unquoted ${fga_server_opts} (line 63) with a bash array (server_opts_args) populated via 'read -ra' and expanded as "${server_opts_args[@]}" to keep arguments properly separated.
3. Replaced ${fga_token:+--api-token ${fga_token}} (line 66) with a bash array (token_args) that properly double-quotes the token value, expanded as "${token_args[@]}".
Also fixed a pre-existing bug: test_file_without_model=mktemp was assigning the literal string 'mktemp' instead of running the command; corrected to test_file_without_model=$(mktemp).

