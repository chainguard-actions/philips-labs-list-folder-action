<!-- markdownlint-disable -->

# Hardening Report: philips-labs--list-folder-action/v2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **philips-labs--list-folder-action/v2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The `inputs.path` value is directly interpolated via `${{ inputs.path }}` inside a `run:` shell command (`cd ${{ inputs.path }}`). Before the shell ever sees the command, GitHub Actions substitutes the raw input string into it, allowing an attacker to inject arbitrary shell commands by supplying a malicious path value (e.g., `.; malicious-command`). The fix is to pass the input through an `env:` variable and double-quote the shell expansion: set `env: INPUT_PATH: ${{ inputs.path }}` and use `cd "$INPUT_PATH"` in the script.

Locations:

- `action.yml:16`

### github-env-injection (severity: high)

The `folders` variable is derived from `tree` output of the attacker-controlled `inputs.path` directory, then written unsanitized to `$GITHUB_OUTPUT` via `echo "folders=$folders" >> $GITHUB_OUTPUT`. A crafted directory name containing newline characters could inject additional key=value pairs into the GITHUB_OUTPUT file, poisoning downstream step outputs. The fix is to sanitize before writing: `safe=$(printf '%s' "$folders" | tr -d '\n\r')` and then `echo "folders=$safe" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:19`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed hardened/action/action.yml: (1) Moved `${{ inputs.path }}` from the run: block into an env: variable `INPUT_PATH` and used `cd "$INPUT_PATH"` to prevent shell injection. (2) Added `safe=$(printf '%s' "$folders" | tr -d '\n\r')` and wrote `$safe` to `$GITHUB_OUTPUT` instead of the raw `$folders` value to prevent newline-based GITHUB_OUTPUT injection. (3) Quoted `$GITHUB_OUTPUT` and `$folders` in echo for robustness.

