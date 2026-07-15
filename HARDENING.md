<!-- markdownlint-disable -->

# Hardening Report: PyO3--maturin-action/v1.51.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **PyO3--maturin-action/v1.51.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct expression interpolation of ${{ matrix.* }} values inside run: shell commands in the test-manylinux job. Three steps are affected:

1. 'setup rust-toolchain' step: `run: echo ${{ matrix.toolchain }} > ${{ matrix.platform.test-crate }}/rust-toolchain` — both matrix.toolchain and matrix.platform.test-crate are interpolated directly into the shell command string.

2. 'setup rust-toolchain.toml' step: A multiline run: block interpolates `${{ matrix.toolchain }}` and `${{ matrix.platform.test-crate }}` directly into shell commands (echo and file redirection).

3. 'setup rust-toolchain with toml content' step: A multiline run: block interpolates `${{ matrix.platform.test-crate }}` (in rm command) and `${{ matrix.toolchain }}` and `${{ matrix.platform.test-crate }}` (in echo/redirect) directly into shell commands.

Any ${{ ... }} expression interpolated directly into a run: block is a script-injection risk because the value is substituted before the shell parses the command, allowing shell metacharacters to be injected.

Locations:

- `.github/workflows/test.yml:228`
- `.github/workflows/test.yml:237`
- `.github/workflows/test.yml:249`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection vulnerabilities in .github/workflows/test.yml in the test-manylinux job:
1. 'setup rust-toolchain' step: moved ${{ matrix.toolchain }} and ${{ matrix.platform.test-crate }} into env: block as TOOLCHAIN and TEST_CRATE, updated run: to use "$TOOLCHAIN" and "$TEST_CRATE/rust-toolchain".
2. 'setup rust-toolchain.toml' step: moved ${{ matrix.toolchain }} and ${{ matrix.platform.test-crate }} into env: block as TOOLCHAIN and TEST_CRATE, updated run: to use "$TOOLCHAIN" and "$TEST_CRATE/rust-toolchain.toml".
3. 'setup rust-toolchain with toml content' step: moved ${{ matrix.toolchain }} and ${{ matrix.platform.test-crate }} into env: block as TOOLCHAIN and TEST_CRATE, updated run: to use "$TEST_CRATE"/rust-toolchain* (for rm) and "$TOOLCHAIN" and "$TEST_CRATE/rust-toolchain" (for echo/redirect).

