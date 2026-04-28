---
doc_id: arch_ptn_005
pattern_name: API Gateway with Backend-for-Frontend (BFF)
category: edge_topology
typical_team_size: 10+
complexity_fit: medium
---

# API Gateway + Backend-for-Frontend (BFF)

## What it is
An edge layer sits between clients and internal services. The API gateway handles cross-cutting concerns — authentication, rate limiting, request logging, TLS termination. One or more BFF services sit behind the gateway, each tailored to a specific client type (web, iOS, Android, partner API). The BFF aggregates calls to internal services and shapes responses for its client.

## When to pick it
- Multiple client types with different needs (mobile apps need smaller payloads; web can handle richer responses).
- Internal service topology is complex and you don't want clients to know about it.
- Authentication, rate limiting, and other cross-cutting concerns should be centralized.
- Different teams own different clients and want control over their own BFF.

## When not to pick it
- Single client type with stable needs — a plain API gateway without per-client BFFs is simpler.
- You only have 2–3 internal services; direct client-to-service calls are fine.
- The BFF layer becomes a dumping ground for business logic that should live in domain services — this is a failure mode, not the pattern's intent.

## Trade-offs
- **Pro:** Decouples clients from internal service topology — internal refactoring doesn't break clients.
- **Pro:** Each client team controls its BFF and can iterate independently.
- **Pro:** Centralized cross-cutting concerns at the gateway.
- **Con:** Extra hop adds latency; the gateway and BFF must be fast.
- **Con:** BFFs can accumulate business logic if not disciplined; they're meant to shape, not decide.
- **Con:** More deploy units to operate.

## Typical stack fit
Gateway: Kong, AWS API Gateway, Apigee, Envoy. BFF: typically Node.js or Go, optimized for network aggregation. Internal services in whatever language is appropriate.

## Related patterns
Usually sits in front of microservices or modular monolith. In a modular monolith, the BFF pattern can still apply at the HTTP layer even if the "backend" is one deploy.
