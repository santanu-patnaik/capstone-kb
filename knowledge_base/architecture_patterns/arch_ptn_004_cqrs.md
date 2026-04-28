---
doc_id: arch_ptn_004
pattern_name: CQRS (Command Query Responsibility Segregation)
category: data_model
typical_team_size: 8+
complexity_fit: medium_to_high
---

# CQRS — Command Query Responsibility Segregation

## What it is
Separate the write model (commands that change state) from the read model (queries). Writes go to a transactional store; reads come from one or more denormalized views optimized for specific query shapes. The read model is updated from the write model, usually asynchronously via events.

## When to pick it
- Read and write workloads have very different characteristics — e.g., complex writes with invariants, but simple high-volume reads, or vice versa.
- You need multiple read projections of the same underlying data (e.g., a dashboard view, a search index, an analytics warehouse).
- Your domain has complex business rules on the write side that don't belong in the read side.

## When not to pick it
- Your read and write shapes are similar and the transactional store serves both well.
- Eventual consistency between write and read is unacceptable for the use case.
- The team hasn't handled eventual consistency before — it changes how you design UX and error handling.

## Trade-offs
- **Pro:** Read and write scale independently; you can optimize each.
- **Pro:** Multiple read models for different query patterns without polluting the domain model.
- **Pro:** Pairs naturally with event-driven architecture — events feed the read projections.
- **Con:** Eventual consistency on the read side; "I just saved this, why isn't it in my list yet?"
- **Con:** Two data models to maintain — schema changes ripple through both.
- **Con:** Infrastructure footprint is larger; simpler alternatives are usually preferable unless the trade-off is clearly worth it.

## Typical stack fit
Transactional store for writes (Postgres, DynamoDB with conditions). Read projections: Elasticsearch for search, Redis for fast lookups, a materialized view in Postgres, or a data warehouse for analytics. Event bus between them.

## Related patterns
Almost always built on top of event-driven architecture. Event sourcing is a stronger variant where the write model is the event log itself.
