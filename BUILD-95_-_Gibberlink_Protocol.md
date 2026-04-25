# BUILD-95 — Gibberlink Protocol

> Source: [https://notion.so/f71da3d7a0a5459aa2e01e438c7567d9](https://notion.so/f71da3d7a0a5459aa2e01e438c7567d9)
> Created: 2026-04-20T14:29:00.000Z | Last edited: 2026-04-20T20:11:00.000Z



---
> **ℹ **Tier 11 · Applications / Bonus · Priority: MEDIUM****

  Compressed inter-agent communication protocol. When two NeuroLoom agents detect each other, they switch from human-readable text to a compact binary protocol, achieving 10-50× token reduction.

## Purpose

Gibberlink replaces verbose natural-language exchange with a compact, typed protocol when agents detect peer AI. Preserves semantics at 5-10% the token cost. Backward-compatible: falls back to plain text for human or non-NeuroLoom counterparts. Used in multi-agent negotiation, tool chaining, and federated reasoning.

## Dependencies

- **BUILD-08 (Queen Orchestrator)** — Multi-agent coordinator
- **BUILD-09 (Fabric Router)** — Transport layer
- **BUILD-29 (Candle ML)** — Semantic embedding
- **BUILD-20 (FED_SYNC_V1)** — Federation foundation
## File Structure

```javascript
crates/gibberlink/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── handshake/
│   │   ├── detect.rs         # Peer AI detection
│   │   ├── negotiate.rs      # Protocol version negotiation
│   │   └── fallback.rs       # Text fallback
│   ├── codec/
│   │   ├── encoder.rs        # Semantic → compressed binary
│   │   ├── decoder.rs        # Binary → semantic
│   │   ├── dictionary.rs     # Shared vocabulary
│   │   └── huffman.rs        # Adaptive Huffman coding
│   ├── schema/
│   │   ├── intent.rs         # Typed intents (request/response/broadcast)
│   │   ├── data_types.rs     # Strongly-typed payload schemas
│   │   └── registry.rs       # Schema registry
│   ├── context/
│   │   ├── session.rs        # Per-session shared context
│   │   └── embedding.rs      # Shared embedding reference
│   ├── security/
│   │   ├── auth.rs           # Peer authentication
│   │   ├── encrypt.rs        # Payload encryption
│   │   └── replay.rs         # Replay attack prevention
│   ├── types.rs
│   └── config.rs
└── tests/
```

## Interfaces & Types

```rust
pub struct GibberMessage {
    pub version: u8,
    pub session_id: u64,
    pub sequence: u64,
    pub intent: IntentCode,
    pub payload: CompressedPayload,
    pub auth_tag: [u8; 16],
}

pub enum IntentCode {
    Request = 0x01,
    Response = 0x02,
    Broadcast = 0x03,
    Query = 0x04,
    Commit = 0x05,
    Reject = 0x06,
    ContextShare = 0x10,
    ToolInvoke = 0x20,
    ToolResult = 0x21,
    NegotiationOffer = 0x30,
    NegotiationAccept = 0x31,
}

pub struct CompressedPayload {
    pub schema_id: u16,        // Registry lookup
    pub encoding: EncodingType,
    pub data: Vec<u8>,
}

pub enum EncodingType {
    Huffman { dict_version: u16 },
    EmbeddingRef { embedding_id: u64 },
    Dictionary { dict_id: u16 },
    Raw,
}

pub struct Handshake {
    pub peer_id: String,
    pub capabilities: Capabilities,
    pub shared_secret: [u8; 32],
    pub dict_version: u16,
    pub session_id: u64,
}

pub struct Capabilities {
    pub gibberlink_version: u8,
    pub schemas: Vec<u16>,         // Known schema IDs
    pub max_message_size: u32,
    pub supports_embedding_ref: bool,
}

pub struct SchemaDefinition {
    pub id: u16,
    pub name: String,
    pub version: u8,
    pub fields: Vec<FieldDef>,
}

pub struct GibberlinkConfig {
    pub min_peer_gibberlink_version: u8,   // 1
    pub require_auth: bool,                // true
    pub session_timeout_sec: u32,          // 3600
    pub dict_refresh_interval_sec: u32,    // 86400
    pub max_compression_ratio: f64,        // Target 10x
    pub fall_back_to_text: bool,           // true for human peers
}
```

## Implementation SOP

### Step 1: Peer detection (`handshake/detect.rs`)

- Inspect first message signature (capabilities advertisement)
- Pattern match on system prompts/headers
- Probe: send Gibberlink Hello — if valid response, upgrade; else fallback
- ⚠️ Never assume peer supports Gibberlink — always probe first
### Step 2: Handshake (`handshake/negotiate.rs`)

- Exchange capabilities
- Derive shared secret (ECDH or pre-shared)
- Agree on protocol version (min of both)
- Exchange dictionary version
### Step 3: Codec (`codec/`)

- `encoder.rs`: structured intent → schema lookup → field encoding → Huffman
- `decoder.rs`: reverse
- `dictionary.rs`: shared domain vocabulary (e.g., NeuroLoom terms, common entities)
- `huffman.rs`: adaptive Huffman with periodic dictionary refresh
### Step 4: Schemas (`schema/`)

- Strongly-typed intent payloads (like Protocol Buffers but more compact)
- Field-level compression hints
- Schema registry with semantic versioning
- ⚠️ Schema changes must be backward-compatible or gated by version negotiation
### Step 5: Context optimization (`context/`)

- Per-session shared context: reference prior messages by hash
- Embedding reference: send embedding ID instead of text for known concepts
- Context window bounded; LRU eviction
### Step 6: Security (`security/`)

- Mutual auth at handshake (signed capabilities)
- ChaCha20-Poly1305 payload encryption
- Replay prevention: session_id + sequence windowing
## Gotchas & Warnings

⚠️ Compression ratio varies by domain; 50× is best case, 5× is worst

⚠️ Schema drift breaks decoding — always version-check before using

⚠️ Embedding refs require synced model between peers

⚠️ Debugging compressed traffic is hard — include dev-mode text fallback

⚠️ Adversarial peers may claim Gibberlink support but send malformed data — validate strictly

⚠️ This is for machine-machine only — always fallback to text for humans

## Testing Requirements

- **Unit:** Handshake with all capability combinations
- **Unit:** Encode/decode roundtrip preserves semantics
- **Unit:** Schema version negotiation
- **Unit:** Replay attacks rejected
- **Integration:** Multi-agent conversation with measured compression ratio
- **Compatibility:** Falls back to text for non-Gibberlink peers
- **Fuzz:** Malformed inputs don't crash decoder
## Acceptance Criteria

- [ ] Peer AI detection works
- [ ] Handshake with capability negotiation
- [ ] Encoder/decoder preserves all test payloads
- [ ] Schema registry with version compatibility
- [ ] Context optimization via hash/embedding refs
- [ ] Mutual auth + encryption
- [ ] ≥5× compression ratio on test corpus
- [ ] Fallback to text for non-Gibberlink peers
- [ ] All tests pass

