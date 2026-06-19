<!-- markdownlint-disable -->

# Hardening Report: openfga--action-openfga-test/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **openfga--action-openfga-test/v0.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block at action.yml line 22 directly interpolates the user-controlled expression `${{ inputs.store-file-path }}` into a shell command string: `run: fga model test --tests ${{ inputs.store-file-path }}`. Because YAML template substitution happens before the shell sees the command, an attacker who controls the `store-file-path` input can inject arbitrary shell commands (e.g. by passing a value containing `;`, `&&`, backticks, or `$(...)` sequences). The fix is to pass the input via an `env:` variable and reference it as a double-quoted shell variable: `env: STORE_FILE_PATH: ${{ inputs.store-file-path }}` then `run: fga model test --tests "$STORE_FILE_PATH"`.

Locations:

- `action.yml:22`

### unpinned-uses (severity: high)

The composite action step `uses: jaxxstorm/action-install-gh-release@v1.10.0` (action.yml line 16) references a mutable version tag (`v1.10.0`) rather than an immutable 40-character commit SHA. If the tag is moved or the repository is compromised, a different (potentially malicious) version of the action could be silently substituted. Pin to a full SHA, e.g. `uses: jaxxstorm/action-install-gh-release@<40-char-sha> # v1.10.0`.

Locations:

- `action.yml:16`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.store-file-path }}" appears directly in run: block of step "Run OpenFGA CLI"; move to env: map

Locations:

- `action.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed action.yml: (1) Pinned jaxxstorm/action-install-gh-release from mutable tag v1.10.0 to full SHA c5ead9a448b4660cf1e7866ee22e4dc56538031a with the tag preserved as a comment. (2) Moved ${{ inputs.store-file-path }} out of the run: block into an env: variable STORE_FILE_PATH and referenced it as double-quoted "$STORE_FILE_PATH" in the shell command, eliminating the script injection vulnerability.

