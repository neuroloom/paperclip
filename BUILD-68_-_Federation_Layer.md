# BUILD-68 — Federation Layer

> Source: [https://notion.so/b4b33417751b42fdb9285c0b8e40e3fb](https://notion.so/b4b33417751b42fdb9285c0b8e40e3fb)
> Created: 2026-04-20T18:29:00.000Z | Last edited: 2026-04-20T20:09:00.000Z



---
> **ℹ **Tier 13 · Scheduler · Cross-scale · Priority: HIGH****

  Unified scheduler that chooses the right scale for a work item: big tasks → Meso, specialized tasks → Micro, primitives → Nano, hot loops → Pico.

## Fold Provenance

*[table: 2 columns]*

## Purpose

Operators and caller code submit generic work items; the Scale-Adaptive Scheduler picks scale + swarm. Heuristics: input size, deadline, cost ceiling, security class.

## Dependencies

- **BUILD-08, BUILD-67, BUILD-68** (ancestors)
- **BUILD-79 (Budget)** — check ceilings
- **BUILD-74 (Registry)** — list swarms
## File Structure

```javascript
crates/scale-sched/
├── src/
│   ├── classify/
│   │   ├── size.rs
│   │   ├── deadline.rs
│   │   └── class.rs
│   ├── pick/
│   │   ├── policy.rs
│   │   └── fallback.rs
│   ├── fold/
│   │   ├── demote.rs
│   │   └── promote.rs
│   └── types.rs
```

## Interfaces & Types

```rust
pub struct Submission {
    pub payload: Bytes,
    pub deadline: HLCTimestamp,
    pub class: SecurityClass,
    pub preferred_scale: Option<SwarmScale>,
}

pub struct Assignment { pub scale: SwarmScale, pub swarm: SwarmId }
```

## Implementation SOP

### Step 1: Classify

- Size → tentative scale
- Deadline → urgency tier
- Class → eligible scales
### Step 2: Pick

- Filter registry for eligible swarms
- Rank by queue depth, budget, affinity
### Step 3: Demote/promote

- If Nano overloaded, promote to Micro
- If Meso underutilized for small task, demote
## Acceptance Criteria

- [ ] Classification correct
- [ ] Registry filtering O(log n)
- [ ] Demote/promote cycles bounded
- [ ] Deadline honored
- [ ] All tests pass with `vitest run`
- [ ] Decision latency ≤ 500 µs
- [ ] Hotspot-aware
- [ ] Fairness guarantee
## Architecture

```mermaid
flowchart LR
	SUB[Submission] --> CLASS[Classify]
	CLASS --> PICK[Pick scale+swarm]
	PICK --> DISPATCH[Dispatch]
```

## Classification Rules

*[table: 4 columns]*

## Extended Types

```rust
pub struct DecisionTrace { pub steps: Vec<String>, pub chosen: Assignment }
```

## Reference — Dispatch

```rust
pub async fn dispatch(s: Submission) -> Result<Assignment> {
    let class = classify::all(&s)?;
    let assign = pick::best(class).await?;
    Ok(assign)
}
```

## Observability

- `sched.decisions_total` by scale
- `sched.promote_total` / `sched.demote_total`
- `sched.decision_us` histogram
## Security

- Class-based gating
- Audit trail per assignment
- Shed non-eligible scales
## Failure Modes

*[table: 3 columns]*

## Operational Runbook

1. **Trace:** `sched trace --sub <id>`.
1. **Bias:** `sched bias --scale nano --weight 1.2`.
1. **Replay:** `sched replay --file trace.jsonl`.
## Integration

- Consumes Budget (BUILD-79)
- Uses Swarm Registry (BUILD-74)
## FAQ

> **Who can override scale?** Any submitter via `preferred_scale`, subject to class checks.

## Changelog

- v0.1.0 — classify, pick, demote/promote
- v0.2.0 (planned) — learned policy
- v0.3.0 (planned) — cost-aware dispatch

