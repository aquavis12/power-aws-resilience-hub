---
name: assess
description: Run a Resilience Hub assessment for your AWS application — validates RTO/RPO targets, discovers dependencies, and reports failure modes with remediation recommendations.
---

# Run Resilience Assessment

## Step 1: Validate AWS session

Verify the user has an active AWS session:
- Run `aws sts get-caller-identity` to confirm credentials are valid.
- If credentials are expired (`ExpiredTokenException`), instruct the user to run `aws login` or `aws sso login --profile <name>` and restart Kiro.
- Confirm the account ID with the user.

## Step 2: Discover Resilience Hub operations

Use the `aws-mcp` tools to discover available `resiliencehub` operations. This is a newly GA service — always verify operation names before calling. Never invent operation names.

## Step 3: Check for existing application

List registered Resilience Hub applications. If one exists, confirm with the user which app to assess. If none exist, offer to onboard one (from CloudFormation stack, resource group, or Terraform state).

## Step 4: Run the assessment

- Execute an assessment against the application's resilience policy.
- Poll the assessment status until COMPLETED (it runs asynchronously).
- Do NOT read results early — wait for completion.

## Step 5: Report results

Produce a table with columns: `Component | Assessed RTO/RPO | Target | Gap | Failure mode | Recommended fix | WAF BP`

Rules:
- Map every recommendation to a remediating service (ARC / DRS / Backup) AND a Well-Architected Reliability best-practice ID (e.g. REL11-BP04).
- Order by blast radius (Region > AZ > component), then by gap size.
- Never modify workload infrastructure — only surface recommendations.

## Guardrails

- Read/assess only. Never create, modify, or delete workload infrastructure.
- If dependency discovery reports zero dependencies for a non-trivial app, flag it as a discovery failure.
- Express all targets as explicit RTO/RPO numbers tied to a DR strategy tier.
