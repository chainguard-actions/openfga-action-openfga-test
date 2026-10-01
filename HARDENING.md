<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openfga--action-openfga-test/v0.1.2** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Run OpenFGA CLI' run: block in action.yml directly interpolates multiple ${{ inputs.* }} expressions into shell command strings (rule a). This means GitHub Actions performs template substitution before the shell ever sees the value, allowing an attacker who controls the inputs to inject arbitrary shell metacharacters and commands. Affected lines include:
- Line 51: `if [[ -z "${{ inputs.fga_server_url }}" ]];`
- Line 54: `echo "...against OpenFGA server ${{ inputs.fga_server_url }}"`
- Line 55: `fga_token="${{ inputs.fga_api_token }}"`
- Line 59: `--api-url "${{ inputs.fga_server_url }}"`
- Line 60: `--store-id "${{ inputs.fga_server_store_id }}"`
- Line 63: `find ${{ inputs.test_path }} -name "${{ inputs.test_files_pattern }}" -print0`
- Line 66: `echo "...path '${{ inputs.test_path }}' and pattern '${{ inputs.test_files_pattern }}'"`
All these inputs should be moved to env: variables and referenced as quoted shell variables (e.g. "$INPUT_VAR") instead.

Locations:

- `action.yml:51`
- `action.yml:54`
- `action.yml:55`
- `action.yml:59`
- `action.yml:60`
- `action.yml:63`
- `action.yml:66`

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

Moved all ${{ inputs.* }} expressions from the 'Run OpenFGA CLI' run: block into an env: block on the same step. The five affected inputs (fga_server_url, fga_api_token, fga_server_store_id, test_path, test_files_pattern) are now exposed as environment variables (INPUT_FGA_SERVER_URL, INPUT_FGA_API_TOKEN, INPUT_FGA_SERVER_STORE_ID, INPUT_TEST_PATH, INPUT_TEST_FILES_PATTERN) and referenced as quoted shell variables throughout the script. This prevents GitHub Actions template substitution from injecting attacker-controlled values directly into the shell command string.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted variable expansions in the 'Run OpenFGA CLI' run block in action.yml:
1. Line 65: Added double-quotes around `${test_file}` and `${test_file_without_model}` in the yq command to prevent word splitting and glob expansion on attacker-controlled file paths.
2. Line 70: Added double-quotes around `${fga_token}` inside the conditional expansion `${fga_token:+--api-token "$fga_token"}` to prevent shell metacharacter injection from the attacker-controlled fga_api_token input.

