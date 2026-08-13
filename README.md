# AWS Resilience Hub — Kiro Power

A Kiro Power that lets your agent assess, improve, and govern AWS application resilience directly from the IDE — powered by the **next-generation AWS Resilience Hub** (GA May 2026) via the managed AWS MCP Server, with integrated AWS Documentation, re:Post community knowledge, and AWS Pricing API cost validation.

## Install in Kiro

**Kiro → Powers panel → Add Custom Power → Import power from GitHub**, then paste:

```
https://github.com/aquavis12/power-aws-resilience-hub
```

Kiro reads `plugin.json` + `mcp.json` and installs the power automatically. Activate it by using keywords like *"resilience", "RTO/RPO", "failover", "disaster recovery", "assessment", "backup", "recovery", "pricing", "cost", "report"* in a conversation.

## What it does

An **umbrella power** combining four tightly-integrated capabilities:

| Capability | What | MCP Server |
|-----------|------|------------|
| **AWS Resilience Hub** | GenAI failure-mode assessments, dependency discovery, RTO/RPO policy validation, structured reporting | `aws-mcp` |
| **Amazon ARC** | Failover orchestration — zonal shift, routing controls, Region switch | `aws-mcp` |
| **Well-Architected Reliability** | The standard assessments grade against | `aws-docs` |
| **AWS Documentation** | Authoritative remediation guidance, console paths, service references | `aws-docs` |
| **AWS re:Post** | Community solutions, known issues, real-world patterns | `aws-repost` |
| **AWS Pricing** | Live cost validation, unit-level pricing breakdowns, remediation cost enrichment | `aws-pricing` |

### Core features

- Next-gen GenAI-powered failure-mode assessments + automatic dependency discovery
- RTO/RPO policy validation against the four DR strategy tiers
- **Enterprise-grade remediation reporting** — executive summary, full resource ARNs, per-component findings across all disruption layers, cost-quantified recommendations validated against AWS Pricing API, operational gaps (alarms/SOPs/FIS tests), direct Resilience Hub console URLs, hidden stateless-classification risks, and a prioritized remediation roadmap
- ARC failover orchestration with production-traffic safety guardrails
- Maps every finding to: DR tier + target RTO/RPO + remediating service + WAF best-practice ID + estimated cost (Pricing API validated) + documentation link
- Cross-references assessment scope against live infrastructure to catch missed resources
- Flags false-positive "stateless" classifications on clearly stateful workloads
- Optional CI/CD gate so resilience regressions fail the build

## Prerequisites

- **AWS CLI >= 2.32.0** — `aws --version`
- **uv/uvx installed** — `uvx --version` (the MCP servers launch via uvx)
- **Active AWS session** — `aws sts get-caller-identity` should succeed

## Skills

| Skill | What it does |
|-------|-------------|
| **assess** | Run a full Resilience Hub assessment for a single app — validate, discover, assess, report with WAF best-practice IDs |
| **account-scan** | Account-wide multi-region resilience scan — all apps, unregistered workloads, aggregated posture report |
| **failover** | ARC failover orchestration — zonal shift, routing controls, or Region switch with explicit safety confirmation |
| **report** | Generate enterprise-grade remediation report from an existing assessment — resource ARNs, cost-quantified recommendations (AWS Pricing API), operational gaps, console URLs, prioritized roadmap |

## MCP Servers

| Server | Purpose | Package |
|--------|---------|---------|
| `aws-mcp` | All AWS API operations (Resilience Hub, ARC, STS, Backup, DRS, etc.) | `mcp-proxy-for-aws` |
| `aws-docs` | AWS Documentation lookups — service guides, best practices, console paths | `awslabs.aws-documentation-mcp-server` |
| `aws-repost` | AWS re:Post search — community solutions, known issues, implementation patterns | `awslabs.aws-repost-mcp-server` |
| `aws-pricing` | AWS Price List API — live cost data, unit pricing, remediation cost validation | `awslabs.aws-pricing-mcp-server` |

## Structure

```
power-aws-resilience-hub/
├── plugin.json                           # Power manifest (name, version 4.0.0, keywords)
├── POWER.md                              # Agent instructions, ground rules, onboarding
├── mcp.json                              # Four MCP servers: aws-mcp, aws-docs, aws-repost, aws-pricing
├── skills/
│   ├── assess/SKILL.md                   # Single-app resilience assessment
│   ├── account-scan/SKILL.md             # Account-wide multi-region scan
│   ├── failover/SKILL.md                 # ARC failover orchestration
│   └── report/SKILL.md                   # Enterprise-grade remediation reporting
├── steering/                             # Steering files (agent guidance)
│   ├── assessment-workflow.md            # Onboard, discover, assess, report, CI/CD gate
│   ├── arc-failover.md                   # Zonal shift / routing controls / Region switch
│   └── wa-reliability.md                 # DR tier selection + Reliability best-practice mapping
└── context-templates/                    # User-owned context templates
    ├── README.md
    ├── rto-rpo-targets.md
    ├── workload-inventory.csv
    └── dependency-notes.md
```

## Safety posture

Read/assess by default. Failover actions (ARC routing controls, zonal shift) move production traffic and **always require explicit user confirmation**. Recovery relies on the data plane, never the control plane (Well-Architected REL11-BP04).

## What's new in v4

- **AWS Pricing MCP** — Remediation cost estimates validated against live AWS Price List API data with unit-level breakdowns
- **Enterprise-grade report skill** — Full resource ARNs, direct Resilience Hub console URLs, per-disruption-layer analysis, operational gap coverage percentages, and prioritized roadmap
- **Cost-quantified recommendations** — Every fix includes unit rate × quantity = monthly total, cross-referenced between Resilience Hub estimates and Pricing API
- **Expanded keywords** — Power activates on pricing, cost, ARN in addition to existing resilience terms
- **Four MCP servers** — aws-mcp, aws-docs, aws-repost, and aws-pricing working together

### Previous highlights (v3)

- Report skill with executive summary and component-level findings
- AWS Documentation MCP for authoritative remediation links
- AWS re:Post MCP for community-validated solutions
- Cross-infrastructure validation catching tag-filter gaps
- Enhanced steering with docs/rePost enrichment guidance

## License

MIT
