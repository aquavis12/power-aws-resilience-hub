# Steering: Well-Architected Reliability & DR Strategy Selection

Load this when choosing or validating a DR strategy tier and mapping resilience findings to AWS Well-Architected Reliability-pillar best practices.

## MCP servers available

- **aws-docs** — Pull the full Well-Architected Reliability pillar documentation, service-specific DR guides, and pricing pages to validate tier selection.
- **aws-repost** — Search for community experiences with DR strategy implementations, known limitations, and cost optimization patterns.

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
- **REL06 — Monitor workload resources** to detect issues before they impact customers (alarms, synthetics, dashboards).
- **REL08 — Implement change management** with deployment strategies that allow fast rollback (alias routing, blue/green, canary).
- **REL09 — Back up data** with tested restore procedures, cross-region copies, and vault lock for immutability.
- **REL10 — Use fault isolation (AZs, Regions, cells)** to limit blast radius.
- **REL11 — Design to withstand component failures** (health checks, automatic healing, failover).
- **REL11-BP04 — Rely on the data plane, not the control plane, during recovery.** Recovery must use a minimal number of control-plane operations. This is why ARC routing controls (data plane) are preferred over control-plane reconfiguration. *Non-negotiable.*
- **REL12 — Test reliability** through fault injection (FIS), game days, and DR drills.
- **REL13 — Plan for disaster recovery** with defined, **tested** RTO/RPO objectives.

## Mapping findings to fixes
When Resilience Hub / GenAI assessment surfaces a failure mode, map it:
- Single-AZ dependency → REL10 → **ARC zonal shift** (`steering/arc-failover.md`)
- No cross-Region failover path → REL11-BP04 / REL13 → **ARC routing controls / Region switch**
- Missing/untested backups → REL09 → **AWS Backup** (vault lock, cross-region copy)
- Stateful server with no replication → REL13 → **AWS DRS** (Pilot Light / Warm Standby)
- RTO/RPO never tested → REL12 → schedule a **DR drill** (DRS drill, ARC failover test)
- No observability → REL06 → **CloudWatch alarms** + **SSM Automation runbooks**
- No rollback mechanism → REL08 → **Lambda aliases / CodeDeploy blue-green / ECS rolling**

## Validation baseline
Before a formal assessment, consider a configuration baseline scan (e.g. AWS Service Screener v2) to catch obvious reliability misconfigurations, then let Resilience Hub do the deep failure-mode analysis.

## Documentation enrichment
When producing recommendations:
- Use `aws-docs` to fetch the official documentation page for each remediating service (Backup, DRS, ARC, DLM, etc.).
- Use `aws-repost` to check for known issues, community workarounds, or cost optimization tips related to the recommended fix.
- Include links in the final report so the user has a direct path to implementation docs.

## Reporting rule
Every recommendation you output must state: **tier + target RTO/RPO + remediating service + WAF best-practice ID + estimated cost + doc link**. No hand-waving "make it highly available".
