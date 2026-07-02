# RTO / RPO Targets

Define the resilience contract for each workload. The agent uses these as the assessment policy targets and to pick the cheapest DR tier that meets them.

## Disruption types (set targets for each)
- **Application** — bug/bad deploy in the app itself
- **Infrastructure (AZ)** — single Availability Zone impairment
- **Region** — full Regional impairment
- **Data** — corruption / accidental deletion (RPO-driven)

## Per-workload targets

| Workload | Business tier | Target RTO | Target RPO | DR strategy tier | Notes |
|----------|---------------|-----------|-----------|------------------|-------|
| `<app-name>` | `<critical/important/standard>` | `<e.g. 15 min>` | `<e.g. 5 min>` | `<Backup&Restore / Pilot Light / Warm Standby / Active-Active>` | `<...>` |

## Tier reference
- Backup & Restore — RTO < 24h, RPO hours
- Pilot Light — RTO ~10 min, RPO minutes
- Warm Standby — RTO minutes, RPO seconds
- Active-Active — RTO ~0, RPO ~0

> Rule: pick the **cheapest tier that meets the required RTO/RPO**. Don't over-provision resilience.
