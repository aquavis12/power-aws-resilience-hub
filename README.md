# AWS Resilience Hub — Kiro Power

A Kiro Power that lets your agent assess, improve, and govern AWS application resilience directly from the IDE — powered by the **next-generation AWS Resilience Hub** (GA May 2026) via the managed AWS MCP Server.

[![Add to Kiro](https://img.shields.io/badge/Add%20to-Kiro-8B5CF6?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://kiro.dev/powers/import?repo=https://github.com/aquavis12/power-aws-resilience-hub)

> One-click install: click **Add to Kiro** above, or in the IDE go to Powers → Add Custom Power → Import from GitHub and paste the repo URL.

It's an **umbrella power**: Resilience Hub is the assessment engine, and it folds in the two capabilities it works with —
- **Amazon Application Recovery Controller (ARC)** — the failover *fix* (zonal shift, routing controls, Region switch)
- **AWS Well-Architected Reliability pillar** — the *standard* it assesses against

## What it does
- Next-gen GenAI-powered failure-mode assessments + automatic dependency discovery
- RTO/RPO policy validation against the four DR strategy tiers
- ARC failover orchestration with production-traffic safety guardrails
- Maps every finding to a remediating service + a Well-Architected Reliability best-practice ID
- Optional CI/CD gate so resilience regressions fail the build

## Install (local test)
1. Open Kiro → **Powers** panel → **Add Custom Power**
2. Choose **Import power from a folder** and select this directory
3. Activate by using keywords like *"resilience", "RTO/RPO", "failover", "zonal shift", "disaster recovery"* in a conversation

## Install (from GitHub)
Add Custom Power → **Import power from GitHub** → paste the repo URL.

## Structure
```
power-aws-resilience-hub/
├── POWER.md                          # Metadata, onboarding, steering mappings, ground rules
├── mcp.json                          # AWS MCP server (aws-mcp) config
├── steering/
│   ├── assessment-workflow.md        # Onboard, discover, assess, report, CI/CD gate
│   ├── arc-failover.md               # Zonal shift / routing controls / Region switch (with safety)
│   └── wa-reliability.md             # DR tier selection + Reliability best-practice mapping
└── context-templates/                # User-owned context (RTO/RPO, inventory, dependencies)
    ├── README.md
    ├── rto-rpo-targets.md
    ├── workload-inventory.csv
    └── dependency-notes.md
```

## Safety posture
Read/assess by default. Failover actions (ARC routing controls, zonal shift) move production traffic and **always require explicit user confirmation**. Recovery relies on the data plane, never the control plane (Well-Architected REL11-BP04).

## Author
**Venkata Pavan Vishnu Rachapudi** — AWS Community Builder (Security), 14× AWS Certified. [github.com/aquavis12](https://github.com/aquavis12)

## License
MIT
