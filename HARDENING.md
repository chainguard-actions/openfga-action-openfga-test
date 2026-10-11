<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openfga--action-openfga-test/v0.1.3** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` expressions are directly interpolated inside the `run:` shell script in the 'Run OpenFGA CLI' step. GitHub Actions performs YAML template substitution before the shell ever sees the string, so an attacker-controlled input value (e.g. containing shell metacharacters like `;`, `|`, `$(...)`, backticks) can break out of the intended command and execute arbitrary code. Offending lines include:
- `if [[ -z "${{ inputs.fga_server_url }}" ]]` — inputs.fga_server_url interpolated directly in a conditional
- `echo "Running FGA test file ${test_file} against OpenFGA server ${{ inputs.fga_server_url }}"` — inputs.fga_server_url in echo
- `fga_token="${{ inputs.fga_api_token }}"` — inputs.fga_api_token assigned to shell variable via direct interpolation
- `--api-url "${{ inputs.fga_server_url }}"` — inputs.fga_server_url as CLI argument
- `--store-id "${{ inputs.fga_server_store_id }}"` — inputs.fga_server_store_id as CLI argument
- `find ${{ inputs.test_path }} -name "${{ inputs.test_files_pattern }}" -print0` — both inputs.test_path and inputs.test_files_pattern interpolated into find command (test_path is also unquoted)
- `echo "No FGA test file found for path '${{ inputs.test_path }}' and pattern '${{ inputs.test_files_pattern }}'"` — both inputs in echo

Fix: Move all inputs into `env:` variables and reference them as quoted shell variables (e.g. `"$FGA_SERVER_URL"`) inside the `run:` block. Never use `${{ }}` expressions directly inside `run:` scripts.

Locations:

- `action.yml:60`
- `action.yml:63`
- `action.yml:64`
- `action.yml:71`
- `action.yml:72`
- `action.yml:78`
- `action.yml:81`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:63`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:67`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_api_token }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:68`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_url }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:77`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.fga_server_store_id }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:78`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_path }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:85`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_files_pattern }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:85`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_path }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:88`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.test_files_pattern }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved all ${{ inputs.* }} expressions from the run: shell script block into the step's env: block in the 'Run OpenFGA CLI' step. Added FGA_SERVER_URL, FGA_SERVER_STORE_ID, FGA_API_TOKEN, TEST_PATH, and TEST_FILES_PATTERN to the env: map. Replaced all inline ${{ inputs.fga_server_url }}, ${{ inputs.fga_server_store_id }}, ${{ inputs.fga_api_token }}, ${{ inputs.test_path }}, and ${{ inputs.test_files_pattern }} references in the run: block with their corresponding shell variable references ($FGA_SERVER_URL, $FGA_SERVER_STORE_ID, $FGA_API_TOKEN, $TEST_PATH, $TEST_FILES_PATTERN). Also added quotes around $TEST_PATH in the find command for correctness. The ${{ inputs.fga_cli_allow_external_files }} was already in the env: block and ${{ inputs.fga_cli_version }} is in a with: block — both are safe and unchanged.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed four script injection issues in action.yml's 'Run OpenFGA CLI' step:
1. `${allow_external_files}` (line 71, no-server branch): Changed to `${allow_external_files:+"$allow_external_files"}` to prevent word splitting/glob expansion.
2. `${allow_external_files}` (line 81, server branch): Same fix applied.
3. `${fga_server_opts}` (line 82): Removed entirely — this variable was never initialized in the script and could be injected from the calling workflow's environment, making it a direct command injection vector.
4. `${fga_token:+--api-token ${fga_token}}` (line 85): Changed to `${fga_token:+--api-token "$fga_token"}` to properly quote the inner expansion of fga_token.

