# BUILD-75 — Build DSL

> Source: [https://notion.so/65336ed5ce2247c6a9f579deac1d9f16](https://notion.so/65336ed5ce2247c6a9f579deac1d9f16)
> Created: 2026-04-20T18:35:00.000Z | Last edited: 2026-04-20T20:10:00.000Z



---
> **ℹ **Tier 14 · Language · User-facing · Priority: MEDIUM****

  Human-authorable DSL that compiles to Cog ISA (BUILD-84). Blends declarative plans with constraints (budget, class, deadline).

## Fold Provenance

*[table: 2 columns]*

## Purpose

Let operators (and LLM planners) express intent at a higher level than raw ISA. Reduces errors; enables version control of cognition recipes.

## Dependencies

- **BUILD-84, BUILD-31** (ancestors)
## File Structure

```javascript
crates/swarm-dsl/
├── src/
│   ├── lex/
│   ├── parse/
│   ├── typecheck/
│   ├── lower/
│   └── types.rs
```

## DSL Example

```plain text
plan "answer_q" budget=10crc deadline=2s {
  let ctx = retrieve(query);
  par {
    a := plan(ctx);
    b := verify(ctx);
  }
  return merge(a, b);
}
```

## Interfaces & Types

```rust
pub struct Plan { pub name: String, pub budget: CRC, pub deadline: Duration, pub body: Block }
```

## Implementation SOP

1. Parse to AST.
1. Typecheck (registers, atomic fn sigs).
1. Lower to ISA Program.
1. Cache (BUILD-84 cache).
## Acceptance Criteria

- [ ] Lex/parse/typecheck/lower pipeline
- [ ] Good error messages
- [ ] LSP support
- [ ] Versioned DSL (semver)
- [ ] All tests pass with `vitest run`
- [ ] Round-trip: DSL → ISA → DSL
- [ ] Budget+deadline in metadata
- [ ] ISA golden tests
## Architecture

```mermaid
flowchart LR
	SRC[.swarm] --> LEX[Lex]
	LEX --> PARSE[Parse]
	PARSE --> TC[Typecheck]
	TC --> LOW[Lower → ISA]
```

## Statement Set

*[table: 2 columns]*

## Extended Types

```rust
pub enum Diag { ParseErr(Span, String), TypeErr(Span, String) }
```

## Reference — Compile

```rust
pub fn compile(src: &str) -> Result<Program> {
    let ast = parse::all(src)?;
    let ty = typecheck::run(&ast)?;
    lower::to_isa(&ty)
}
```

## Observability

- `dsl.compile_ms` histogram
- `dsl.errors_total` by class
## Security

- Syntactic gates on dangerous fns
- LSP surfaces capability requirements
## Failure Modes

*[table: 3 columns]*

## Operational Runbook

1. **Compile:** `swarmc plan.swarm`.
1. **Disasm:** `swarmc --emit isa plan.swarm`.
## Integration

- Emits ISA (BUILD-84)
- CLI integration (BUILD-31)
## FAQ

> **Can LLMs write this?** Yes — the DSL was designed to be easy for both humans and LLMs.

## Changelog

- v0.1.0 — lex, parse, typecheck, lower, LSP
- v0.2.0 (planned) — macros
- v0.3.0 (planned) — formal semantics proof

