---
doc_id: tech_stack_decision_log
title: Tech Stack Decision Log (Architecture Decision Records — condensed)
version: 2.7
updated: 2026-02-10
---

# Tech Stack Decision Log (condensed ADRs)

Each decision below records the context, options considered, choice, and — where available — the outcome. Used to calibrate future decisions and to surface lessons when a similar choice comes up.

## TSD-001 — Primary backend language for internal APIs
- Context: New default stack for internal tools and APIs after sunset of the previous Ruby on Rails standard.
- Options: Python (FastAPI), Node.js (NestJS), Go (chi).
- Decision: Python (FastAPI) and Node.js both approved Tier 1; Go Tier 2 for high-throughput infra only.
- Rationale: Team skill distribution was heaviest in Python and JS; Go reserved for cases where its profile is actually required.
- Outcome: Adoption successful. ~70% of new services Python, ~25% Node, ~5% Go.

## TSD-002 — React vs Angular for new frontends
- Context: Deprecating legacy Angular 1/2 apps; picking a forward standard.
- Options: React (with TS), Angular 17, Svelte.
- Decision: React with TypeScript.
- Rationale: Ecosystem depth, team experience, Next.js server-rendering for customer/member surfaces.
- Outcome: Strong. Build times and DX improved; team hiring easier.

## TSD-003 — Relational default (Postgres vs MySQL vs Oracle)
- Context: Multi-year Oracle deprecation and need for a go-forward relational default.
- Options: PostgreSQL on RDS Aurora, MySQL on RDS, stay on Oracle.
- Decision: PostgreSQL on RDS Aurora.
- Rationale: Feature parity for 95%+ of workloads; JSONB handles semi-structured without needing MongoDB; cost roughly 1/3 of Oracle at scale; GovCloud-eligible.
- Outcome: Strong. Migration on track for 2027 EOL.

## TSD-004 — Messaging: Kafka vs SQS+SNS
- Context: New event-driven workloads standardizing on one messaging approach.
- Options: SQS + SNS (AWS-native), Kafka (MSK), EventBridge.
- Decision: SQS + SNS as Tier 1 default; Kafka Tier 2 when event sourcing, replay, or high-throughput streaming are actual requirements.
- Rationale: SQS/SNS operational simplicity wins for most workloads. Kafka's power is wasted on typical async task queues.
- Outcome: Mixed. Some teams over-reached for Kafka; tightened governance in v3 of this log.

## TSD-005 — Vector database for RAG workloads
- Context: Need a managed vector DB for RAG-based agentic workflows.
- Options: Pinecone, Qdrant (self-managed), Weaviate (managed), pgvector in Postgres.
- Decision: Pinecone as Tier 2 standard. pgvector approved for POCs and lightweight embeddings where data stays in existing Postgres instances.
- Rationale: Pinecone's managed operational footprint, strong metadata filtering, and namespace support outweigh cost. pgvector is fine for <1M vectors and teams already using Postgres.
- Outcome: Pinecone adoption has been clean on 4 projects to date; pgvector used successfully on 2 POCs.

## TSD-006 — LLM access gateway
- Context: Multiple teams wanting direct LLM vendor SDK access; security and cost visibility concerns.
- Options: Allow direct SDK usage, build an internal gateway, use Bedrock as the unified front.
- Decision: All LLM calls go through an internal API gateway that proxies to Bedrock and (for models not in Bedrock) directly to OpenAI/Anthropic. Direct SDK usage not permitted.
- Rationale: Cost attribution, prompt logging for audit, rate limiting, PII redaction at the gateway, and model-swap flexibility.
- Outcome: Strong. Saved meaningful cost via caching and logging surfaced two near-incidents with sensitive data in prompts.

## TSD-007 — Agent orchestration framework for multi-agent workflows
- Context: Several teams exploring multi-agent systems.
- Options: LangChain, LangGraph, custom Python with asyncio, n8n (low-code), Langflow (low-code visual).
- Decision: Approved for internal/POC use: LangGraph (code track), n8n + Langflow (low-code track). LangChain allowed as a dependency, not as primary orchestrator for complex flows. Custom asyncio orchestrators need architecture review.
- Rationale: LangGraph's explicit state graph fits hub-and-spoke and message-graph patterns; n8n/Langflow give non-engineers a path in and make demo videos clear.
- Outcome: Too early to evaluate — reviewing after 3 completed projects.

## TSD-008 — Feature flag platform
- Context: Multiple teams using multiple feature flag approaches.
- Options: LaunchDarkly, Flagsmith, Unleash (self-hosted), home-grown.
- Decision: LaunchDarkly for user-facing feature flags. Code-level config flags acceptable for infrastructure rollouts.
- Rationale: Per-user targeting, experimentation, and audit trails that home-grown solutions consistently cut corners on. Cost justified at scale.
- Outcome: Strong. Now used on every customer-facing launch.

## TSD-009 — CI/CD: Jenkins → GitHub Actions + ArgoCD
- Context: Jenkins operational burden and poor developer experience.
- Options: Stay on Jenkins, migrate to GitHub Actions + ArgoCD, move to CircleCI.
- Decision: GitHub Actions for CI; ArgoCD for Kubernetes CD; CodeDeploy for Lambda.
- Rationale: Closer to source control, less infrastructure to operate, better DX.
- Outcome: Mid-migration. Faster feedback loops on pilot repos; full cutover targeted Q3 2026.

## TSD-010 — Identity provider: Ping vs Okta
- Context: Post-acquisition, two identity providers active (Okta + Ping). Consolidation decision.
- Options: Consolidate on Okta, consolidate on Ping, keep both federated.
- Decision: Okta for employees, Ping Identity for customers/members. Not consolidated to one because customer IAM and workforce IAM have different scale and feature needs, and Ping's existing customer footprint is large.
- Rationale: Migration cost of moving 1.2M members to Okta outweighs the benefit of a single provider.
- Outcome: Stable two-provider model; documented as long-term state.

## TSD-011 — Observability: Datadog vs Dynatrace vs self-hosted
- Context: Existing Dynatrace contract ending; decision on replacement.
- Options: Renew Dynatrace, move to Datadog, self-host Prometheus + Grafana + Loki + Tempo.
- Decision: Datadog for metrics, traces, and APM. Self-hosted Prometheus allowed for platform teams. Logs on the org log platform (not Datadog Logs, cost).
- Rationale: Datadog's trace quality and incident response tooling, single-vendor consolidation for most observability domains.
- Outcome: Cost is higher than Dynatrace was, but mean time to resolution is measurably better.

## TSD-012 — Mobile: React Native vs native
- Context: Member mobile app replacement decision.
- Options: React Native, Flutter, native Swift + Kotlin.
- Decision: Native Swift + Kotlin for member-facing mobile apps. React Native acceptable for internal / employee-facing apps.
- Rationale: App Store performance, accessibility feature depth, and platform-specific features (wallet, health integrations) are difficult to match in cross-platform frameworks for member apps.
- Outcome: Native app launched with stronger accessibility scores than React Native prototype showed; holding the line.

## TSD-013 — API style: REST vs GraphQL
- Context: Several teams proposing GraphQL for new APIs.
- Options: Default to REST, default to GraphQL, case-by-case.
- Decision: REST (OpenAPI 3.1) is default. GraphQL allowed with architecture review approval for BFFs serving multiple client types with genuinely different shape needs.
- Rationale: GraphQL's power is wasted on single-client APIs; REST's tooling (gateway, caching, observability) is much more mature. BFF with GraphQL is a legitimate niche.
- Outcome: Steady. 2 GraphQL BFFs approved and delivered; both appropriate fits.

## TSD-014 — Container platform: ECS vs EKS
- Context: Container platform standardization.
- Options: ECS Fargate, EKS, a mix.
- Decision: ECS Fargate for most workloads; EKS for platform-owned workloads and workloads needing Kubernetes-specific ecosystem (service mesh, GPU scheduling, advanced autoscaling).
- Rationale: ECS operational simplicity for application teams; Kubernetes where its ecosystem earns its complexity.
- Outcome: Stable. About 80% on ECS, 20% on EKS.

## TSD-015 — Python typing discipline
- Context: Grew from "optional" to "strongly preferred" in code style.
- Options: Optional type hints, mandatory on all new code, mypy strict.
- Decision: Mandatory type hints on all new code; mypy in CI at strict mode for new modules and non-strict for legacy.
- Rationale: Catches real bugs, improves IDE support, meaningfully reduces review comments.
- Outcome: Strong after initial adoption friction. Refactoring velocity up; incident rate on typed code lower.
