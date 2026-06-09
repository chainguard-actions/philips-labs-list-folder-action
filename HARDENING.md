# Hardening Report: philips-labs--list-folder-action/v3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **philips-labs--list-folder-action/v3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in the `set-folders` step directly interpolates `${{ inputs.path }}` into a shell command (`cd ${{ inputs.path }}`). An attacker who controls the `path` input can inject arbitrary shell commands (e.g., by supplying a value containing shell metacharacters or command substitution). The input should be assigned to an environment variable (via `env:`) and referenced as `$ENV_VAR` in the shell instead.

Locations:

- `action.yml:19`

### github-env-injection (severity: high)

The `set-folders` step writes `echo "folders=$folders" >> $GITHUB_OUTPUT` where `$folders` is derived from a directory listing rooted at the attacker-controlled `${{ inputs.path }}` input. No sanitization (`printf '%s' ... | tr -d '\n\r'`) is applied before the write. A crafted `path` value could inject newlines into `$GITHUB_OUTPUT`, allowing an attacker to overwrite or inject additional output variables.

Locations:

- `action.yml:22`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed action.yml `set-folders` step: (1) Moved `${{ inputs.path }}` from the `run:` block into an `env:` block as `INPUT_PATH` and referenced it as `"$INPUT_PATH"` in the shell — eliminating shell injection via the path input. (2) Added newline sanitization (`printf '%s' "$folders" | tr -d '\n\r'`) before writing the folders value to `$GITHUB_OUTPUT` — preventing GITHUB_OUTPUT injection via crafted directory names or path values.

