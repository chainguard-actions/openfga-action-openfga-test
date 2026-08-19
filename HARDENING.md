<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **openfga--action-openfga-test/v0.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in action.yml directly interpolates the expression `${{ inputs.store-file-path }}` into the shell command string: `fga model test --tests ${{ inputs.store-file-path }}`. This value is attacker-controlled (supplied by the calling workflow) and is substituted into the shell command before the shell parses it, enabling command injection. The fix is to pass the input via an `env:` variable and double-quote it in the shell: `env:\n  STORE_FILE_PATH: ${{ inputs.store-file-path }}\nrun: fga model test --tests "$STORE_FILE_PATH"`.

Locations:

- `action.yml:19`

### unpinned-uses (severity: high)

Two `uses:` references are pinned to mutable tags rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved:
- `action.yml` line 14: `jaxxstorm/action-install-gh-release@v1.10.0` (tag)
- `.github/workflows/test.yml` line 11: `actions/checkout@v4` (tag)
Each should be replaced with the full SHA of the intended commit, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:14`
- `.github/workflows/test.yml:11`

### permissions (severity: medium)

missing-permissions: `.github/workflows/test.yml` has no top-level `permissions:` key and the single job `test` also has no `permissions:` key. Without an explicit permissions block the workflow inherits the repository's default token permissions, which may be broader than necessary. Add a top-level `permissions: {}` (or the minimal required scopes) to restrict the GITHUB_TOKEN.

Locations:

- `.github/workflows/test.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.store-file-path }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, permissions, static-inline-injection

**Notes:**

Fixed all four findings: (1) Moved `${{ inputs.store-file-path }}` from the run: shell string into an env: block as STORE_FILE_PATH, referencing it as "$STORE_FILE_PATH" in the shell command to prevent script injection. (2) Pinned jaxxstorm/action-install-gh-release@v1.10.0 to full SHA c5ead9a448b4660cf1e7866ee22e4dc56538031a in action.yml. (3) Pinned actions/checkout@v4 to full SHA 34e114876b0b11c390a56381ad16ebd13914f8d5 in .github/workflows/test.yml. (4) Added top-level `permissions: {}` to .github/workflows/test.yml to restrict the GITHUB_TOKEN.

