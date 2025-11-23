# Introducing Sovereign Composable Tools: Reclaiming Agency in the Information Age

*Building the anti-algorithm stack - for humans AND agents*

## The Platform Trap

You check your feed. Not because you want to, but because the algorithm trained you to.

Your creative work sits behind someone else's algorithm, shown to your audience only if it maximizes their engagement metrics. Your data, your keys, your identity - all rented from platforms that can revoke access at any time.

And if you're building AI agents? They're forced to scrape brittle HTML, reverse-engineer APIs, or pay for expensive platform access that can be cut off whenever the terms change.

**This isn't sustainable. It's not even desirable.**

## A Different Future

In 1997, "The Sovereign Individual" predicted that cryptography would enable individuals to escape institutional control in the information age. The technology arrived exactly as predicted - cryptographic identities, sovereign money, composable protocols.

But then the platforms captured it.

Decentralized protocols exist (Nostr, RSS, Bitcoin), but the tools to use them are either consumer apps that hide the primitives, or developer APIs that exclude non-programmers. We need something different.

**Sovereign Composable Tools (SCT)** implements the vision the book predicted, with one critical extension: these tools serve both humans seeking agency AND agents needing composable primitives. The same interface. The same sovereignty.

Because the future isn't humans OR agents - it's humans WITH agents.

## The Five Axioms

SCT tools follow five core principles:

1. **Sovereignty** - Users own their data, keys, and execution (no platform lock-in)
2. **Composability** - Tools compose via text streams using structured I/O
3. **Locality** - Offline-first, local storage is the source of truth
4. **Portability** - Deploy anywhere: WASM-capable, cross-platform
5. **Openness** - Only decentralized, open protocols (no corporate APIs)

These aren't abstract principles. They're enforced by architecture: pure WASM cores, Unix stdin/stdout, local-first SQLite, and zero corporate API dependencies.

## The Tools

### Plurcast - The OUTPUT Layer

**Cast to many.** Post to Nostr, Mastodon, and SSB from the command line. No algorithms deciding who sees your work. No ads. No engagement farming. Just intentional distribution to your chosen audiences.

```bash
# Your words, your platforms, your rules
echo "Building tools for sovereignty" | plur-post --platform nostr,mastodon

# Compose with any Unix tool
cat weekly-update.md | sed 's/draft/final/g' | plur-post
```

Your audience isn't held hostage. Your posts are stored locally in SQLite. The keys are in your OS keyring. Everything composes.

### rss-wasm - The INPUT Layer

**Intentional consumption.** Subscribe to the sources you choose - blogs, podcasts, news sites - without algorithms, ads, or surveillance.

Why RSS? Because it's **token-efficient for agents**. Monitoring AI announcements via RSS costs ~2K tokens vs 50K+ for HTML scraping. This enables multi-agent systems to stay informed while minimizing costs.

```bash
# Parse any feed format
curl https://blog.anthropic.com/feed.xml | rss-wasm parse

# Filter and query
rss-wasm parse feed.xml | rss-wasm filter --since=24h | jq '.items[].title'

# Multi-agent workflows
rss-wasm parse openai-feed.xml | agent analyze-impact | agent draft-response
```

Pure WASM (2MB), runs anywhere, composes beautifully. Built for humans reading intentionally AND agents monitoring efficiently.

### sct-npow - The DECENTRALIZED INPUT Layer (Experimental)

**Read Nostr by computational commitment, not algorithms.** Filter the decentralized firehose using proof-of-work as a quality signal.

On open protocols like Nostr, anyone can publish anything. Proof-of-work adds computational cost - high-PoW events signal intentional, committed content. No social graphs needed, no central moderation, just verifiable work.

```bash
# Fetch high-quality content from Nostr
sct-npow fetch --relay wss://relay.damus.io --min-pow 20 --since 24h

# Merge with RSS for unified intentional feed
(rss-wasm parse feeds.opml && sct-npow fetch --min-pow 20) | \
  jq -s 'sort_by(.created_at) | reverse'

# Amplify quality content cross-platform
sct-npow fetch --min-pow 25 --limit 1 | plurcast publish --platform mastodon
```

Experimental, WASM-first, composable with everything.

### sct-vec - The INTELLIGENCE Layer (Planned Q1 2026)

**Local-first vector search** for semantic querying across all your intentional feeds. RSS archives, Nostr commentary, historical context - all semantically searchable, all local-first.

```bash
# Monitor AI announcements and draft responses
(rss-wasm parse openai-feed.xml && sct-npow fetch --min-pow 20) | \
  sct-vec index --collection=ai-news

# Query historical context
sct-vec query "previous model releases" --k=20 | \
  agent analyze-trends | \
  agent draft-commentary | \
  plurcast publish
```

Complete the stack: INPUT (rss-wasm + sct-npow) + OUTPUT (Plurcast) + INTELLIGENCE (sct-vec).

## Why This Matters

**For Humans:**
- ✊ **Intentional consumption** - Subscribe to what matters, not what's trending
- ✊ **Sovereign creation** - Own your audience, not rent it from platforms
- ✊ **Local-first data** - Your information lives on your machine
- ✊ **Composable workflows** - Build your own tools, don't wait for features

**For AI Agents:**
- 🤖 **Token-efficient primitives** - RSS vs HTML saves 25x on tokens
- 🤖 **Testable during development** - Agents can verify their own code works
- 🤖 **Multi-agent orchestration** - Compose tools for emergent behaviors
- 🤖 **Human-agent collaboration** - Same interface, different users

Traditional tools force a choice: consumer apps for humans, or APIs for agents. SCT tools serve both, using the same sovereign, composable primitives.

## The Deeper Vision

RSS and Unix philosophy were designed for machine-readable, composable data flows. With LLMs and AI agents, we can finally complete this vision - tools that compose seamlessly with both humans and machines.

"The Sovereign Individual" focused on empowered humans. These tools extend that to human-agent collaboration. The book described the destination. These tools build the roads.

## What's Next

**Current Status:**
- ✅ Plurcast - Multi-platform posting (Nostr, Mastodon, experimental SSB)
- ✅ rss-wasm - Pure WASM RSS/Atom/JSON Feed parser (57 tests passing)
- 🧪 sct-npow - Nostr PoW reader (experimental, WASM-targeted)
- 🔮 sct-vec - Local vector search (planned Q1 2026)

**Future Plans:**
- Open source stable releases
- Complete WASM portability (wasm32-wasip2)
- Community-driven protocol support
- Documentation and examples for agentic workflows

## Join the Movement

This isn't about building "better platforms." It's about building **composable primitives** that make platforms unnecessary.

Your attention. Your audience. Your data.

---

**Connect:**
- 🌐 [EverythingSings.Art](http://EverythingSings.Art)
- 💬 Follow for updates on decentralized tools, WASM, and agentic workflows
- 📫 Open to collaboration on SCT-aligned projects
- ₿ Support: bc1qskq3l4gy73ew78hye8uq6yfkkp5cxswducmvkamw2vxun7wms04qneezd5
- ⚡ Lightning: EverythingSings@primal.net

---

*Building composable primitives for intentional information sovereignty.*

*The technology arrived. The platforms captured it. These tools reclaim it.*
