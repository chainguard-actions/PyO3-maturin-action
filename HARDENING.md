<!-- markdownlint-disable -->

# Hardening Report: PyO3--maturin-action/v1.51.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **PyO3--maturin-action/v1.51.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `run:` blocks in the `test-manylinux` job directly interpolate `${{ matrix.* }}` expressions into shell commands. GitHub Actions performs YAML template substitution before the shell ever sees the string, so any shell metacharacters in the matrix values are interpreted by the shell.

1. Step "setup rust-toolchain" (line ~222): `run: echo ${{ matrix.toolchain }} > ${{ matrix.platform.test-crate }}/rust-toolchain` — both `matrix.toolchain` and `matrix.platform.test-crate` are interpolated directly into the shell command.

2. Step "setup rust-toolchain.toml" (line ~231): multi-line `run:` block contains `channel = \"${{ matrix.toolchain }}\"` and `> ${{ matrix.platform.test-crate }}/rust-toolchain.toml`.

3. Step "setup rust-toolchain with toml content" (line ~243): multi-line `run:` block contains `rm ${{ matrix.platform.test-crate }}/rust-toolchain*`, `channel = \"${{ matrix.toolchain }}\"`, and `> ${{ matrix.platform.test-crate }}/rust-toolchain`. Fix: move the values into `env:` variables and reference them as quoted shell variables, e.g. `env: { TOOLCHAIN: "${{ matrix.toolchain }}", TEST_CRATE: "${{ matrix.platform.test-crate }}" }` and use `"$TOOLCHAIN"` / `"$TEST_CRATE"` in the script.

Locations:

- `.github/workflows/test.yml:222`
- `.github/workflows/test.yml:231`
- `.github/workflows/test.yml:243`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in `.github/workflows/test.yml` in the `test-manylinux` job:
1. Step 'setup rust-toolchain' (line ~222): moved `${{ matrix.toolchain }}` and `${{ matrix.platform.test-crate }}` into `env:` as `TOOLCHAIN` and `TEST_CRATE`, updated run command to use `"$TOOLCHAIN"` and `"$TEST_CRATE"`.
2. Step 'setup rust-toolchain.toml' (line ~231): same env block approach, updated multi-line run script to use `"$TOOLCHAIN"` and `"$TEST_CRATE"`.
3. Step 'setup rust-toolchain with toml content' (line ~243): same env block approach, updated multi-line run script to use `"$TOOLCHAIN"` and `"$TEST_CRATE"` throughout (rm command and echo redirect).

