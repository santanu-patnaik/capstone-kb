---
doc_id: arch_ptn_002
pattern_name: Microservices
category: deployment_topology
typical_team_size: 15+
complexity_fit: medium_to_high
---

# Microservices

## What it is
The system is decomposed into small, independently deployable services, each owning its own data and exposing a network API. Services communicate over HTTP/gRPC or asynchronously via a message bus. Each service is owned end-to-end by one team.

## When to pick it
- Organization has multiple engineering teams that need to deploy independently.
- Different components have genuinely different scaling, availability, or compliance profiles.
- You have the SRE investment for a distributed system — tracing, service mesh, on-call rotations, deploy pipelines per service.
- Business domain is stable enough that service boundaries won't need frequent renegotiation.

## When not to pick it
- You have fewer than ~15 engineers.
- Domain boundaries aren't yet well understood — premature microservices lock in bad boundaries.
- Latency budget is tight (chained network calls add up fast).
- You haven't solved observability, CI/CD, and secrets management at platform level.

## Trade-offs
- **Pro:** Team autonomy — independent deploys, independent tech choices within guardrails, independent scaling.
- **Pro:** Blast radius isolation — a bad deploy in service A doesn't take down service B.
- **Pro:** Compliance-by-isolation — regulated data can live in a narrower network.
- **Con:** Distributed systems complexity — cascading failures, partial availability, eventual consistency.
- **Con:** Operational cost multiplier — more services means more dashboards, more on-call rotations, more pipelines.
- **Con:** Cross-service refactoring is expensive; boundary mistakes are expensive.
- **Con:** Latency penalty from network hops — chains of synchronous calls become tail-latency risks.

## Typical stack fit
Kubernetes or equivalent container orchestration, service mesh (Istio, Linkerd), async messaging (Kafka, SQS/SNS, RabbitMQ), API gateway, centralized tracing (OpenTelemetry), per-service datastore.

## Related patterns
Often combined with event-driven architecture for async workflows. Modular monolith is the prerequisite — extract services out of a well-modularized monolith, not a greenfield build.
