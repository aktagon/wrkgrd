# Changelog

All notable changes to wrkgrd are documented in this file. The format follows
[Keep a Changelog](https://keepachangelog.com/), and the project adheres to
[Semantic Versioning](https://semver.org/). wrkgrd guards its own releases with
the pre-push release-guard verifier (ADR-017): a tag's version must match
`Cargo.toml`, this changelog must carry an entry for it, the tree must be clean,
and the tag must be annotated and version-monotonic.

## [Unreleased]

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
