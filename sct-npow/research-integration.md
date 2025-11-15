# Nostr PoW Reader: Infrastructure Integration

**Date:** 2025-11-15
**Purpose:** Design how the Nostr PoW reader composes with existing SCT infrastructure

---

## The Complete SCT Ecosystem

```
┌─────────────────────────────────────────────────────────────────┐
│                         INPUT LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│  rss-wasm          │  Parse RSS/Atom/JSON feeds                 │
│  nostr-reader      │  Read Nostr events (PoW-focused)          │
├─────────────────────────────────────────────────────────────────┤
│                      INTELLIGENCE LAYER                          │
├─────────────────────────────────────────────────────────────────┤
│  sct-vec           │  Vector search, semantic RAG (Q1 2026)    │
├─────────────────────────────────────────────────────────────────┤
│                        OUTPUT LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│  plurcast          │  Post to Nostr/Mastodon/SSB               │
└─────────────────────────────────────────────────────────────────┘

              All tools share: SQLite storage + JSON I/O
```

---

## Integration Pattern 1: INPUT → INTELLIGENCE

### Nostr Reader → sct-vec (Vector Database)

**Use Case:** Index high-quality Nostr content for semantic search

```bash
# Index high-PoW Nostr events in vector database
nostr-reader \
  --relay wss://relay.damus.io \
  --min-pow 20 \
  --kind 1,30023 \
  --since 24h \
  --output jsonl | \
  sct-vec index --collection nostr-pow

# Semantic search across archived Nostr content
echo "rust wasm component model" | \
  sct-vec query --collection nostr-pow --k 10 | \
  jq '.[] | {author, content, pow}'
```

**Data Flow:**
1. Nostr reader fetches events from relays
2. Filters by PoW difficulty threshold
3. Outputs JSONL stream (one event per line)
4. sct-vec indexes content + metadata in sqlite-vec
5. Semantic queries return relevant high-PoW content

**Schema Alignment:**
```json
{
  "id": "event_id_hex",
  "pubkey": "author_pubkey_hex",
  "created_at": 1731629200,
  "kind": 1,
  "content": "Event text content",
  "pow_difficulty": 22,
  "tags": [...],
  "sig": "signature_hex"
}
```

---

## Integration Pattern 2: INPUT → OUTPUT

### Nostr Reader → Plurcast (Cross-Platform Posting)

**Use Case:** Amplify high-quality Nostr content to other platforms

```bash
# Find high-PoW content and cross-post to Mastodon
nostr-reader \
  --relay wss://relay.damus.io \
  --min-pow 25 \
  --kind 1 \
  --author <trusted_pubkey> \
  --limit 1 \
  --format text | \
  plurcast publish --platform mastodon

# Content curation workflow
nostr-reader \
  --relay wss://nos.lol \
  --min-pow 20 \
  --since 1h \
  --format json | \
  jq '.[] | select(.pow_difficulty > 22) | .content' | \
  plurcast publish --platform nostr,mastodon --draft
```

**Use Cases:**
- **Content Curation:** Reshare high-quality posts from Nostr to other platforms
- **Cross-Platform Presence:** Mirror your high-PoW Nostr content elsewhere
- **Quality Amplification:** Only boost content with computational commitment

**Reverse Flow (OUTPUT → INPUT):**
Plurcast already posts to Nostr. With the reader, you can:
- Verify your own posts propagated
- Check PoW on posts you created
- Monitor responses to your content

---

## Integration Pattern 3: INPUT + INPUT

### rss-wasm + Nostr Reader (Unified Feed)

**Use Case:** Merge intentional sources (RSS + Nostr) into unified timeline

```bash
# Combine RSS feeds and high-PoW Nostr into single stream
(rss-wasm parse blog-feeds.opml --format jsonl && \
 nostr-reader --min-pow 20 --format jsonl) | \
  jq -s 'sort_by(.created_at) | reverse | .[] |
    {time: .created_at, source: .source, title: .title, content: .content}'
```

**Common Output Schema:**
Both tools output compatible JSON for composition:

```json
// Unified format
{
  "id": "unique_id",
  "source": "rss|nostr",
  "created_at": 1731629200,
  "author": "name or pubkey",
  "title": "Optional title",
  "content": "Body text",
  "url": "Optional link",
  "metadata": {
    // RSS: feed info, enclosures
    // Nostr: pow, kind, tags
  }
}
```

**Benefits:**
- **Intentional consumption** from both RSS blogs AND Nostr accounts
- **PoW as quality filter** for decentralized content
- **Agent-friendly**: Single timeline format for multi-agent workflows
- **Offline-capable**: Both tools cache locally

---

## Integration Pattern 4: Shared Storage Layer

### SQLite as Common Infrastructure

All SCT tools use SQLite for local-first storage. Nostr reader follows the pattern:

**Plurcast Schema (Reference):**
```sql
-- Plurcast stores posts with platform metadata
CREATE TABLE posts (
  id INTEGER PRIMARY KEY,
  content TEXT NOT NULL,
  platform TEXT NOT NULL,
  account_id TEXT,
  created_at INTEGER,
  metadata JSON
);
```

**Nostr Reader Schema (Proposed):**
```sql
-- Store fetched events with PoW metrics
CREATE TABLE nostr_events (
  id TEXT PRIMARY KEY,           -- Event ID (hex)
  pubkey TEXT NOT NULL,          -- Author pubkey
  created_at INTEGER NOT NULL,   -- Unix timestamp
  kind INTEGER NOT NULL,         -- Event kind (NIP-01)
  content TEXT NOT NULL,         -- Event content
  pow_difficulty INTEGER,        -- Leading zero bits
  tags JSON,                     -- Event tags
  sig TEXT NOT NULL,             -- Event signature
  fetched_at INTEGER,            -- Local timestamp
  relay TEXT                     -- Source relay
);

CREATE INDEX idx_pow ON nostr_events(pow_difficulty DESC);
CREATE INDEX idx_created ON nostr_events(created_at DESC);
CREATE INDEX idx_author ON nostr_events(pubkey);

-- Track relay connections
CREATE TABLE relays (
  url TEXT PRIMARY KEY,
  last_connected INTEGER,
  status TEXT,
  metadata JSON
);
```

**Cross-Tool Queries:**
```bash
# How many high-quality items across all inputs?
sqlite3 ~/.local/share/sct/sct.db "
  SELECT
    'rss' as source, COUNT(*) as items
  FROM rss_items
  UNION ALL
  SELECT
    'nostr' as source, COUNT(*) as items
  FROM nostr_events WHERE pow_difficulty >= 20
"
```

**Benefits:**
- **Single source of truth**: All tools write to `~/.local/share/sct/sct.db`
- **Cross-tool analytics**: Query across RSS + Nostr content
- **Local-first**: Works offline, no cloud dependency
- **sqlite-vec integration**: Ready for sct-vec semantic search

---

## Integration Pattern 5: Multi-Agent Workflows

### Complete Pipeline Example

**Use Case:** Monitor AI/crypto announcements, analyze, and respond

```bash
#!/bin/bash
# Agent workflow: Monitor → Analyze → Respond → Publish

# 1. Gather intelligence from multiple sources
monitor_sources() {
  # RSS: Official blogs
  rss-wasm parse openai-feed.xml anthropic-feed.xml \
    --format jsonl > /tmp/rss-updates.jsonl

  # Nostr: High-quality community commentary (PoW filter for quality)
  nostr-reader \
    --relay wss://relay.damus.io \
    --min-pow 20 \
    --kind 1,30023 \
    --since 1h \
    --search "claude|gpt|llm" \
    --format jsonl > /tmp/nostr-updates.jsonl

  # Combine streams
  cat /tmp/rss-updates.jsonl /tmp/nostr-updates.jsonl | \
    jq -s 'sort_by(.created_at) | reverse'
}

# 2. Index for semantic search
index_content() {
  monitor_sources | \
    sct-vec index --collection ai-news --upsert
}

# 3. Analyze trends (hypothetical agent)
analyze_trends() {
  sct-vec query "major model releases" \
    --collection ai-news \
    --k 20 | \
    agent analyze-impact --context "previous releases"
}

# 4. Draft response
draft_response() {
  analyze_trends | \
    agent draft-commentary --tone professional
}

# 5. Publish to your platforms
publish() {
  draft_response | \
    plurcast publish --platform nostr,mastodon
}

# Run complete pipeline
index_content && publish
```

**Agent Benefits:**
- **Token efficiency**: RSS + filtered Nostr vs scraping web pages
- **Quality signal**: PoW filters spam, keeps signal high
- **Composability**: Each step is testable, replaceable
- **Context**: Vector search provides historical grounding

---

## Integration Pattern 6: PoW as Infrastructure Primitive

### How PoW Serves the Ecosystem

**For rss-wasm:**
- Could add PoW to RSS items before publishing to Nostr
- Bridge RSS → Nostr with computational commitment
- Signal "I vouch for this RSS content" via PoW

**For Plurcast:**
- When posting to Nostr, add PoW difficulty parameter
- `plurcast publish --platform nostr --pow 20`
- Mark your own content as high-quality via computation

**For sct-vec:**
- Use PoW as ranking signal for semantic search
- Weight high-PoW content higher in results
- Filter indexed content by minimum PoW threshold

**Cross-Tool PoW:**
```bash
# Plurcast publishes with PoW
echo "Important announcement" | \
  plurcast publish --platform nostr --pow 22

# Nostr reader verifies it propagated
nostr-reader \
  --author <your_pubkey> \
  --min-pow 20 \
  --since 1m | \
  jq '.[] | select(.content | contains("Important"))'

# Index in vector DB with PoW weighting
nostr-reader --author <your_pubkey> --format jsonl | \
  sct-vec index --collection my-posts \
    --weight-field pow_difficulty
```

---

## Output Format Standardization

### JSON Schema (for composition)

All INPUT tools should output compatible JSON for downstream composition:

```json
{
  "version": "sct-input-v1",
  "source": {
    "type": "nostr|rss|mastodon",
    "identifier": "relay_url|feed_url|instance"
  },
  "item": {
    "id": "unique_identifier",
    "created_at": 1731629200,
    "author": {
      "id": "pubkey|email|handle",
      "name": "Display name",
      "url": "Profile URL"
    },
    "content": {
      "title": "Optional title",
      "body": "Main content",
      "content_type": "text/plain|text/markdown|text/html"
    },
    "metadata": {
      // Source-specific fields
      "pow_difficulty": 22,        // Nostr
      "categories": ["tech"],      // RSS
      "boost_count": 15           // Mastodon
    },
    "links": [
      {"rel": "canonical", "href": "..."},
      {"rel": "attachment", "href": "..."}
    ]
  }
}
```

**Benefits:**
- Tools can consume each other's output
- jq filters work across all sources
- Agents see unified data model
- Future tools integrate easily

---

## CLI Interface Design

### Following rss-wasm and Plurcast Patterns

**rss-wasm pattern:**
```bash
rss-wasm <command> [options]
# Commands: parse, filter, merge, normalize, validate
```

**Plurcast pattern:**
```bash
plurcast <command> [options]
# Commands: publish, history, accounts
```

**Nostr reader pattern:**
```bash
nostr-reader <command> [options]

# Commands:
nostr-reader fetch      # Fetch events from relays
nostr-reader verify     # Verify PoW on events
nostr-reader subscribe  # Real-time subscription
nostr-reader relays     # Manage relay list

# Options:
--relay <url>           # Relay to connect to
--min-pow <difficulty>  # Minimum PoW threshold
--kind <kinds>          # Event kinds (comma-separated)
--author <pubkey>       # Filter by author
--since <time>          # Since timestamp/duration
--limit <n>             # Max events to return
--output <format>       # json|jsonl|text|markdown
--storage <path>        # SQLite database path
```

**Composition examples:**
```bash
# Fetch and filter
nostr-reader fetch --min-pow 20 --kind 1 | jq '.[] | .content'

# Subscribe and index
nostr-reader subscribe --relay wss://relay.damus.io | \
  sct-vec index --collection nostr-live

# Verify local events
cat saved-events.jsonl | nostr-reader verify --min-pow 15

# Merge with RSS
(rss-wasm parse feeds.opml && nostr-reader fetch) | \
  jq -s 'sort_by(.created_at)'
```

---

## Shared Configuration

### Following SCT Patterns

**Directory structure:**
```
~/.config/sct/
  config.toml              # Global SCT config
  rss-wasm.toml           # rss-wasm settings
  plurcast.toml           # Plurcast settings
  nostr-reader.toml       # Nostr reader settings

~/.local/share/sct/
  sct.db                  # Shared SQLite database
  cache/                  # WASM modules, temp files
```

**nostr-reader.toml:**
```toml
[relays]
default = [
  "wss://relay.damus.io",
  "wss://nos.lol",
  "wss://relay.snort.social"
]

[filters]
min_pow = 15              # Default PoW threshold
kinds = [1, 30023]        # Text notes + long-form

[storage]
database = "~/.local/share/sct/sct.db"
cache_events = true
retention_days = 90

[output]
format = "jsonl"
include_metadata = true
```

**Cross-tool config:**
```toml
# ~/.config/sct/config.toml
[storage]
database = "~/.local/share/sct/sct.db"

[output]
default_format = "jsonl"  # All tools default to JSONL

[agents]
enabled = true
context_size = 8192

[integrations]
# Tools can discover each other
rss_wasm = "~/.cargo/bin/rss-wasm"
plurcast = "~/.cargo/bin/plurcast"
nostr_reader = "~/.cargo/bin/nostr-reader"
sct_vec = "~/.cargo/bin/sct-vec"
```

---

## Implementation Phases

### Phase 1: Core Reader (Layer 1 + 2)
- PoW verification (WASM core)
- Basic relay client
- JSON/JSONL output
- CLI interface

**Integration:** Can pipe to jq, sct-vec, plurcast

### Phase 2: Storage (Layer 3)
- SQLite database matching Plurcast schema patterns
- Local event caching
- Relay management

**Integration:** Shared database with other tools

### Phase 3: Advanced Composition
- Unified output schema with rss-wasm
- Cross-tool configuration
- Agent workflows documented

**Integration:** Full ecosystem composability

### Phase 4: Intelligence Features
- Work with sct-vec for semantic search
- PoW-weighted rankings
- Multi-source RAG workflows

**Integration:** Complete INPUT → INTELLIGENCE → OUTPUT pipeline

---

## Why PoW Completes the Vision

### The Anti-Algorithm Stack

**Problem:** Centralized platforms use algorithms to:
- Control what you see (engagement maximization)
- Amplify low-quality content (clickbait wins)
- Lock you into walled gardens

**Solution:** Sovereign Composable Tools with PoW primitive

1. **INPUT (Intentional):**
   - rss-wasm: Subscribe to sources you choose
   - nostr-reader: Filter by computational commitment (PoW)
   - Result: Quality content from decentralized sources

2. **INTELLIGENCE (Local):**
   - sct-vec: Semantic search your own archive
   - PoW as quality signal for ranking
   - Result: Find what matters in your own data

3. **OUTPUT (Sovereign):**
   - Plurcast: Publish to platforms you choose
   - Optional PoW on your own posts
   - Result: Reach your audience without platform control

### PoW as Universal Quality Primitive

**For humans:**
- "Show me posts someone thought hard about" (high PoW)
- Spam costs compute, quality signals commitment
- No central moderator needed

**For agents:**
- Filter signal from noise without ML
- Verifiable quality metric (objective)
- Compose across tools (PoW everywhere)

**Example agent workflow:**
```bash
# Agent monitors high-quality discussions
nostr-reader \
  --min-pow 25 \
  --search "wasm component model" | \
  sct-vec index --collection wasm-discourse

# Agent finds context for response
echo "best practices component model" | \
  sct-vec query --collection wasm-discourse | \
  agent synthesize-response | \
  plurcast publish --pow 20  # Agent's response also has PoW
```

---

## Next Steps

1. **Verify WASM compatibility** - Test rust-nostr with wasm32-wasip2
2. **Build minimal reader** - Core PoW verification + relay client
3. **Design output schema** - Align with rss-wasm patterns
4. **Test composition** - Pipe to existing tools (jq, plurcast)
5. **Add storage layer** - SQLite matching Plurcast patterns
6. **Document workflows** - Example pipelines for humans and agents

**The goal:** By adding PoW-focused Nostr reading, we complete the anti-algorithm stack for both humans seeking quality content and agents needing composable primitives.
