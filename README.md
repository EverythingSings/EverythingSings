# 👋 Hi, I'm @EverythingSings

Building **Sovereign Composable Tools** for the decentralized web.

🌐 [EverythingSings.Art](http://EverythingSings.Art)

## Philosophy: The Five Axioms

I'm building tools that respect user sovereignty and follow Unix philosophy, guided by five core principles:

1. **Sovereignty** - Users own their data, keys, and execution (no platform lock-in)
2. **Composability** - Tools compose via text streams using structured I/O
3. **Locality** - Offline-first, local storage is the source of truth
4. **Portability** - Deploy anywhere: WASM-capable, cross-platform
5. **Openness** - Only decentralized, open protocols (no corporate APIs)

## Current Projects

> **Note:** These projects will be open sourced once they reach stable releases.

### 🌐 Plurcast
**Cast to many** - Unix tools for the decentralized social web

Post to Nostr, Mastodon, and SSB from the command line. Follows Unix philosophy: reads from stdin, outputs to stdout, composes with pipes.

```bash
# Post to multiple platforms
echo "Hello decentralized world!" | plur-post

# Pipe composition
cat draft.txt | sed 's/foo/bar/g' | plur-post --platform nostr,mastodon

# Query history with semantic filtering
plur-history --since=7d --format=json | jq '.[] | select(.platform == "nostr")'
```

**Features:**
- ✅ Multi-platform posting (Nostr, Mastodon, experimental SSB)
- ✅ Multi-account support with OS keyring
- ✅ Local SQLite database
- ✅ Agent-friendly JSON output
- ✅ Unix-composable (stdin/stdout, semantic exit codes)

### 📡 rss-wasm
**A local-first, deploy-anywhere RSS/Atom reader** built with Rust and WebAssembly

Pure WASM parser that compiles to < 1MB, runs anywhere, and composes beautifully with Unix tools.

```bash
# Parse any feed format
curl https://blog.com/feed.xml | rss-wasm parse

# Filter and query
rss-wasm parse feed.xml | rss-wasm filter --since=24h | jq '.items[].title'

# Agent-driven workflows
rss-wasm parse feed.xml | rss-wasm filter --since=7d | llm summarize
```

**Features:**
- ✅ Pure WASM parser (2.0MB optimized)
- ✅ All formats (RSS 0.91/1.0/2.0, Atom, JSON Feed)
- ✅ Composable commands (parse, filter, merge, normalize, validate)
- ✅ Multiple output formats (JSON, JSONL, CSV)
- ✅ 57 automated tests passing
- ✅ HTTP-agnostic core (accepts stdin for max composability)

### 🔮 sct-vec (Planned Q1 2026)
**Vector search for the decentralized web** - The intelligence layer

Completing the ecosystem: **Read** (rss-wasm) + **Write** (Plurcast) + **Search** (sct-vec)

```bash
# The complete loop
rss-wasm fetch --feed=blog.xml | vectordb index --collection=feeds
echo "rust wasm tools" | vectordb query --collection=feeds --k=10
vectordb query < prompt.txt | agent process | plurcast publish
```

**Vision:**
- Local-first semantic search using sqlite-vec
- WASM-portable, offline-capable
- Agent-friendly RAG support
- Protocol-agnostic (works with any content source)

## Why This Matters

**Completing an unfinished vision:** RSS and Unix philosophy were designed for machine-readable, composable data flows. With LLMs and AI agents, we can finally realize this vision.

These tools enable:
- 🤖 **Agentic workflows** - LLMs composing with Unix tools
- 🔐 **User sovereignty** - Your data, your keys, your control
- 🌍 **Decentralization** - No corporate platforms, no lock-in
- 📦 **True portability** - One WASM binary, deploy anywhere
- 🔗 **Maximum composability** - Pipe everything, script everything

## The Complete Specification

All tools follow the SCT (Sovereign Composable Tools) specification with:

- **Layer 1 (Core):** Pure WASM, zero I/O, maximum portability
- **Layer 2 (Protocol):** Optional network (feature-flagged)
- **Layer 3 (Storage):** Optional persistence (feature-flagged)
- **Layer 4 (Application):** CLI with stdin/stdout interface

## Tech Stack

- **Language:** Rust (for WASM portability and safety)
- **Target:** wasm32-wasip2 (Component Model)
- **Storage:** SQLite / sqlite-vec (when persistence needed)
- **Philosophy:** Unix philosophy + Local-first software + Decentralized protocols

## Connect

- 🌐 Website: [EverythingSings.Art](http://EverythingSings.Art)
- 💬 Interested in decentralized tools, WASM, and agentic workflows
- 📫 Open to collaboration on SCT-aligned projects

---

*Building the primitives for a sovereign, composable, decentralized web.*
