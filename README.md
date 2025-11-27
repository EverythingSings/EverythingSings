# 🌀 Hi, I'm @EverythingSings

Building **Sovereign Composable Tools** for intentional information sovereignty.

🌐 [EverythingSings.Art](http://EverythingSings.Art)

## The Vision

**Reclaiming agency from algorithms and platforms - for humans AND agents.**

Modern platforms have stolen our attention and creative output:
- Consumption feeds driven by engagement algorithms, not intention
- Creative work trapped behind platform walls and ad-driven feeds
- Your audience, your content, your data - all held hostage

I'm building the **anti-algorithm stack** - tools that restore agency over both what you consume and what you create:

- **INPUT:** Subscribe to sources intentionally, not algorithmically
- **OUTPUT:** Publish to your chosen audiences, not platform silos
- **SOVEREIGNTY:** Own your data, keys, and execution

In 1997, "The Sovereign Individual" predicted that cryptography would enable individuals to escape institutional control in the information age. These tools implement that vision - cryptographic identities (Nostr), sovereign money (Bitcoin/Lightning), and composable primitives that make platforms unnecessary.

**These tools serve both humans seeking agency AND agents needing composable primitives.** The same interface works for both - because the future isn't humans OR agents, it's humans WITH agents.

## Philosophy: The Five Axioms

These tools follow five core principles that respect user sovereignty and embrace Unix philosophy:

1. **Sovereignty** - Users own their data, keys, and execution (no platform lock-in)
2. **Composability** - Tools compose via text streams using structured I/O
3. **Locality** - Offline-first, local storage is the source of truth
4. **Portability** - Platform agnostic
5. **Openness** - Only decentralized, open protocols (no corporate APIs)

**These tools don't lock you into an ecosystem - not even ours.** Each tool is independently useful and composes with anything. Standard formats (JSON, SQLite, OPML), standard protocols (Nostr, RSS), standard tools (Unix pipes, rsync, git). You can use one SCT tool without any others, and escape the ecosystem anytime.

See [SCT specification](/docs/sct-spec.md) for complete architectural details.

## Current Projects

> **Note:** These projects will be open sourced once they reach stable releases.

### 🌐 Plurcast - The OUTPUT Layer
**Cast to many** - Intentional creation instead of platform capture

Post to Nostr, Mastodon, and SSB from the command line. Create once, reach your chosen audiences. No algorithms deciding who sees your work. No ads. No engagement farming. Just intentional distribution to decentralized platforms.

**Features:**
- Multi-platform posting (Nostr, Mastodon, experimental SSB)
- Multi-account support with OS keyring
- Local SQLite database
- Agent-friendly JSON output
- Unix-composable (stdin/stdout, semantic exit codes)

### 📡 sct-rss - The INPUT Layer
**Intentional consumption for humans, token-efficient primitives for agents**

Subscribe to the sources you choose - blogs, podcasts, news sites - without algorithms, ads, or surveillance. RSS is orders of magnitude more token-efficient than HTML parsing (~2K tokens vs 50K+), enabling multi-agent systems to monitor information efficiently.

Pure WASM parser that compiles to <1MB, runs anywhere, and composes beautifully with Unix tools.

**Features:**
- Pure WASM parser (2.0MB optimized)
- All formats (RSS 0.91/1.0/2.0, Atom, JSON Feed)
- Composable commands (parse, filter, merge, normalize, validate)
- Multiple output formats (JSON, JSONL, CSV)
- HTTP-agnostic core (accepts stdin for max composability)

### 🧪 sct-npow (Experimental)
**Nostr Proof-of-Work Reader** - The INPUT Layer (Decentralized)

Read Nostr by computational commitment, not algorithms. Filter the decentralized firehose using NIP-13 Proof of Work as a quality primitive. High-PoW events signal intentional, committed content - no social graphs needed, no central moderation, just verifiable computational work.

**Experimental Features:**
- PoW-based content filtering (NIP-13)
- Multi-relay event fetching
- Local SQLite caching
- WASM-first (targeting wasm32-wasip2)

See [sct-npow spec](/docs/sct-npow-spec.md) for full design.

### 🔮 sct-vec (Planned Q1 2026)
**Vector search for the decentralized web** - The INTELLIGENCE Layer

Local-first semantic search across all your intentional feeds. RSS archives, Nostr commentary, historical context - all semantically searchable, all local.

**Vision:**
- Local-first semantic search using sqlite-vec
- WASM-portable, offline-capable
- Agent-friendly RAG support
- Protocol-agnostic (works with any content source)

See [usage examples](/docs/examples.md) for composition patterns.

## Why This Matters

"The Sovereign Individual" (Davidson & Rees-Mogg, 1997) predicted this transition - written before Bitcoin, before platform capitalism, before AI agents. They foresaw the information age would enable individual sovereignty through cryptography. **The technology arrived. The platforms captured it. These tools reclaim it.**

**For Humans:**
- ✊ Intentional consumption - Subscribe to what matters, not what's trending
- ✊ Sovereign creation - Own your audience, not rent it from platforms
- ✊ Local-first data - Your information lives on your machine
- ✊ Composable workflows - Build your own tools, don't wait for features

**For AI Agents:**
- 🤖 Token-efficient primitives - RSS vs HTML parsing saves 25x on tokens
- 🤖 Testable during development - Agents can verify their own code works
- 🤖 Multi-agent orchestration - Compose tools for emergent behaviors
- 🤖 Human-agent collaboration - Same interface, different users

Traditional tools force a choice: consumer apps for humans, or APIs for agents. These tools serve both, because **the future isn't humans OR agents - it's humans WITH agents**, using the same sovereign, composable primitives.

## Documentation

- [SCT Specification](/docs/sct-spec.md) - Complete architectural specification
- [Usage Examples](/docs/examples.md) - Composition patterns and workflows
- [Why Rust?](/docs/why-rust.md) - Language choice and agentic development
- [sct-npow Spec](/docs/sct-npow-spec.md) - Nostr PoW reader design

## Tech Stack

- **Language:** Rust ([why?](/docs/why-rust.md))
- **Storage:** SQLite / sqlite-vec (when persistence needed)
- **Philosophy:** Unix philosophy + Local-first software + Decentralized protocols
- **Target Platforms:** Linux, macOS, Windows (native binaries)

## Connect

- 🌐 Website: [EverythingSings.Art](http://EverythingSings.Art)
- 💬 Interested in decentralized tools, WASM, and agentic workflows
- 📫 Open to collaboration on SCT-aligned projects
- ₿ Value-for-value: bc1qskq3l4gy73ew78hye8uq6yfkkp5cxswducmvkamw2vxun7wms04qneezd5 (Bitcoin)
- ⚡ Value-for-value: EverythingSings@primal.net (Lightning)

---

*Building composable primitives for intentional information sovereignty.*

*Your attention. Your audience. Your data.*
