# Nostr Libraries Research: PoW-Focused Reader

**Date:** 2025-11-15
**Branch:** `claude/sct-nostr-reader-01EHSVxZDDpJcddWbVDtT5yt`
**Purpose:** Research Rust libraries for building an SCT Nostr reader with Proof of Work (NIP-13) support

---

## Executive Summary

**Primary Library:** `rust-nostr` (https://github.com/rust-nostr/nostr)
**Status:** ALPHA - working but expect breaking API changes
**PoW Support:** ✅ NIP-13 fully supported
**WASM Support:** ⚠️ `wasm32-unknown-unknown` (browser), **NOT** `wasm32-wasip2` (Component Model)

### Critical Compatibility Issue

The `rust-nostr` ecosystem targets **browser WASM** (`wasm32-unknown-unknown`), while Everything Sings targets **Component Model WASM** (`wasm32-wasip2`). This is a potential blocker requiring investigation.

---

## Library Ecosystem Overview

### rust-nostr Monorepo Structure

The project is organized as a workspace with modular crates:

#### Core Protocol
- **`nostr`** - Low-level protocol implementation
  - `no_std` capable (see `embedded/` examples)
  - Best choice for constrained environments
  - Pure protocol logic, event building, NIP implementations

- **`nostr-sdk`** - High-level client library
  - Relay connection management
  - Event subscription/publishing
  - Full standard library required

#### Storage Backends
- `nostr-database` - Storage abstraction layer
- `nostr-lmdb` - LMDB backend
- `nostr-ndb` - NDB backend (performance-focused)
- `nostr-indexeddb` - Browser storage (WASM-specific)

#### Authentication & Signing
- `nostr-browser-signer` - NIP-07 (window.nostr)
- `nostr-connect` - NIP-46 (remote signing)
- `nostr-keyring` - Secure key storage

#### Additional Components
- `nostr-relay-pool` - Multi-relay management
- `nwc` - Nostr Wallet Connect (NIP-47)
- `nostr-blossom` - File storage protocol
- `nostr-gossip` - Gossip protocol traits

---

## NIP-13 Proof of Work Support

### Implementation

rust-nostr provides first-class PoW support through the `EventBuilder` API:

```rust
use nostr::{EventBuilder, Keys};

let keys = Keys::generate();

// Create event with PoW difficulty of 20 leading zero bits
let event = EventBuilder::text_note("POW text note from rust-nostr")
    .pow(20)  // Specify difficulty
    .sign_with_keys(&keys)?;
```

### How It Works

1. **Difficulty:** Number of leading zero bits in NIP-01 event ID
2. **Mining Process:** Library automatically:
   - Updates nonce tag
   - Recalculates event ID hash
   - Repeats until difficulty target met
3. **Verification:** Receivers count leading zeros in event ID

### Use Cases for PoW Reader

- **Spam Filtering:** Only display events meeting minimum difficulty threshold
- **Priority Ranking:** Sort feed by computational commitment
- **Quality Signal:** Higher PoW = stronger anti-spam commitment
- **Resource Control:** Limit relay/bandwidth to high-PoW events

---

## WASM Compatibility Analysis

### Current rust-nostr WASM Support

✅ **Supported:** `wasm32-unknown-unknown` (browser)
- Example: https://github.com/rust-nostr/nostr-sdk-wasm-example
- Build: `cargo build --target wasm32-unknown-unknown`
- Use case: Web apps, browser extensions
- Special feature: NIP-07 window.nostr (browser-only)

⚠️ **Limitation:** NIP-03 (OpenTimestamps) not WASM-compatible

### Everything Sings Target

🎯 **Required:** `wasm32-wasip2` (Component Model)
- WASI 0.2 (Preview 2)
- WebAssembly Component Model
- Tier 2 support (Rust 1.82+, Oct 2024)
- Server-side, CLI, embedded runtimes
- Sockets & networking in WASI 0.2 (not in 0.1)

### The Gap

**Problem:** rust-nostr documented for `wasm32-unknown-unknown`, no evidence of `wasm32-wasip2` testing

**Potential Paths:**

1. **Use `nostr` crate (not SDK)** - Lower-level, `no_std` capable
   - May compile to wasip2 with feature flags
   - Skip networking layers that assume browser
   - Build relay connections manually

2. **Evaluate Dependencies** - Check if blockers exist:
   - HTTP clients (many lack WASI socket support)
   - Crypto libraries (some use browser-specific APIs)
   - Storage backends (IndexedDB browser-only)

3. **Hybrid Approach** - Use nostr for protocol logic, custom WASI networking
   - `nostr` crate for event parsing, PoW verification
   - Custom relay client using WASI-compatible HTTP/WebSocket

4. **Upstream Contribution** - Add wasip2 support to rust-nostr
   - Long-term investment
   - Benefits broader ecosystem

---

## Recommended Architecture

### Layer 1: Core (Pure WASM, Zero I/O)

```rust
// Use `nostr` crate for protocol logic
use nostr::{Event, EventId, Filter};

pub struct PowFilter {
    min_difficulty: u8,
}

impl PowFilter {
    /// Check if event meets PoW requirement
    pub fn verify_pow(&self, event: &Event) -> bool {
        count_leading_zeros(&event.id) >= self.min_difficulty
    }
}

fn count_leading_zeros(id: &EventId) -> u8 {
    // NIP-13 implementation
    // Count leading zero bits in event ID
}
```

### Layer 2: Protocol (Feature-Flagged Networking)

Options to investigate:

1. **Try `nostr-sdk` with wasip2**
   - May work if dependencies support WASI sockets
   - Need to audit dependency tree

2. **Build custom relay client**
   - Use WASI-compatible HTTP/WebSocket libraries
   - `nostr-relay-pool` as reference implementation
   - Direct socket access via WASI 0.2

### Layer 3: Storage (Optional Persistence)

- **SQLite** - Already in tech stack, WASI-compatible
- Store events locally after PoW verification
- `nostr-database` trait as inspiration, custom implementation

### Layer 4: Application (CLI)

```bash
# Example usage
nostr-reader \
  --relay wss://relay.damus.io \
  --min-pow 20 \
  --kind 1 \
  --limit 50 \
  --output json
```

Output composable with other SCT tools via stdin/stdout.

---

## Next Steps

### Immediate Actions

1. **Verify wasip2 Compatibility**
   - Clone rust-nostr
   - Attempt build with `--target wasm32-wasip2`
   - Document which crates/features fail
   - Identify problematic dependencies

2. **Evaluate Alternatives**
   - Check if lighter Nostr libraries exist
   - Consider building minimal protocol implementation
   - Search for WASI-native WebSocket/HTTP clients

3. **Prototype Core Layer**
   - Build PoW verification without networking
   - Use sample Nostr events (JSON)
   - Prove concept with pure `nostr` crate

### Decision Points

**If rust-nostr works with wasip2:**
→ Use `nostr-sdk` with careful feature selection

**If dependency conflicts exist:**
→ Use `nostr` crate only, build custom relay layer

**If incompatible:**
→ Minimal protocol implementation referencing rust-nostr

---

## Alternative Libraries

### nostro2-nips
- **URL:** https://lib.rs/crates/nostro2-nips
- **Description:** Simple Nostr tools
- **WASM:** Claims WASM compilation support
- **Status:** Unknown wasip2 compatibility
- **Consideration:** Likely smaller dependency tree

### Custom Implementation
- **Pros:** Full control, minimal dependencies, wasip2-first
- **Cons:** More work, reinventing wheel
- **Scope:** NIP-01 (basic protocol), NIP-13 (PoW) only
- **Reference:** rust-nostr for correctness

---

## Technical Notes

### WASI 0.2 Advantages for This Use Case

- **Sockets:** WASI 0.1 lacked networking, 0.2 has it
- **Component Model:** Clean interfaces, composability
- **CLI-first:** Perfect for SCT unix-pipe architecture
- **Offline-capable:** Local execution, local storage

### NIP-13 Implementation Complexity

**Low complexity** - core algorithm is simple:
1. Parse event JSON
2. Calculate SHA-256 of event ID
3. Count leading zero bits
4. Compare to threshold

Verification can be implemented in ~50 lines of Rust using standard crypto libraries.

### PoW Mining (Future)

If we want to **create** PoW events (not just read):
- More complex (mining loop)
- CPU-intensive
- rust-nostr's `.pow()` API is excellent reference
- Component Model allows WASM-native execution

---

## Questions for Consideration

1. **Read-only vs Read-Write?**
   - Just consume events, or also publish?
   - Publishing needs key management, signing

2. **Relay Strategy:**
   - Connect to specific relays (CLI arg)?
   - Discover relays (NIP-65)?
   - Gossip protocol?

3. **PoW Granularity:**
   - Binary filter (yes/no)?
   - Difficulty tiers (bronze/silver/gold)?
   - Display PoW metrics alongside content?

4. **Composition with Other Tools:**
   - Pipe to `jq` for JSON processing?
   - Feed into `sct-vec` for semantic search?
   - Output to `plurcast` for re-posting?

5. **Performance Target:**
   - How many events/second?
   - Relay connection count?
   - Local storage size limits?

---

## References

- **rust-nostr Repository:** https://github.com/rust-nostr/nostr
- **Rust Nostr Book:** https://rust-nostr.org/
- **NIP-13 Spec:** https://github.com/nostr-protocol/nips/blob/master/13.md
- **WASI 0.2 Announcement:** https://blog.rust-lang.org/2024/11/26/wasip2-tier-2/
- **wasm32-wasip2 Platform:** https://doc.rust-lang.org/rustc/platform-support/wasm32-wasip2.html

---

## Conclusion

**rust-nostr is the right library** for Nostr protocol + PoW support, but **WASM target compatibility needs verification**. The modular architecture allows using just the `nostr` core crate if `nostr-sdk` has wasip2 blockers.

**Recommended immediate next step:** Build a minimal PoW verification prototype using the `nostr` crate targeting `wasm32-wasip2` to identify any compatibility issues early.
