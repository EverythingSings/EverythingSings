# sct-npow: Nostr Proof-of-Work Reader

**Status:** 🧪 **EXPERIMENTAL** - Design phase, not yet implemented

Read Nostr by computational commitment, not algorithms.

## What is this?

`sct-npow` is a command-line Nostr client focused on consuming high-quality content through NIP-13 Proof-of-Work filtering. Instead of relying on social graphs or algorithmic feeds, it uses computational commitment as a quality signal.

**Core idea:** High-PoW events = Serious content

## Quick Example

```bash
# Fetch high-PoW events from Nostr
sct-npow fetch --relay wss://relay.damus.io --min-pow 20 --since 24h

# Pipe to vector search
sct-npow fetch --min-pow 20 | sct-vec index --collection nostr

# Merge with RSS for unified feed
(rss-wasm parse feeds.opml && sct-npow fetch --min-pow 20) | \
  jq -s 'sort_by(.created_at)'
```

## Documentation

- **Full Specification:** [/docs/sct-npow-spec.md](/docs/sct-npow-spec.md)
- **Integration Design:** [research-integration.md](./research-integration.md)
- **Library Research:** [research-libraries.md](./research-libraries.md)

## Current Status

**Phase: Research & Design**

- ✅ Research rust-nostr library for PoW support
- ✅ Design 4-layer SCT architecture
- ✅ Define CLI interface and integration patterns
- ✅ Document experimental specification
- ⏳ Verify wasm32-wasip2 compatibility
- ⏳ Build minimal PoW verification prototype
- ⏳ Implement core + CLI layers
- ⏳ Add network layer (relay client)
- ⏳ Add storage layer (SQLite caching)

## Why PoW?

On open, permissionless protocols like Nostr:
- **Anyone can publish anything** (good for freedom)
- **Spam has no cost** (bad for quality)
- **Social graphs help** (but require network effects)

Proof of Work adds:
- ✅ **Computational cost** to publishing
- ✅ **Quality signal** (high-PoW = commitment)
- ✅ **Objective filtering** (verifiable, no reputation needed)
- ✅ **Anti-spam primitive** (no central moderation)

## Architecture

Following the SCT 4-layer specification:

1. **Layer 1 (Core):** Pure WASM - PoW verification, event parsing
2. **Layer 2 (Protocol):** Optional network - Relay connections
3. **Layer 3 (Storage):** Optional persistence - SQLite caching
4. **Layer 4 (Application):** CLI - Unix-composable interface

**Target:** `wasm32-wasip2` (WebAssembly Component Model)

## Integration

Completes the SCT ecosystem:

```
INPUT          INTELLIGENCE      OUTPUT
────────       ────────────      ──────────
rss-wasm   →   sct-vec      →   plurcast
sct-npow   →                →
```

**Examples:**
- Index high-PoW Nostr in vector DB
- Amplify quality content cross-platform
- Merge RSS + Nostr into unified feed
- Multi-agent intelligence workflows

## Tech Stack

- **Language:** Rust
- **WASM Target:** wasm32-wasip2 (Component Model)
- **Nostr Library:** rust-nostr (evaluating)
- **Storage:** SQLite (shared with other SCT tools)
- **Output:** JSON/JSONL for composition

## Next Steps

1. **Verify compatibility:** Test rust-nostr with wasm32-wasip2
2. **Build prototype:** Core PoW verification (Layer 1)
3. **Implement CLI:** Basic command interface (Layer 4)
4. **Add networking:** Relay client (Layer 2)
5. **Add storage:** SQLite integration (Layer 3)

## Contributing

Not yet accepting contributions - still in experimental design phase.

Once the implementation stabilizes, this will be open sourced following the SCT specification.

## Related Projects

Part of the **Everything Sings** ecosystem:

- **rss-wasm** - RSS/Atom/JSON feed parser (INPUT)
- **plurcast** - Multi-platform publisher (OUTPUT)
- **sct-vec** - Vector search (INTELLIGENCE) - Planned Q1 2026

## License

TBD (will be open source)

---

**Built with the SCT philosophy:** Sovereignty, Composability, Locality, Portability, Openness.

Read Nostr by computational commitment, not algorithms.
