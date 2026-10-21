# wrkgrd

A verification gate for source trees. wrkgrd detects which language stacks a
repository uses, runs each stack's build and tests, and reports only failures.
The output is compact and color-free, so a coding agent or a CI job can read it
cheaply.

wrkgrd is the code-verification member of the `*grd` family. The family shares
one CLI shape, config format and exit-code contract:

| Tool       | Checks                                  |
| ---------- | --------------------------------------- |
| `ctxgrd`   | Documents: ADRs, PRDs, READMEs          |
| `trtlgrd`  | RDF graphs: Turtle, N-Triples           |
| `wrkgrd`   | Code: build, tests, lint configuration  |

This repository holds the release binaries and the guides. The source is not
published.

## Install

```sh
brew install aktagon/tap/wrkgrd
```

Or download the archive for your platform from the
[releases page](https://github.com/aktagon/wrkgrd/releases) and check it against
`checksums.txt`. Releases cover macOS (Apple silicon and Intel) and Linux
(x86_64, statically linked).

## Use

```sh
wrkgrd init      # write a starter wrkgrd.toml
wrkgrd rules     # show the verification tools for each detected stack
wrkgrd verify    # run build and tests; exit 0 clean, 1 failures, 2 misuse
wrkgrd features  # list wrkgrd's capabilities
wrkgrd docs      # print the full guide
```

`wrkgrd verify --harness claude` wraps the result in the Claude Code Stop hook
contract. `wrkgrd hooks claude` prints the hook configuration.

## Documentation

- [Getting started](docs/guides/001-getting-started.md)
- [Changelog](CHANGELOG.md)

## License

MIT. See [LICENSE](LICENSE).
