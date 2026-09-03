<!-- markdownlint-disable -->

# Hardening Report: mjun0812--setup-cuda/v1.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **mjun0812--setup-cuda/v1.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable version tags instead of full 40-character commit SHAs. Failing references: 'actions/checkout@v7' and 'voidzero-dev/setup-vp@v1' appear in ci.yml, release.yml, _test.yml, and _test-container.yml. These should be pinned to exact commit SHAs (e.g., actions/checkout@<40-hex-sha>) to prevent supply-chain attacks.

Locations:

- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:19`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:23`
- `.github/workflows/_test.yml:40`
- `.github/workflows/_test.yml:43`
- `.github/workflows/_test-container.yml:57`
- `.github/workflows/_test-container.yml:60`

### script-injection (severity: high)

Multiple run: blocks directly interpolate GitHub Actions expressions (${{ ... }}) into shell commands, violating sub-rule (a). In release.yml: (1) line 40 interpolates ${{github.ref_name}} directly into a gh CLI command (`gh release create ${{github.ref_name}} --generate-notes`); (2) line 44 interpolates ${{ github.ref_name }} into a shell variable assignment (`GITHUB_REF_NAME=${{ github.ref_name }}`). In _test.yml: lines 57–58, 84–85, and 113–114 interpolate ${{ steps.setup-cuda.outputs.version }} and ${{ steps.setup-cuda.outputs.cuda-path }} directly into echo commands inside run: blocks. In _test-container.yml: lines 75–76 do the same. These expressions are substituted by the template engine before the shell sees them, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:44`
- `.github/workflows/_test.yml:57`
- `.github/workflows/_test.yml:58`
- `.github/workflows/_test.yml:84`
- `.github/workflows/_test.yml:85`
- `.github/workflows/_test.yml:113`
- `.github/workflows/_test.yml:114`
- `.github/workflows/_test-container.yml:75`
- `.github/workflows/_test-container.yml:76`

### missing-permissions (severity: medium)

Several workflow files have no top-level 'permissions:' block and no job-level 'permissions:' blocks on any of their jobs. This means the workflows run with the default (potentially broad) GITHUB_TOKEN permissions. Affected files: ci.yml, full-test.yml, _test.yml, _test-container.yml, and container-test.yml. Each should declare minimal required permissions (e.g., 'permissions: contents: read') at the top level or per job.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/full-test.yml:1`
- `.github/workflows/_test.yml:1`
- `.github/workflows/_test-container.yml:1`
- `.github/workflows/container-test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding types: (1) Pinned actions/checkout@v7 to SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 and voidzero-dev/setup-vp@v1 to SHA 250f29ce396baf5e8f24498e17c0dfdebabc26eb in ci.yml, release.yml, _test.yml, and _test-container.yml. (2) Fixed script injection in release.yml by moving github.ref_name into REF_NAME env var in both Create Release and Update Major Tag steps; fixed script injection in _test.yml and _test-container.yml by moving steps.setup-cuda.outputs.version and steps.setup-cuda.outputs.cuda-path into CUDA_VERSION and CUDA_PATH_OUTPUT env vars. (3) Added 'permissions: contents: read' to ci.yml, full-test.yml, _test.yml, _test-container.yml, and container-test.yml; release.yml already had the necessary 'permissions: contents: write' for its release/tag operations.

