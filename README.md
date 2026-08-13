# AWS Resilience Hub — Kiro Power

A Kiro Power that lets your agent assess, improve, and govern AWS application resilience directly from the IDE — powered by the **next-generation AWS Resilience Hub** (GA May 2026) via the managed AWS MCP Server.

## Install in Kiro

**Kiro → Powers panel → Add Custom Power → Import power from GitHub**, then paste:

```
https://github.com/aquavis12/power-aws-resilience-hub
```

Kiro reads `plugin.json` + `mcp.json` and installs the power automatically. Activate it by using keywords like *"resilience", "RTO/RPO", "failover", "zonal shift", "disaster recovery"* in a conversation.

It's an **umbrella power**: Resilience Hub is the assessment engine, and it folds in the two capabilities it works with —
- **Amazon Application Recovery Controller (ARC)** — the failover *fix* (zonal shift, routing controls, Region switch)
- **AWS Well-Architected Reliability pillar** — the *standard* it assesses against

## What it does

- Next-gen GenAI-powered failure-mode assessments + automatic dependency discovery
- RTO/RPO policy validation against the four DR strategy tiers
- ARC failover orchestration with production-traffic safety guardrails
- Maps every finding to a remediating service + a Well-Architected Reliability best-practice ID
- Optional CI/CD gate so resilience regressions fail the build

## Prerequisites

- **AWS CLI >= 2.32.0** — `aws --version`
- **uv/uvx installed** — `uvx --version` (the MCP proxy launches via uvx)
- **Active AWS session** — `aws sts get-caller-identity` should succeed

## Install (local test)

1. Open Kiro → **Powers** panel → **Add Custom Power**
2. Choose **Import power from a folder** and select this directory
3. Activate by using keywords like *"resilience", "RTO/RPO", "failover", "zonal shift", "disaster recovery"* in a conversation

## Install (from GitHub)

Add Custom Power → **Import power from GitHub** → paste the repo URL.

## Structure

```
power-aws-resilience-hub/
├── plugin.json                           # Agent Plugins manifest (name, keywords, author)
├── POWER.md                              # Agent instructions, ground rules, onboarding
├── mcp.json                              # AWS MCP server (aws-mcp) config
├── skills/
│   ├── assess/
│   │   └── SKILL.md                      # Run resilience assessment workflow
│   └── failover/
│       └── SKILL.md                      # ARC failover orchestration with safety
├── dev.kiro/
│   └── steering/
│       ├── assessment-workflow.md         # Onboard, discover, assess, report, CI/CD gate
│       ├── arc-failover.md               # Zonal shift / routing controls / Region switch
│       └── wa-reliability.md             # DR tier selection + Reliability best-practice mapping
├── steering/                             # Legacy steering location (backward compat)
│   ├── assessment-workflow.md
│   ├── arc-failover.md
│   └── wa-reliability.md
└── context-templates/                    # User-owned context (RTO/RPO, inventory, dependencies)
    ├── README.md
    ├── rto-rpo-targets.md
    ├── workload-inventory.csv
    └── dependency-notes.md
```

## Safety posture

Read/assess by default. Failover actions (ARC routing controls, zonal shift) move production traffic and **always require explicit user confirmation**. Recovery relies on the data plane, never the control plane (Well-Architected REL11-BP04).

## Author

**Venkata Pavan Vishnu Rachapudi** — AWS Community Builder (Security), 14x AWS Certified. [github.com/aquavis12](https://github.com/aquavis12)

## License

MIT
