# BUILD-77 — Memory Protocol

> Source: [https://notion.so/c9f756669b3548688c1742a776f121fb](https://notion.so/c9f756669b3548688c1742a776f121fb)
> Created: 2026-04-20T18:35:00.000Z | Last edited: 2026-04-20T20:10:00.000Z



---
> **ℹ **Tier 14 · Memory · Cross-scale · Priority: HIGH****

  Memory is tiered by both semantic layer (L0 raw → L7 executive) and scale (Meso shared → Pico register). Defines canonical placements and hot paths.

## Fold Provenance

*[table: 2 columns]*

## Purpose

Prevents cache thrash + duplication across scales. Every memory item has a canonical tier × scale cell; copies are tracked.

## Dependencies

- **BUILD-24, BUILD-49, BUILD-46** (ancestors)
## File Structure

```javascript
crates/mem-hier/
├── src/
│   ├── placement/
│   │   ├── rule.rs
│   │   └── copy.rs
│   ├── recall/
│   │   └── path.rs
│   ├── fold/
│   │   ├── promote.rs
│   │   └── evict.rs
│   └── types.rs
```

## Interfaces & Types

```rust
pub struct Cell { pub tier: MemTier, pub scale: SwarmScale }
pub enum MemTier { L0, L1, L2, L3, L4, L5, L6, L7 }
```

## Implementation SOP

1. Canonical cell per item; copies annotated as caches.
1. Recall walks tiers bottom-up (Pico → Meso).
1. Promotion on hit frequency; eviction on cold.
1. Consistency via HLC + invalidation gossip.
## Acceptance Criteria

- [ ] Placement deterministic
- [ ] Recall path bounded
- [ ] Invalidations arrive ≤ 1 s
- [ ] Evictions auditable
- [ ] All tests pass with `vitest run`
- [ ] Duplication ≤ 2x
- [ ] Cross-scale hit-ratio gauge
- [ ] Privacy scope honored
## Architecture

```mermaid
flowchart LR
	HIT[Recall] --> PICO[Pico reg]
	PICO --> NANO[Nano cache]
	NANO --> MICRO[Micro shmem]
	MICRO --> MESO[Meso]
	MESO --> L0[L0 store]
```

## Placement Matrix (canonical)

*[table: 2 columns]*

## Extended Types

```rust
pub struct Copy { pub src: Cell, pub at: Cell, pub ttl: Duration }
```

## Reference — Recall

```rust
pub async fn recall(k: &Key) -> Option<Bytes> { path::walk_up(k).await }
```

## Observability

- `mem.hits_by_scale_tier` counter
- `mem.evictions_total` by tier
- `mem.dup_factor` gauge
## Security

- Scope: per-tenant / per-tier
- Invalidation signed
## Failure Modes

*[table: 3 columns]*

## Operational Runbook

1. **Stats:** `mem stats --tier L4 --scale meso`.
1. **Evict:** `mem evict --key <k>`.
## Integration

- Consumed by Agent runtimes; Oracle inspects stats
## FAQ

> **Can copies live in Pico?** Only as registers during active op.

## Changelog

- v0.1.0 — placement, recall, promote/evict
- v0.2.0 (planned) — compression per tier
- v0.3.0 (planned) — predictive prefetch

