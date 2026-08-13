---
name: report
description: Generate a comprehensive, enterprise-grade resilience assessment report with executive summary, resource ARNs, per-component findings across all disruption layers, cost-quantified remediation recommendations (using AWS Pricing API), operational gaps (alarms/SOPs/FIS tests), direct Resilience Hub console URLs, and a prioritized remediation roadmap — from an existing Resilience Hub assessment.
---

# Generate Resilience Assessment Report

## Step 1: Validate AWS session

- Run `aws sts get-caller-identity` to confirm credentials are valid.
- If expired, instruct the user to re-authenticate (`aws login` or `aws sso login`).
- Record the account ID and region — these are needed for console URL construction.

## Step 2: Identify the assessment

Determine which assessment to report on:
- If the user provides an assessment ARN or app name, use that.
- If not, list recent assessments (`list-app-assessments`) and let the user pick.
- Read the full assessment results: compliance status, component scores, recommendations, cost estimates.
- Record the assessment ID, app ARN, app version, policy name, and completion timestamp.

## Step 3: Cross-reference with live infrastructure

Use `aws-mcp` to validate the assessment scope against live resources:
- Compare assessed resources against actual tagged resources in the account.
- Flag any resources that exist in the account but are NOT in the assessment scope.
- Note any resources in the assessment that no longer exist (drift).
- For each resource, collect the **full ARN** (e.g., `arn:aws:ec2:us-east-1:380459294095:instance/i-0ad5a1107058f233d`).
- Identify Auto Scaling Groups at Min=Max=Desired=0 (effectively disabled — call out explicitly).
- Identify ASGs with active scaling policies (expected drift from autoscaling is normal, not data quality issue).

## Step 4: Gather supporting context

Use the available MCP servers to enrich the report:
- **aws-docs**: Pull relevant AWS documentation for recommended services (Backup, DRS, ARC, DLM, S3 Versioning, NAT Gateway HA) to provide authoritative remediation guidance.
- **aws-repost**: Search for community solutions, known issues, and real-world implementation patterns related to the findings.
- Cross-reference findings against Well-Architected Reliability best practices (see `steering/wa-reliability.md`).

## Step 5: Gather pricing data for remediation costing

Use the **AWS Pricing MCP** (`aws-pricing`) to validate and enrich cost estimates:
- For NAT Gateway redundancy: query `AmazonEC2` pricing for NAT Gateway hourly + per-GB charges in the target region.
- For AWS Backup: query `AWSBackup` pricing for EBS snapshot storage, S3 backup storage, and RDS backup storage.
- For DRS (Elastic Disaster Recovery): query `AWSElasticDisasterRecovery` pricing if DRS is recommended.
- For S3 versioning storage overhead: query `AmazonS3` pricing for standard storage in the target region.
- Cross-reference Resilience Hub's own cost estimates against live pricing — note any discrepancies.
- Present costs as monthly estimates with unit breakdowns (e.g., "$0.045/GB-month for EBS snapshots").
- If Resilience Hub provides its own cost estimate for a recommendation, use that as the primary figure and note the Pricing API rate as supporting context.

## Step 6: Produce the report

Generate a comprehensive, structured report matching the following format. The report should be professional, enterprise-grade, and suitable for executive and engineering stakeholders.

---

### Title Page

```
AWS Resilience Hub Assessment Report — {Application Name}

Resilience Assessment Report — {Environment} Environment

{Date}

Document Version {X.X}
```

---

### 1. Executive Summary

- Overall resiliency score (percentage and decimal)
- Policy compliance status (Met / Breached — with severity: CRITICAL / HIGH / MEDIUM)
- Total breach signal count broken down by disruption type (Application, Infrastructure, AZ)
- RTO status at whole-application level (with targets per category)
- RPO status at whole-application level (with targets per category)
- Total AppComponents assessed and breach count
- Monthly cost to close breaches (from Resilience Hub cost estimates + Pricing API validation)
- Monthly cost for full best-available architecture (Multi-AZ/Multi-Site across estate)

#### 1.1 Key Findings Table

| Metric | Status |
|--------|--------|
| Overall Assessment Status | {e.g., Policy Breached — CRITICAL} |
| Resiliency Score | {X%} ({decimal} / 1.00) |
| Total Policy Breach Signals | {N} total ({breakdown by category}) |
| RTO | {status} (Targets: {per-category targets}) |
| RPO | {status} (Targets: {per-category targets}) |
| AppComponents Assessed | {N} total, covering {M} physical resources ({breached} / {total} with breaches) |
| Monthly Cost to Close Breaches | ${X} / month |
| Monthly Cost for Full Multi-AZ | ${X} / month |

---

### 1.2 Assessment Overview

#### Assessment Details

- **Report**: {report name} (assessment ID {full UUID})
- **Application**: {app name} (App Version {N})
- **Account**: {account ID}
- **Region**: {region display name} — {region code}
- **Resiliency Policy**: {policy name}
- **Assessment Completed**: {date}

#### Direct Console URL

```
https://{region}.console.aws.amazon.com/resiliencehub/home?region={region}#/application/{app-arn}/assessment/{assessment-arn}
```

Provide the exact navigation path:
> AWS Resilience Hub → Applications → {App Name} → Assessment reports → {Report Name}

#### Application Composition & Resource Coverage

Explain the resource validation methodology (cross-check against live infrastructure, not just tag-based Resource Group). Note any resources added by ARN that the Resource Group missed.

| Resource Type | Count | Headline Finding |
|---------------|-------|------------------|
| {type} | {N} | {one-line summary} |

Include ALL resource types: RDS, DynamoDB, SQS, SNS, Lambda, S3, API Gateway, Auto Scaling Groups, EC2, ECS, NAT Gateway, ALB/NLB, etc.

For each resource, note the **ARN** in the detailed sections below.

---

### 2. RTO and RPO Analysis

#### Policy Targets

State the per-category targets clearly:
- Application (Software): RTO {X}, RPO {X}
- Infrastructure (Hardware): RTO {X}, RPO {X}
- Availability Zone: RTO {X}, RPO {X}

#### Current State Analysis

**Recovery Time Objective (RTO)**: Explain whether the whole-application level meets or breaches policy, and WHY (which components pull it to Unrecoverable vs. how many meet policy).

**Recovery Point Objective (RPO)**: Same analysis — distinguish between components with no recovery, insufficient frequency, and compliant ones.

#### Critical Findings

Cross-reference breach detail to identify **distinct failure patterns** (not just a list of components):
1. No recovery path at all (no Backup, DLM, or DRS)
2. Backup too infrequent (exists but cadence exceeds target)
3. Single-AZ / no versioning (managed service misconfiguration)

#### Policy Breach Context Table

| Disruption Category | Components Breached | RTO Signals | RPO Signals | Total Signals |
|--------------------|--------------------:|------------:|------------:|--------------:|
| Application | | | | |
| Infrastructure | | | | |
| Availability Zone | | | | |

#### Implications

Explain that "Unrecoverable at whole-application level" does NOT mean the entire estate lacks DR — explain the weakest-link scoring model and what percentage of components are actually compliant.

---

### 3. Per-Layer Component Assessment

For EACH disruption layer, produce:

#### 3.1 Application (Software) Layer

**Breached Components Table:**

| Component | Resource ARN | Current RTO | Current RPO | Root Cause |
|-----------|-------------|-------------|-------------|------------|

**Remediation Detail** — Group by fix type (not individual component):

For each fix group:
- **Fix description** with console path
- **Recommended Alarms** table (Alarm ID | What it triggers when)
- **Recommended SOPs** table (SOP ID | What it does)
- **Estimated Cost** (from Resilience Hub + Pricing API validation)

**Assessment Summary**: X of Y components (Z%) compliant. Where the gap is isolated vs. systemic.

#### 3.2 Infrastructure (Hardware) Layer

Same structure. Note where fixes overlap with Application layer (fixing once resolves both).

#### 3.3 Availability Zone Layer

Same structure. Call out AZ-specific fixes (NAT Gateway redundancy, cross-AZ replication) separately from backup fixes.

---

### 4. Database & Messaging Layer Findings

#### RDS

| Component | ARN | Layer | RTO | RPO | Basis |
|-----------|-----|-------|-----|-----|-------|

Distinguish between primary (Multi-AZ, PITR) and replicas (often missing PITR). Call out when Resilience Hub's recommendation engine CANNOT close a gap (e.g., PITR on read replicas) — flag for manual tracking.

#### DynamoDB

| Component | ARN | Finding | Fix |
|-----------|-----|---------|-----|

Distinguish between "PITR enabled but restore time exceeds target" (mechanics issue) vs. "no backup at all" (configuration gap).

#### SQS

Call out DLQs that exist but aren't wired (no redrive policy on source queues). This is a common operational gap.

#### SNS

Call out topics with zero subscriptions (alerts silently lost).

---

### 5. Storage Layer Findings

Identify **systemic** S3 gaps (same root cause × N buckets) vs. isolated issues:
- Count of buckets breaching policy and the uniform root cause
- Explain this is ONE gap repeated N times, not N separate problems
- Provide the uniform fix (versioning + tag-based Backup plan)
- **Prioritization**: rank buckets by data sensitivity (financial/contractual first, logs/maintenance last)
- **Cost**: explain pricing model (per-GB backed up, not per-bucket)

---

### 6. Hidden Risk: Stateless Classification Review

Flag instances where Resilience Hub marks RPO=0 ("stateless — nothing to lose") on clearly stateful workloads. Cross-reference against:
- CloudWatch alarm names in the account
- Instance tags and names
- Known service roles (Redis, Postgres, SFTP, ML models)

| Instance | ARN | Role (from alarms/tags) | Resilience Hub Verdict |
|----------|-----|-------------------------|----------------------|

Recommend manual review of all stateless-classified instances before relying on the RPO=0 for compliance.

---

### 7. Recommendations

#### Categorized by Severity

**Category 1: No Recovery Mechanism (CRITICAL)**
- Components | Current state | Fix | Expected outcome | Cost (from Pricing API)

**Category 2: Backup Too Infrequent (HIGH)**
- Note when Resilience Hub's recommendation engine cannot fully close the gap

**Category 3: Single-AZ Resources (HIGH)**
- NAT Gateways, single-AZ databases, etc. | Cost per unit

**Category 4: Missing Versioning/Backup on S3 (HIGH)**
- Systemic fix | Total cost

**Category 5: Database & Messaging Gaps (HIGH)**
- Configuration-only fixes (no new infrastructure)

**Category 6: Compliant Components (MAINTAIN)**
- Count | No action required

#### Cost Summary Table

| Remediation Scope | Monthly Cost | Breaches Resolved |
|-------------------|-------------|-------------------|
| Minimum to clear policy breaches | ${X} | {N} of {M} |
| Full best-available architecture | ${X} | All |

Include unit cost breakdowns validated against AWS Pricing API:
- NAT Gateway: ${X}/month per gateway (hourly + data processing)
- EBS Snapshots: ${X}/GB-month
- S3 Backup: ${X}/GB-month
- etc.

---

### 8. Operational Recommendations

#### Gap Summary

| Category | Implemented | Outstanding | Coverage % |
|----------|-------------|-------------|------------|
| CloudWatch Alarms | {X} of {Y} | {Z} | {%} |
| SOPs | {X} of {Y} | {Z} | {%} |
| FIS Tests | {X} of {Y} | {Z} | {%} |

#### Per-Resource-Type Alarm Tables

For EACH resource type (EC2, ECS, Lambda, RDS, DynamoDB, SQS, SNS, S3, ALB, API Gateway, NAT Gateway):

| Alarm | Triggers when… |
|-------|----------------|

#### Per-Resource-Type SOP Requirements

List required SOPs per resource type with Resilience Hub SOP IDs.

#### Per-Resource-Type FIS Test Requirements

List required FIS experiments per resource type.

#### Fastest Implementation Path

Document the CloudFormation template generation workflow:
> Resilience Hub console → Applications → {App} → Operational recommendations → Select items → Create CloudFormation template

---

### 9. Appendix A: Component-Level Gap Checklist

One row per assessed component:

| Component | Type | Resource ARN | RTO/RPO Status | Alarms Outstanding | SOPs Outstanding | Tests Outstanding |
|-----------|------|-------------|----------------|-------------------|-----------------|-------------------|

---

### 10. Appendix B: Console Navigation Guide

Step-by-step navigation for accessing the full interactive assessment:
1. Open AWS Console → Resilience Hub
2. Applications → {App Name}
3. Assessment reports → {Report Name}

**Direct URL**: `https://{region}.console.aws.amazon.com/resiliencehub/home?region={region}#/application/{url-encoded-app-arn}/assessment/{url-encoded-assessment-arn}`

List what's available in the console view:
- Detailed RTO/RPO per component and disruption type
- Complete alarm/SOP/FIS recommendations with implementation steps
- Least Cost / Least Change / Best-AZ-Recovery comparison
- Per-component cost breakdown
- Implementation progress tracking

---

### 11. Appendix C: Resource ARN Reference

Full ARN listing for all resources in scope, grouped by type:

| Resource Type | Name/ID | Full ARN |
|---------------|---------|----------|
| EC2 Instance | {name} | arn:aws:ec2:{region}:{account}:instance/{id} |
| S3 Bucket | {name} | arn:aws:s3:::{bucket-name} |
| RDS Instance | {name} | arn:aws:rds:{region}:{account}:db:{id} |
| ... | ... | ... |

---

## Step 7: Save the report

Save to `resilience-context/` with a descriptive filename:
- Pattern: `{app-name}-assessment-report-{version}-{date}.md`
- If user specifies DOCX format: produce the markdown with proper heading hierarchy and note that it can be converted using pandoc or similar tooling.
- Include the document version number in the filename and title page.

---

## Reporting rules

1. Every recommendation MUST name: (1) DR strategy tier, (2) target RTO/RPO, (3) remediating service, (4) WAF Reliability best-practice ID, (5) estimated cost (from Resilience Hub + Pricing API validation).
2. Order by blast radius (Region > AZ > component), then by gap size.
3. Distinguish systemic gaps (same root cause repeated N times) from isolated issues — present systemic gaps as ONE finding with a count, not N separate findings.
4. Flag components with false-positive "stateless" classification (RPO=0 on clearly stateful workloads) in a dedicated section.
5. Include the **full resource ARN** for every resource mentioned in breach tables and the appendix.
6. Include **direct Resilience Hub console URLs** for the assessment, the application, and the operational recommendations page.
7. Include console navigation paths for every remediation action (e.g., "AWS Backup → Backup plans → Create Backup plan → assign resources by instance ID").
8. Reference AWS documentation links for each recommended service.
9. Validate cost figures against AWS Pricing API — use Resilience Hub's estimates as primary, note Pricing API rates as supporting context.
10. For Auto Scaling Groups scaled to zero: explicitly call out that these are effectively disabled and cannot launch instances without manual intervention.
11. For ASGs with active scaling policies: note that instance membership drift in point-in-time snapshots is expected and normal.
12. When Resilience Hub's recommendation engine cannot fully close a gap (e.g., PITR on read replicas, hourly backup vs. 15-min target), explicitly flag this as requiring manual tracking outside Resilience Hub.

## Pricing integration rules

- Use the `aws-pricing` MCP server for cost validation and enrichment.
- Query pricing for: `AmazonEC2` (NAT Gateway), `AWSBackup`, `AmazonS3`, `AmazonRDS`, `AWSElasticDisasterRecovery`, `AmazonDynamoDB` as needed.
- Always filter by the assessment's region.
- Present costs as: unit rate × quantity = monthly total.
- If Pricing API is unavailable, fall back to Resilience Hub's own estimates and note the limitation.

## Console URL construction

Build Resilience Hub console URLs using this pattern:
```
https://{region}.console.aws.amazon.com/resiliencehub/home?region={region}#/application/{url-encoded-app-arn}/assessment/{url-encoded-assessment-arn}
```

For operational recommendations:
```
https://{region}.console.aws.amazon.com/resiliencehub/home?region={region}#/application/{url-encoded-app-arn}/operational-recommendations
```

## Guardrails

- Read/report only. Never modify workload infrastructure.
- Use aws-docs and aws-repost to provide authoritative guidance, not invented solutions.
- If Resilience Hub's recommendation engine cannot close a gap, explicitly call this out as requiring manual tracking.
- Cost figures come from Resilience Hub's own estimates as primary source — Pricing API validates but does not override.
- Never invent resource ARNs — only include ARNs retrieved from actual API calls.
- Do not expose AWS account credentials or session tokens in the report.
