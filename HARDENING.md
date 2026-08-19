<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openfga--action-openfga-test/v0.1.1** was hardened automatically. 12 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates multiple `${{ inputs.* }}` expressions into shell commands (rule a). This allows an attacker who controls the calling workflow's inputs to inject arbitrary shell commands. Offending lines include:
- `if [[ -z "${{ inputs.fga_server_url }}" ]];`
- `echo "Running FGA test file ${test_file} against OpenFGA server ${{ inputs.fga_server_url }}"`
- `fga_token="${{ inputs.fga_api_token }}"`
- `--api-url "${{ inputs.fga_server_url }}"`
- `--store-id "${{ inputs.fga_server_store_id }}"`
- `find ${{ inputs.test_path }} -name "${{ inputs.test_files_pattern }}" -print0`
- `echo "No FGA test file found for path '${{ inputs.test_path }}' and pattern '${{ inputs.test_files_pattern }}'"` 
All these values should be passed via `env:` variables and then referenced as properly double-quoted shell variables (e.g. `"$INPUT_VAR"`) instead of being interpolated directly.

Locations:

- `action.yml:40`
- `action.yml:44`
- `action.yml:45`
- `action.yml:50`
- `action.yml:51`
- `action.yml:56`
- `action.yml:57`
- `action.yml:61`
- `action.yml:62`

### unpinned-uses (severity: high)

Several `uses:` references are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

In action.yml:
- `jaxxstorm/action-install-gh-release@v1.12.0` (tag, not SHA)
- `chrisdickinson/setup-yq@v1.0.1` (tag, not SHA)

In .github/workflows/test.yml:
- `actions/checkout@v4` (tag, not SHA)
- `jaxxstorm/action-install-gh-release@v1.11.0` (tag, not SHA — appears twice)
- `dcarbone/install-jq-action@v2` (tag, not SHA)

Locations:

- `action.yml:32`
- `action.yml:35`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:37`
- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:50`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test.yml` has no top-level `permissions:` key and neither of its jobs (`test`, `test_conditions_support`) defines a job-level `permissions:` block. This means the workflow runs with the default GitHub token permissions, which may be broader than necessary (e.g. write access to contents on some repository configurations). A minimal explicit `permissions:` block (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/test.yml:1`

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

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings in action.yml and .github/workflows/test.yml:

1. script-injection/static-inline-injection (action.yml): Moved all ${{ inputs.* }} expressions out of the run: block into an env: block with named variables (FGA_SERVER_URL, FGA_SERVER_STORE_ID, FGA_API_TOKEN, TEST_PATH, TEST_FILES_PATTERN). Shell script now uses properly double-quoted env vars.

2. unpinned-uses (action.yml): Pinned jaxxstorm/action-install-gh-release@v1.12.0 to SHA 25d5e2dd555cd74f1fab9ac1e6ea117acde2c0c4 and chrisdickinson/setup-yq@v1.0.1 to SHA 3d931309f27270ebbafd53f2daee773a82ea1822.

3. unpinned-uses (.github/workflows/test.yml): Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262, jaxxstorm/action-install-gh-release@v1.11.0 to SHA 71d17cb091aa850acb2a1a4cf87258d183eb941b (both occurrences), and dcarbone/install-jq-action@v2 to SHA 8867ddb4788346d7c22b72ea2e2ffe4d514c7bcb.

4. missing-permissions (.github/workflows/test.yml): Added top-level permissions: contents: read block.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:

1. `.github/workflows/test.yml` (lines 94-101): Moved `${{ matrix.test.conditions_supported }}`, `${{ steps.tests.outcome }}`, and `${{ matrix.test.openfga_tag }}` out of the `run:` shell block into a step-level `env:` block as `CONDITIONS_SUPPORTED`, `TESTS_OUTCOME`, and `OPENFGA_TAG`. The shell script now references these as plain environment variables, eliminating the injection vector.

2. `action.yml` (lines 61, 66): (a) Fixed missing command substitution: `test_file_without_model=mktemp` → `test_file_without_model=$(mktemp)`. (b) Added double-quotes around `${test_file}` and `${test_file_without_model}` to prevent word-splitting and glob expansion. (c) Quoted the inner token expansion: `${fga_token:+--api-token ${fga_token}}` → `${fga_token:+--api-token "$fga_token"}` to prevent word-splitting on attacker-controlled API token values.

