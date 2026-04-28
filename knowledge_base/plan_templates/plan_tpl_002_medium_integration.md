---
doc_id: plan_tpl_002
title: Medium Integration-Heavy Project — Standard Plan Template
applicability: enterprise integration (Guidewire, ServiceNow, Salesforce), team size 6–10, duration 16–24 weeks, moderate regulatory scope
complexity: medium
source: org_pmo_library
version: 3.0
---

# Plan Template: Medium Integration-Heavy Project

## When to use this template
- Meaningful integration with enterprise system of record (Guidewire, ServiceNow, Salesforce, SAP, etc.).
- Team size 6–10 including EM and SRE.
- Moderate regulatory exposure (SOX, SOC 2, PCI — but not HIPAA/HITRUST full scope).
- May include ML or advanced analytics component.
- Expected duration 16–24 weeks.

## Phase structure

### Phase 1 — Discovery & Architecture (3 weeks)
- Business requirements workshop (3 sessions).
- Integration discovery: map every upstream and downstream system; inventory APIs, rate limits, data contracts, SLAs.
- Architecture design — document pattern choice (event-driven vs request-response), data flow, NFR mapping.
- Compliance pre-review — identify controls impact and evidence needs.
- Risk register with top 10 risks and mitigations.
- Exit criteria: ADR signed; integration inventory complete; compliance scope confirmed.

### Phase 2 — Foundation & Contracts (3–4 weeks)
- Build integration adapters as independent modules against vendor sandbox/staging.
- Lock down all API contracts (OpenAPI or equivalent) and publish to org registry.
- Set up observability baseline (logging schema, tracing, SLIs).
- Security baseline — secrets management, network segmentation, audit logging wiring.
- Exit criteria: adapters passing contract tests against sandbox; observability emitting to prod dashboards.

### Phase 3 — Core Build (5–8 weeks)
- Build core business logic in 2-week iterations.
- Integrate adapters into core flows one system at a time.
- Weekly demo; bi-weekly stakeholder review.
- Feature flags on every integration point for staged rollout.
- Exit criteria: end-to-end flows working in staging against vendor staging/sandbox.

### Phase 4 — Hardening, Performance & Compliance (3–4 weeks)
- Performance test against NFR targets — including CAT/peak scenarios.
- Validate throughput against real vendor production rate limits (not sandbox numbers).
- Compliance evidence collection — audit logs, change records, access reviews.
- Third-party security review / pen test.
- DR runbook creation and tabletop exercise.
- Exit criteria: NFRs met in prod-equivalent environment; compliance sign-off; DR runbook validated.

### Phase 5 — Pilot, UAT & Launch (2–3 weeks)
- Pilot with controlled cohort (e.g., 10% of volume or one region).
- 2-week soak in pilot; decision gate at day 10.
- Staged rollout 25 → 50 → 100%.
- Post-launch hypercare for 2 weeks.
- Exit criteria: P0/P1 bug count zero; business owner sign-off; full rollout complete.

## Standard risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Vendor API rate limits below assumption | High | High | Confirm prod rate limits in writing during Phase 1; load-test against prod limits in Phase 4, not sandbox. |
| Vendor staging environment diverges from prod | Medium | High | Include a vendor-prod smoke test in Phase 4; budget buffer for prod-only surprises. |
| Regulatory scope expands mid-project | Medium | High | Compliance pre-review in Phase 1; re-review at end of Phase 2 and Phase 4. |
| ML model underperforms in prod vs offline | Medium | Medium | Shadow-mode deployment for minimum 2 weeks before recommendations are written back. |
| Legacy data quality worse than audit indicated | Medium | Medium | Data profiling task in Phase 2 independent of feature work; dedicated reconciliation engineer if multi-source. |
| Peak-load scenario not exercised | High | High | Phase 4 must include CAT/peak simulation at 2x expected peak. |

## Standard milestones
- M1 (end of Phase 1): Architecture signed; risk register reviewed with stakeholders.
- M2 (end of Phase 2): All adapters contract-compliant against vendor sandbox.
- M3 (end of Phase 3): End-to-end in staging.
- M4 (end of Phase 4): NFRs met + compliance signed.
- M5 (pilot day 10): Go/No-Go for staged rollout.
- M6 (full launch): 100% rollout complete.

## Team composition (typical)
- 1 EM (75%)
- 1 staff engineer / tech lead
- 3–5 senior engineers
- 1 integration specialist
- 1 SRE
- 1 data scientist (if ML component)
- 0.25 designer
- 0.25 compliance engineer (more if HIPAA/HITRUST)

## Typical variance band
Historical projects using this template have delivered within +10% to +25% of planned duration. The #1 overrun source is vendor integration surprises discovered in Phase 4; the #2 source is compliance evidence collection underscoped. Plan buffer of 20% is typical and recommended.
