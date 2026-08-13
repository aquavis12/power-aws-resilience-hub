---
name: "aws-resilience-hub"
displayName: "AWS Resilience Hub (next-gen)"
description: "Assess, improve, and govern the resilience of AWS applications from your IDE — next-gen Resilience Hub GenAI failure-mode assessments, dependency discovery, RTO/RPO policy validation, Amazon Application Recovery Controller (ARC) failover orchestration, and AWS Well-Architected Reliability-pillar guidance — via the managed AWS MCP Server."
keywords: ["resilience", "resilience hub", "disaster recovery", "dr", "rto", "rpo", "failover", "arc", "application recovery controller", "zonal shift", "routing control", "reliability", "well-architected", "high availability", "resiliency", "failure mode"]
author: "Venkata Pavan Vishnu Rachapudi"
---

# AWS Resilience Hub Power

This power lets the Kiro agent operate **next-generation AWS Resilience Hub** (GA May 2026) — AWS's service for proactively assessing and improving application resilience — through the managed **AWS MCP Server** (`aws-mcp`). It folds in two closely-related capabilities:

- **Amazon Application Recovery Controller (ARC)** — the *fix* Resilience Hub recommends for failover: zonal shift, routing controls, and Region switch.
- **AWS Well-Architected Reliability pillar** — the *standard* Resilience Hub assesses against.

## Ground rules (always apply)

1. **Assess, don't mutate**: This power's default posture is read/assess. Never create, modify, or delete workload infrastructure (EC2, RDS, Lambda, networking, DNS). Permitted writes are limited to Resilience Hub resources (apps, policies, assessments), ARC recovery-readiness resources, and workspace state files.
2. **Failover actions are high-risk — always confirm**: ARC routing-control state changes and zonal shifts can move production traffic. NEVER trigger a failover, zonal shift, or routing-control flip without showing the user the exact target state and getting explicit confirmation. Prefer `--dry-run` / readiness-check reads first.
3. **Data plane over control plane during recovery**: Per Well-Architected REL11-BP04, recovery actions must rely on the **data plane** (ARC routing-control data plane, Route 53 health checks), not control-plane APIs. Never recommend a recovery path that depends on a control-plane operation succeeding during an impairment.
4. **Discover, don't assume**: Next-gen Resilience Hub is newly GA and its API surface differs from the legacy hub. Before the first Resilience Hub operation in a session, discover the actual operation names via the `aws-mcp` tools. NEVER invent operation names — verify, then call.
5. **RTO/RPO are contractual**: Always express resilience targets as explicit RTO (recovery time) and RPO (data loss) numbers tied to a DR strategy tier. Never hand-wave "highly available" — map it to a tier (see `context-templates/rto-rpo-targets.md`).
6. **One MCP server**: Use only `aws-mcp`. Do NOT add the legacy `aws-api-mcp-server` or `aws-knowledge-mcp-server` alongside it — AWS recommends removing them to avoid tool conflicts.

## Onboarding

### Step 1: Validate tools work

Before using this power, ensure the following:

- **AWS CLI >= 2.32.0**: Verify with `aws --version`
- **uv/uvx installed**: Verify with `uvx --version` (the MCP proxy launches via uvx)
- **Active AWS session**: This power authenticates via SigV4 through `mcp-proxy-for-aws`. Recommend `aws login` (auto-rotates credentials, valid up to 12 h). Verify with `aws sts get-caller-identity` and confirm the account ID with the user.
- **CRITICAL**: If credentials are missing or expired (`ExpiredTokenException`), DO NOT proceed — instruct the user to run `aws login` (or `aws sso login --profile <name>`) and restart Kiro.

### Step 2: Verify the Resilience Hub service is reachable

Discover `resiliencehub` (and `arc` / `route53-recovery-*`) operations via the `aws-mcp` tools. Confirm the caller has resilience read permissions, then check whether an application is already registered (list-apps) before offering to onboard one.

### Step 3: Set up workspace state

Capture the resilience app ARN, region(s), resilience policy tier, and target RTO/RPO into `.kiro/resilience-hub.json`. Copy `context-templates/` into the user's workspace as `resilience-context/` and offer to help fill the placeholders (workload tier, RTO/RPO targets, dependency inventory).

## Skills

This power provides two skills:

- **assess** — Run a full Resilience Hub assessment: validate session, discover operations, onboard or select an app, run assessment, and report results as a structured table with WAF best-practice IDs.
- **failover** — Orchestrate ARC failover with safety guardrails: zonal shift, routing controls, or Region switch with explicit user confirmation.

## Best Practices

### Tool usage

- All operations go through the `aws-mcp` server tools: Resilience Hub service calls, ARC/Route 53 recovery-controller calls, STS identity checks, and read-only inspection of assessed resources.
- Prefer next-gen Resilience Hub for: dependency discovery, GenAI failure-mode assessments, modular resilience policies, and organization-wide reporting. Use direct service calls only for quick ad-hoc checks.
- Wire assessments into CI/CD as a gate (fail the pipeline if assessed RTO/RPO regresses below policy).

### Reporting conventions

- Every recommendation must name: (1) the DR strategy tier, (2) the target RTO/RPO, (3) the remediating service (ARC / DRS / Backup), and (4) the Well-Architected Reliability best-practice ID it satisfies (e.g. REL11-BP04).
- Frame failure modes by blast radius: component → AZ → Region.

## Troubleshooting

- `ExpiredTokenException` / MCP tools not loading → SigV4 session expired. Re-run `aws login` and restart the MCP client.
- `AccessDeniedException` → caller lacks the Resilience Hub or ARC operator policy. Point the user to the IAM setup guide; never attach policies without explicit instruction.
- Unknown service/operation → next-gen API not yet exposed through the managed server; fall back to console instructions.
- Assessment stuck / empty results → dependency discovery runs asynchronously; poll assessment status until COMPLETED before reading results.
- ARC routing-control change has no effect → confirm you targeted the **data plane** endpoint (region-specific), not the control plane.

## License and support

This power is licensed under MIT (SPDX: `MIT`). It integrates with the AWS MCP Server (`mcp-proxy-for-aws`, SPDX: `Apache-2.0`), a managed AWS service.
- [Privacy Policy](https://aws.amazon.com/privacy/)
- [Support](mailto:rachapudivishnu9@gmail.com) (rachapudivishnu9@gmail.com) · [GitHub Issues](https://github.com/aquavis12/power-aws-resilience-hub/issues)
- Author: Venkata Pavan Vishnu Rachapudi — AWS Community Builder (Security)
