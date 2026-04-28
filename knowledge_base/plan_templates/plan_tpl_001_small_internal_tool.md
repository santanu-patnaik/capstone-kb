---
doc_id: plan_tpl_001
title: Small Internal Tool — Standard Plan Template
applicability: internal tools, CRUD apps, team size 3–5, duration 6–10 weeks, low regulatory scope
complexity: low
source: org_pmo_library
version: 2.1
---

# Plan Template: Small Internal Tool

## When to use this template
- Internal-only audience (no external customer exposure).
- CRUD-heavy; minimal ML or heavy integration.
- Team size 3–5 including one EM.
- No new compliance regime (SOC 2 / HIPAA already covered by platform).
- Expected duration 6–10 weeks.

## Phase structure

### Phase 1 — Discovery & Design (1 week)
- Requirements walkthrough with business owner.
- Clickable prototype (Figma) reviewed with 3 end-users.
- Technical design doc (1 page) — data model, API surface, auth approach.
- Exit criteria: design doc signed off; backlog of stories for Phase 2 groomed.

### Phase 2 — Core Build (3–5 weeks)
- Build in 1-week iterations; demo to business owner weekly.
- Target 70% of functional scope delivered before Phase 3.
- Exit criteria: main happy paths working end-to-end in staging; one real end-user has clicked through.

### Phase 3 — Hardening & Integration (1–2 weeks)
- Integration tests, SSO wiring, audit logging, accessibility pass.
- Performance test against NFR targets.
- Security review with platform team (not full pen test for internal tools).
- Exit criteria: NFRs met; platform sign-off on security checklist.

### Phase 4 — UAT & Launch (1 week)
- Business owner + 5 pilot users run structured UAT.
- Bug triage and fix; launch readiness review.
- Production deploy behind feature flag; 48-hour monitoring watch.
- Exit criteria: zero P0/P1 bugs; business owner sign-off; launched.

## Standard risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| SSO integration edge cases (SCIM, legacy tenants) | Medium | Medium | Budget 15% of total plan as buffer; test against a non-trivial tenant in Phase 2. |
| Scope creep from stakeholders seeing the demo | High | Medium | Weekly demo is also weekly scope review; new requests go to Phase 2 backlog or later. |
| End-user adoption weaker than expected | Medium | Low | Pilot with 5 users in Phase 4; don't announce to full org until pilot shows usage. |

## Standard milestones
- M1 (end of week 1): Design doc signed off.
- M2 (end of Phase 2): Demo of core flows.
- M3 (end of Phase 3): NFRs met + security sign-off.
- M4 (launch): Feature flag enabled for full audience.

## Team composition (typical)
- 1 EM (50%)
- 2–3 full-stack engineers
- 0.5 designer (Phase 1 heavy)
- 0.25 SRE/platform (Phase 3 heavy)

## Typical variance band
Historical projects using this template have delivered within ±15% of planned duration. The most common overrun source is SSO/identity integration, followed by stakeholder-driven scope additions.
