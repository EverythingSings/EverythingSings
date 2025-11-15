# sct-npow Roadmap

**Status:** 🧪 Experimental - Design Phase

---

## Phase 1: Core + CLI (Target: Q4 2024)

### Research
- [x] Research rust-nostr library and NIP-13 PoW support
- [x] Design 4-layer SCT architecture
- [x] Define CLI interface
- [x] Document integration patterns
- [ ] Verify wasm32-wasip2 compatibility with rust-nostr
- [ ] Identify WASI-compatible WebSocket library

### Implementation
- [ ] Set up Rust project with wasm32-wasip2 target
- [ ] Implement Layer 1 (Core): PoW verification
  - [ ] Event parsing (using `nostr` crate)
  - [ ] PoW difficulty calculation (count leading zeros)
  - [ ] Event signature validation
  - [ ] Filter construction
- [ ] Implement Layer 4 (CLI): Basic commands
  - [ ] `sct-npow verify` - Verify PoW on events from stdin
  - [ ] JSON/JSONL output formats
  - [ ] Text output format
  - [ ] Command-line argument parsing (clap)

### Testing
- [ ] Unit tests for PoW verification
- [ ] Integration tests with sample events
- [ ] Composition tests (pipe to jq, other tools)

**Deliverable:** `sct-npow verify` - Verify PoW on events from stdin

---

## Phase 2: Network Layer (Target: Q1 2025)

### Research
- [ ] Evaluate WASI-compatible WebSocket libraries
- [ ] Test async runtime (tokio/async-std) with WASI
- [ ] Benchmark relay connection performance

### Implementation
- [ ] Implement Layer 2 (Protocol): Relay client
  - [ ] WebSocket connection to single relay
  - [ ] Send REQ message (NIP-01)
  - [ ] Receive EVENT messages
  - [ ] Handle EOSE (end of stored events)
  - [ ] Connection timeout/retry logic
- [ ] Expand CLI commands
  - [ ] `sct-npow fetch` - Fetch events from relay
  - [ ] Multi-relay support
  - [ ] Relay URL validation
  - [ ] Connection status reporting

### Testing
- [ ] End-to-end relay fetching tests
- [ ] Error handling (connection failures, timeouts)
- [ ] Multiple relay tests

**Deliverable:** `sct-npow fetch` - Fetch events from Nostr relays

---

## Phase 3: Storage Layer (Target: Q1 2025)

### Implementation
- [ ] Implement Layer 3 (Storage): Local caching
  - [ ] SQLite schema (events, relays, authors)
  - [ ] Insert/query events
  - [ ] Index by PoW, timestamp, author, kind
  - [ ] Cache management (retention, cleanup)
- [ ] Expand CLI commands
  - [ ] `sct-npow relays` - Manage relay list
  - [ ] `sct-npow authors` - Track high-PoW authors
  - [ ] `sct-npow stats` - Show PoW statistics
  - [ ] `--cache` flag for fetch command

### Testing
- [ ] SQLite integration tests
- [ ] Cache expiration tests
- [ ] Concurrent access tests
- [ ] Cross-tool database sharing tests (with plurcast schema)

**Deliverable:** Persistent storage, offline-capable queries

---

## Phase 4: Advanced Features (Target: Q2 2025)

### Implementation
- [ ] Real-time subscriptions
  - [ ] `sct-npow subscribe` - Live event stream
  - [ ] WebSocket keep-alive
  - [ ] Automatic reconnection
- [ ] Search functionality
  - [ ] Full-text search in event content
  - [ ] Regex pattern matching
  - [ ] Tag-based filtering
- [ ] Statistics and analytics
  - [ ] PoW distribution charts (text-based)
  - [ ] Author rankings
  - [ ] Relay performance metrics
- [ ] Advanced output formats
  - [ ] Markdown output
  - [ ] HTML output (for static sites)
  - [ ] Custom templates

### Integration Examples
- [ ] Document rss-wasm + sct-npow unified feed
- [ ] Document sct-npow → sct-vec indexing
- [ ] Document sct-npow → plurcast amplification
- [ ] Multi-agent workflow examples
- [ ] Create example scripts

### Performance
- [ ] Optimize PoW verification (SIMD?)
- [ ] Parallel relay connections
- [ ] Event batching
- [ ] Memory profiling

**Deliverable:** Feature-complete 1.0 release

---

## Phase 5: Polish & Release (Target: Q2-Q3 2025)

### Documentation
- [ ] User guide
- [ ] API reference
- [ ] Tutorial videos
- [ ] Blog post announcement

### Quality
- [ ] Security audit
- [ ] Performance benchmarks
- [ ] Cross-platform testing (Linux, macOS, Windows)
- [ ] WASM runtime testing (wasmtime, wasmer, wazero)

### Community
- [ ] Open source release
- [ ] Contribution guidelines
- [ ] Issue templates
- [ ] CI/CD setup

**Deliverable:** Public 1.0 release

---

## Future Possibilities

### Additional NIPs
- [ ] NIP-11: Relay information document
- [ ] NIP-40: Expiration timestamps
- [ ] NIP-65: Relay list metadata (for discovery)
- [ ] NIP-50: Search capability (relay-side)

### Advanced PoW Features
- [ ] PoW mining (create high-PoW events)
- [ ] Difficulty prediction/recommendation
- [ ] PoW cost calculator (time/energy)
- [ ] Adaptive difficulty thresholds

### Integration Enhancements
- [ ] Native sct-vec plugin
- [ ] Plurcast PoW publishing support
- [ ] RSS → Nostr bridge with PoW
- [ ] Nostr → RSS bridge (high-PoW as feed)

### Performance
- [ ] GPU acceleration for PoW mining
- [ ] Distributed relay queries
- [ ] Bloom filters for event deduplication
- [ ] Delta compression for storage

---

## Open Questions

### Technical Decisions Needed

1. **Default PoW threshold?**
   - Research: What's typical on live relays?
   - Hypothesis: 15-20 for quality, 25+ for premium
   - Action: Empirical testing needed

2. **WebSocket library?**
   - Must support wasm32-wasip2
   - Options need research
   - Fallback: Manual implementation?

3. **Async runtime?**
   - Tokio vs async-std for WASI
   - Performance testing needed

4. **Storage limits?**
   - Time-based (90 days default?)
   - Count-based (10k events default?)
   - Both with config override?

### Product Decisions Needed

1. **Relay discovery?**
   - Start with manual list
   - Add NIP-65 later?
   - Gossip protocol integration?

2. **Author reputation?**
   - Track average PoW per author
   - Build local web-of-trust?
   - Keep it simple (just PoW)?

3. **Output schema standardization?**
   - Align with rss-wasm exactly?
   - Create SCT-wide input schema?
   - Keep tool-specific for now?

---

## Success Metrics

### Technical
- [ ] WASM binary < 2MB (optimized)
- [ ] Fetch 1000+ events/second
- [ ] PoW verification < 1ms per event
- [ ] Support 10+ concurrent relays
- [ ] Works on all major WASM runtimes

### User Experience
- [ ] Easy to install (cargo install)
- [ ] Works offline with cache
- [ ] Composes with Unix tools (jq, grep)
- [ ] Clear error messages
- [ ] Helpful documentation

### Ecosystem
- [ ] Used in multi-agent workflows
- [ ] Integration examples exist
- [ ] Other tools depend on it
- [ ] Community contributions

---

**Last updated:** 2025-11-15
**Current phase:** Phase 1 (Research & Design)
**Next milestone:** wasm32-wasip2 compatibility verification
