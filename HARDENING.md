# Hardening Report: philips-labs--list-folder-action/v3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **philips-labs--list-folder-action/v3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in the `set-folders` step directly interpolates `${{ inputs.path }}` into a shell command (`cd ${{ inputs.path }}`). An attacker who controls the `path` input can inject arbitrary shell commands. The value should be assigned to an environment variable (e.g., `env: INPUT_PATH: ${{ inputs.path }}`) and then referenced as `$INPUT_PATH` in the shell script.

Locations:

- `action.yml:17`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed both script injection findings in action.yml. Moved `${{ inputs.path }}` from the `run:` block into an `env:` block as `INPUT_PATH: ${{ inputs.path }}`, and updated the shell command from `cd ${{ inputs.path }}` to `cd "$INPUT_PATH"`. This prevents attacker-controlled input from being interpreted as shell commands.

