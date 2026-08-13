# AWS Resilience Hub — Kiro Power

A Kiro Power that lets your agent assess, improve, and govern AWS application resilience directly from the IDE — powered by the **next-generation AWS Resilience Hub** (GA May 2026) via the managed AWS MCP Server, with integrated AWS Documentation and re:Post community knowledge.

## Install in Kiro

**Kiro → Powers panel → Add Custom Power → Import power from GitHub**, then paste:

```
https://github.com/aquavis12/power-aws-resilience-hub
```

Kiro reads `plugin.json` + `mcp.json` and installs the power automatically. Activate it by using keywords like *"resilience", "RTO/RPO", "failover", "disaster recovery", "assessment", "backup", "recovery"* in a conversation.

## What it does

An **umbrella power** combining three tightly-integrated capabilities:

| Capability | What | MCP Server |
|-----------|------|------------|
| **AWS Resilience Hub** | GenAI failure-mode assessments, dependency discovery, RTO/RPO policy validation, structured reporting | `aws-mcp` |
| **Amazon ARC** | Failover orchestration — zonal shift, routing controls, Region switch | `aws-mcp` |
| **Well-Architected Reliability** | The standard assessments grade against | `aws-docs` |
| **AWS Documentation** | Authoritative remediation guidance, console paths, service references | `aws-docs` |
| **AWS re:Post** | Community solutions, known issues, real-world patterns | `aws-repost` |

### Core features

- Next-gen GenAI-powered failure-mode assessments + automatic dependency discovery
- RTO/RPO policy validation against the four DR strategy tiers
- **Structured remediation reporting** — executive summary, per-component findings, cost-quantified recommendations, operational gaps (alarms/SOPs/FIS tests), prioritized roadmap
- ARC failover orchestration with production-traffic safety guardrails
- Maps every finding to: DR tier + target RTO/RPO + remediating service + WAF best-practice ID + estimated cost + documentation link
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
| **report** | Generate structured remediation report from an existing assessment — enriched with AWS docs and re:Post community knowledge |

## MCP Servers

| Server | Purpose | Package |
|--------|---------|---------|
| `aws-mcp` | All AWS API operations (Resilience Hub, ARC, STS, Backup, DRS, etc.) | `mcp-proxy-for-aws` |
| `aws-docs` | AWS Documentation lookups — service guides, best practices, console paths | `awslabs.aws-documentation-mcp-server` |
| `aws-repost` | AWS re:Post search — community solutions, known issues, implementation patterns | `awslabs.aws-repost-mcp-server` |

## Structure

```
power-aws-resilience-hub/
├── plugin.json                           # Power manifest (name, version 3.0.0, keywords)
├── POWER.md                              # Agent instructions, ground rules, onboarding
├── mcp.json                              # Three MCP servers: aws-mcp, aws-docs, aws-repost
├── skills/
│   ├── assess/SKILL.md                   # Single-app resilience assessment
│   ├── account-scan/SKILL.md             # Account-wide multi-region scan
│   ├── failover/SKILL.md                 # ARC failover orchestration
│   └── report/SKILL.md                   # Structured remediation reporting
├── dev.kiro/steering/                    # Steering files (dev context)
│   ├── assessment-workflow.md            # Onboard, discover, assess, report, CI/CD gate
│   ├── arc-failover.md                   # Zonal shift / routing controls / Region switch
│   └── wa-reliability.md                 # DR tier selection + Reliability best-practice mapping
├── steering/                             # Steering files (power distribution)
│   ├── assessment-workflow.md
│   ├── arc-failover.md
│   └── wa-reliability.md
├── context-templates/                    # User-owned context templates
│   ├── README.md
│   ├── rto-rpo-targets.md
│   ├── workload-inventory.csv
│   └── dependency-notes.md
└── resilience-context/                   # Generated reports and filled-in context
    ├── account-posture-report.md
    ├── cross-scout-assessment-report.md
    └── ...
```

## Safety posture

Read/assess by default. Failover actions (ARC routing controls, zonal shift) move production traffic and **always require explicit user confirmation**. Recovery relies on the data plane, never the control plane (Well-Architected REL11-BP04).

## What's new in v3

- **Report skill** — Generate structured assessment reports with executive summary, component-level findings, cost-quantified remediation, and operational gap analysis
- **AWS Documentation MCP** — Every recommendation backed by authoritative AWS docs with direct links
- **AWS re:Post MCP** — Community-validated solutions and known gotchas surfaced alongside recommendations
- **Expanded keywords** — Power activates on backup, recovery, report, assessment, remediation, PITR, multi-AZ, cross-region
- **Enhanced steering** — All steering files now guide the agent to use docs/rePost for enrichment
- **Cross-infrastructure validation** — Assessment scope checked against live resources to catch tag-filter gaps

## License

MIT

