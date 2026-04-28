---
doc_id: brd_002
title: P&C Claims Auto-Triage Engine
domain: insurance_pc
complexity: medium
team_size_historical: 9
duration_actual_weeks: 22
duration_estimated_weeks: 18
status: delivered
year: 2024
tags: [ML_ops, integration_heavy, regulated, Guidewire]
---

# BRD-002: P&C Claims Auto-Triage Engine

## 1. Business Context
The Claims Operations team processes roughly 8,000 first notice of loss (FNOL) submissions per month across auto and property lines. Today, every FNOL is manually reviewed by a triage adjuster who assigns severity (low/medium/high), complexity, and fraud suspicion — then routes to the appropriate queue. The average triage cycle is 4.2 hours; severity mis-assignments cause downstream rework in ~11% of cases. Leadership wants an ML-assisted triage engine integrated with Guidewire ClaimCenter that recommends severity, complexity, fraud risk, and adjuster pool within 90 seconds of FNOL intake.

## 2. Functional Requirements
- FR-1: Consume FNOL events from Guidewire ClaimCenter via the Cloud API when a claim is created.
- FR-2: Enrich with policy data (coverage, deductible, claims history) pulled from the PolicyCenter integration layer.
- FR-3: Score the claim across four dimensions: severity (L/M/H), complexity (L/M/H), fraud risk (0–100), adjuster pool recommendation (standard / specialist / SIU).
- FR-4: Write the recommendation back to ClaimCenter as a structured note plus tags; the human adjuster retains final authority (recommendation, not decision).
- FR-5: Expose an audit trail view showing model inputs, score, confidence, and model version for every claim.
- FR-6: Adjusters can override any recommendation; override reason is captured in a structured dropdown + free text and fed back to the training set.
- FR-7: Weekly model performance report (precision/recall per dimension, override rate, drift indicators).

## 3. Non-Functional Requirements
- NFR-1: End-to-end latency ≤ 90 seconds from FNOL creation to recommendation posted.
- NFR-2: 99.9% availability during extended business hours (6 AM – 10 PM PT, 7 days).
- NFR-3: Model explainability: top 3 feature contributions exposed for every score (regulatory requirement).
- NFR-4: Data retention 7 years; all training data and model artifacts versioned and reproducible.
- NFR-5: PII protection — no raw claimant PII leaves the VPC; all model features must be transformed or pseudonymized before model inference.
- NFR-6: Handle peak volume of 400 FNOLs/hour during CAT events (vs. ~15/hour normal).

## 4. Constraints
- Guidewire ClaimCenter 10.x on PolicyCenter 10.x — must integrate via Cloud API, not direct DB access.
- Compliance sign-off required from Legal, Privacy, and State Insurance Commissioner liaison before go-live.
- Approved cloud: AWS (GovCloud-compatible region for PII workloads).
- No new vendor contracts; must use existing MLOps tooling (SageMaker + existing MLflow registry).

## 5. Success Criteria
- Average triage cycle reduced from 4.2 hours to under 30 minutes.
- Severity mis-assignment rate reduced from 11% to under 5%.
- Fraud detection catch rate on SIU-referred claims improved by at least 15% vs. current rule-based triage.
- Adjuster override rate at steady state ≤ 20% (higher means model isn't trusted or is wrong).

## 6. Out of Scope
- Full claims settlement automation (triage only).
- Integration with third-party fraud vendors (Phase 2).
- Commercial lines (personal auto and property only in Phase 1).

## 7. Stakeholders
- Executive sponsor: Chief Claims Officer
- Business owner: VP Claims Operations
- Technical owner: Director of Claims Platform Engineering
- Compliance: Privacy Officer, Chief Compliance Officer
- Delivery team: 9 engineers (2 data scientists, 3 backend, 2 integration, 1 SRE, 1 EM)

## 8. Lessons Captured at Close
- 4-week overrun came almost entirely from Guidewire Cloud API rate limits discovered late — the integration team had estimated throughput off a staging tenant that was throttled differently than prod.
- Explainability (NFR-3) added meaningful scope that wasn't costed; regulators pushed for feature contribution UI, not just logs.
- MLflow registry worked well; would reuse. SageMaker endpoint cold starts required pre-warming during CAT events.
- Recommendation for similar projects: treat Guidewire integration throughput as a first-class risk item, not an implementation detail.
