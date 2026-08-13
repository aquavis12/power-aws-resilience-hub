---
name: report
description: Generate a structured resilience assessment report with executive summary, per-component findings, cost-quantified remediation recommendations, operational gaps (alarms/SOPs/FIS tests), and a prioritized remediation roadmap — from an existing Resilience Hub assessment.
---

# Generate Resilience Assessment Report

## Step 1: Validate AWS session

- Run `aws sts get-caller-identity` to confirm credentials are valid.
- If expired, instruct the user to re-authenticate.

## Step 2: Identify the assessment

Determine which assessment to report on:
- If the user provides an assessment ARN or app name, use that.
- If not, list recent assessments (`list-app-assessments`) and let the user pick.
- Read the full assessment results (compliance, component scores, recommendations).

## Step 3: Cross-reference with live infrastructure

Use `aws-mcp` to validate the assessment scope against live resources:
- Compare assessed resources against actual tagged resources in the account.
- Flag any resources that exist in the account but are NOT in the assessment scope.
- Note any resources in the assessment that no longer exist (drift).

## Step 4: Gather supporting context

Use the available MCP servers to enrich the report:
- **aws-docs**: Pull relevant AWS documentation for recommended services (Backup, DRS, ARC, DLM) to provide authoritative remediation guidance.
- **aws-repost**: Search for community solutions, known issues, and real-world implementation patterns related to the findings.
- Cross-reference findings against Well-Architected Reliability best practices (see `steering/wa-reliability.md`).

## Step 5: Produce the report

Generate a structured report with the following sections:

### Executive Summary
- Overall score, policy compliance status, total breach count
- Key metrics: RTO/RPO targets vs actual, components assessed, breach count
- Monthly cost to remediate (from Resilience Hub cost estimates)

### Assessment Overview
- Assessment ID, app name, account, region, policy details
- Resource composition table (type → count → headline finding)

### RTO/RPO Analysis
- Per-disruption-category tables (Application, Infrastructure, AZ)
- Current state vs target for each category
- Breached components with root cause

### Per-Layer Component Assessment
For each disruption layer (Application, Infrastructure, Availability Zone):
- Breached components table with current RTO/RPO and root cause
- Specific remediation steps per component
- Recommended alarms and SOPs (with Resilience Hub recommendation IDs)

### Database & Messaging Findings (if applicable)
- RDS, DynamoDB, SQS, SNS findings with specific fixes

### Storage Layer Findings (if applicable)
- S3 versioning/backup gaps — systemic patterns, not per-bucket repetition

### Recommendations
- Categorized by severity (Critical / High / Medium / Maintain)
- Each recommendation includes: DR tier, target RTO/RPO, remediating service, WAF best-practice ID, estimated cost
- Cost summary table

### Operational Recommendations
- Missing alarms count by resource type
- Missing SOPs count
- Missing FIS tests count
- Per-resource-type alarm/SOP/test tables

### Appendix: Component-Level Gap Checklist
- One row per assessed component: type, RTO/RPO status, alarms/SOPs/tests outstanding

## Step 6: Save the report

Save to `resilience-context/` with a descriptive filename:
- Pattern: `{app-name}-assessment-report-{date}.md`
- If the user specifies a different format (PDF outline, DOCX structure), produce the markdown and note conversion instructions.

## Reporting rules

- Every recommendation MUST name: (1) DR strategy tier, (2) target RTO/RPO, (3) remediating service, (4) WAF Reliability best-practice ID, (5) estimated cost where available.
- Order by blast radius (Region > AZ > component), then by gap size.
- Distinguish systemic gaps (same root cause repeated N times) from isolated issues.
- Flag components with false-positive "stateless" classification (RPO=0 on clearly stateful workloads).
- Include console navigation paths for remediation actions.
- Reference AWS documentation links for each recommended service.

## Guardrails

- Read/report only. Never modify workload infrastructure.
- Use aws-docs and aws-repost to provide authoritative guidance, not invented solutions.
- If Resilience Hub's recommendation engine cannot close a gap (e.g., PITR on read replicas), explicitly call this out as requiring manual tracking.
- Cost figures come from Resilience Hub's own estimates — never invent pricing.
