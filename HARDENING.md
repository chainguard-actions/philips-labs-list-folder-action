<!-- markdownlint-disable -->

# Hardening Report: philips-labs--list-folder-action/v2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **philips-labs--list-folder-action/v2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The expression `${{ inputs.path }}` is directly interpolated inside a `run:` shell command: `cd ${{ inputs.path }}`. This substitution happens before the shell parses the command, so a malicious caller can supply a path value containing shell metacharacters (e.g. `; malicious-command`) to achieve arbitrary command execution. The fix is to pass the input via an `env:` variable and reference it as a quoted shell variable: `env: INPUT_PATH: ${{ inputs.path }}` then `cd "$INPUT_PATH"`.

Locations:

- `action.yml:18`

### github-env-injection (severity: high)

The `run:` block writes `folders=$folders` to `$GITHUB_OUTPUT` (line 21) without the required sanitization step (`printf '%s' "$folders" | tr -d '\n\r'`). The `$folders` variable is derived from `tree` output executed in a directory controlled by the attacker-supplied `inputs.path`. A crafted directory structure could inject newlines into the output, allowing an attacker to smuggle additional key=value pairs into `$GITHUB_OUTPUT` and potentially override subsequent step outputs.

Locations:

- `action.yml:21`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed action.yml step 'set-folders': (1) Moved `${{ inputs.path }}` into an `env:` block as `INPUT_PATH` and replaced `cd ${{ inputs.path }}` with `cd "$INPUT_PATH"` to prevent shell metacharacter injection. (2) Added newline sanitization before writing to $GITHUB_OUTPUT: `safe=$(printf '%s' "$folders" | tr -d '\n\r')` and then `echo "folders=$safe" >> "$GITHUB_OUTPUT"` to prevent newline injection attacks via crafted directory structures.

