# Hardening Report: philips-labs--list-folder-action/v2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `c40cfe5fa14e08549b1b988e7e5a26da4816abf0`

**Test Policy SHA:** `f2e7d85641cde4267138117189b8eba7ba2bfbde`

Action **philips-labs--list-folder-action/v2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In action.yml, the composite action step directly interpolates the user-supplied input `${{ inputs.path }}` into a `run:` shell command: `cd ${{ inputs.path }}`. This allows an attacker who controls the `path` input to inject arbitrary shell commands via shell metacharacters or command substitution. The value should instead be assigned to an environment variable (e.g., `INPUT_PATH: ${{ inputs.path }}`) and then referenced as `$INPUT_PATH` in the shell script.

Locations:

- `action.yml:19`

### github-env-injection (severity: high)

In action.yml, the attacker-controlled input `${{ inputs.path }}` is interpolated directly into the `run:` block (line 19: `cd ${{ inputs.path }}`), and the resulting computed value (`folders`) is subsequently written to `$GITHUB_OUTPUT` (line 22: `echo "folders=$folders" >> $GITHUB_OUTPUT`) without any newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled `path` value containing newline characters could inject additional key=value pairs into `$GITHUB_OUTPUT`, poisoning the outputs consumed by downstream steps. The value must be sanitized before being written to the special environment file.

Locations:

- `action.yml:19`
- `action.yml:22`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all three high-severity findings in action.yml:
1. Moved `${{ inputs.path }}` from the `run:` block into the step's `env:` block as `INPUT_PATH: ${{ inputs.path }}`, then referenced it as `"$INPUT_PATH"` in the shell script (fixes script-injection and static-inline-injection).
2. Added newline sanitization for the `folders` output using `printf '%s' "$folders" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT` (fixes github-env-injection).
3. Also quoted `$GITHUB_OUTPUT` and `$folders`/`$INPUT_PATH` references for robustness.

