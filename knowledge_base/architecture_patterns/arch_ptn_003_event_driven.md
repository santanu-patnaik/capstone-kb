---
doc_id: arch_ptn_003
pattern_name: Event-Driven Architecture
category: communication_style
typical_team_size: 10+
complexity_fit: medium_to_high
---

# Event-Driven Architecture

## What it is
Services communicate primarily by publishing and consuming events on a durable message broker. State changes in one service become events that other services react to asynchronously. No direct synchronous calls between services for core flows.

## When to pick it
- Workflows span multiple services and don't need synchronous round-trip (e.g., fraud check triggered by new claim, notification triggered by order completion).
- Throughput or burst tolerance matters — the broker absorbs spikes.
- You want loose coupling — producers don't know their consumers, and new consumers can be added without producer changes.
- Audit / replay is a requirement — the event log is the system of record.

## When not to pick it
- The use case is fundamentally request-response (e.g., a user query that needs a synchronous answer).
- Your team has no experience with distributed systems — eventual consistency surprises people.
- You can't invest in a durable, well-operated broker.

## Trade-offs
- **Pro:** Loose coupling — new consumers added without producer changes; producers protected from slow consumers.
- **Pro:** Natural burst absorption; the broker flattens spikes.
- **Pro:** Audit and replay — event log is a durable record.
- **Con:** Eventual consistency surprises users and developers; "I did X, why isn't Y showing it yet?"
- **Con:** Debugging is harder — tracing a business flow across N services and events requires distributed tracing and event correlation IDs.
- **Con:** Schema evolution on events is a first-class concern; breaking event contracts breaks every consumer.
- **Con:** Exactly-once semantics are a lie; design for idempotency.

## Typical stack fit
Kafka, AWS SNS+SQS, EventBridge, or NATS. Schema registry (Confluent, AWS Glue Schema Registry). Strong observability with trace propagation.

## Related patterns
Often combined with microservices. CQRS and event sourcing are specializations. The Saga pattern is the standard way to handle multi-step business transactions across event-driven services.
