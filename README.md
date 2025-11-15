# 👋 Hi, I'm @EverythingSings

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

**These tools serve both humans seeking agency AND agents needing composable primitives.** The same interface works for both - because the future isn't humans OR agents, it's humans WITH agents.

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
**Intentional consumption for humans, token-efficient primitives for agents**

Subscribe to the sources you choose - blogs, podcasts, news sites - without algorithms, ads, or surveillance. Built with Rust and WebAssembly for both human consumption and autonomous agent workflows.

**Why RSS for agents?** RSS is orders of magnitude more token-efficient than HTML parsing. Monitoring OpenAI/Anthropic announcements via RSS uses ~2K tokens vs 50K+ for HTML scraping. This enables multi-agent systems to respond to news quickly while minimizing costs.

Pure WASM parser that compiles to < 1MB, runs anywhere, and composes beautifully with Unix tools.

```bash
# Parse any feed format (human or agent)
curl https://blog.com/feed.xml | rss-wasm parse

# Filter and query
rss-wasm parse feed.xml | rss-wasm filter --since=24h | jq '.items[].title'

# Multi-agent workflows
rss-wasm parse openai-feed.xml | agent analyze-impact | agent draft-response

# Archive for historical context
rss-wasm parse feed.xml | vectordb index --collection=ai-news
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
# Monitor and respond to AI announcements (multi-agent workflow)
rss-wasm parse openai-feed.xml anthropic-feed.xml | \
  rss-wasm filter --since=1h | \
  vectordb index --collection=ai-news

# Query historical context and draft response
vectordb query "previous model releases" --collection=ai-news | \
  agent analyze-trends | \
  agent draft-commentary | \
  plurcast publish --platform nostr,mastodon

# Semantic search across all archived content
echo "rust wasm performance" | vectordb query --collection=feeds --k=10
```

**Vision:**
- Local-first semantic search using sqlite-vec
- WASM-portable, offline-capable
- Agent-friendly RAG support
- Protocol-agnostic (works with any content source)

## Why This Matters

**You should control your attention and your audience, not algorithms. And your agents need composable primitives, not brittle integrations.**

Every platform wants to:
- Control what you see (algorithmic feeds maximize engagement, not value)
- Control who sees you (your audience is their hostage)
- Monetize your attention (ads, tracking, surveillance capitalism)
- Force agents to scrape, hack, or use expensive APIs they control

This isn't sustainable. It's not even desirable.

**These tools restore agency for humans:**
- ✊ **Intentional consumption** - Subscribe to what matters, not what's trending
- ✊ **Sovereign creation** - Own your audience, not rent it from platforms
- ✊ **Local-first data** - Your information lives on your machine
- ✊ **Composable workflows** - Build your own tools, don't wait for features

**And provide foundations for agents:**
- 🤖 **Token-efficient primitives** - RSS vs HTML parsing saves 25x on tokens
- 🤖 **Testable during development** - Agents can verify their own code works
- 🤖 **Multi-agent orchestration** - Compose tools for emergent behaviors
- 🤖 **Human-agent collaboration** - Same interface, different users

**The deeper vision:** RSS and Unix philosophy were designed for machine-readable, composable data flows. With LLMs and AI agents, we can finally complete this vision - tools that compose seamlessly with both humans and machines.

Traditional tools force a choice: consumer apps for humans, or APIs for agents. These tools serve both, because **the future isn't humans OR agents - it's humans WITH agents**, using the same sovereign, composable primitives.

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
