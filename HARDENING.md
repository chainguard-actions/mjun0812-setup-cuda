<!-- markdownlint-disable -->

# Hardening Report: mjun0812--setup-cuda/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **mjun0812--setup-cuda/v1.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the upstream action tag is moved or compromised. Unpinned references found:
- `actions/checkout@v5`
- `pnpm/action-setup@v4`
- `actions/setup-node@v6`
All should be replaced with their full SHA digests (e.g. `actions/checkout@<40-char-sha> # v5`).

Locations:

- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:22`
- `.github/workflows/_test.yml:38`
- `.github/workflows/_test.yml:41`
- `.github/workflows/_test.yml:44`
- `.github/workflows/_test-container.yml:68`
- `.github/workflows/_test-container.yml:71`
- `.github/workflows/_test-container.yml:74`
- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:26`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands (rule a). This allows the expression value to be parsed by the shell before quoting can protect it, enabling command injection.

**release.yml — 'Create Release' step (line 43):** `gh release create ${{github.ref_name}} --generate-notes` — `github.ref_name` is interpolated directly into the shell command without quoting.

**release.yml — 'Update Major Tag' step (line 47):** `GITHUB_REF_NAME=${{ github.ref_name }}` — `github.ref_name` is interpolated directly into a shell variable assignment.

**_test.yml — 'Verify CUDA Installation (Linux)' step (lines 63–64):** `echo "CUDA Version: ${{ steps.setup-cuda.outputs.version }}"` and `echo "CUDA Path: ${{ steps.setup-cuda.outputs.cuda-path }}"` — step outputs interpolated directly into shell.

**_test.yml — 'Verify CUDA Installation (Windows)' step (lines 82–83):** Same pattern in PowerShell `run:` block.

**_test.yml — 'Verify CUDA Installation' step (lines 100–101):** Same pattern in final bash verification step.

**_test-container.yml — 'Verify CUDA Installation (Linux)' step (lines 89–90):** Same step output interpolation pattern.

Fix: move values into `env:` variables and reference them as `"$VAR"` in the shell script.

Locations:

- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:47`
- `.github/workflows/_test.yml:63`
- `.github/workflows/_test.yml:64`
- `.github/workflows/_test.yml:82`
- `.github/workflows/_test.yml:83`
- `.github/workflows/_test.yml:100`
- `.github/workflows/_test.yml:101`
- `.github/workflows/_test-container.yml:89`
- `.github/workflows/_test-container.yml:90`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, workflows run with the default token permissions, which may be overly broad (e.g. `write` on `contents` for workflows triggered by `push` or `pull_request`). Affected files:
- `ci.yml`: no top-level permissions; `Format-Lint-TypeCheck`, `Test`, and `ci-check` jobs all lack job-level permissions.
- `_test.yml`: no top-level or job-level permissions on the `Test` job.
- `_test-container.yml`: no top-level or job-level permissions on the `TestContainer` job.
- `container-test.yml`: no top-level or job-level permissions on the `Container-Test` job.
- `full-test.yml`: no top-level or job-level permissions on the `Full-Test` job.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/_test.yml:1`
- `.github/workflows/_test-container.yml:1`
- `.github/workflows/container-test.yml:1`
- `.github/workflows/full-test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding types across 6 workflow files:

**unpinned-uses**: Pinned all three actions to full commit SHAs in ci.yml, _test.yml, _test-container.yml, and release.yml:
- actions/checkout@v5 → @fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09 # v5
- pnpm/action-setup@v4 → @b906affcce14559ad1aafd4ab0e942779e9f58b1 # v4
- actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 # v6

**script-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks:
- release.yml: github.ref_name → REF_NAME env var in 'Create Release' and 'Update Major Tag' steps
- _test.yml: step outputs → CUDA_VERSION and CUDA_PATH_OUTPUT env vars in all three verification steps (Linux bash, Windows PowerShell, final bash)
- _test-container.yml: step outputs → CUDA_VERSION and CUDA_PATH_OUTPUT env vars in the Linux verification step

**missing-permissions**: Added permissions: {} to all affected files:
- ci.yml: top-level + job-level on Format-Lint-TypeCheck and ci-check
- _test.yml: job-level on Test job
- _test-container.yml: job-level on TestContainer job
- container-test.yml: top-level
- full-test.yml: top-level
- release.yml already had contents: write (required for gh release create and git push --force tags)

