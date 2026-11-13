# Changelog

All notable changes to wrkgrd are documented in this file. The format follows
[Keep a Changelog](https://keepachangelog.com/), and the project adheres to
[Semantic Versioning](https://semver.org/). wrkgrd guards its own releases with
the pre-push release-guard verifier (ADR-017): a tag's version must match
`Cargo.toml`, this changelog must carry an entry for it, the tree must be clean,
and the tag must be annotated and version-monotonic.

## [Unreleased]

## [0.8.1] - 2026-11-13

### Fixed

- **A ruff that is not installed warns, and no longer blocks as lint errors
  (BUG-021).** wrkgrd runs ruff only when it is in the project's `.venv` or on
  PATH. Otherwise it warns "ruff is not installed" and skips the step.

## [0.8.0] - 2026-11-12

### Added

- **A failing blocking lint says how to turn it off (ROADMAP-012).** Its help
  line says to fix the errors, and to ask the user before setting
  `[tools.<name>] enabled = false` when they predate the change. Builds,
  tests, type checks and the secret scan get no such line.

## [0.7.0] - 2026-11-12

### Added

- **`verify --pre-commit`, the pre-commit hook path (ADR-037 WRK-195,
  WRK-197).** It runs the per-turn and pre-commit runners, the builds
  included, and no test runner. Plain `verify` still runs the tests. The
  fragment `hooks install` writes now ends with `exec wrkgrd verify
  --pre-commit`; rerun `wrkgrd hooks install` to get it.
- **A stale hook warns (ADR-038 WRK-199).** The new `stale-hook` audit warns
  once per hook wrkgrd wrote that this version writes differently (the
  pre-commit fragment, `commit-msg`, `pre-push`, and the Claude Stop hook
  entry), and points at `grd doctor --fix`. It never fails the run.
- **`hooks install --dry-run --format json` (ADR-038 WRK-205)** prints one
  object with each file's path, body and whether the installed copy matches.
- **A failure names its command (ADR-038 WRK-201)** in the text header and in
  a JSON `command` field.
- **Rust diagnostics carry their location (ADR-038 WRK-202).** `cargo-clippy`
  and `cargo-test` run with `--message-format=json`; the diagnostic sets
  `file`, `line` and `col` from the first error.

### Changed

- **Two scoped mode flags together are misuse (ADR-038 WRK-198)** and exit 2.
- **A passing `--pre-commit` run says the tests did not run (ADR-038
  WRK-200).**
- **The end-of-turn block reason holds errors only (ADR-038 WRK-203).**
- **The help text is for users (ADR-038 WRK-204):** no document ids or
  implementation notes; the examples list `verify --pre-commit`, and `verify
  --help` states the exit codes.

## [0.6.1] - 2026-11-01

### Changed

- **The end-of-turn run no longer builds (ADR-037 WRK-192).** `node-build`,
  `go-build` and `swift-build` move to the `pre-commit-hook` context, and a
  SHACL shape rejects any other build at the end of a turn. `verify
  --harness claude` runs type check and lint only, plus `cargo check` when
  clippy is off, because it generates no code. Plain `verify` still runs
  each build before its tests, so `skip_tests_if_build_fails` still applies.

## [0.6.0] - 2026-11-01

### Added

- **A run on the same tree waits for the run already going (ADR-037
  WRK-190).** Plain `verify` holds a lock from its first key to its record.
  A second plain run on the same key prints `wrkgrd: waiting for wrkgrd run
  <pid> (verify, started <n> s ago) on the same tree` to stderr, waits, and
  skips when that run ended green. A run on a different key does not wait.
  Each key build also holds a short lock on the private index. The locks
  live in `<git-dir>/grd-skip/lock-*`; the OS frees them when a run exits.

### Changed

- **The end-of-turn run no longer runs the test runners (ADR-037 WRK-186,
  WRK-187).** Every test tool moves to the `pre-commit-hook` context, and a
  SHACL shape rejects one at the end of a turn. `verify --harness claude`
  runs build, lint and type check only. Plain `verify` (pre-commit, manual,
  CI) still runs every stack runner. Under `--harness`, `cargo clippy`
  stands in for `cargo check` when it is on PATH (WRK-188).
- **The end-of-turn run never skips (ADR-037 WRK-189).** `verify --harness`
  builds no CR-019 key, reads, writes or deletes no record, takes no lock
  and prints no skip line. A red turn no longer deletes plain `verify`'s
  record.
- **Scoped modes name the check that ran (ADR-037 WRK-191).** With no
  diagnostics, `--pre-push` prints `wrkgrd: release guard passed (stack
  runners not run)`, in place of `wrkgrd: all detected stacks passed`.
  `--staged`, `--commit-msg`, `--anchors`, `--links`, `--bugs`, `--reqs` and
  `--stdlib-only` print the same shape. JSON output is unchanged.

## [0.5.0] - 2026-10-31

### Added

- **Dependency cooldown audits (ADR-005 WRK-180 to WRK-182).** uv, npm,
  pnpm, Bun, Yarn and Bundler each get a per-turn row and a SessionStart row.
  They warn when the client's release-age cooldown is missing or under 14
  days, or when the client is too old to have one. Node audits follow
  `packageManager`, then the lockfile, then PATH.
- **Install-script audits (ADR-036 WRK-183 to WRK-185).** npm, pnpm and Yarn
  warn when the project config turns install-script blocking off, or when
  the client is older than the first release that blocks by default.
- **Requirement tests (ADR-034).** In a namespace that binds ctxgrd's
  `core.requirement-status`, each Implemented requirement is checked for a
  test named after its key (`adr_033_wrk_150_*`). `wrkgrd reqs`, `wrkgrd reqs
  test <KEY>` and `verify --reqs` report the matrix. Plain `verify` folds the
  missing tests into one summary warning; `verify --reqs` errors on each.
- **clippy, go vet and tsc run per turn** from the catalog, each honouring
  `[tools.<name>] enabled`. `cargo clippy` replaces `cargo check` as the
  Rust build step when it is on PATH. tsc runs only from the project's own
  `node_modules/.bin/tsc` with a `tsconfig.json`, never through npx.

### Fixed

- **A root `verify` checked stacks at the root only (BUG-011, ADR-035).**
  `wrkgrd.toml` gains `roots` and `skip_roots`. A root run verifies each
  listed root under that root's own `wrkgrd.toml`, and
  `nested-manifest-skipped` warns once per tracked manifest no root
  verifies.

### Changed

- **The CR-019 skip runs on grd's `grd-skip` crate (grd ADR-010).** The key is
  unchanged. The record moves to
  `<git-dir>/grd-skip/wrkgrd-<root key>.json`, so the first run after the
  upgrade runs in full. Building wrkgrd needs grd checked out beside it, at
  `../grd`.
- **The skip line is the grd family's (grd ADR-010 GRD-071).** A skipped
  `verify` prints `wrkgrd: skipped: unchanged since clean run at <UTC>, key
  <7 hex>` in place of `wrkgrd: verify skipped: code unchanged since green run
  at …, hash …`. Under `--harness claude` a clean skip now sends that line as
  the `systemMessage`, with no `decision`; CR-019 kept it silent. Other notes
  still travel only beside a block.
- **A run that could not skip says why (grd ADR-010 GRD-072, Open Question
  9).** With the skip on, `verify` prints `wrkgrd: not skipped: <reason>` on
  stderr, in place of `wrkgrd: verify not skipped: …`. Outside a git work tree
  it now prints `wrkgrd: not skipped: not inside a git work tree`; before, it
  printed nothing. Under `--harness claude` the note goes to stderr only and
  never into the reply, beside a block included.

## [0.4.0] - 2026-10-21

### Added

- **Release binaries and a Homebrew formula.** `brew install aktagon/tap/wrkgrd`
  installs a prebuilt binary for macOS (Apple silicon, Intel) and Linux
  (x86_64, statically linked). The archives and `checksums.txt` are attached to
  the releases on the public repository, `aktagon/wrkgrd`, which holds the
  binaries and the guides. The source stays private.

### Changed

- **`--format json` is built from the family's generated types** (grd
  `ADR-003`). `src/wire.rs` is written by grd's `codegen/generate.py`; `make
  check` fails when it is stale, and `make wire-codegen` regenerates it. Two
  visible changes: `diagnostics` now precedes `summary`, and a diagnostic with
  no help text leaves `help` out instead of printing `null` (grd GRD-066).
  The parsed content is otherwise unchanged.
- **`hooks install` no longer overwrites a pre-commit hook it did not write**
  (grd `ADR-006`). It used to replace any `.githooks/pre-commit` whose bytes
  differed from the current runner, exit 0, which destroyed a user's own hook.
  It now checks the file against the family list of shipped runners
  (`src/known-runners.txt`). An older wrkgrd or sibling runner is upgraded. Any
  other file is refused: nothing is written and the exit is 2, with `io.exists`
  under `--format json`. `--force` replaces it. When `core.hooksPath` already
  names another directory (husky's `.husky`), install writes there and no
  longer resets `core.hooksPath` to `.githooks`.
- **`hooks install` no longer overwrites a `commit-msg` or `pre-push` hook it
  did not write** (grd `ADR-006`). In a husky repository it replaced the user's
  `.husky/commit-msg`. Each file is now checked against the bodies wrkgrd has
  shipped (`src/known-commit-msg.txt`, `src/known-pre-push.txt`) under the same
  rule as the pre-commit runner. All three files are checked before anything is
  written, so one foreign file means nothing is written; `--force` replaces
  them all.

## [0.3.0] - 2026-10-07

### Changed

- **Diagnostics now carry `file`, `line`, and `help` in their own JSON fields**
  (BUG-017). `message` no longer embeds a position. Secret findings set `file`,
  `line`, and a stable `detector` id such as `aws-access-key`; `code` stays
  `secret-scan`. The `commit-attribution` check emits one diagnostic per
  offending line, with the attribution form in `form` and the fix in `help`.
  Text output prints `file:line` before the message and `help` after it.

- **`verify --format json` now emits the `*grd` family wire object** — one JSON
  object per invocation carrying `exit_code`, `diagnostics` and `summary`
  (ctxgrd `ADR-086` § WIRE-002..005, schema
  `ctxgrd/docs/schema/grd-output.schema.json`). This replaces the previous JSONL
  stream, under which `wrkgrd verify --format json | jq` was an error for anyone
  who read the documented contract and trusted it (BUG-015).

  Breaking for any JSON consumer, and deliberately taken while wrkgrd is
  pre-release. What changed:

  - one object, not one line per finding plus a trailing summary line;
  - `exit_code` is present, at the top level;
  - each diagnostic carries `code`, a **stable identifier** from the generated
    tool catalog (`cargo-check`, `branch-scope`, `secret-scan`, …), spelled the
    same as the config key. The human label stays available as `check`, which is
    prose and may be reworded;
  - `severity` spells the advisory tier `warning`, not `warn`;
  - `file`, `line` and `col` are present and null rather than absent, so a
    consumer reads a value instead of testing for a key;
  - `summary` carries `errors`/`warnings`/`infos`/`files`; the old `failed` key
    is gone, replaced by `errors`.

  `summary.files` is 0: wrkgrd's unit of input is a detected stack run through
  its toolchain, not a file it reads, and reporting a stack count under a key
  named `files` would repurpose a canonical field. The verdict a consumer needs
  is in `exit_code` and `diagnostics`.

  The internal `Diagnostic` is unchanged apart from the added `code` — it remains
  the in-process plugin contract, and the wire shape is now a separate
  projection. Serialising the internal type directly was the root defect.

### Added

- `wrkgrd scan secrets <path>...` runs the staged secret-scan rule set over
  files and directories (CR-018, ADR-015 WRK-141). It walks each directory,
  skipping `.git` and binary files, and runs gitleaks too when it is on PATH.
  `[secret-scan] allow_paths` match the path relative to each scan root.

- `--harness <name>` is accepted at the **top level**, resolving to the default `verify` (CR-017). `wrkgrd --harness claude` and `wrkgrd verify --harness claude` are now the same invocation. Purely additive — the subcommand form is unchanged, and its value wins if both positions carry one. wrkgrd was the family's only single-position member; ctxgrd and trtlgrd already accepted the flag in either place, so a family-wide caller had to know which tool was the exception
- Release-guard verifier (ADR-017): a Tier-1/Block check on the git stack that
  runs in the pre-push hook (`wrkgrd verify --pre-push`) and rejects a pushed tag
  whose version disagrees with the declared version file, whose version has no
  changelog entry, whose tree is dirty, or whose tag is lightweight or
  non-monotonic.

### Fixed

- `verify --staged` and `verify --pre-push` now run when `--root` sits below
  the repository top (BUG-019). They used to find no `.git` at the root, run
  nothing, and exit 0, so a nested project's pre-commit hook let a staged
  secret through. The staged scan covers the whole staged diff and reports
  `file` relative to the root; a root inside no git work tree now blocks.

- The high-entropy secret check no longer treats a credential keyword inside a
  longer word as a credential name (BUG-018). `keychain`, `keyboard`, and
  `tokenizer` pass; `api_keys`, `key2`, and `apikey` still match.

- `core.hooksPath` is now read from the git **common** directory, so a wired
  `.githooks/pre-commit` is no longer reported as inactive inside a linked git
  worktree (BUG-013). A worktree's `.git` is a file, not a directory, so the old
  `<root>/.git/config` read always missed and the default fell back to a
  `<root>/.git/hooks` that cannot exist — making the warning fire in every
  worktree, while that very hook was running the audit. Resolution stays
  filesystem-only (no `git` subprocess); a relative `hooksPath` remains
  working-tree-relative, so one shared `.githooks` still serves the primary tree
  and every worktree at once.

## [0.1.0]

### Added

- Initial verification binary (ADR-001): ontology-driven tool catalog (ADR-002)
  verifying Go, Node, Python, Rust, and Ruby stacks, with a ctxgrd-mirrored CLI
  (`verify` / `init` / `rules`), git-hook integration, the staged secret scan
  (ADR-015), the Conventional Commits message check (ADR-008), the branch-scope
  advisory (ADR-014), and the composable pre-commit drop-in protocol (ADR-016).
