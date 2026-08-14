<p align="center">
  <img src="assets/banner.png" alt="AWS Resilience Hub + Amazon ARC — Kiro Power" width="100%">
</p>

<h1 align="center">AWS Resilience Hub + ARC — Kiro Power</h1>

<p align="center">
  Assess, improve, and govern AWS application resilience from inside the IDE.<br>
  Next-gen <b>AWS Resilience Hub</b> (GA May 2026) and <b>Amazon ARC</b> failover, wired through four MCP servers.
</p>

---

## Install

**Kiro → Powers panel → Add Custom Power → Import power from GitHub**

```
https://github.com/aquavis12/power-aws-resilience-hub
```

Kiro reads `plugin.json` + `mcp.json` and installs automatically. The power activates on keywords like *resilience, RTO/RPO, failover, disaster recovery, assessment, backup, pricing, cost, report*.

**Prerequisites:** AWS CLI ≥ 2.32.0 · `uvx` installed · a working `aws sts get-caller-identity`.

## What it does

- **GenAI failure-mode assessments** with automatic dependency discovery, plus RTO/RPO validation against the four DR tiers
- **ARC failover orchestration** — zonal shift, routing controls, Region switch
- **Enterprise remediation reports** — full resource ARNs, per-disruption-layer findings, operational gaps (alarms/SOPs/FIS), console URLs, prioritized roadmap
- **Cost-quantified fixes** — unit rate × quantity = monthly total, validated against the live AWS Price List API
- Every finding maps to DR tier, target RTO/RPO, remediating service, Well-Architected best-practice ID, cost, and a docs link
- Cross-checks assessment scope against live infrastructure to catch resources the tag filter missed, and flags false-positive "stateless" classifications
- Optional CI/CD gate so resilience regressions fail the build

## Skills

| Skill | What it does |
|-------|-------------|
| `assess` | Full assessment for one app — validate, discover, assess, report with WAF best-practice IDs |
| `account-scan` | Account-wide multi-region scan — all apps, unregistered workloads, aggregated posture |
| `failover` | ARC zonal shift / routing controls / Region switch, with explicit safety confirmation |
| `report` | Enterprise remediation report from an existing assessment — ARNs, costs, gaps, roadmap |

## MCP servers

| Server | Purpose | Package |
|--------|---------|---------|
| `aws-mcp` | All AWS API calls — Resilience Hub, ARC, STS, Backup, DRS | `mcp-proxy-for-aws` |
| `aws-docs` | Service guides, best practices, console paths | `awslabs.aws-documentation-mcp-server` |
| `aws-repost` | Community solutions, known issues, real-world patterns | `awslabs.aws-repost-mcp-server` |
| `aws-pricing` | Live Price List data for remediation cost validation | `awslabs.aws-pricing-mcp-server` |

## Safety

Read and assess by default. Failover actions move production traffic and **always require explicit confirmation**. Recovery runs through the data plane, never the control plane (Well-Architected REL11-BP04).

## Changelog

**v4** — AWS Pricing MCP, enterprise `report` skill, cost-quantified recommendations, pricing/cost/ARN keywords.
**v3** — Report skill, docs + re:Post MCP enrichment, cross-infrastructure validation.

## License

MIT
