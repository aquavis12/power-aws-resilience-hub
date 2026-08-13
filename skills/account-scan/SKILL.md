---
name: account-scan
description: Perform an account-wide resilience assessment across all regions — discover all Resilience Hub apps, assess each, flag unregistered workloads, and produce an aggregated posture report enriched with AWS documentation and community knowledge.
---

# Account-Wide Resilience Assessment

Scan an entire AWS account for resilience posture across all active regions.

## Step 1: Validate AWS session

- Run `aws sts get-caller-identity` to confirm credentials and note the account ID.
- Ask the user which regions to scan. If they say "all", use the common active regions:
  `us-east-1, us-east-2, us-west-1, us-west-2, eu-west-1, eu-west-2, eu-central-1, ap-southeast-1, ap-southeast-2, ap-northeast-1`
- If the user specifies particular regions, use only those.

## Step 2: Discover apps in each region

For each target region:
- Use `aws-mcp` tools to call Resilience Hub `list-apps` with the region parameter.
- Collect all registered applications across all regions.
- Present the user a summary: region → app count → app names.

## Step 3: Assess each application

For each discovered app:
- Check if there's a recent assessment (< 24h old). If yes, read it instead of re-running.
- If no recent assessment exists, trigger one and poll until COMPLETED.
- Record: app name, region, compliance status, resilience score, policy breaches.

## Step 4: Identify unregistered workloads

Use `aws-mcp` to list resources that exist in the account but are NOT registered in Resilience Hub:
- List CloudFormation stacks per region.
- List resource groups per region.
- Compare against registered apps.
- Flag unregistered workloads as potential gaps.

## Step 5: Enrich with documentation and community knowledge

- Use `aws-docs` to pull documentation for each recommended remediation service.
- Use `aws-repost` to search for community patterns related to common findings (e.g., "DynamoDB PITR restore time", "S3 CRR setup gotchas").
- Include relevant documentation links in the final report.

## Step 6: Produce aggregated report

Generate a report with:

### Account Summary Table
```
| Region | Apps Registered | Compliant | Breached | Avg Score | Unregistered Stacks |
```

### Per-App Detail (ordered by severity)
```
| App Name | Region | Score | Worst Breach | Gap | DR Tier | Fix | WAF BP | Doc Link |
```

### Cross-Region Observations
- Apps with no DR region configured
- Single-region dependencies across multi-region apps
- Region pairs that share a failure domain (e.g., same AZ naming)

### Recommendations (ordered by blast radius)
Every recommendation must include: DR tier + target RTO/RPO + remediating service + WAF best-practice ID + estimated cost + documentation link.

## Step 7: Save report

Save the report to `resilience-context/account-posture-report-{account-id}-{date}.md`.

## Region handling

The `aws-mcp` proxy inherits the region from your AWS CLI configuration (`AWS_REGION` or `AWS_DEFAULT_REGION`). To query a different region:
- Pass the `--region` parameter in each Resilience Hub API call.
- The power does NOT hardcode a region — it uses whatever region the user's AWS session targets, and can override per-call.

## Guardrails

- Read/assess only. Never modify workload infrastructure.
- If a region returns AccessDenied, note it and continue with other regions (not all regions may be enabled).
- Rate-limit assessments — don't trigger more than 5 concurrent assessments to avoid throttling.
- For organization-wide (multi-account) scanning, the user needs a delegated admin role. Confirm this before attempting cross-account calls.
- Use `aws-docs` and `aws-repost` for authoritative guidance — never invent remediation steps.
