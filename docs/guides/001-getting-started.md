---
diataxis:
  type: reference
title: wrkgrd getting started
---

# wrkgrd — getting started

wrkgrd orchestrates a project's build/test toolchain behind one command. It
detects which language stacks a repository uses, runs each stack's canonical
build and test steps, and reports only failures — compact, color-free output a
coding agent or a CI job can read cheaply. It is the code-verification sibling
of ctxgrd (documents) and turtleguard in the *grd family.

wrkgrd never runs command strings from config and never formats your code; it
only verifies and audits.

## Install

wrkgrd is a single binary. Install it with Homebrew on macOS or Linux:

```sh
brew install aktagon/tap/wrkgrd
```

Or download the archive for your platform from the
[releases page](https://github.com/aktagon/wrkgrd/releases), check it against
`checksums.txt`, and put `wrkgrd` on your `PATH`. Confirm the install with
`wrkgrd --help`, whose command surface mirrors ctxgrd.

## Commands

- `wrkgrd verify` (default) — verify every detected stack's build and tests. A
  failing build skips the redundant test step. Advisory config audits (weak
  lint/compiler configuration) warn but never fail the run.
- `wrkgrd init` — write a starter `wrkgrd.toml` to the project root. `--force`
  overwrites an existing file.
- `wrkgrd rules` — list the verification-tool catalog grouped by stack (tool
  name, check-kind, tier, severity, context, implemented), then show per-stack
  detection state. `wrkgrd rules <tool>` prints one tool's full classification.
- `wrkgrd hooks install` — wire the tracked pre-commit hook (see below).
  `--dry-run` prints the plan and writes nothing; `--force` overwrites an
  existing `.githooks/pre-commit`.
- `wrkgrd docs` — print the canonical guide.

## Global flags

- `--root <PATH>` — project root for all commands (default: the nearest
  ancestor holding a `wrkgrd.toml`, else the current directory).
- `--format text|json` — `text` (default) is human-readable; `json` emits
  machine-readable output for CI pipelines.

## wrkgrd.toml toggles

Config is declarative toggles only — never command strings. Two keys are read:

- `skip_tests_if_build_fails` (default `true`) — skip the test step when the
  build/check step already failed, since it re-reports the same compile error.
- `disable` (default `[]`) — stacks to skip entirely, by name: `go`, `node`,
  `python`, `rust`, `ruby`.

The `[secret-scan]` table tunes the always-on native staged-secret scan:

- `allow_paths` (default `[]`) — repo-root-relative paths the native scan skips,
  for files of deliberate fake secrets (test fixtures) that would otherwise need
  `--no-verify`. Matching is path-prefix, not glob: an exact path
  (`"src/secrets.rs"`), or a `dir/`-suffixed entry that exempts everything under
  it (`"tests/"`). Example:

  ```toml
  [secret-scan]
  allow_paths = ["src/secrets.rs", "tests/fixtures/"]
  ```

Any other table (e.g. `[commands]`) is ignored by design.

## Git pre-commit hook

`wrkgrd hooks install` installs the tracked convention: a committed
`.githooks/pre-commit` that runs `wrkgrd verify --staged` (a staged-diff secret
scan) and then `wrkgrd verify --pre-commit` (the builds and lint, no tests; run
plain `wrkgrd verify` for the test suite), plus a `.githooks/commit-msg` that checks the
Conventional Commits subject, a `scripts/setup-hooks.sh` bootstrap, and
`core.hooksPath .githooks` set so the hooks are active immediately. The hooks
hard-gate: a clone that has activated them but lacks wrkgrd on `PATH` aborts the
commit (exit 1) rather than silently skipping the check, so a missing binary
cannot bypass the gate. A fresh clone activates the shared hooks with
`scripts/setup-hooks.sh` (or `git config core.hooksPath .githooks`).

## Exit codes

- `0` — clean (no failing checks).
- `1` — diagnostics (one or more checks failed).
- `2` — misuse (bad flags, config error, or a refused `hooks install`).
