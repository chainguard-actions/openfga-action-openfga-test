<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openfga--action-openfga-test/v0.1.2** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Run OpenFGA CLI' run: block directly interpolates multiple ${{ inputs.* }} expressions inside shell commands (rule a). This allows an attacker who controls the calling workflow to inject arbitrary shell commands. Offending lines:
- Line 48: `if [[ -z "${{ inputs.fga_server_url }}" ]]` — inputs.fga_server_url interpolated directly into shell test
- Line 51: `echo "...against OpenFGA server ${{ inputs.fga_server_url }}"` — inputs.fga_server_url in echo
- Line 52: `fga_token="${{ inputs.fga_api_token }}"` — inputs.fga_api_token assigned directly to shell variable
- Line 55: `--api-url "${{ inputs.fga_server_url }}"` — inputs.fga_server_url in CLI argument
- Line 56: `--store-id "${{ inputs.fga_server_store_id }}"` — inputs.fga_server_store_id in CLI argument
- Line 59: `find ${{ inputs.test_path }} -name "${{ inputs.test_files_pattern }}"` — both inputs interpolated directly into find command (unquoted test_path is also an unquoted expansion)
- Lines 62-63: `'${{ inputs.test_path }}'` and `'${{ inputs.test_files_pattern }}'` in echo
All of these should be moved to env: variables and the shell expansions must be double-quoted.

Locations:

- `action.yml:48`
- `action.yml:51`
- `action.yml:52`
- `action.yml:55`
- `action.yml:56`
- `action.yml:59`
- `action.yml:62`
- `action.yml:63`

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

Moved all ${{ inputs.* }} expressions from the 'Run OpenFGA CLI' run: block into a step-level env: block (INPUT_FGA_SERVER_URL, INPUT_FGA_API_TOKEN, INPUT_FGA_SERVER_STORE_ID, INPUT_TEST_PATH, INPUT_TEST_FILES_PATTERN). Updated all shell references to use the corresponding environment variable names, double-quoted where appropriate. This eliminates all script-injection and static-inline-injection findings.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions in the 'Run OpenFGA CLI' step of action.yml:
1. Line 65: Quoted `${test_file}` and `${test_file_without_model}` in the yq command: `yq 'del(.model_file, .model)' "${test_file}" > "${test_file_without_model}"` — prevents word-splitting/glob expansion on filenames with shell metacharacters.
2. Line 70: Quoted the inner `${fga_token}` in the conditional expansion: `${fga_token:+--api-token "$fga_token"}` — prevents word-splitting on the API token value.

