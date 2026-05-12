# Hardening Report: subosito--flutter-action/v2.19.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **subosito--flutter-action/v2.19.0** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Set action inputs' run: step in action.yaml directly interpolates multiple inputs.* expressions into the shell command string without first assigning them to environment variables. Most critically, `${{ inputs.channel }}` is interpolated completely unquoted (no surrounding quotes), enabling arbitrary shell command injection. The other inputs (`inputs.flutter-version`, `inputs.flutter-version-file`, `inputs.architecture`, `inputs.cache-key`, `inputs.cache-path`, `inputs.pub-cache-key`, `inputs.pub-cache-path`, `inputs.git-source`) are single-quoted but still directly interpolated. All should be passed via `env:` variables and referenced as `$VAR` in the shell.

Locations:

- `action.yaml:97`

### github-env-injection (severity: high)

In setup.sh, values derived from action inputs (passed as shell arguments from `inputs.*` expressions in action.yaml) are written to $GITHUB_OUTPUT, $GITHUB_ENV, and $GITHUB_PATH without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically: CHANNEL, VERSION, ARCHITECTURE, CACHE-KEY, CACHE-PATH, PUB-CACHE-KEY, and PUB-CACHE-PATH are written to $GITHUB_OUTPUT; FLUTTER_ROOT and PUB_CACHE are written to $GITHUB_ENV; and Flutter bin paths are written to $GITHUB_PATH — all without newline sanitization. An attacker-controlled input containing newlines could inject arbitrary environment variables or path entries.

Locations:

- `setup.sh:230`
- `setup.sh:243`
- `setup.sh:249`

### unpinned-uses (severity: high)

Two `uses:` references in action.yaml use the mutable tag `@v4` instead of a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the referenced action's tag is moved to a different (potentially malicious) commit. Failing references: `actions/cache@v4` (Cache Flutter step) and `actions/cache@v4` (Cache pub dependencies step).

Locations:

- `action.yaml:108`
- `action.yaml:114`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flutter-version }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:100`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.flutter-version-file }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:101`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.architecture }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:102`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-key }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:103`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cache-path }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:104`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pub-cache-key }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:105`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pub-cache-path }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:106`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.git-source }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:107`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.channel }}" appears directly in run: block of step "Set action inputs"; move to env: map

Locations:

- `action.yml:108`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all findings in action.yaml and setup.sh:
1. script-injection / static-inline-injection: Moved all `${{ inputs.* }}` expressions in the 'Set action inputs' step to an `env:` block (INPUT_FLUTTER_VERSION, INPUT_FLUTTER_VERSION_FILE, INPUT_ARCHITECTURE, INPUT_CACHE_KEY, INPUT_CACHE_PATH, INPUT_PUB_CACHE_KEY, INPUT_PUB_CACHE_PATH, INPUT_GIT_SOURCE, INPUT_CHANNEL). Also fixed the 'Run setup script' step to use env: variables for step outputs. All shell references now use quoted "$VAR" syntax.
2. github-env-injection: In setup.sh, sanitized all values written to $GITHUB_OUTPUT (CHANNEL, VERSION, ARCHITECTURE, CACHE-KEY, CACHE-PATH, PUB-CACHE-KEY, PUB-CACHE-PATH) and to $GITHUB_ENV (FLUTTER_ROOT, PUB_CACHE) and $GITHUB_PATH using `printf '%s' "$VAR" | tr -d '\n\r'` before writing.
3. unpinned-uses: Pinned both `actions/cache@v4` references to `actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4` using the resolved commit SHA.

