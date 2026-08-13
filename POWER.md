---
name: "aws-resilience-hub"
displayName: "AWS Resilience Hub"
description: "Assess, improve, and govern the resilience of AWS applications from your IDE — next-gen Resilience Hub GenAI failure-mode assessments, dependency discovery, RTO/RPO policy validation, structured remediation reporting, Amazon Application Recovery Controller (ARC) failover orchestration, and AWS Well-Architected Reliability-pillar guidance — powered by the managed AWS MCP Server with integrated AWS Documentation and re:Post knowledge."
keywords: ["resilience", "resilience hub", "disaster recovery", "dr", "rto", "rpo", "failover", "arc", "application recovery controller", "zonal shift", "routing control", "reliability", "well-architected", "high availability", "resiliency", "failure mode", "backup", "recovery", "report", "assessment", "remediation", "pitr", "multi-az", "cross-region"]
author: "Venkata Pavan Vishnu Rachapudi"
---

# AWS Resilience Hub Power

This power lets the Kiro agent operate **next-generation AWS Resilience Hub** (GA May 2026) — AWS's service for proactively assessing and improving application resilience — through four integrated MCP servers:

- **aws-mcp** — The managed AWS MCP Server for all Resilience Hub, ARC, and AWS service API calls (SigV4 authenticated).
- **aws-docs** — AWS Documentation MCP Server for pulling authoritative service docs, best-practice guides, and remediation references.
- **aws-repost** — AWS re:Post MCP Server for searching community solutions, known issues, and real-world implementation patterns.
- **aws-pricing** — AWS Pricing MCP Server for querying live AWS Price List API data to validate and enrich cost estimates in remediation recommendations.

It folds in two closely-related capabilities:

- **Amazon Application Recovery Controller (ARC)** — the *fix* Resilience Hub recommends for failover: zonal shift, routing controls, and Region switch.
- **AWS Well-Architected Reliability pillar** — the *standard* Resilience Hub assesses against.

## Ground rules (always apply)

1. **Assess, don't mutate**: This power's default posture is read/assess. Never create, modify, or delete workload infrastructure (EC2, RDS, Lambda, networking, DNS). Permitted writes are limited to Resilience Hub resources (apps, policies, assessments), ARC recovery-readiness resources, and workspace state files.
2. **Failover actions are high-risk — always confirm**: ARC routing-control state changes and zonal shifts can move production traffic. NEVER trigger a failover, zonal shift, or routing-control flip without showing the user the exact target state and getting explicit confirmation. Prefer `--dry-run` / readiness-check reads first.
3. **Data plane over control plane during recovery**: Per Well-Architected REL11-BP04, recovery actions must rely on the **data plane** (ARC routing-control data plane, Route 53 health checks), not control-plane APIs. Never recommend a recovery path that depends on a control-plane operation succeeding during an impairment.
4. **Discover, don't assume**: Next-gen Resilience Hub is newly GA and its API surface differs from the legacy hub. Before the first Resilience Hub operation in a session, discover the actual operation names via the `aws-mcp` tools. NEVER invent operation names — verify, then call.
5. **RTO/RPO are contractual**: Always express resilience targets as explicit RTO (recovery time) and RPO (data loss) numbers tied to a DR strategy tier. Never hand-wave "highly available" — map it to a tier (see `context-templates/rto-rpo-targets.md`).
6. **Enrich with documentation**: Use `aws-docs` to pull authoritative AWS documentation for remediation guidance and `aws-repost` for community-validated solutions. Never invent guidance — cite sources.
7. **MCP server usage**: Use `aws-mcp` for all AWS API operations. Use `aws-docs` for documentation lookups. Use `aws-repost` for community knowledge. Use `aws-pricing` for cost validation and enrichment. Do NOT add legacy servers (`aws-api-mcp-server`, `aws-knowledge-mcp-server`) — they conflict.

## Onboarding

### Step 1: Validate tools work

Before using this power, ensure the following:

- **AWS CLI >= 2.32.0**: Verify with `aws --version`
- **uv/uvx installed**: Verify with `uvx --version` (the MCP proxy launches via uvx)
- **Active AWS session**: This power authenticates via SigV4 through `mcp-proxy-for-aws`. Recommend `aws login` (auto-rotates credentials, valid up to 12 h). Verify with `aws sts get-caller-identity` and confirm the account ID with the user.
- **CRITICAL**: If credentials are missing or expired (`ExpiredTokenException`), DO NOT proceed — instruct the user to run `aws login` (or `aws sso login --profile <name>`) and restart Kiro.

### Step 2: Verify the Resilience Hub service is reachable

Discover `resiliencehub` (and `arc` / `route53-recovery-*`) operations via the `aws-mcp` tools. Confirm the caller has resilience read permissions, then check whether an application is already registered (list-apps) before offering to onboard one.

For account-wide scans, ask the user which regions to cover (or offer to scan all enabled regions). The agent will iterate per-region using the `--region` parameter on each API call.

### Step 3: Set up workspace state

Capture the resilience app ARN, region(s), resilience policy tier, and target RTO/RPO into `.kiro/resilience-hub.json`. Copy `context-templates/` into the user's workspace as `resilience-context/` and offer to help fill the placeholders (workload tier, RTO/RPO targets, dependency inventory).

## Region handling

This power does NOT hardcode a region. The `aws-mcp` proxy inherits the region from your AWS CLI configuration (`AWS_REGION`, `AWS_DEFAULT_REGION`, or profile default). To target a specific region, pass `--region` in each API call. For account-wide scans, the agent iterates across all user-specified regions automatically.

## Skills

This power provides four skills:

- **assess** — Run a full Resilience Hub assessment for a single application: validate session, discover operations, onboard or select an app, run assessment, and report results as a structured table with WAF best-practice IDs.
- **account-scan** — Perform an account-wide resilience assessment across all (or selected) regions: discover all registered apps, assess each, flag unregistered workloads, and produce an aggregated posture report.
- **failover** — Orchestrate ARC failover with safety guardrails: zonal shift, routing controls, or Region switch with explicit user confirmation.
- **report** — Generate a comprehensive, enterprise-grade remediation report from an existing assessment: executive summary, resource ARNs, per-component findings across all disruption layers, cost-quantified recommendations (validated against AWS Pricing API), operational gaps (alarms/SOPs/FIS), direct Resilience Hub console URLs, hidden stateless-classification risks, and a prioritized roadmap — enriched with AWS documentation and re:Post community solutions.

## MCP Server Usage

### aws-mcp (primary)
All AWS API operations: Resilience Hub service calls, ARC/Route 53 recovery-controller calls, STS identity checks, read-only inspection of assessed resources, AWS Backup, DRS, and other service APIs.

### aws-docs (documentation)
Pull authoritative AWS documentation for:
- Service-specific remediation guidance (e.g., "how to enable PITR on DynamoDB")
- Well-Architected best-practice references
- Console navigation paths
- CLI/API reference for recommended actions
- Pricing and feature comparison pages

### aws-repost (community knowledge)
Search for:
- Community-validated solutions for specific resilience patterns
- Known issues and workarounds with AWS services
- Real-world implementation experiences and gotchas
- Troubleshooting guides from practitioners

### aws-pricing (cost validation & enrichment)
Query the AWS Price List API to:
- Validate Resilience Hub's cost estimates against live pricing data
- Provide unit-level cost breakdowns (e.g., "$0.045/GB-month for NAT Gateway data processing")
- Cost remediation recommendations for services not covered by Resilience Hub's estimator
- Compare pricing across regions for multi-region DR strategies
- Support services: `AmazonEC2` (NAT Gateway, instances), `AWSBackup`, `AmazonS3`, `AmazonRDS`, `AWSElasticDisasterRecovery`, `AmazonDynamoDB`, `AWSLambda`

## Best Practices

### Tool usage

- All AWS API operations go through `aws-mcp`.
- Use `aws-docs` to back up every recommendation with authoritative documentation links.
- Use `aws-repost` to surface community patterns and known pitfalls before recommending a remediation path.
- Prefer next-gen Resilience Hub for: dependency discovery, GenAI failure-mode assessments, modular resilience policies, and organization-wide reporting.
- Wire assessments into CI/CD as a gate (fail the pipeline if assessed RTO/RPO regresses below policy).

### Reporting conventions

- Every recommendation must name: (1) the DR strategy tier, (2) the target RTO/RPO, (3) the remediating service (ARC / DRS / Backup), (4) the Well-Architected Reliability best-practice ID it satisfies (e.g. REL11-BP04), and (5) estimated monthly cost validated against AWS Pricing API where available.
- Include the **full resource ARN** for every resource in breach tables and appendices.
- Include **direct Resilience Hub console URLs** for the assessment, application, and operational recommendations.
- Frame failure modes by blast radius: component → AZ → Region.
- Distinguish systemic gaps (same root cause × N resources) from isolated issues.
- Flag false-positive stateless classifications on clearly stateful workloads in a dedicated section.
- Include AWS documentation links for each recommended service action.
- Validate cost figures against AWS Pricing API — present as unit rate × quantity = monthly total.
- When Resilience Hub's recommendation engine cannot fully close a gap, explicitly flag for manual tracking.

## Troubleshooting

- `ExpiredTokenException` / MCP tools not loading → SigV4 session expired. Re-run `aws login` and restart the MCP client.
- `AccessDeniedException` → caller lacks the Resilience Hub or ARC operator policy. Point the user to the IAM setup guide; never attach policies without explicit instruction.
- Unknown service/operation → next-gen API not yet exposed through the managed server; fall back to console instructions.
- Assessment stuck / empty results → dependency discovery runs asynchronously; poll assessment status until COMPLETED before reading results.
- ARC routing-control change has no effect → confirm you targeted the **data plane** endpoint (region-specific), not the control plane.
- aws-docs or aws-repost not responding → these are optional enrichment servers. Continue with aws-mcp for core operations and note that documentation references may be limited.

## License and support

This power is licensed under MIT (SPDX: `MIT`). It integrates with:
- AWS MCP Server (`mcp-proxy-for-aws`, SPDX: `Apache-2.0`)
- AWS Documentation MCP Server (`awslabs.aws-documentation-mcp-server`, SPDX: `Apache-2.0`)
- AWS re:Post MCP Server (`awslabs.aws-repost-mcp-server`, SPDX: `Apache-2.0`)
- AWS Pricing MCP Server (`awslabs.aws-pricing-mcp-server`, SPDX: `Apache-2.0`)

- [Privacy Policy](https://aws.amazon.com/privacy/)
- [Support](mailto:rachapudivishnu9@gmail.com) · [GitHub Issues](https://github.com/aquavis12/power-aws-resilience-hub/issues)
- Author: Venkata Pavan Vishnu Rachapudi — AWS Community Builder (Security)
