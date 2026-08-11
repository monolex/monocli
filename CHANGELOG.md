# Changelog

All notable changes to monocli. Versions follow SemVer.

## [Unreleased]

### Changed
- Documentation: example session ids throughout are now documented placeholders
  rather than ids drawn from a real machine.
- Corrected two claims about how the tool ships its help: bare `monocli` opens
  the TUI (or prints the list when piped); `monocli --help` prints the condensed
  reference, and `initiate.md` ships beside the binary.

### Removed
- Site screenshots and the linked walkthrough recording. Both were rendered from
  a live session store and are being replaced by material produced against a
  synthetic one.

## [0.2.16] — 2026-07-24

First public binary release: macOS (Apple Silicon, Developer ID-signed), Linux
x86_64, and Windows x86_64. Cross-CLI session browsing, search, export, resume,
and lossless native transcode across Claude Code, Codex, Grok, OpenCode, and
Antigravity, plus read-only Cursor IDE sessions.
