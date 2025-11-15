# sct-npow: Nostr Proof-of-Work Reader

**Status:** 🧪 EXPERIMENTAL
**Version:** 0.1.0-alpha
**Layer:** INPUT
**Repository:** TBD

---

## Overview

`sct-npow` (Sovereign Composable Tools - Nostr Proof-of-Work) is a command-line Nostr client focused on consuming high-quality content through computational proof-of-work filtering (NIP-13).

Unlike general-purpose Nostr clients, `sct-npow` treats **Proof of Work as a quality primitive** - filtering the firehose of decentralized content by computational commitment rather than social graphs or algorithms.

### Design Philosophy

Following the SCT specification:
- **Sovereignty:** Read Nostr without platform intermediaries
- **Composability:** Unix pipes, JSON I/O, works with other SCT tools
- **Locality:** Local-first caching, offline-capable
- **Portability:** WASM-first (wasm32-wasip2), runs anywhere
- **Openness:** Decentralized protocol, no corporate APIs

---

## Why sct-npow?

### The Problem

Nostr is **open and permissionless** - anyone can publish anything. This is powerful but creates noise:
- Spam has no cost (publish to any relay)
- Quality signals require social graphs (who you follow)
- No computational commitment to filter by

### The Solution

**Proof of Work as a quality filter:**
- PoW adds **computational cost** to publishing
- High-PoW events signal **intentional, committed content**
- PoW is **objective and verifiable** (no reputation needed)
- Works for **both humans and agents**

### Use Cases

**For Humans:**
- Read high-quality Nostr content without building social graph
- Filter spam without centralized moderation
- Discover committed creators (high-PoW = serious content)

**For Agents:**
- Token-efficient content consumption (filter before LLM processing)
- Objective quality metric (PoW difficulty is verifiable)
- Compose with other tools (pipe to sct-vec, plurcast)

---

## Architecture

Following the 4-layer SCT specification:

### Layer 1: Core (Pure WASM, Zero I/O)

**Purpose:** Protocol logic, PoW verification, event parsing

```rust
// Core functionality (no I/O)
pub struct PowFilter {
    min_difficulty: u8,
}

impl PowFilter {
    pub fn verify_pow(&self, event: &Event) -> bool {
        count_leading_zeros(&event.id) >= self.min_difficulty
    }
}

pub fn count_leading_zeros(id: &EventId) -> u8 {
    // NIP-13: Count leading zero bits in event ID
    // Pure computation, no I/O
}
```

**Features:**
- Event parsing (NIP-01)
- PoW verification (NIP-13)
- Filter construction (NIP-01)
- Signature validation
- no_std compatible

### Layer 2: Protocol (Optional Network)

**Purpose:** Relay connections, event fetching

**Feature flag:** `network`

```rust
#[cfg(feature = "network")]
pub struct RelayClient {
    url: String,
    // WebSocket connection (WASI-compatible)
}

impl RelayClient {
    pub async fn fetch_events(&self, filter: Filter) -> Vec<Event> {
        // Connect to relay
        // Send REQ message
        // Receive EVENTS
        // Close connection
    }
}
```

**WASI 0.2 sockets:**
- Uses WASI socket APIs (not browser WebSocket)
- Compatible with wasm32-wasip2 target
- Offline-first (optional feature)

### Layer 3: Storage (Optional Persistence)

**Purpose:** Local caching, relay management

**Feature flag:** `storage`

**Database:** SQLite (shared with other SCT tools)

**Schema:**
```sql
-- Events table
CREATE TABLE nostr_events (
  id TEXT PRIMARY KEY,           -- Event ID (hex)
  pubkey TEXT NOT NULL,          -- Author pubkey
  created_at INTEGER NOT NULL,   -- Unix timestamp
  kind INTEGER NOT NULL,         -- Event kind (NIP-01)
  content TEXT NOT NULL,         -- Event content
  pow_difficulty INTEGER,        -- Leading zero bits
  tags JSON,                     -- Event tags
  sig TEXT NOT NULL,             -- Event signature
  fetched_at INTEGER,            -- Local cache timestamp
  relay TEXT,                    -- Source relay
  verified BOOLEAN DEFAULT 0     -- PoW verified flag
);

CREATE INDEX idx_pow ON nostr_events(pow_difficulty DESC);
CREATE INDEX idx_created ON nostr_events(created_at DESC);
CREATE INDEX idx_author ON nostr_events(pubkey);
CREATE INDEX idx_kind ON nostr_events(kind);

-- Relay tracking
CREATE TABLE relays (
  url TEXT PRIMARY KEY,
  last_connected INTEGER,
  status TEXT CHECK(status IN ('active', 'error', 'disabled')),
  error_count INTEGER DEFAULT 0,
  metadata JSON
);

-- Author tracking (optional)
CREATE TABLE authors (
  pubkey TEXT PRIMARY KEY,
  name TEXT,
  about TEXT,
  avg_pow REAL,               -- Average PoW of their events
  event_count INTEGER,
  last_seen INTEGER
);
```

**Location:** `~/.local/share/sct/sct.db` (shared with rss-wasm, plurcast)

### Layer 4: Application (CLI)

**Purpose:** User interface, composition with other tools

**Binary:** `sct-npow`

**Commands:**
```bash
sct-npow fetch      # Fetch events from relays
sct-npow verify     # Verify PoW on events (from stdin or storage)
sct-npow subscribe  # Real-time event subscription
sct-npow relays     # Manage relay list
sct-npow authors    # Track high-PoW authors
sct-npow stats      # Show PoW statistics
```

---

## CLI Interface

### Command: `fetch`

Fetch events from Nostr relays with PoW filtering.

```bash
sct-npow fetch [OPTIONS]

Options:
  --relay <URL>              Relay to fetch from (repeatable)
  --min-pow <DIFFICULTY>     Minimum PoW difficulty (default: 0)
  --max-pow <DIFFICULTY>     Maximum PoW difficulty (optional)
  --kind <KINDS>             Event kinds, comma-separated (default: 1)
  --author <PUBKEY>          Filter by author pubkey (repeatable)
  --since <TIME>             Since timestamp or duration (e.g., "24h", "2d")
  --until <TIME>             Until timestamp or duration
  --search <TEXT>            Search in event content (optional)
  --limit <N>                Maximum events to return (default: 100)
  --output <FORMAT>          Output format: json|jsonl|text|markdown (default: jsonl)
  --cache                    Use local cache (requires storage feature)
  --verify-all               Verify PoW on all events (slower)

Examples:
  # Fetch high-PoW text notes from last 24h
  sct-npow fetch --relay wss://relay.damus.io --min-pow 20 --since 24h

  # Fetch from specific author
  sct-npow fetch --author <pubkey> --min-pow 15 --kind 1,30023

  # Multiple relays
  sct-npow fetch \
    --relay wss://relay.damus.io \
    --relay wss://nos.lol \
    --min-pow 20 --limit 50
```

### Command: `verify`

Verify PoW on events (from stdin or storage).

```bash
sct-npow verify [OPTIONS]

Options:
  --min-pow <DIFFICULTY>     Minimum PoW to pass (default: 0)
  --input <FILE>             Input file (default: stdin)
  --output <FORMAT>          Output format (default: jsonl)
  --show-difficulty          Include PoW difficulty in output
  --strict                   Fail on invalid signatures

Examples:
  # Verify events from file
  cat events.jsonl | sct-npow verify --min-pow 20

  # Verify and show difficulty
  sct-npow verify --input events.json --show-difficulty
```

### Command: `subscribe`

Real-time event subscription with PoW filtering.

```bash
sct-npow subscribe [OPTIONS]

Options:
  --relay <URL>              Relay to subscribe to
  --min-pow <DIFFICULTY>     Minimum PoW threshold
  --kind <KINDS>             Event kinds to subscribe to
  --author <PUBKEY>          Filter by author
  --output <FORMAT>          Output format (default: jsonl)

Examples:
  # Subscribe to high-PoW events
  sct-npow subscribe \
    --relay wss://relay.damus.io \
    --min-pow 22 \
    --kind 1

  # Pipe to sct-vec for live indexing
  sct-npow subscribe --min-pow 20 | \
    sct-vec index --collection nostr-live
```

### Command: `relays`

Manage relay list.

```bash
sct-npow relays [SUBCOMMAND]

Subcommands:
  list                       List configured relays
  add <URL>                  Add relay to list
  remove <URL>               Remove relay from list
  test <URL>                 Test relay connection
  stats                      Show relay statistics

Examples:
  # Add relay
  sct-npow relays add wss://relay.snort.social

  # List relays with stats
  sct-npow relays list --with-stats
```

### Command: `authors`

Track high-PoW authors.

```bash
sct-npow authors [SUBCOMMAND]

Subcommands:
  list                       List tracked authors
  show <PUBKEY>              Show author details
  top                        Show authors by average PoW

Examples:
  # Show top PoW authors
  sct-npow authors top --limit 20

  # Show specific author
  sct-npow authors show <pubkey>
```

### Command: `stats`

Show PoW statistics from local cache.

```bash
sct-npow stats [OPTIONS]

Options:
  --by-author                Group by author
  --by-kind                  Group by event kind
  --by-difficulty            Show difficulty distribution
  --since <TIME>             Since timestamp or duration

Examples:
  # Overall PoW statistics
  sct-npow stats

  # Difficulty distribution
  sct-npow stats --by-difficulty
```

---

## Output Format

### JSONL (Default)

One event per line, compatible with jq and other SCT tools:

```jsonl
{"id":"abc123...","pubkey":"def456...","created_at":1731629200,"kind":1,"content":"Event text","pow_difficulty":22,"tags":[],"sig":"789..."}
{"id":"ghi789...","pubkey":"jkl012...","created_at":1731629300,"kind":1,"content":"Another event","pow_difficulty":20,"tags":[],"sig":"345..."}
```

### JSON

Pretty-printed array of events:

```json
[
  {
    "id": "abc123...",
    "pubkey": "def456...",
    "created_at": 1731629200,
    "kind": 1,
    "content": "Event text",
    "pow_difficulty": 22,
    "tags": [],
    "sig": "789..."
  }
]
```

### Text

Human-readable format:

```
[PoW: 22] 2024-11-15 10:00:00 - def456...
Event text

[PoW: 20] 2024-11-15 10:01:40 - jkl012...
Another event
```

### Markdown

Formatted for reading:

```markdown
## Event abc123...
**Author:** def456...
**Created:** 2024-11-15 10:00:00
**PoW Difficulty:** 22

Event text

---

## Event ghi789...
**Author:** jkl012...
**Created:** 2024-11-15 10:01:40
**PoW Difficulty:** 20

Another event
```

---

## Integration Patterns

### With sct-vec (Vector Search)

Index high-PoW events for semantic search:

```bash
# Index Nostr events
sct-npow fetch --min-pow 20 --format jsonl | \
  sct-vec index --collection nostr-pow

# Semantic search
echo "rust wasm component model" | \
  sct-vec query --collection nostr-pow --k 10
```

### With Plurcast (Cross-Platform Publishing)

Amplify high-quality content:

```bash
# Cross-post high-PoW event to Mastodon
sct-npow fetch \
  --author <trusted_pubkey> \
  --min-pow 25 \
  --limit 1 \
  --format text | \
  plurcast publish --platform mastodon
```

### With rss-wasm (Unified Feed)

Merge RSS and Nostr into single timeline:

```bash
# Combined intentional sources
(rss-wasm parse feeds.opml --format jsonl && \
 sct-npow fetch --min-pow 20 --format jsonl) | \
  jq -s 'sort_by(.created_at) | reverse'
```

### Multi-Agent Workflows

Complete intelligence pipeline:

```bash
#!/bin/bash
# Monitor → Index → Analyze → Respond → Publish

# Gather intelligence
(rss-wasm parse ai-feeds.opml && \
 sct-npow fetch --min-pow 20 --search "llm|ai") | \
  sct-vec index --collection ai-intel

# Analyze and respond
sct-vec query "major announcements" --k 20 | \
  agent analyze-trends | \
  agent draft-response | \
  plurcast publish --platform nostr,mastodon
```

---

## Configuration

### Config File Location

`~/.config/sct/sct-npow.toml`

### Example Configuration

```toml
[relays]
# Default relays for fetch/subscribe
default = [
  "wss://relay.damus.io",
  "wss://nos.lol",
  "wss://relay.snort.social"
]

# Relay-specific settings
[relays.custom."wss://relay.damus.io"]
timeout = 10
max_events = 500

[filters]
# Default PoW threshold
min_pow = 15

# Default event kinds
kinds = [1, 30023]  # Text notes + long-form

# Search settings
search_case_sensitive = false

[storage]
# Database path (shared with other SCT tools)
database = "~/.local/share/sct/sct.db"

# Cache settings
cache_enabled = true
retention_days = 90
max_events = 10000

# Auto-cleanup
auto_cleanup = true

[output]
# Default output format
format = "jsonl"

# Include metadata in output
include_metadata = true
include_relay_info = false

# Text output settings
[output.text]
show_difficulty = true
show_timestamp = true
truncate_content = false

[verification]
# Verify all events by default
verify_all = false

# Strict mode (fail on invalid signatures)
strict = false

[network]
# Connection timeout (seconds)
timeout = 10

# Retry settings
max_retries = 3
retry_delay = 2

# User agent
user_agent = "sct-npow/0.1.0"
```

---

## Implementation Details

### Dependencies

**Core (Layer 1):**
- `nostr` - Protocol implementation, PoW verification

**Network (Layer 2):**
- WASI-compatible WebSocket library (TBD - needs verification)
- `async-std` or `tokio` (with WASI support)

**Storage (Layer 3):**
- `rusqlite` - SQLite bindings
- `serde_json` - JSON serialization

**CLI (Layer 4):**
- `clap` - Command-line argument parsing
- `serde` - Serialization framework

### WASM Target

**Primary target:** `wasm32-wasip2`
- WASI 0.2 (Preview 2)
- WebAssembly Component Model
- Requires Rust 1.82+ (Tier 2 support)

**Build command:**
```bash
cargo build --release --target wasm32-wasip2
```

### Feature Flags

```toml
[features]
default = ["network", "storage", "cli"]

# Layer 2: Network access
network = ["dep:async-std", "dep:wasi-sockets"]

# Layer 3: Persistence
storage = ["dep:rusqlite"]

# Layer 4: CLI
cli = ["dep:clap"]

# All features
full = ["network", "storage", "cli"]

# Minimal (just core verification)
minimal = []
```

---

## NIPs Supported

### Implemented

- ✅ **NIP-01:** Basic protocol flow encoding
- ✅ **NIP-13:** Proof of Work

### Planned

- 🔄 **NIP-11:** Relay information document
- 🔄 **NIP-40:** Expiration timestamps
- 🔄 **NIP-65:** Relay list metadata

### Out of Scope (Reader-only tool)

- ❌ **NIP-07:** window.nostr (browser-only)
- ❌ **NIP-46:** Nostr Connect (signing, not reading)
- ❌ **NIP-47:** Wallet Connect (payments)

---

## Testing Strategy

### Unit Tests

Core PoW verification:
```rust
#[test]
fn test_pow_verification() {
    let event = Event::from_json(sample_event_json);
    let filter = PowFilter::new(20);
    assert!(filter.verify_pow(&event));
}

#[test]
fn test_leading_zeros_count() {
    let id = EventId::from_hex("0000abcd...");
    assert_eq!(count_leading_zeros(&id), 16);
}
```

### Integration Tests

End-to-end relay fetching:
```bash
# Test fetch command
sct-npow fetch \
  --relay wss://relay.damus.io \
  --min-pow 15 \
  --limit 10 \
  --output json > /tmp/events.json

# Verify output
cat /tmp/events.json | jq 'length'
cat /tmp/events.json | jq '.[].pow_difficulty | select(. < 15)' | wc -l  # Should be 0
```

### Composition Tests

Test with other SCT tools:
```bash
# Test piping to jq
sct-npow fetch --min-pow 20 | jq '.[] | .content' > /dev/null

# Test piping to sct-vec
sct-npow fetch --min-pow 20 | \
  sct-vec index --collection test --dry-run

# Test piping to plurcast
sct-npow fetch --limit 1 --format text | \
  plurcast publish --platform nostr --dry-run
```

---

## Roadmap

### Phase 1: Core + CLI (Q4 2024)
- ✅ Research rust-nostr library
- ✅ Design architecture
- ⏳ WASM compatibility verification
- ⏳ Core PoW verification (Layer 1)
- ⏳ Basic CLI (Layer 4)
- ⏳ JSON/JSONL output

**Deliverable:** `sct-npow verify` - verify PoW on events from stdin

### Phase 2: Network (Q1 2025)
- ⏳ Relay client (Layer 2)
- ⏳ Event fetching
- ⏳ WebSocket connection management
- ⏳ Multiple relay support

**Deliverable:** `sct-npow fetch` - fetch events from relays

### Phase 3: Storage (Q1 2025)
- ⏳ SQLite integration (Layer 3)
- ⏳ Local caching
- ⏳ Relay management
- ⏳ Author tracking

**Deliverable:** Persistent storage, offline-capable

### Phase 4: Advanced Features (Q2 2025)
- ⏳ Real-time subscriptions
- ⏳ Search functionality
- ⏳ Statistics and analytics
- ⏳ Integration examples with other SCT tools

**Deliverable:** Feature-complete 1.0 release

---

## Success Metrics

### For Humans

- Can read high-quality Nostr content without social graph
- PoW filtering reduces spam effectively
- Easy to discover committed creators
- Composes with other tools (jq, grep, etc.)

### For Agents

- Token-efficient content filtering
- Reliable PoW verification
- Composable with multi-agent workflows
- Low-latency event fetching

### Technical

- WASM binary < 2MB (optimized)
- Fetch 1000+ events/second
- PoW verification < 1ms per event
- Supports 10+ concurrent relay connections

---

## Open Questions

### Technical

1. **WASI compatibility:** Does rust-nostr work with wasm32-wasip2?
   - **Action:** Build prototype to verify

2. **WebSocket library:** Which WASI-compatible WS library?
   - **Candidates:** Research needed

3. **Async runtime:** Tokio vs async-std for WASI?
   - **Action:** Performance testing

### Product

1. **Default PoW threshold:** What's a good default min-pow?
   - **Hypothesis:** 15-20 for quality content
   - **Action:** Empirical testing on live relays

2. **Relay discovery:** Should we auto-discover relays?
   - **Options:** Manual list, NIP-65, relay hints
   - **Decision:** Start with manual, add discovery later

3. **Storage limits:** How many events to cache locally?
   - **Options:** Time-based (90 days), count-based (10k events)
   - **Decision:** Both with config override

---

## Contributing

**Status:** Not yet open source (experimental)

Once stable, we'll welcome contributions for:
- Additional NIPs support
- Performance optimization
- WASM runtime testing
- Integration examples
- Documentation

---

## License

TBD (will be open source upon stable release)

---

## References

- **Nostr Protocol:** https://github.com/nostr-protocol/nips
- **NIP-13 (PoW):** https://github.com/nostr-protocol/nips/blob/master/13.md
- **rust-nostr:** https://github.com/rust-nostr/nostr
- **WASI 0.2:** https://blog.rust-lang.org/2024/11/26/wasip2-tier-2/
- **SCT Philosophy:** /README.md

---

**sct-npow** - Read Nostr by computational commitment, not algorithms.
