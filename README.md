# monocli

**The canonical session layer for AI CLIs.**

Claude Code, Codex, Grok, OpenCode, and Antigravity each speak a private
session dialect — different files, different schemas, different tool-call
encodings. monocli normalizes all of them into one **complete** canonical
model: every message, every tool call and its result, thinking blocks,
images, timestamps, native IDs, token usage, git context, session lineage.

Complete enough to browse and search everything in one place — and complete
enough to **write a whole session back into another CLI's own store** and
keep working there. Translation has no fidelity levels and no text fallback:
every block transfers, verified one-by-one against the source, or the write
fails closed.

Your sessions stop being app-locked artifacts and become portable data you
own. monocli is the session plane of [Monolex](https://monolex.ai) — one
concept layer of the AI-native terminal, shipped as a standalone CLI.

> 🎬 **Demo video:** [watch the 2-minute demo](https://drive.google.com/file/d/1yfJIDcbOtL9j4aF5vqPfpaUTt5HRJK3N/view)
>
> 📚 **Docs:** [docs.monolex.ai/ai-clis/monocli](https://docs.monolex.ai/ai-clis/monocli)

```
  ┌────────────┬────────────┬────────────┬────────────┬────────────┐
  │ Claude Code│   Codex    │    Grok    │  OpenCode  │Antigravity │
  │  ~/.claude │  ~/.codex  │   ~/.grok  │ opencode.db│  ~/.gemini │
  └─────┬──────┴─────┬──────┴─────┬──────┴─────┬──────┴─────┬──────┘
        │            │            │            │            │
        └────────────┴────────────┼────────────┴────────────┘
                                  ▼
        ┌───────────────────────────────────────────────────┐
        │                 provider adapters                 │
        │  one decoder per native format (JSONL/SQLite/pb)  │
        └─────────────────────────┬─────────────────────────┘
                                  ▼
        ┌───────────────────────────────────────────────────┐
        │              canonical session model              │
        │  messages · tool calls · images · git · lineage   │
        └─────────────────────────┬─────────────────────────┘
                                  ▼
        ┌───────────────────────────────────────────────────┐
        │  browse (TUI) · search · read · export · resume   │
        │  translate --native → a real session written      │
        │  into another CLI's own store, verified lossless  │
        └───────────────────────────────────────────────────┘
```

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

- **A complete canonical model, not a lowest common denominator** — roles,
  tool calls *and their results*, thinking blocks, images, native IDs,
  request metadata, token usage, git branch/commit context, forks and
  parent edges. If a provider recorded it, the canonical form carries it;
  unmapped provider tools are preserved verbatim (`ToolCall::Unknown` keeps
  the raw name and args) instead of being dropped.
- **Universal native transcode** — `translate --native` writes a *real*
  session into the target CLI's own store, block by block. Every message,
  tool call, tool result, timestamp, and image is reconstructed in the
  target's native encoding, then the written session is reparsed through the
  target adapter and verified against the source before it lands. No
  fidelity levels, no text fallback — a session that cannot be represented
  losslessly fails closed instead of being silently truncated.
- **Interactive TUI** — a fast 2-panel terminal browser over every session on
  the machine. Bare `monocli` opens it; piping falls back to script-friendly
  list output.
- **Cross-CLI full-text search** — BM25-ranked FTS5 index over all providers
  at once, with a bounded linear-scan fallback when unindexed.
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

# Full CLI reference

The complete built-in help (`initiate.md`) — running bare `monocli` with no
arguments prints this reference. Every OpenCLIs tool ships its full help this
way so both humans and AI agents can discover the entire surface offline.

## USAGE

```
monocli                              TUI when stdout is a tty, list when piped
monocli tui [--provider P] [--all]   interactive 2-panel browser
monocli list [FILTERS]               enumerate sessions
monocli show <ID>                    meta + first/last message preview
monocli read <ID>                    full conversation, markdown
monocli cat <ID>                     raw native-format passthrough
monocli search <QUERY> [FILTERS]     full-text search across sessions
                                     (FTS5 BM25 when indexed, linear scan
                                     fallback; aliases: grep, find)
monocli index [--rebuild]            build/refresh the incremental FTS index
monocli export <ID> --format json    archival/export view
monocli resume <ID>                  emit native resume command
monocli translate <ID> --to agy --native
monocli test auto                    PTY self-test for the TUI
monocli providers                    show adapter availability
```

## ID FORMAT

```
<provider>/<short>          claude/7244c5a3, codex/019de9e0,
                            grok/05f90c2e, opencode/ses_54c2, agy/67c5c950
<full-uuid>                 also accepted for any provider
```

## FILTERS (`list`)

```
--provider claude|codex|grok|agy|opencode|cursor
                                  restrict to one provider
--cwd /path                       sessions whose cwd starts with PATH
                                  (also accepts ~/path)
--since YYYY-MM-DDTHH:MM         ISO lower bound (LOCAL timezone)
--until YYYY-MM-DDTHH:MM         ISO upper bound (LOCAL timezone)
--limit N                         cap output (default 30, --all to disable)
--json                            emit JSON instead of table
--include-sidechains              include Claude subagent files
```

## TUI

```
monocli tui [--cwd PATH] [--all] [--provider P] [--limit N]
monocli browse                    alias for tui
```

Bare `monocli` opens the TUI when stdout is a terminal. When stdout is piped or
redirected, it prints the non-interactive list output for scripts.

Environment:

```
MONOCLI_DEFAULT=list|tui          force default action
MONOCLI_TUI_PROVIDER=P            default provider filter for bare monocli / tui
```

Self-test:

```
monocli test auto                 expected: 23 passed / 0 failed
monocli test spawn                show initial TUI screen from PTY harness
monocli test key q                spawn, send q, show resulting screen
```

## PROVIDERS

| Provider | Status | Storage |
|---|---|---|
| claude | implemented | `~/.claude/projects/<encoded-cwd>/<uuid>.jsonl` |
| codex | implemented | `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl` |
| grok | implemented | `~/.grok/sessions/<encoded-cwd>/<uuid>/` (4-file split) |
| opencode | implemented | `~/.local/share/opencode/opencode.db` (SQLite; platform data-dir fallback) |
| agy / antigravity | implemented | `~/.gemini/antigravity-cli/brain/*` mirror + `conversations/*.{pb,db}` native state |

## EXAMPLES

```
monocli                              TUI in a terminal, list when piped
monocli list --provider claude       Claude sessions only
monocli list --provider agy          Antigravity canonical mirror sessions
monocli list --cwd ~/Projects/foo    sessions started under this directory
monocli list --since 2026-05-28T20:00 last 8 hours (local time)
monocli show claude/7244c5a3         session metadata + preview
monocli read codex/019de9e0          full conversation as markdown
monocli cat claude/7244c5a3 > out.jsonl   raw jsonl dump
monocli resume codex/019de9e0        emit `codex resume -C <cwd> <uuid>`
monocli translate claude/7244c5a3 --to agy --native
monocli test auto                    verify TUI render, keys, picker, scope
```

## OUTPUT (LIST)

```
PROVIDER  ID          WHEN              MSGS   MODEL             TITLE/SLUG               CWD
claude    7244c5a3    2026-05-28 14:32  127    claude-opus-4-7   wondrous-popping-hedgehog .../app-monolex
codex     019de9e0    2026-05-28 12:08   85    openai            lib-niia-core refactor    .../app-monolex
```

## CANONICAL MODEL

monocli builds on `lib-monosession`'s canonical types:

```
SessionMeta { id, started_at, last_at, cwd, model, title, slug,
              parent, forks, msg_count, git_branch, git_commit, git_repo,
              originator, cli_version, tokens, cost_usd, is_sidechain, agent_id,
              canonical_source }

Message { role: User|Assistant|System|Tool, ts, content: Vec<Block>, model, native_id,
          parent_native_id, request_id, usage }

Block { Text, Thinking, ToolUse{call: ToolCall, native_id},
        ToolResult{native_id, value, is_error}, Image }

ToolCall { Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch,
           ApplyPatch, AgentSpawn, Mcp, Unknown }
```

`ToolCall::Unknown { provider, raw_name, raw_args }` preserves original tool name + args
when no canonical mapping exists — used for future provider-specific tools.

## FLOW

```
                                  ┌──────────────────┐
                  monocli list ──→│  enumerate all   │
                                  │  registered      │
                                  │  adapters        │
                                  └────────┬─────────┘
                                           │
                  ┌────────────────────────┼─────────────────────┐
                  │                        │                     │
            ClaudeAdapter             CodexAdapter         Grok/OpenCode/Agy
            ~/.claude/projects        ~/.codex/sessions          │
                  │                        │                     │
            decode dir+content      decode payload events        │
                  │                        │                     │
                  └────────────────────────┼─────────────────────┘
                                           ↓
                              canonical Vec<SessionMeta>
                                           ↓
                          sort by last_at, apply --limit
                                           ↓
                                     render table
```

## UNIVERSAL NATIVE TRANSCODE

`monocli translate <id> --to <provider> --native` writes a real target
session into the target CLI's own store. Claude, Codex, Grok, OpenCode, and
Agy/Antigravity preserve every canonical message field, including empty messages,
roles, timestamps, native IDs, model/request metadata, usage, thinking, native tool
provenance, tool results, and images. Each provider also carries an immutable exact
snapshot of the first source `SessionMeta`. The resumed target keeps its own active
session ID/model/cwd, while repeated A→B→C translations retain the original snapshot
without nesting intermediate sessions.

Every translation is reparsed through the target adapter and checked against the source.
Native-visible records and the exact canonical messages/source metadata must both pass
before a native write; the installed target session is audited again afterward.
Image bytes from a local or temporary path are embedded during transcode; the target
session does not retain that path. A missing path aborts the whole write instead of
dropping the image.

Grok stores a semantic snapshot of the emitted native prefix beside the exact canonical
sidecar. After Grok resumes, monocli appends only turns that follow a semantically identical JSON
prefix; a rewritten or shortened prefix fails closed instead of being joined by message
count. OpenCode's temporary import export is removed after every native import attempt, and all
native writes remove their audited dry-run preflight output before touching the target store.

Translation has no fidelity levels and no text fallback. It consumes the complete session; use
`monocli export` when an intentionally textual or truncated artifact is required.

Agy/Antigravity uses `agy --print` to mint native `.pb/.db` state, hydrates the exact
canonical mirror, and writes byte-identical image files into that conversation's
`.user_uploaded` directory. Mirror attachment records link every image block to its
durable file. monocli does not fabricate opaque native state or replace an image with
a placeholder. URL-only images fail closed if their bytes cannot be materialized.

## INSTALL / VERIFY

The installed OpenCLIs binary is normally:

```
~/.openclis/bin/monocli
```

Do not assume source and installed binaries match just because `monocli version`
prints the same version. Verify the installed binary directly:

```
command -v monocli
monocli version
monocli list --help
monocli providers --help
monocli test auto
monocli list --provider claude --limit 1
monocli list --provider codex --limit 1
monocli list --provider grok --limit 1
monocli list --provider opencode --limit 1
monocli list --provider agy --limit 1
```

Current expected installed help behavior: every command accepts `--help`,
including `list`, `show`, `read`, `cat`, `tree`, `providers`, `tui`, `test`,
and `title`.

## WINDOWS / LINUX

- Linux: supported; `monocli test auto` uses Unix PTY APIs and should pass.
- Windows: supported when built with the Windows SQLite toolchain. The TUI uses
  crossterm. The built-in `monocli test` PTY harness is Unix-only and is not the
  Windows support boundary.
- Paths remain home-relative for Claude, Codex, Grok, Agy, and the monocli
  sidecar. OpenCode uses `~/.local/share/opencode/opencode.db` first and then
  platform data-dir fallbacks such as `%LOCALAPPDATA%\opencode`.

`[NEXT] monocli show <id>   monocli read <id>   monocli providers`

---

Built on `lib-monosession`, the canonical AI-session model behind the
Monolex AI terminal.

A **[Monolex AI](https://monolex.ai)** project.
