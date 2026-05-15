# Hardening Report: subosito--flutter-action/v2.21.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **subosito--flutter-action/v2.21.0** was hardened automatically. 11 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Set action inputs' run: block in action.yaml directly interpolates multiple attacker-controlled `inputs.*` expressions into shell command arguments without first assigning them to environment variables. Specifically: `${{ inputs.flutter-version }}`, `${{ inputs.flutter-version-file }}`, `${{ inputs.architecture }}`, `${{ inputs.cache-key }}`, `${{ inputs.cache-path }}`, `${{ inputs.pub-cache-key }}`, `${{ inputs.pub-cache-path }}`, `${{ inputs.git-source }}`, and `${{ inputs.channel }}` (the last one is completely unquoted) are all interpolated directly into the shell command string. An attacker supplying a malicious value (e.g. containing shell metacharacters or newlines) can break out of the argument context and execute arbitrary commands.

Locations:

- `action.yaml:100`

### unpinned-uses (severity: high)

Two `uses:` references in action.yaml use mutable version tags instead of pinned full-length SHA commit hashes, making the action vulnerable to supply-chain attacks if the referenced tag is moved or compromised: `actions/cache@v4` (used twice, for 'Cache Flutter' and 'Cache pub dependencies' steps).

Locations:

- `action.yaml:107`
- `action.yaml:113`

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

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed in action.yaml: (1) Moved all 9 ${{ inputs.* }} expressions from the 'Set action inputs' run: block into an env: block (INPUT_FLUTTER_VERSION, INPUT_FLUTTER_VERSION_FILE, INPUT_ARCHITECTURE, INPUT_CACHE_KEY, INPUT_CACHE_PATH, INPUT_PUB_CACHE_KEY, INPUT_PUB_CACHE_PATH, INPUT_GIT_SOURCE, INPUT_CHANNEL), referencing them as double-quoted shell variables to prevent script injection. (2) Pinned both actions/cache@v4 references to the full commit SHA actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4. Note: findings referenced both action.yaml and action.yml but only action.yaml exists — both sets of findings apply to the same file.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed setup.sh to sanitize all user-controlled values before writing to $GITHUB_OUTPUT, $GITHUB_ENV, and $GITHUB_PATH. In the PRINT_ONLY block, all 7 output values (CHANNEL, VERSION, ARCHITECTURE, CACHE-KEY, CACHE-PATH, PUB-CACHE-KEY, PUB-CACHE-PATH) are now sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before being echoed to $GITHUB_OUTPUT. Similarly, FLUTTER_ROOT and PUB_CACHE values written to $GITHUB_ENV, and the three path entries written to $GITHUB_PATH, now use sanitized safe_cache_path and safe_pub_cache variables. The TEST_MODE branch (stdout only, not written to special files) was left as-is since it doesn't affect the runner environment.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the 'Run setup script' step in action.yaml by moving all ${{ steps.flutter-action.outputs.* }} expressions (VERSION, ARCHITECTURE, CACHE-PATH, PUB-CACHE-PATH, CHANNEL) out of the run: shell command string and into an env: block. The shell script now references these as properly double-quoted environment variables ($FLUTTER_VERSION, $FLUTTER_ARCHITECTURE, $FLUTTER_CACHE_PATH, $FLUTTER_PUB_CACHE_PATH, $FLUTTER_CHANNEL), preventing shell command injection — especially critical for CHANNEL which was previously completely unquoted.

