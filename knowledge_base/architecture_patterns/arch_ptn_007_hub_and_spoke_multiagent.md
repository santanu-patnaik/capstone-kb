---
doc_id: arch_ptn_007
pattern_name: Hub-and-Spoke Multi-Agent Orchestration
category: agentic_systems
typical_team_size: 3–8
complexity_fit: medium
---

# Hub-and-Spoke Multi-Agent Orchestration

## What it is
A central Orchestrator agent receives input, decomposes it into sub-tasks, dispatches each sub-task to a specialist agent, and aggregates the results. The Orchestrator owns state, retries, error handling, and the sequence of handoffs. Specialist agents are stateless with respect to each other — they only know their own contract.

## When to pick it
- You have clear specialist roles (planner, architect, estimator, critic) that map cleanly to sub-tasks.
- You need centralized state and auditability — one place to see what happened in the run.
- Workflow has a known shape; sub-tasks are largely independent and can often run in parallel.
- Error handling, retries, and fallback logic belong in one place.

## When not to pick it
- Specialist agents need to negotiate with each other (e.g., two agents iteratively refining a joint artifact). Use message graph instead.
- The Orchestrator becomes a bottleneck because it does non-trivial work itself — keep it thin.
- The workflow is fundamentally sequential with no fan-out; a sequential pipeline is simpler.

## Trade-offs
- **Pro:** Clear responsibility boundaries — one role, one agent, one output contract per specialist.
- **Pro:** Easy to add or swap specialists without touching the others.
- **Pro:** Centralized observability; the Orchestrator's state is the audit trail.
- **Pro:** Retries, timeouts, and fallbacks are centralized.
- **Con:** Orchestrator is a single point of complexity; keep its logic thin.
- **Con:** Not well-suited to iterative multi-agent negotiation.
- **Con:** Fan-in aggregation logic (what to do if one specialist fails but others succeed) is a real design decision.

## Typical stack fit
- Low-code: n8n or Langflow — visual DAG with a central router node and specialist sub-flows.
- Code: LangGraph (explicit state graph), LangChain (sequential chains with central dispatcher), custom Python with asyncio.

## Related patterns
- **Sequential pipeline:** A → B → C, no dispatcher. Simpler when there's no fan-out.
- **Hierarchical:** Orchestrator dispatches to sub-orchestrators, each with their own specialists. Useful when agents naturally cluster into groups.
- **Message graph:** Peer-to-peer, agents emit messages consumed by others. Best for negotiation-style workflows.

## When to combine with a Critic
Almost always. A Critic agent inserted between the specialist fan-in and the final output gives you a rubric-based quality gate and a revision loop. The Critic is itself a specialist — it just happens to evaluate other specialists' outputs rather than produce new artifacts.
