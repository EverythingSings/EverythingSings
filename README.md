# 👋 Hi, I'm @EverythingSings

Building **Sovereign Composable Tools** for intentional information sovereignty.

🌐 [EverythingSings.Art](http://EverythingSings.Art)

## The Vision

**Reclaiming agency from algorithms and platforms.**

Modern platforms have stolen our attention and creative output:
- Consumption feeds driven by engagement algorithms, not intention
- Creative work trapped behind platform walls and ad-driven feeds
- Your audience, your content, your data - all held hostage

I'm building the **anti-algorithm stack** - tools that restore agency over both what you consume and what you create:

- **INPUT:** Subscribe to sources intentionally, not algorithmically
- **OUTPUT:** Publish to your chosen audiences, not platform silos
- **SOVEREIGNTY:** Own your data, keys, and execution

This isn't about building "better platforms." It's about building **composable primitives** that make platforms unnecessary.

## Philosophy: The Five Axioms

These tools respect user sovereignty and follow Unix philosophy, guided by five core principles:

1. **Sovereignty** - Users own their data, keys, and execution (no platform lock-in)
2. **Composability** - Tools compose via text streams using structured I/O
3. **Locality** - Offline-first, local storage is the source of truth
4. **Portability** - Deploy anywhere: WASM-capable, cross-platform
5. **Openness** - Only decentralized, open protocols (no corporate APIs)

## Current Projects

> **Note:** These projects will be open sourced once they reach stable releases.

### 🌐 Plurcast - The OUTPUT Layer
**Cast to many** - Intentional creation instead of platform capture

Post to Nostr, Mastodon, and SSB from the command line. Create once, reach your chosen audiences. No algorithms deciding who sees your work. No ads. No engagement farming. Just intentional distribution to decentralized platforms.

Follows Unix philosophy: reads from stdin, outputs to stdout, composes with pipes.

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

### 📡 rss-wasm - The INPUT Layer
**Intentional consumption, not algorithmic feeds**

Subscribe to the sources you choose - blogs, podcasts, news sites - without algorithms, ads, or surveillance. A local-first RSS/Atom reader built with Rust and WebAssembly. Your attention, your rules.

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
**Vector search for the decentralized web** - The INTELLIGENCE Layer

Completing the ecosystem: **INPUT** (rss-wasm) + **OUTPUT** (Plurcast) + **INTELLIGENCE** (sct-vec)

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

### 🌉 Future: Bridging RSS ↔ Nostr
**Completing the circle between protocols**

Exploring a bridge to connect RSS (federated web) with Nostr (decentralized social):
- **RSS → Nostr:** Follow traditional blogs/podcasts on Nostr relays
- **Nostr → RSS:** Read Nostr streams in your RSS reader

This would enable true **protocol sovereignty** - consume and publish on your terms, regardless of the underlying protocol.

## Why This Matters

**You should control your attention and your audience, not algorithms.**

Every platform wants to:
- Control what you see (algorithmic feeds maximize engagement, not value)
- Control who sees you (your audience is their hostage)
- Monetize your attention (ads, tracking, surveillance capitalism)

This isn't sustainable. It's not even desirable.

**These tools restore agency:**
- ✊ **Intentional consumption** - Subscribe to what matters, not what's trending
- ✊ **Sovereign creation** - Own your audience, not rent it from platforms
- ✊ **Composable workflows** - Build your own tools, don't wait for features
- ✊ **Local-first data** - Your information lives on your machine
- ✊ **Agent-friendly** - Let AI work for you, not platforms

**The deeper vision:** RSS and Unix philosophy were designed for machine-readable, composable data flows. With LLMs and AI agents, we can finally complete this vision - tools that compose seamlessly with both humans and machines.

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

*Building composable primitives for intentional information sovereignty.*

*Your attention. Your audience. Your data.*
