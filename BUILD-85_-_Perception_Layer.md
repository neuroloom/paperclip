# BUILD-85 — Perception Layer

> Source: [https://notion.so/0b1c3be1dde84338b623cb48a8e43cfc](https://notion.so/0b1c3be1dde84338b623cb48a8e43cfc)
> Created: 2026-04-20T18:39:00.000Z | Last edited: 2026-04-20T20:11:00.000Z



---
> **ℹ **Tier 15 · Perception · Cross-scale · Priority: HIGH****

  Sensor-to-knowledge pipeline. Bytes → features → chunks → concepts, with scale-appropriate agents at each step.

## Fold Provenance

*[table: 2 columns]*

## Purpose

Provide a canonical perception pipeline that every modality plugs into. Consistent observability, provenance, and backpressure.

## Dependencies

- **BUILD-24, BUILD-39, BUILD-41, BUILD-68** (ancestors)
## File Structure

```javascript
crates/perception/
├── src/
│   ├── pipeline/
│   │   ├── stage.rs
│   │   └── dag.rs
│   ├── modalities/
│   │   ├── text.rs
│   │   ├── audio.rs
│   │   ├── vision.rs
│   │   └── events.rs
│   ├── fold/
│   │   ├── backpressure.rs
│   │   └── drop.rs
│   └── types.rs
```

## Interfaces & Types

```rust
pub struct Stage { pub name: String, pub fn_sig: AtomicSig, pub scale: SwarmScale }
pub struct Pipeline { pub modality: Modality, pub stages: Vec<Stage> }
```

## Implementation SOP

1. Land raw in L0 (scale: Meso).
1. Stages run on Nano Swarms.
1. Backpressure propagates upstream.
1. Emit to KG Spine + provenance.
## Acceptance Criteria

- [ ] 4 modalities supported
- [ ] Backpressure works
- [ ] Provenance end-to-end
- [ ] Drop policy explicit
- [ ] All tests pass with `vitest run`
- [ ] Modality extensibility
- [ ] p99 stage latency bounded
- [ ] Quality metrics per modality
## Architecture

```mermaid
flowchart LR
	IN[Sensor] --> L0[L0]
	L0 --> S1[feat]
	S1 --> S2[chunk]
	S2 --> S3[concept]
	S3 --> KG[KG Spine]
```

## Modality Stage Table (seed)

*[table: 2 columns]*

## Extended Types

```rust
pub enum Modality { Text, Audio, Vision, Events }
pub struct DropPolicy { pub on_full: DropKind, pub ttl: Duration }
pub enum DropKind { OldestFirst, LowestPriority, Never }
```

## Reference — Enqueue

```rust
pub async fn enqueue(p: &Pipeline, payload: Bytes) -> Result<()> { /* ... */ Ok(()) }
```

## Observability

- `perc.stage.duration_ms` by stage
- `perc.drops_total` by reason
- `perc.quality_score` gauge
## Security

- Per-tenant pipelines
- PII redaction at extract
## Failure Modes

*[table: 3 columns]*

## Operational Runbook

1. **Tail:** `perc tail --pipeline text`.
1. **Scale:** `perc scale --stage embed --lanes 4`.
## Integration

- Output → KG Spine, Agents
## FAQ

> **Can pipelines be custom per tenant?** Yes — stages composable from registry.

## Changelog

- v0.1.0 — pipelines, backpressure, drop
- v0.2.0 (planned) — on-device pre-filters
- v0.3.0 (planned) — adaptive sampling

