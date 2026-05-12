# Hardening Report: subosito--flutter-action/v2.22.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **subosito--flutter-action/v2.22.0** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Set action inputs' run: block in action.yaml directly interpolates multiple inputs.* expressions into the shell command without first assigning them to environment variables. Most critically, `${{ inputs.channel }}` is completely unquoted at the end of the command line (line ~109), enabling arbitrary shell command injection. All other inputs (flutter-version, flutter-version-file, architecture, cache-key, cache-path, pub-cache-key, pub-cache-path, git-source) are also interpolated directly in the run: block, only protected by single quotes which can be escaped by an attacker supplying a value containing a single quote character.

Locations:

- `action.yaml:98`

### github-env-injection (severity: high)

setup.sh writes values derived from user-controlled inputs (passed as shell arguments from action.yaml's inputs.*) to $GITHUB_OUTPUT, $GITHUB_ENV, and $GITHUB_PATH without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Specifically: (1) CHANNEL, VERSION, ARCHITECTURE, CACHE-KEY, CACHE-PATH, PUB-CACHE-KEY, PUB-CACHE-PATH are written to $GITHUB_OUTPUT without sanitization; (2) FLUTTER_ROOT and PUB_CACHE are written to $GITHUB_ENV without sanitization; (3) CACHE_PATH/bin paths and PUB_CACHE/bin are written to $GITHUB_PATH without sanitization. An attacker can inject newline characters via inputs to add arbitrary entries to these special environment files.

Locations:

- `setup.sh:236`
- `setup.sh:253`
- `setup.sh:258`

### unpinned-uses (severity: high)

Two steps in action.yaml reference `actions/cache@v5` using a mutable version tag instead of a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the tag is moved to point to a different (potentially malicious) commit. Failing references: `actions/cache@v5` (Cache Flutter step) and `actions/cache@v5` (Cache pub dependencies step).

Locations:

- `action.yaml:112`
- `action.yaml:119`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flutter-version }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:107`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flutter-version-file }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:108`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.architecture }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:109`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-key }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:110`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-path }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:111`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pub-cache-key }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:112`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pub-cache-path }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:113`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.git-source }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:114`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.channel }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:115`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed action.yaml: (1) Moved all ${{ inputs.* }} expressions in 'Set action inputs' step to an env: block (INPUT_FLUTTER_VERSION, INPUT_FLUTTER_VERSION_FILE, INPUT_ARCHITECTURE, INPUT_CACHE_KEY, INPUT_CACHE_PATH, INPUT_PUB_CACHE_KEY, INPUT_PUB_CACHE_PATH, INPUT_GIT_SOURCE, INPUT_CHANNEL) and referenced them as plain env vars in the shell script with proper double-quoting. (2) Also fixed 'Run setup script' step to use env: block for step output expressions. (3) Pinned both actions/cache@v5 references to actions/cache@27d5ce7f107fe9357f9df03efb73ab90386fccae # v5. Fixed setup.sh: Sanitized all values written to $GITHUB_OUTPUT (CHANNEL, VERSION, ARCHITECTURE, CACHE-KEY, CACHE-PATH, PUB-CACHE-KEY, PUB-CACHE-PATH), $GITHUB_ENV (FLUTTER_ROOT, PUB_CACHE), and $GITHUB_PATH (CACHE_PATH/bin paths and PUB_CACHE/bin) using printf '%s' | tr -d '\n\r' to prevent newline injection attacks.

