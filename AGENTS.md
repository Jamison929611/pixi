# Repository guidance

## Scope and structure

- This file applies to the whole repository. Check for a more specific `AGENTS.md` before editing a subtree.
- Pixi is a Rust workspace (`Cargo.toml`, Rust 1.95.0 from `rust-toolchain`). Workspace crates live in `crates/`; the `pixi` binary is `crates/pixi`, and CLI behavior is primarily in `crates/pixi_cli`.
- Python integration tests live in `tests/integration_python`; fixtures and sample workspaces live in `tests/data`. Documentation is in `docs`, schemas in `schema`, helper scripts in `scripts`, and standalone build-backend projects in `pixi-build-backends`.
- `pixi.toml` is the authoritative catalog of development environments and tasks. `lefthook.yaml` maps repository pre-commit and pre-push checks to those tasks.

## Working practices

- Keep changes focused. Do not reformat, regenerate, or edit unrelated files, and preserve existing user changes in a dirty worktree.
- Follow existing module boundaries, error types, naming, async patterns, and test style. Avoid broad refactors in bug fixes.
- For behavior changes and bug fixes, use test-driven development: add the smallest regression test, run it and confirm the expected failure, implement the minimal fix, then rerun the test and relevant suites.
- Prefer unit tests beside Rust code when they exercise a narrow contract. Use Python integration tests for CLI/end-to-end behavior and existing fixtures/helpers rather than bespoke shell scripts.
- Do not hand-edit generated artifacts. In particular, regenerate CLI reference docs with `pixi run generate-cli-docs` and JSON schemas with `pixi run generate-schema` when their sources change.
- Rust code denies `clippy::dbg_macro` and `clippy::unwrap_used` in the CLI crate. Propagate errors with the repository's existing `miette`/typed-error patterns instead of adding `unwrap` or `dbg!`.

## Build and test commands

- Build the binary: `pixi run build-debug` (or `cargo build --workspace --bin pixi`).
- Run one Rust test while iterating: `cargo test -p <crate> <test_name>`.
- Run the fast Rust suite: `pixi run test-fast`.
- Run one Python integration test: `pixi run test-specific-test-debug "path/to/test.py::test_name"`.
- Run pixi-build integration tests: `pixi run test-pixi-build-debug`; use the release variant `pixi run test-pixi-build` when release behavior matters.
- Run all fast tests: `pixi run test-all-fast`. Slow/online suites are `pixi run test-all-slow` and should be used when the affected behavior requires them.
- The repository anchors Cargo output at `target/pixi` when using Pixi tasks. Direct Cargo commands may need the same `CARGO_TARGET_DIR` if consistency matters.

## Formatting and linting

- Format Rust: `pixi run cargo-fmt`; verify without mutation with `cargo fmt --all -- --check`.
- Lint Rust: `pixi run cargo-clippy` (`cargo clippy --all-targets --workspace -- -D warnings`).
- Run all fast pre-commit checks: `pixi run lint-fast`; all checks: `pixi run lint`.
- Python uses Ruff (`pixi run ruff-lint`, `pixi run ruff-fmt`) and `ty` (`pixi run typecheck-python`). TOML uses Tombi, YAML uses dprint, shell uses shfmt/shellcheck, and spelling uses `pixi run typos`.
- Before committing, run checks proportionate to the changed files plus `git diff --check`. Before pushing Rust changes, run the relevant tests, formatting check, and Clippy for the affected crate or the workspace when feasible.

## CI and pull requests

- `.github/workflows/ci.yml` runs repository lint, Rust tests across Linux/macOS/Windows, docs/link checks, integration tests, and other change-sensitive jobs. Local success does not replace monitoring all required PR checks.
- PR titles must be semantic/conventional (for example, `fix(build): forward verbosity to backends`). Use the repository PR template: describe the issue and fix, list exact tests, disclose AI assistance honestly, and complete only applicable checklist items.
- Keep commits reviewable and conventional-style. Reference the issue in the PR body with `Fixes #<number>` when the change fully resolves it.
- Do not commit credentials, caches, build output, local environments, editor files, or personal planning artifacts.
