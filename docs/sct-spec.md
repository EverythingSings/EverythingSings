# Sovereign Composable Tools (SCT) Specification

The complete architectural specification for SCT tools.

## The Five Axioms

All SCT tools adhere to five core principles:

1. **Sovereignty** - Users own their data, keys, and execution (no platform lock-in)
2. **Composability** - Tools compose via text streams using structured I/O
3. **Locality** - Offline-first, local storage is the source of truth
4. **Portability** - Deploy anywhere: WASM-capable, cross-platform
5. **Openness** - Only decentralized, open protocols (no corporate APIs)

## Four-Layer Architecture

All tools follow this layered specification:

### Layer 1 (Core)
**Pure WASM, zero I/O, maximum portability**

- Pure computation, no side effects
- WASM-first design (targeting wasm32-wasip2)
- No network, no filesystem access
- Testable in isolation
- Examples: RSS parsing logic, PoW verification, vector operations

### Layer 2 (Protocol)
**Optional network (feature-flagged)**

- Decentralized protocol implementations
- Feature-gated network access
- Must work without network (using cached/local data)
- Examples: Nostr relay communication, RSS feed fetching, Mastodon API

### Layer 3 (Storage)
**Optional persistence (feature-flagged)**

- Local-first SQLite storage
- Feature-gated persistence
- Must work without persistence (streaming only)
- Standard schemas, queryable by any tool
- Examples: Post history, feed cache, event storage

### Layer 4 (Application)
**CLI with stdin/stdout interface**

- Unix philosophy: do one thing well
- Composable via pipes
- Structured output (JSON/JSONL/CSV)
- Semantic exit codes
- Examples: `plur-post`, `rss-wasm parse`, `sct-npow fetch`

## Interface Requirements

### Input
- Accept data from stdin when possible
- Support file arguments when necessary
- Never require interactive input (fully scriptable)

### Output
- Primary output to stdout
- Errors and logs to stderr
- Structured formats (JSON, JSONL, CSV)
- Human-readable when piped to terminal (detect TTY)

### Exit Codes
- 0: Success
- 1: General error
- 2: Invalid input
- 3: Network error (if network-enabled)
- 4: Storage error (if persistence-enabled)

## Data Sovereignty

### Local Storage
- SQLite for structured data
- Local files for large/binary content
- Standard locations: `~/.local/share/sct/`
- User-queryable, user-exportable

### Cryptographic Keys
- OS keyring integration (optional)
- File-based keys (always supported)
- User controls key material
- Never upload keys

### Data Export
Every tool must provide:
- Full data export in standard formats
- Import from standard formats
- Direct SQLite access (documented schemas)
- No proprietary lock-in

## Composability Patterns

### Pipe-Friendly Design
```bash
# Single tool
curl feed.xml | rss-wasm parse

# Tool chains
rss-wasm parse feed.xml | jq '.items[0]' | plur-post

# Multi-source
(rss-wasm parse a.xml && rss-wasm parse b.xml) | jq -s 'flatten'
```

### Standard Formats
- JSON: Primary structured format
- JSONL: Streaming line-delimited JSON
- CSV: Tabular data export
- OPML: Feed lists (RSS standard)

### Agent-Friendly
- Deterministic output
- Machine-readable errors
- Token-efficient formats
- Testable during development

## Anti-Ecosystem Principle

**Each tool is independently useful and composes with anything.**

- Standard formats (JSON, SQLite, OPML)
- Standard protocols (Nostr, RSS, Mastodon)
- Standard tools (Unix pipes, rsync, git)
- No SCT-specific dependencies between tools
- Can use one tool without any others

Users can:
- Backup with standard tools (rsync, borg, restic)
- Query with standard tools (sqlite3, jq)
- Sync with standard tools (Syncthing, git)
- Escape the ecosystem anytime

## Tech Stack

- **Language:** Rust (see [why-rust.md](why-rust.md))
- **Storage:** SQLite / sqlite-vec (when persistence needed)
- **Protocols:** Nostr, RSS/Atom, Mastodon, SSB
- **Target Platforms:** Linux, macOS, Windows (native binaries)
- **Future:** WASM components (wasm32-wasip2)

## Design Philosophy

Built on the intersection of:
- **Unix philosophy** - Do one thing well, compose via text streams
- **Local-first software** - Offline-capable, user owns data
- **Decentralized protocols** - No corporate gatekeepers
- **Human-agent collaboration** - Same tools for both

Inspired by "The Sovereign Individual" (Davidson & Rees-Mogg, 1997) - implementing cryptographic sovereignty for the information age.
