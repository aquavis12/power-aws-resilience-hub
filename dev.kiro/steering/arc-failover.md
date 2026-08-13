# Steering: Amazon Application Recovery Controller (ARC) Failover

Load this for failover orchestration — zonal shift, routing controls, recovery readiness checks, and Region switch. ARC is the service Resilience Hub recommends to actually *execute* failover for Warm Standby and Active-Active tiers.

## CRITICAL SAFETY
Routing-control state changes and zonal shifts **move production traffic**. Before ANY state change:
1. Show the user the exact current state and the exact target state.
2. Confirm which resources (ALB/NLB, Route 53 records, ASGs) are affected.
3. Get explicit "yes" confirmation. Never flip a control on your own initiative.
4. Prefer reading readiness checks and doing dry-runs first.

## The three ARC capabilities

### 1. Zonal shift (AZ impairment)
- Temporarily shifts traffic **away from one AZ** for a supported resource (ALB, NLB).
- Fastest, lowest-blast-radius recovery action. Use for a single-AZ gray failure.
- Reversible; set an expiry. Always confirm the target AZ and resource ARN.

### 2. Routing controls (Region failover)
- Simple on/off switches (data plane) that gate traffic between Regions via Route 53 health checks.
- **Data plane, not control plane**: flip routing controls through the region-specific data-plane endpoints (per REL11-BP04). Never depend on the control plane during an active impairment.
- Use **safety rules** (assertion/gating rules) to prevent invalid states (e.g. both Regions off, or turning primary off before standby is on).

### 3. Region switch (orchestrated multi-step failover)
- Structured, multi-step plans that orchestrate a full Regional failover (routing + scaling + data promotion, e.g. DynamoDB Global Tables, Aurora Global).
- Use for Active-Active / Warm Standby apps that need a repeatable, testable failover runbook.

## Recovery readiness checks
- Continuously validate that the standby is actually ready (capacity, quotas, config parity) **before** you need to fail over.
- A green assessment in Resilience Hub + green ARC readiness checks = a failover you can trust. Read these first.

## Workflow
1. Read current routing-control / readiness state (data plane).
2. Confirm target state with the user (see CRITICAL SAFETY).
3. Execute the smallest sufficient action: zonal shift (AZ) < routing control (Region) < Region switch (full).
4. Verify traffic moved (health checks, target-group health).
5. Report: action taken, resources affected, new state, and the rollback command.

## Guardrails
- Never leave routing controls in an inconsistent state — verify safety rules exist.
- Never trigger Region switch without a tested plan and user confirmation.
