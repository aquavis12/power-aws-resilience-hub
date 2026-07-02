# Dependency Notes

Automatic dependency discovery is good but not omniscient. List dependencies it may miss so the assessment reflects reality.

## External / third-party
- `<e.g. Stripe API — payment path, no Regional failover, hard dependency>`

## Cross-account / cross-Region
- `<e.g. shared services account 1234... hosts the KMS key used by the DR Region>`

## On-prem / hybrid
- `<e.g. Direct Connect to DC-East; single link = single point of failure>`

## Stateful data stores (RPO-critical)
- `<e.g. Aurora Global — replica lag typically < 1s; verify before claiming RPO=seconds>`

> Flag anything that is a single point of failure or that discovery cannot see from inside the AWS account.
