# monocli

**One terminal browser for all your AI CLI sessions.**

Claude Code, Codex, Grok, OpenCode, and Antigravity each keep their session
transcripts in their own private format, in their own corner of your disk.
monocli reads them all through one canonical interface — browse, search,
read, export, resume, and even *move sessions between CLIs*.

> 🎬 **Demo video:** [watch the 2-minute demo](https://drive.google.com/file/d/1yfJIDcbOtL9j4aF5vqPfpaUTt5HRJK3N/view)
>
> 📚 **Docs:** [docs.monolex.ai/ai-clis/monocli](https://docs.monolex.ai/ai-clis/monocli)

---

## What it does

```
monocli                              TUI when stdout is a tty, list when piped
monocli tui                          interactive 2-panel session browser
monocli list [FILTERS]               enumerate sessions across every CLI
monocli show <ID>                    metadata + first/last message preview
monocli read <ID>                    full conversation as markdown
monocli cat <ID>                     raw native-format passthrough
monocli search <QUERY>               full-text search (FTS5 BM25 when indexed)
monocli export <ID> --format json    archival/export view
monocli resume <ID>                  emit the provider-native resume command
monocli translate <ID> --to <cli> --native
                                     lossless native transcode into another CLI
monocli providers                    show adapter availability on this machine
```

Sessions are addressed as `<provider>/<short-id>` — `claude/7244c5a3`,
`codex/019de9e0`, `grok/05f90c2e`, `opencode/ses_54c2`, `agy/67c5c950` —
and filtered by provider, working directory, or time window:

```
monocli list --provider claude --cwd ~/Projects/foo --since 2026-08-01T09:00
```

## Supported CLIs

| Provider | Status | Native store |
|---|---|---|
| Claude Code | full | `~/.claude/projects/**.jsonl` |
| Codex | full | `~/.codex/sessions/**.jsonl` |
| Grok | full | `~/.grok/sessions/**` |
| OpenCode | full | `~/.local/share/opencode/opencode.db` |
| Antigravity (agy) | full | `~/.gemini/antigravity-cli/**` |
| Cursor (IDE agent) | read-only | Cursor local store |

Every adapter normalizes into one canonical model — roles, tool calls, tool
results, thinking blocks, images, token usage, git context, and session
lineage — so every command works identically across providers.

## Highlights

- **Interactive TUI** — a fast 2-panel terminal browser over every session on
  the machine. Bare `monocli` opens it; piping falls back to script-friendly
  list output.
- **Cross-CLI full-text search** — BM25-ranked FTS5 index over all providers
  at once, with a bounded linear-scan fallback when unindexed.
- **Universal native transcode** — `translate --native` writes a *real*
  session into the target CLI's own store. Every message, tool block,
  timestamp, and image is preserved; each write is reparsed through the
  target adapter and verified against the source before it lands. No fidelity
  levels, no text fallback — a session that cannot be represented losslessly
  fails closed instead of being silently truncated.
- **Session lineage** — fork sessions across CLIs, link parent/child edges
  between providers, and reconcile pending forks into a cross-CLI timeline.
- **Resume anywhere** — `monocli resume` emits the exact provider-native
  resume command for any session it can see.
- **Usage-aware** — per-session token and cost columns via the Monometer
  usage engine when installed.
- **Self-testing** — `monocli test auto` drives the TUI through a real PTY
  harness (23 checks) so a broken build tells you before a demo does.

## Platform

macOS and Linux are fully supported; Windows is supported with the Windows
SQLite toolchain (the TUI runs on crossterm — only the Unix PTY self-test
harness is platform-bound).

## Status

**Experimental public preview.** monocli ships today as part of the
[Monolex](https://monolex.ai) / OpenCLIs toolchain; a standalone source
release from this repository is in preparation. Star or watch the repo to
follow along.

---

Built on `lib-monosession`, the canonical AI-session model behind the
Monolex AI terminal. © Umzikim Inc.
