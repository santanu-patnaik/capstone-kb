---
doc_id: brd_003
title: Member Health Portal Unification
domain: healthcare
complexity: high
team_size_historical: 18
duration_actual_weeks: 46
duration_estimated_weeks: 36
status: delivered
year: 2024
tags: [HIPAA, multi_tenant, legacy_migration, high_regulated, member_facing]
---

# BRD-003: Member Health Portal Unification

## 1. Business Context
Following three acquisitions in 2021–2023, the health plan now operates four distinct member portals — each with its own authentication, claims display, provider directory, ID card, and secure messaging. Members with coverage spanning multiple acquired plans must maintain separate logins, which drives 38% of inbound call center volume ("I can't see my claim" or "I need a new ID card"). Leadership has mandated a unified member experience: one portal, one login, all lines of business, with parity or better for every feature in the four legacy portals.

## 2. Functional Requirements
- FR-1: Unified SSO — a single member account spanning all lines of business (commercial, Medicaid, Medicare Advantage, dental).
- FR-2: Identity proofing on first login matching CMS Identity Assurance Level 2 (IAL2).
- FR-3: Claims view across all lines of business with filters (status, date, provider, amount).
- FR-4: Digital ID cards (view, share via Apple Wallet / Google Pay, request replacement).
- FR-5: Provider directory with in-network search, specialty filter, telehealth flag, accepting-new-patients flag; backed by a consolidated provider data service.
- FR-6: Secure messaging with plan (HIPAA-compliant), threaded, with attachments and read receipts.
- FR-7: Benefits and coverage summary with year-to-date deductible and out-of-pocket tracking.
- FR-8: Explanation of Benefits (EOB) retrieval, download, and email-to-self.
- FR-9: Claims appeal initiation flow (form + document upload + tracking).
- FR-10: Accessibility compliance — WCAG 2.1 AA across every member-facing surface.
- FR-11: English and Spanish at launch; architecture must support additional languages in subsequent phases.

## 3. Non-Functional Requirements
- NFR-1: 99.95% availability; planned maintenance windows only outside 6 AM – 10 PM local time, any US timezone.
- NFR-2: Page load ≤ 2.5 seconds p95 for authenticated member dashboard.
- NFR-3: Support 1.2M monthly active members; 150k concurrent peak during open enrollment.
- NFR-4: HIPAA Security Rule compliance — encryption, audit controls, integrity, person/entity authentication, transmission security.
- NFR-5: Full PHI audit log with 7-year retention; immutable append-only storage.
- NFR-6: Disaster recovery RPO ≤ 15 min, RTO ≤ 4 hours; active-passive across two AWS regions.
- NFR-7: Zero data loss during migration from legacy portals; members must not lose message history, saved providers, or in-flight appeals.

## 4. Constraints
- Three of the four legacy portals run on on-prem infrastructure that must be decommissioned within 6 months of unified portal go-live.
- HITRUST CSF certification required.
- All PHI processing must occur in the existing HIPAA-compliant AWS landing zone (GovCloud-eligible accounts).
- Existing enterprise identity provider (Ping Identity) must be extended, not replaced.
- Member data migration requires CMS notification for Medicare Advantage members 60 days prior to cutover.

## 5. Success Criteria
- Inbound "can't see my claim / ID card" call volume reduced by 60% within 6 months post-launch.
- Member NPS on portal experience ≥ 40 (current range across legacy portals: −5 to 22).
- Portal authentication success rate ≥ 98% (current: 89% across legacy portals).
- All four legacy portals retired within 6 months of unified launch.
- Zero HIPAA compliance incidents related to the unification project.

## 6. Out of Scope
- Provider-facing portal unification (separate initiative).
- Broker-facing portal (Phase 2).
- In-app telehealth video (redirect to existing partner platform).
- Wearable / device integrations.

## 7. Stakeholders
- Executive sponsor: Chief Operating Officer
- Business owners: VP Member Experience, VP Operations (one per line of business, 4 total)
- Technical owners: VP Engineering, Chief Information Security Officer
- Compliance: HIPAA Privacy Officer, HITRUST Program Manager, State Regulatory Liaison
- Delivery team: 18 engineers across 3 SAFe teams (Identity, Claims/EOB, Provider/Messaging) + 1 RTE

## 8. Lessons Captured at Close
- 10-week overrun sources: (1) provider directory data reconciliation across four source systems took 14 weeks, not 6 — data quality was far worse than the initial audit suggested; (2) HITRUST evidence collection required a dedicated compliance engineer we hadn't staffed; (3) two legacy portal cutovers slipped because of member communication lead time.
- What worked: SAFe ART structure with three feature teams kept dependencies visible. PI planning was essential.
- What to do differently: data reconciliation should be its own track with its own DRI, not a sub-task of the feature team. HITRUST evidence collection should start at kickoff, not at pre-prod.
- Stack decision held up well: React + Next.js + Node + Postgres + Elasticsearch (for provider search) + Redis. SageMaker was not used; no ML component in scope.
