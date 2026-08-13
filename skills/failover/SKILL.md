---
name: failover
description: Orchestrate ARC failover operations — zonal shift, routing controls, and Region switch — with production-traffic safety guardrails, backed by AWS documentation for data-plane endpoint validation.
---

# ARC Failover Orchestration

## CRITICAL SAFETY

Routing-control state changes and zonal shifts **move production traffic**. Before ANY state change:
1. Show the user the exact current state and the exact target state.
2. Confirm which resources (ALB/NLB, Route 53 records, ASGs) are affected.
3. Get explicit "yes" confirmation. Never flip a control on your own initiative.
4. Prefer reading readiness checks and doing dry-runs first.

## Step 1: Validate AWS session

Run `aws sts get-caller-identity` to confirm credentials. If expired, instruct the user to re-authenticate.

## Step 2: Read current state

Use `aws-mcp` tools to read the current routing-control and recovery-readiness state via ARC data-plane endpoints. Present this clearly to the user.

## Step 3: Validate with documentation

Use `aws-docs` to confirm:
- The correct data-plane endpoint format for the target region.
- Safety rule configuration requirements.
- Any service-specific failover prerequisites (e.g., DynamoDB Global Tables promotion, Aurora failover steps).

Use `aws-repost` to check for:
- Known timing issues with the failover type being attempted.
- Community-reported gotchas for the specific resource types involved.

## Step 4: Determine the action

Choose the smallest sufficient action:
- **Zonal shift** (AZ impairment) — shifts traffic away from one AZ for ALB/NLB. Fastest, lowest blast radius.
- **Routing control** (Region failover) — on/off switches (data plane) gating traffic between Regions.
- **Region switch** (full orchestrated failover) — multi-step plan for Active-Active / Warm Standby.

## Step 5: Confirm with user

Display:
- Current state of all affected resources
- Proposed target state
- Which resources will be affected
- The rollback command
- Relevant AWS documentation links for the operation

Wait for explicit "yes" before proceeding.

## Step 6: Execute and verify

- Execute the action through data-plane endpoints (per REL11-BP04 — never rely on control plane during recovery).
- Verify traffic moved (health checks, target-group health).
- Report: action taken, resources affected, new state, rollback command.

## Guardrails

- Never leave routing controls in an inconsistent state.
- Never trigger Region switch without a tested plan and user confirmation.
- Always use data-plane endpoints, not control-plane APIs.
- Use `aws-docs` to validate endpoint formats before executing — never guess data-plane URLs.
