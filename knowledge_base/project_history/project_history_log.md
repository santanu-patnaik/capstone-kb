---
doc_id: project_history_log
title: Organization Project History Log
description: Anonymized history of delivered projects. Used to calibrate estimates, identify variance patterns, and match new BRDs to historical analogs.
updated: 2025-09-30
version: 1.3
---

# Project History Log (Anonymized)

Each row below is a delivered project with its estimated vs actual outcome. This log is used to calibrate new estimates. The narrative after each row captures the top drivers of variance so retrieval can surface lessons relevant to a new BRD.

## PH-001 — Customer Feedback Portal
- Domain: customer_experience
- Complexity: low
- Team size: 4
- Estimated weeks: 8
- Actual weeks: 10
- Variance: +25%
- Status: delivered, healthy
- Top stack: React, Node.js, Postgres, Okta SSO
- Variance driver: SSO SCIM edge cases in a legacy tenant discovered in Phase 3. No other scope changes.
- Lesson: Budget 15% buffer for SSO on internal tools with non-vanilla identity setups.

## PH-002 — P&C Claims Auto-Triage Engine
- Domain: insurance_pc
- Complexity: medium
- Team size: 9
- Estimated weeks: 18
- Actual weeks: 22
- Variance: +22%
- Status: delivered, healthy
- Top stack: Python, FastAPI, SageMaker, MLflow, AWS, Guidewire Cloud API
- Variance driver: Guidewire Cloud API production rate limits were more restrictive than sandbox; explainability UI scope was added late.
- Lesson: Confirm vendor prod rate limits in writing during Phase 1. Explainability is regulatory, not optional — cost it from the start.

## PH-003 — Member Health Portal Unification
- Domain: healthcare
- Complexity: high
- Team size: 18
- Estimated weeks: 36
- Actual weeks: 46
- Variance: +28%
- Status: delivered, healthy
- Top stack: React, Next.js, Node.js, Postgres, Elasticsearch, Redis, Ping Identity, AWS
- Variance driver: Provider directory data reconciliation across four source systems took 14 weeks vs. 6 estimated. HITRUST evidence collection underscoped.
- Lesson: Data reconciliation must be a dedicated track with its own DRI, not a sub-task. HITRUST evidence starts at kickoff, not pre-prod.

## PH-004 — Agent Productivity Suite (ServiceNow Integration)
- Domain: itsm
- Complexity: medium
- Team size: 7
- Estimated weeks: 14
- Actual weeks: 15
- Variance: +7%
- Status: delivered, healthy
- Top stack: ServiceNow, Node.js, Postgres, Okta, AWS Lambda
- Variance driver: Minor — one ServiceNow scoped-app permission issue cost a week.
- Lesson: Projects scoped tightly to a known platform with an experienced team deliver close to plan. Use this as an estimation benchmark for ServiceNow work.

## PH-005 — Fraud Signal Platform
- Domain: insurance_pc
- Complexity: high
- Team size: 12
- Estimated weeks: 26
- Actual weeks: 34
- Variance: +31%
- Status: delivered, quality concerns
- Top stack: Python, Kafka, Flink, Snowflake, AWS, Feast (feature store)
- Variance driver: Feature store choice (Feast) was new to the team; 6 weeks of learning curve. Drift monitoring scope was added in the middle of the project after compliance review.
- Lesson: New infrastructure choices on critical path need a spike and a skills plan. Do not add regulatory monitoring mid-project without a scope conversation.

## PH-006 — Broker Commission Engine
- Domain: insurance_life
- Complexity: medium
- Team size: 8
- Estimated weeks: 20
- Actual weeks: 19
- Variance: −5%
- Status: delivered, healthy
- Top stack: Java, Spring Boot, Oracle, Kafka, Jenkins, on-prem
- Variance driver: Came in slightly ahead because team reused a calculation library from a prior project and dropped 2 weeks of design work.
- Lesson: Reuse opportunities should be surfaced in Phase 1. Stack familiarity plus reusable code is the single biggest accelerator.

## PH-007 — Provider Directory Search v2
- Domain: healthcare
- Complexity: medium
- Team size: 6
- Estimated weeks: 16
- Actual weeks: 24
- Variance: +50%
- Status: delivered, quality concerns
- Top stack: Node.js, Elasticsearch, Postgres, React, AWS
- Variance driver: Data quality was the killer — source data from three provider systems required dedicated reconciliation work. Team initially estimated as a search-UX project; it was actually a data project with a search UX on top.
- Lesson: If a project touches provider or member master data, scope as a data project first, UX project second. Reconciliation dominates.

## PH-008 — Underwriting Decision Support Tool
- Domain: insurance_life
- Complexity: medium
- Team size: 10
- Estimated weeks: 22
- Actual weeks: 23
- Variance: +5%
- Status: delivered, healthy
- Top stack: Python, FastAPI, Postgres, React, AWS SageMaker
- Variance driver: Minor. One week spent on explainability requirement surfaced in compliance review.
- Lesson: Underwriting projects reliably surface explainability requirements. Include a 1-week explainability task by default.

## PH-009 — Customer Identity Consolidation
- Domain: customer_experience
- Complexity: high
- Team size: 14
- Estimated weeks: 30
- Actual weeks: 42
- Variance: +40%
- Status: delivered, quality concerns
- Top stack: Node.js, Postgres, Redis, Ping Identity, Kafka, AWS
- Variance driver: Identity migrations touching multiple tenants and source systems were underestimated; communication lead time to regulated populations (broker, fiduciary) was not in the plan.
- Lesson: Identity migrations with regulated populations require 45–60 days of comms lead time as a critical path item. This is the #1 overrun source in identity projects.

## PH-010 — Call Center AI Assistant (Internal POC)
- Domain: customer_experience
- Complexity: low
- Team size: 3
- Estimated weeks: 8
- Actual weeks: 7
- Variance: −12%
- Status: POC complete
- Top stack: Python, LangChain, Pinecone, OpenAI GPT-4, FastAPI, React
- Variance driver: POC scope was tightly defined; team had prior LLM experience; no integration dependencies.
- Lesson: Tightly-scoped POCs with experienced teams reliably land on or ahead of schedule. Use 8 weeks as the default POC budget for focused LLM work.

## PH-011 — Document Ingestion Service
- Domain: shared_infra
- Complexity: medium
- Team size: 5
- Estimated weeks: 12
- Actual weeks: 14
- Variance: +17%
- Status: delivered, healthy
- Top stack: Python, AWS Textract, S3, SQS, Postgres
- Variance driver: Textract accuracy on low-quality scans required a retry + human-in-loop path that wasn't originally scoped.
- Lesson: Document ingestion projects need a "bad document" scenario in Phase 1 scoping. OCR quality is never 100%.

## PH-012 — Agentic BRD Intelligence POC
- Domain: internal_platform
- Complexity: medium
- Team size: 4
- Estimated weeks: 10
- Actual weeks: 10
- Variance: 0%
- Status: POC complete
- Top stack: Python, LangChain, n8n, Pinecone, OpenAI, FastAPI
- Variance driver: Scope was well-defined. Team used a plan template + knowledge-base pattern from prior projects.
- Lesson: Agentic POCs with multi-agent orchestration land on schedule when (a) the orchestration pattern is committed up front and (b) the RAG corpus is scoped before building.
