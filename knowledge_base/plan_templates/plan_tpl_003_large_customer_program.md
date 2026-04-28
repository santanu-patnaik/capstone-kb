---
doc_id: plan_tpl_003
title: Large Customer-Facing Multi-Team Program — Standard Plan Template
applicability: customer-facing unification/migration, multi-team (SAFe ART), team size 12–25, duration 9–14 months, heavy regulatory
complexity: high
source: org_pmo_library
version: 2.4
---

# Plan Template: Large Customer-Facing Multi-Team Program

## When to use this template
- Customer- or member-facing experience at scale (>500k MAU).
- Multi-team structure required (SAFe ART with 2–4 feature teams).
- Heavy regulatory exposure (HIPAA, HITRUST, PCI-DSS L1, SOX combined).
- Legacy migration or multi-system unification in scope.
- Expected duration 9–14 months.

## Phase structure

### Phase 1 — Program Inception (4 weeks)
- RTE assigned; ART composition confirmed.
- Executive steering committee formed with monthly cadence.
- Epic-level business case signed by executive sponsor.
- Architecture runway built — foundational patterns selected (identity, data, observability).
- Compliance kickoff — identify all applicable regimes; engage external auditor early.
- Exit criteria: program charter signed; ART roster complete; first PI planning scheduled.

### Phase 2 — PI 1 (Foundation) (10 weeks, two PIs)
- PI planning ceremonies (2 per 10-week PI).
- Build shared foundation: identity, data platform, observability, security controls, DR pattern.
- Each team delivers enabler features; no customer-facing features in PI 1.
- System demo at end of each iteration; Inspect & Adapt at end of PI.
- Exit criteria: foundation services in prod (used by no one yet); DR tabletop passed; compliance evidence collection tooling online.

### Phase 3 — PI 2 (Vertical Slice) (10 weeks)
- Each team delivers a vertical slice of its feature area end-to-end.
- First HITL / pilot user cohort engaged (employees, family & friends, or similar) at end of PI.
- Parallel: data migration dry-runs on non-production data.
- Exit criteria: vertical slice demoed to exec steering; pilot cohort has positive signal; migration dry run successful.

### Phase 4 — PIs 3–4 (Feature Completion) (20 weeks)
- Feature completion across all teams, prioritized by business value.
- Beta cohort expanded progressively (1k → 10k → 50k users).
- External pen test during this phase.
- Compliance evidence collection ongoing — dedicated compliance engineer on ART.
- Legacy decommission planning begins; lead time for member/customer communication is long (45–60 days minimum for regulated populations).
- Exit criteria: feature parity + UX improvements demonstrated; pen test findings remediated; HITRUST/HIPAA interim audit clean; legacy decommission comms sent.

### Phase 5 — Staged Migration & Launch (10 weeks)
- Production migration in cohorts (weekly waves).
- Legacy system runs in parallel for minimum 30 days per cohort.
- Real-time migration health dashboard monitored by SRE + business ops.
- Hypercare escalation path from day 1.
- Exit criteria: 100% of users migrated; legacy systems decommissioned; 30-day post-launch health review clean.

## Standard risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Data quality worse than audit indicated (multi-source reconciliation) | High | High | Dedicate a reconciliation track with its own DRI from Phase 1, not Phase 3. |
| Compliance evidence underscoped | High | High | Embedded compliance engineer from Phase 1; evidence collection is a feature, not a documentation task. |
| Legacy decommission slips due to communication lead time | Medium | High | Send regulated-population communication at Phase 4 start; treat comms lead time as a critical path item. |
| PI cadence can't absorb executive scope changes | High | Medium | Executive steering monthly (not per-PI); scope changes enter backlog at Program Increment boundaries only. |
| Identity/SSO complexity underestimated | High | High | Identity is a PI 2 vertical slice with its own team; don't distribute identity work across feature teams. |
| Performance at peak (open enrollment, CAT event) not simulated | High | High | Phase 4 must include 2x-peak simulation; any red flag blocks Phase 5 entry. |
| External auditor engagement late | Medium | High | Engage external auditor at Phase 1 kickoff; don't wait until pre-prod. |
| HITL / pilot cohort too small to surface real problems | Medium | Medium | Pilot cohorts must be ≥1% of target population and include representative diversity. |

## Standard milestones
- M1 (end of Phase 1): Program chartered, ART formed, PI 1 scheduled.
- M2 (end of PI 1): Foundation in prod; DR tabletop passed.
- M3 (end of PI 2): Vertical slice demoed; pilot cohort engaged.
- M4 (mid PI 4): Pen test clean; HITRUST interim clean.
- M5 (end of PI 4): Beta at 50k users; legacy decommission comms sent.
- M6 (Phase 5 kickoff): First migration wave cutover.
- M7 (end of Phase 5): 100% migration; legacy retired.

## Team composition (typical)
- 1 RTE
- 3 EMs (one per feature team)
- 1 program architect / chief architect
- 3 tech leads (one per team)
- 12–18 engineers across teams
- 2 SREs (shared across ART)
- 1 data engineer (migration track)
- 1 compliance engineer (embedded)
- 1 QA lead / test automation
- 1 designer lead + 1–2 designers
- 0.5 technical writer (regulated populations need plain-language docs)

## Typical variance band
Historical programs using this template have delivered within +15% to +35% of planned duration. Primary overrun sources in order:
1. Data reconciliation / migration complexity (nearly every program underestimates this)
2. Compliance evidence collection
3. Legacy decommission communication lead time
4. PI planning absorbing executive scope shifts

A 25% schedule buffer is standard. Programs without that buffer have historically slipped more than programs with it, not less.
