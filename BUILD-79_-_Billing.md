# BUILD-79 — Billing

> Source: [https://notion.so/ed3a01bc421f4485ab04bec4222076d6](https://notion.so/ed3a01bc421f4485ab04bec4222076d6)
> Created: 2026-04-20T18:37:00.000Z | Last edited: 2026-04-20T20:10:00.000Z



---
> **ℹ **Tier 15 · Billing · Cross-scale · Priority: MEDIUM****

  Meters CRC usage per tenant and converts to billable line items. Drives invoices, alerts, and budget guards.

## Fold Provenance

*[table: 2 columns]*

## Purpose

Turn raw CRC records into billable usage. Enables tiered pricing, committed-use discounts, and trustable invoices.

## Dependencies

- **BUILD-47, BUILD-90, BUILD-96** (ancestors)
## File Structure

```javascript
crates/billing/
├── src/
│   ├── meter/
│   │   ├── collect.rs
│   │   └── aggregate.rs
│   ├── price/
│   │   ├── list.rs
│   │   └── discount.rs
│   ├── invoice/
│   │   └── render.rs
│   └── types.rs
```

## Interfaces & Types

```rust
pub struct UsageLine { pub tenant: TenantId, pub sku: String, pub qty: f64, pub unit: String }
```

## Implementation SOP

1. Meter: stream Provenance events → tenant bucket.
1. Aggregate: daily SKU totals.
1. Price: apply rate card + discounts.
1. Invoice: render, sign, dispatch.
## Acceptance Criteria

- [ ] End-to-end reconciliation to CRC
- [ ] Rate card versioned
- [ ] Discounts auditable
- [ ] Invoice re-derivable
- [ ] All tests pass with `vitest run`
- [ ] Dispute workflow
- [ ] Prepay + postpay supported
- [ ] Currency-agnostic
## Architecture

```mermaid
flowchart LR
	PROV[Provenance] --> MET[Meter]
	MET --> AGG[Aggregate]
	AGG --> PRICE[Price]
	PRICE --> INV[Invoice]
```

## SKU Catalog (seed)

*[table: 3 columns]*

## Extended Types

```rust
pub struct RateCard { pub version: Semver, pub entries: Vec<(String, f64)> }
```

## Reference — Aggregate

```rust
pub async fn aggregate(day: Date) -> Vec<UsageLine> { meter::sum_by_tenant_sku(day).await }
```

## Observability

- `billing.usage_total` by tenant/sku
- `billing.invoices_total`, `disputes_total`
## Security

- Invoices signed
- Rate-card change requires dual-approval
## Failure Modes

*[table: 3 columns]*

## Operational Runbook

1. **Run:** `bill run --day 2025-01-01`.
1. **Dispute:** `bill dispute --invoice <i>`.
## Integration

- Input: Provenance; Output: external billing system
## FAQ

> **Can tenants self-serve usage?** Yes — read-only API.

## Changelog

- v0.1.0 — meter, price, invoice
- v0.2.0 (planned) — committed-use
- v0.3.0 (planned) — real-time meter

