# Hardening Report: subosito--flutter-action/v2.23.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **subosito--flutter-action/v2.23.0** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Set action inputs' step in action.yaml directly interpolates multiple `inputs.*` expressions inside the `run:` shell command string without first assigning them to environment variables. For example: `${{ inputs.flutter-version }}`, `${{ inputs.flutter-version-file }}`, `${{ inputs.architecture }}`, `${{ inputs.cache-key }}`, `${{ inputs.cache-path }}`, `${{ inputs.pub-cache-key }}`, `${{ inputs.pub-cache-path }}`, `${{ inputs.git-source }}`, and `${{ inputs.channel }}` are all interpolated directly as shell arguments. An attacker-controlled input value containing shell metacharacters (e.g. single-quote breaking out of the quoted argument, or unquoted `${{ inputs.channel }}`) can execute arbitrary shell commands. These values must be passed via `env:` and referenced as `$ENV_VAR` in the run block.

Locations:

- `action.yaml:113`

### github-env-injection (severity: high)

setup.sh writes values derived from attacker-controlled inputs (passed as shell arguments from `inputs.*` in action.yaml) to $GITHUB_OUTPUT, $GITHUB_ENV, and $GITHUB_PATH without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically: (1) CHANNEL, VERSION, ARCHITECTURE, CACHE-KEY, CACHE-PATH, PUB-CACHE-KEY, and PUB-CACHE-PATH are written to $GITHUB_OUTPUT (line ~185 in setup.sh); (2) FLUTTER_ROOT and PUB_CACHE are written to $GITHUB_ENV; (3) flutter/bin paths are written to $GITHUB_PATH. A newline injected into any of these values (e.g. via `inputs.channel` or `inputs.git-source`) can inject arbitrary environment variables or PATH entries.

Locations:

- `setup.sh:185`
- `setup.sh:200`
- `setup.sh:205`

### unpinned-uses (severity: high)

action.yaml references `actions/cache@v5` twice using a mutable version tag instead of a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to point to a different (potentially malicious) commit. Both the 'Cache Flutter' and 'Cache pub dependencies' steps are affected. These should be pinned to a full SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v4`.

Locations:

- `action.yaml:120`
- `action.yaml:126`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flutter-version }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:111`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flutter-version-file }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:112`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.architecture }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:113`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-key }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:114`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-path }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:115`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pub-cache-key }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:116`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pub-cache-path }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:117`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.git-source }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:118`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.channel }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:119`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all findings in action.yaml and setup.sh: (1) Moved all ${{ inputs.* }} expressions in the 'Set action inputs' step to an env: block and referenced them as $ENV_VAR in the run: block to prevent shell injection. (2) Sanitized all values written to $GITHUB_OUTPUT, $GITHUB_ENV, and $GITHUB_PATH in setup.sh using `printf '%s' "$var" | tr -d '\n\r'` to prevent newline injection. (3) Pinned both actions/cache@v5 references to the full SHA actions/cache@27d5ce7f107fe9357f9df03efb73ab90386fccae # v5.

