# Steering: Resilience Hub Assessment Workflow

Load this when onboarding an application, running dependency discovery, executing assessments, defining resilience policies, or validating RTO/RPO.

## Next-gen model (what changed)
Next-generation Resilience Hub (GA May 2026) introduces:
- **New application modeling hierarchy** — model at the application level with nested components.
- **Automatic dependency discovery** — the service maps dependencies instead of you declaring them manually.
- **GenAI-powered failure-mode assessments** — analysis grounded in the AWS Well-Architected Reliability pillar identifies weaknesses and proposes fixes.
- **Modular resilience policies** — availability, disaster recovery, and data recovery targets can vary per workload.
- **Organization-wide reporting** — aggregated resilience posture across accounts without logging into each.

Because this is newly GA, always **discover operation names** via `aws-mcp` before calling. If an operation isn't exposed yet, fall back to the console (Resilience Hub → Manage / Assess / Report).

## Workflow

### 1. Onboard the application
- Register the app (name + description). Prefer importing from an existing source (CloudFormation stack, resource group, Terraform state) so dependency discovery has a starting point.
- Attach a **resilience policy** defining target RTO/RPO for the four disruption types: Application, Infrastructure (AZ), Region, and Cloud (see `context-templates/rto-rpo-targets.md`).

### 2. Run dependency discovery
- Trigger discovery and **poll until COMPLETED** (asynchronous). Do not read results early.
- Review discovered dependencies with the user — flag anything untracked or unexpected (untested dependencies are the #1 cause of missed RTO).

### 3. Run the assessment
- Execute an assessment against the policy.
- Read back: assessed RTO/RPO per component vs target, and the GenAI-identified failure modes.

### 4. Report (required format)
Produce a table: `Component | Assessed RTO/RPO | Target | Gap | Failure mode | Recommended fix | WAF BP`.
- Map every recommended fix to a remediating service and a Well-Architected Reliability best-practice ID (see `steering/wa-reliability.md`).
- Order recommendations by blast radius (Region > AZ > component) then by gap size.

### 5. (Optional) CI/CD gate
Wire the assessment into a pipeline: fail the build if assessed RTO/RPO regresses below policy. This turns resilience into a release gate rather than an audit afterthought.

## Guardrails
- Read/assess only. Never modify workload infrastructure to "fix" a finding — surface the recommendation and let the user act.
- If dependency discovery reports zero dependencies for a non-trivial app, treat it as a discovery failure, not a clean bill of health.
