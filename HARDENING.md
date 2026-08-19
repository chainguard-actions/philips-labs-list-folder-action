<!-- markdownlint-disable -->

# Hardening Report: philips-labs--list-folder-action/v3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **philips-labs--list-folder-action/v3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) and (b): The `run:` block on line 19 directly interpolates the user-controlled input `${{ inputs.path }}` into a shell command: `cd ${{ inputs.path }}`. This allows an attacker to inject arbitrary shell commands by supplying a crafted `path` value (e.g., `.; malicious_command`). The value is also unquoted, compounding the risk. The expression must be moved to an `env:` variable and double-quoted: `env: INPUT_PATH: ${{ inputs.path }}` then `cd "$INPUT_PATH"`.

Locations:

- `action.yml:19`

### github-env-injection (severity: high)

The `run:` block writes `$folders` to `$GITHUB_OUTPUT` on line 22 (`echo "folders=$folders" >> $GITHUB_OUTPUT`) without sanitization. The `$folders` variable is derived from processing a directory path supplied via the untrusted `inputs.path` input. No `printf '%s' "$folders" | tr -d '\n\r'` sanitization step is applied before the write, allowing newline injection that could poison subsequent steps reading from `$GITHUB_OUTPUT`. Additionally, `$folders` is unquoted on line 21 (`echo $folders`), which is also unsafe.

Locations:

- `action.yml:22`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed action.yml step 'set-folders':
1. Moved `${{ inputs.path }}` from the run block into an `env:` variable `INPUT_PATH` and double-quoted it in the shell (`cd "$INPUT_PATH"`), eliminating shell injection risk.
2. Quoted `$folders` in the diagnostic `echo` call.
3. Added `safe=$(printf '%s' "$folders" | tr -d '\n\r')` sanitization before writing to `$GITHUB_OUTPUT` to prevent newline injection, and quoted `$GITHUB_OUTPUT` in the redirection.

