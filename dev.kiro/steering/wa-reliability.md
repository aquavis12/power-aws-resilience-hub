# Steering: Well-Architected Reliability & DR Strategy Selection

Load this when choosing or validating a DR strategy tier and mapping resilience findings to AWS Well-Architected Reliability-pillar best practices.

## The four DR strategy tiers
Increasing cost/complexity, decreasing RTO/RPO:

| Tier | RTO | RPO | How | Remediating services |
|------|-----|-----|-----|----------------------|
| **Backup & Restore** | < 24h | hours | Restore from backups into recovery Region | AWS Backup |
| **Pilot Light** | ~10 min | minutes | Core data replicated, minimal always-on; scale up on failover | AWS DRS, AWS Backup |
| **Warm Standby** | minutes | seconds | Scaled-down full stack always running; scale up + shift traffic | AWS DRS, Amazon ARC |
| **Active-Active** | ~0 | ~0 | Both Regions serve traffic; failover = drain one Region | Amazon ARC (routing controls, Region switch) |

Pick the **cheapest tier that meets the RTO/RPO the business actually requires** — over-provisioning resilience is a common waste driver. Record the chosen tier in `.kiro/resilience-hub.json`.

## Key Reliability-pillar best practices to check
- **REL11-BP04 — Rely on the data plane, not the control plane, during recovery.** Recovery must use a minimal number of control-plane operations. This is why ARC routing controls (data plane) are preferred over control-plane reconfiguration. *Non-negotiable.*
- **REL10 — Use fault isolation (AZs, Regions, cells)** to limit blast radius.
- **REL11 — Design to withstand component failures** (health checks, automatic healing, failover).
- **REL13 — Plan for disaster recovery** with defined, **tested** RTO/RPO objectives.

## Mapping findings to fixes
When Resilience Hub / GenAI assessment surfaces a failure mode, map it:
- Single-AZ dependency → REL10 → **ARC zonal shift** (`steering/arc-failover.md`)
- No cross-Region failover path → REL11-BP04 / REL13 → **ARC routing controls / Region switch**
- Missing/untested backups → REL13 → **AWS Backup** (vault lock, cross-region copy)
- Stateful server with no replication → REL13 → **AWS DRS** (Pilot Light / Warm Standby)
- RTO/RPO never tested → REL13 → schedule a **DR drill** (DRS drill, ARC failover test)

## Validation baseline
Before a formal assessment, consider a configuration baseline scan (e.g. AWS Service Screener v2) to catch obvious reliability misconfigurations, then let Resilience Hub do the deep failure-mode analysis.

## Reporting rule
Every recommendation you output must state: **tier + target RTO/RPO + remediating service + WAF best-practice ID**. No hand-waving "make it highly available".
