# BUILD-69 — Agent ISA

> Source: [https://notion.so/698713f11cf049139859d30d4dbd5859](https://notion.so/698713f11cf049139859d30d4dbd5859)
> Created: 2026-04-20T18:31:00.000Z | Last edited: 2026-04-20T20:09:00.000Z



---
> **ℹ **Tier 13 · Lineage · Cross-scale · Priority: MEDIUM****

  Canonical library of role genomes (DNA segments encoding agent behaviors). Instead of hand-coding roles, agents are *grown* from genomes selected by Continuum evolution.

## Fold Provenance

*[table: 2 columns]*

## Purpose

Roles (planner, executor, verifier, …) are themselves genotypes. The Role Genome Library stores canonical and candidate genomes, versions them, and feeds the best into live agents.

## Dependencies

- **BUILD-56, BUILD-65, BUILD-69, BUILD-72** (ancestors)
## File Structure

```javascript
crates/role-genome/
├── src/
│   ├── catalog/
│   │   ├── canonical.rs
│   │   └── candidates.rs
│   ├── express/
│   │   ├── translate.rs
│   │   └── splice.rs
│   ├── fold/
│   │   ├── selection.rs
│   │   └── drift.rs
│   └── types.rs
```

## Interfaces & Types

```rust
pub struct RoleGenome {
    pub role: String,
    pub version: Semver,
    pub segments: Vec<Segment>,
    pub fitness: Fitness,
}

pub struct Segment { pub gene: String, pub locus: u32, pub payload: Vec<u8> }
```

## Implementation SOP

### Step 1: Canonical

- Shipped genomes per role (planner-001, executor-001, …)
- Immutable; tagged with provenance
### Step 2: Candidates

- Evolved variants
- Continuum scores them; top-k promoted
### Step 3: Expression

- Translate genome → agent config
- Splice compatible variants into live agents
### Step 4: Drift control

- Max divergence from canonical per generation
- Quarantine pathological genomes
## Acceptance Criteria

- [ ] Canonical genomes immutable
- [ ] Candidate expression deterministic
- [ ] Drift bounded
- [ ] Selection uses Continuum scores
- [ ] All tests pass with `vitest run`
- [ ] Expression ≤ 10 ms
- [ ] Quarantine works
- [ ] Version rollback supported
## Architecture

```mermaid
flowchart LR
	CAN[Canonical] --> EXP[Express]
	CAND[Candidates] --> EXP
	EXP --> AGENT[Live Agent]
	AGENT --> CONT[Continuum score]
	CONT -. feedback .-> CAND
```

## Role Catalog (seed)

*[table: 4 columns]*

## Extended Types

```rust
pub struct Fitness { pub success_rate: f32, pub cost_ratio: f32, pub novelty: f32 }
```

## Reference — Express

```rust
pub fn express(g: &RoleGenome) -> AgentConfig {
    let mut cfg = AgentConfig::default();
    for s in &g.segments { translate::apply(&mut cfg, s); }
    cfg
}
```

## Observability

- `genome.express_total` counter
- `genome.fitness.latest` gauge per role
- `genome.quarantine_total` counter
## Security

- Genome supply-chain signed
- Splicing capability-gated
- Drift rate capped
## Failure Modes

*[table: 3 columns]*

## Operational Runbook

1. **List:** `genome ls --role planner`.
1. **Promote:** `genome promote planner-003`.
1. **Rollback:** `genome rollback planner`.
## Integration

- Consumed by Agent Runtime (BUILD-69)
- Evolved via Continuum (BUILD-65)
## FAQ

> **Why not hard-code roles?** Roles drift with task distribution; evolution adapts faster.

## Changelog

- v0.1.0 — catalog, expression, drift
- v0.2.0 (planned) — cross-role splicing
- v0.3.0 (planned) — meta-genomes

