---
doc_id: arch_ptn_006
pattern_name: Saga Pattern
category: distributed_transactions
typical_team_size: 10+
complexity_fit: medium_to_high
---

# Saga Pattern

## What it is
A long-running business transaction that spans multiple services, implemented as a sequence of local transactions where each step publishes an event. If a step fails, compensating transactions undo previous steps. Two flavors: choreography (each service reacts to events from others) and orchestration (a central coordinator directs the sequence).

## When to pick it
- Business workflows span 3+ services and each owns its own data.
- Traditional ACID transactions across services are not an option.
- The workflow has clearly defined steps and clearly defined compensations (e.g., reserve inventory / charge card / create order; compensations are cancel reservation / refund / cancel order).

## When not to pick it
- The transaction can fit in a single service with a local ACID transaction — do that instead.
- You can't define reasonable compensating actions (some things can't be undone, like sending a physical package).
- The workflow is simple two-step; a synchronous call with retry is usually enough.

## Orchestration vs choreography
- **Orchestration:** Central coordinator tracks state and issues commands. Easier to debug, centralizes workflow logic, but introduces a central component that must be kept simple and stateless where possible.
- **Choreography:** No coordinator; services react to events. Maximum decoupling but the workflow logic is distributed — "where did this transaction get stuck?" becomes harder to answer.

Use orchestration when workflows are complex or change often. Use choreography when workflows are short (2–3 steps) and stable.

## Trade-offs
- **Pro:** Handles long-running transactions across services without distributed locks.
- **Pro:** Explicit failure and compensation handling is better than hoping transactions don't fail.
- **Con:** Designing compensations correctly is hard — especially for partial failures and idempotency.
- **Con:** Eventual consistency throughout; users must understand that a transaction isn't "done" even if they got a success response.
- **Con:** Debugging a stuck saga requires good tracing and a state view.

## Typical stack fit
Orchestration: a workflow engine (Temporal, AWS Step Functions, Camunda). Choreography: event bus with strong schema discipline (Kafka + Schema Registry).

## Related patterns
Usually combined with event-driven architecture. Often applied within a microservices setup.
