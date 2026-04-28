---
doc_id: arch_ptn_001
pattern_name: Modular Monolith
category: deployment_topology
typical_team_size: 3–10
complexity_fit: low_to_medium
---

# Modular Monolith

## What it is
A single deployable application organized internally into strongly-bounded modules with explicit contracts between them. Each module owns its own data and exposes an internal API; cross-module calls go through the API, not shared tables. Ships as one artifact, runs as one process.

## When to pick it
- Team size under 10 engineers.
- Business domain still evolving — boundaries will shift.
- You don't have the SRE investment to run a distributed system safely.
- Latency and operational simplicity matter more than independent deploy velocity.

## When not to pick it
- Teams need to deploy independently and frequently.
- Different parts of the system have radically different scaling profiles (e.g., one component needs GPU, another needs horizontal web scale).
- Compliance boundary requires separate blast radius (e.g., PHI-handling module must run in isolated VPC).

## Trade-offs
- **Pro:** Fastest path to delivery; one CI/CD pipeline; one observability stack; refactoring across boundaries is cheap while boundaries are still being discovered.
- **Pro:** Operational simplicity — one service to monitor, one deploy to coordinate.
- **Con:** Doesn't scale organizationally past ~10 engineers without explicit module ownership discipline.
- **Con:** A bug in one module can take down the whole app.
- **Con:** Deploy cadence is shared — one team's hotfix can block another team's release.

## Typical stack fit
Language-native module systems (Go packages, Java modules, .NET projects), shared Postgres with schema-per-module, single container.

## Related patterns
If you outgrow this, the natural next step is either Service-Oriented (keep a shared DB, split deploys) or Microservices (full split).
