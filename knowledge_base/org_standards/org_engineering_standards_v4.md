---
doc_id: org_standards_v4
title: Engineering Standards & Approved Stacks
version: 4.1
owner: Office of the CTO
last_updated: 2026-01-15
---

# Engineering Standards & Approved Stacks (v4.1)

These standards apply to all new engineering projects. Exceptions require written approval from the architecture review board.

## 1. Approved Technology Stacks

### 1.1 Backend Services
| Tier | Language / Runtime | Framework | Use Case |
|---|---|---|---|
| Tier 1 (default) | Python 3.11+ | FastAPI | General backend services, ML-adjacent work, APIs |
| Tier 1 (default) | Node.js LTS (20+) | Express or NestJS | Frontend-adjacent APIs, BFF, integration services |
| Tier 1 | Java 21 LTS | Spring Boot 3 | Regulated systems, long-running batch, Guidewire adjacent |
| Tier 2 | Go 1.22+ | stdlib + chi | High-throughput services, platform infrastructure |
| Restricted | .NET 8 | ASP.NET Core | ServiceNow-adjacent only |
| Restricted | Ruby on Rails | — | Legacy maintenance only; no new projects |

### 1.2 Frontend
| Tier | Framework | Notes |
|---|---|---|
| Tier 1 (default) | React 18+ with TypeScript | Web apps, internal tools |
| Tier 1 | Next.js 14+ | Server-rendered member/customer-facing experiences |
| Tier 2 | React Native | Cross-platform mobile only; native Swift/Kotlin preferred for member apps |
| Restricted | Angular | Legacy maintenance only |

### 1.3 Data Stores
| Tier | Store | Use Case |
|---|---|---|
| Tier 1 (default) | PostgreSQL 15+ (RDS Aurora) | Transactional, general purpose |
| Tier 1 | Redis 7+ (ElastiCache) | Caching, session, rate limiting |
| Tier 1 | Elasticsearch / OpenSearch | Full-text and faceted search |
| Tier 1 | S3 | Blob, document, data lake |
| Tier 2 | DynamoDB | High-throughput key-value, known access patterns |
| Tier 2 | Snowflake | Analytics and reporting warehouse |
| Restricted | MongoDB | Case-by-case — use Postgres JSONB by default |
| Restricted | Oracle | Legacy only; migrations to Postgres in progress |

### 1.4 Messaging
| Tier | Platform | Use Case |
|---|---|---|
| Tier 1 (default) | Amazon SQS + SNS | Task queues, fan-out, standard async work |
| Tier 1 | Amazon EventBridge | Event routing, cross-service integration |
| Tier 2 | Kafka (MSK) | High-throughput event streaming, event sourcing |

### 1.5 ML / AI Platform
| Tier | Platform / Tool | Use Case |
|---|---|---|
| Tier 1 (default) | SageMaker | Training, inference, endpoints |
| Tier 1 | MLflow (existing registry) | Model registry, experiment tracking |
| Tier 1 | Bedrock / OpenAI via approved gateway | LLM inference for internal tools |
| Tier 2 | Pinecone | Vector database (managed) |
| Tier 2 | Feast | Feature store (evaluate skills before adopting) |
| Tier 2 | LangChain / LangGraph / n8n | Agent orchestration — approved for POC and internal tools |
| Restricted | Direct LLM vendor SDKs without gateway | Must go through approved API gateway |

### 1.6 Cloud
AWS is the primary cloud. GovCloud-eligible accounts are required for all PHI, PII, and regulated workloads. Multi-region active-passive is the standard for customer-facing workloads with DR requirements. No new GCP or Azure workloads without architecture review board approval.

## 2. CI/CD Standards

- Source control: GitHub Enterprise. One repo per service; monorepo allowed with approval.
- Branching: trunk-based with short-lived feature branches. PRs require at least one approval and all checks passing.
- CI: GitHub Actions. Required checks: lint, unit tests (≥70% coverage for new code), static analysis (SonarQube), SAST (Snyk Code), dependency scan (Snyk Open Source), secret scan (GitLeaks).
- CD: GitHub Actions → ArgoCD for Kubernetes; GitHub Actions → CodeDeploy/Lambda for serverless.
- Environments: dev → staging → prod. Prod deploys require change ticket; automated for everything except regulated workloads.
- Feature flags: LaunchDarkly for user-facing changes; config flags in-code acceptable for infrastructure changes.
- Rollback: every deploy must be rollback-tested in staging. No "forward-fix only" culture.

## 3. Observability

Every service must emit:
- Structured JSON logs via the org log framework.
- Metrics — RED (Rate, Errors, Duration) for every endpoint; USE (Utilization, Saturation, Errors) for every resource.
- Traces — OpenTelemetry, propagated through all internal calls.
- SLOs — at least one SLO per service, tracked in the SLO dashboard.

Centralized in Datadog + CloudWatch for metrics/traces and in the org-managed log platform for logs. Direct writes to Splunk are not permitted; use the log framework.

## 4. Security

### 4.1 Authentication / Authorization
- SSO: Okta for employees; Ping Identity for customers/members.
- Service-to-service: mTLS inside the VPC where possible; short-lived tokens from the org secrets service otherwise.
- No passwords in code; no long-lived credentials. Secrets managed in AWS Secrets Manager + org rotation pipeline.

### 4.2 Data Classification
- Public / Internal / Confidential / Restricted. Restricted = PII, PHI, PCI, financial.
- Restricted data must be encrypted at rest with customer-managed keys (KMS), in transit with TLS 1.3.
- Restricted data in logs must be hashed or redacted; no raw PII in log streams, ever.

### 4.3 Reviews
- Architecture review board review required for: new cloud regions, new vendors, new data stores, new external integrations, any Restricted data flow.
- Security review required for: customer/member-facing endpoints, any handling of Restricted data, any service running outside the VPC.
- Pen test required before go-live for customer/member-facing surfaces.

## 5. Architecture Review Criteria

New architectures are evaluated against the following criteria. A proposal should explicitly address each.

1. **Scalability** — What is the expected load at launch, 1 year, 3 years? How does the design scale each tier?
2. **Availability & DR** — What is the availability target? What is the DR pattern (active-passive cross-region, multi-AZ within region, etc.)? What is RPO/RTO?
3. **Security & compliance** — What data classifications are involved? Which regulatory regimes apply? Where are the trust boundaries?
4. **Operational complexity** — How many services? How many teams? What's the on-call story?
5. **Cost profile** — What is the expected monthly cost at launch and at scale? What are the cost drivers?
6. **Integration surface** — Which internal and external systems are touched? What are their SLAs? What is the blast radius if one is down?
7. **Team familiarity** — Does the proposing team have experience with the chosen stack? If not, what's the skills plan?
8. **Exit strategy** — If this architecture doesn't work out, what is the migration path?

## 6. Coding Standards (Summary)
- Language-specific style: Black + Ruff for Python; Prettier + ESLint for JS/TS; gofmt for Go; Google Java Format for Java.
- Commit messages: Conventional Commits.
- Reviews: focused on correctness, testability, and readability — not style (style is a pre-commit concern).
- Documentation: every service has a README covering purpose, run instructions, dependencies, on-call runbook link.

## 7. Deprecations (Current)

| Item | Status | Target EOL | Migration |
|---|---|---|---|
| Oracle DB | Deprecated | Q4 2027 | Postgres |
| Angular 1 and 2 | Deprecated | Q2 2026 | React |
| On-prem Jenkins | Deprecated | Q3 2026 | GitHub Actions + ArgoCD |
| MongoDB in new projects | Restricted | — | Postgres JSONB |
| .NET Framework (not Core) | Deprecated | Q1 2027 | .NET 8 or migrate off |
