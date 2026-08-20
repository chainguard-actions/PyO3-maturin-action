<!-- markdownlint-disable -->

# Hardening Report: PyO3--maturin-action/v1.50.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **PyO3--maturin-action/v1.50.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Three `run:` steps in the `test-manylinux` job directly interpolate `${{ matrix.* }}` expressions inside shell commands, violating rule (a). This allows shell metacharacter injection if matrix values are attacker-influenced.

1. "setup rust-toolchain" step: `run: echo ${{ matrix.toolchain }} > ${{ matrix.platform.test-crate }}/rust-toolchain` — both `${{ matrix.toolchain }}` and `${{ matrix.platform.test-crate }}` are interpolated directly into the shell command.

2. "setup rust-toolchain.toml" step: multi-line run block contains `channel = \"${{ matrix.toolchain }}\"` and `" > ${{ matrix.platform.test-crate }}/rust-toolchain.toml` — direct expression interpolation in shell.

3. "setup rust-toolchain with toml content" step: multi-line run block contains `rm ${{ matrix.platform.test-crate }}/rust-toolchain*`, `channel = \"${{ matrix.toolchain }}\"`, and `" > ${{ matrix.platform.test-crate }}/rust-toolchain` — direct expression interpolation in shell.

All three steps should route matrix values through `env:` variables and reference them as double-quoted shell variables (e.g., `"$MATRIX_TOOLCHAIN"`) instead of using `${{ }}` directly in the run script.

Locations:

- `.github/workflows/test.yml:217`
- `.github/workflows/test.yml:228`
- `.github/workflows/test.yml:240`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in `.github/workflows/test.yml` in the `test-manylinux` job:

1. **"setup rust-toolchain" step** (line 217): Moved `${{ matrix.toolchain }}` and `${{ matrix.platform.test-crate }}` into an `env:` block as `MATRIX_TOOLCHAIN` and `MATRIX_TEST_CRATE`. The `run:` now uses `echo "$MATRIX_TOOLCHAIN" > "$MATRIX_TEST_CRATE/rust-toolchain"`.

2. **"setup rust-toolchain.toml" step** (line 228): Same env variables added. The multi-line `run:` now uses `$MATRIX_TOOLCHAIN` and `"$MATRIX_TEST_CRATE/rust-toolchain.toml"` instead of direct `${{ }}` interpolation.

3. **"setup rust-toolchain with toml content" step** (line 240): Same env variables added. The `rm` command now uses `"$MATRIX_TEST_CRATE/rust-toolchain"*` (glob outside quotes) and the echo uses `$MATRIX_TOOLCHAIN` and `"$MATRIX_TEST_CRATE/rust-toolchain"` instead of direct `${{ }}` interpolation.

