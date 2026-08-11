# Contributing

## Where the source lives

This repository is the published face of monocli. The canonical source is an
internal build closure; the source drop lands here as a mirror, so a change made
only in this repository would be overwritten on the next sync. Open an issue
first and we will land the change upstream and mirror it back.

## Building

```sh
cargo build --locked --workspace
cargo test --workspace
```

**Prerequisite:** `rusqlite` is built with its `bundled` feature, which compiles
SQLite from source — you need a working C toolchain (Xcode command line tools on
macOS, `build-essential` on Debian/Ubuntu, MSVC build tools on Windows). This is
not documented anywhere else and is the most common first-build failure.

## Feature lanes

The session library carries two mutually exclusive SQLite lanes; a change to the
adapters should be checked against all three:

```sh
cargo test -p lib-monosession                                   # default: rusqlite
cargo test -p lib-monosession --no-default-features             # parse-only
cargo test -p lib-monosession --no-default-features --features libsql-opencode
```

## Screenshots, logs, and bug reports

`monocli test spawn` and `monocli test key` render **your** sessions — titles,
working directories, branch names. Add `--demo` for any output that leaves your
machine:

```sh
monocli test spawn --demo
```

The same applies to `export`: `--redact` masks vendor-marked credentials, JWTs,
PEM blocks, provider reasoning signatures and your home path, but it is
best-effort. Read the output before you send it.
