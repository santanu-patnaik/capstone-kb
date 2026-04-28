---
doc_id: brd_001
title: Customer Feedback Portal
domain: customer_experience
complexity: low
team_size_historical: 4
duration_actual_weeks: 10
duration_estimated_weeks: 8
status: delivered
year: 2023
tags: [internal_tool, CRUD, small_scale]
---

# BRD-001: Customer Feedback Portal

## 1. Business Context
The Customer Success team currently collects feedback via scattered email threads, Slack DMs, and a shared spreadsheet. This makes trend analysis impossible and response SLAs inconsistent. Leadership has mandated a centralized portal where customers submit structured feedback and CS managers triage, respond, and report on it.

## 2. Functional Requirements
- FR-1: Customers authenticate via existing SSO (Okta) and submit feedback categorized by product area, severity, and sentiment.
- FR-2: CS managers view a triage dashboard with filters (category, severity, age, assigned rep).
- FR-3: Managers assign tickets to reps; reps respond in-thread; customers receive email notifications on status change.
- FR-4: Weekly rollup report exported to CSV and emailed to leadership every Monday 9 AM PT.
- FR-5: Basic search across feedback body, customer name, company.

## 3. Non-Functional Requirements
- NFR-1: 99.5% uptime during business hours (8 AM – 8 PM PT).
- NFR-2: Page load under 2 seconds for dashboards with up to 500 open tickets.
- NFR-3: SOC 2 Type II controls — audit logging, encryption at rest and in transit.
- NFR-4: Support 200 concurrent CS users; 5,000 customer submissions/month.

## 4. Constraints
- Must integrate with existing Okta and Salesforce (for customer lookup).
- Infrastructure must run on approved AWS stack (no new cloud vendors).
- Go-live before end of Q2 2023.

## 5. Success Criteria
- 80% of feedback tickets acknowledged within 24 hours (vs. 40% current).
- Manager time-to-triage reduced from 12 min to under 3 min.
- Leadership report automated end-to-end (zero manual spreadsheet work).

## 6. Out of Scope
- Customer-facing analytics.
- Mobile native apps (web-responsive only).
- Integration with external review sites (G2, Trustpilot).

## 7. Stakeholders
- Executive sponsor: VP Customer Success
- Product owner: Senior CS Ops Manager
- Engineering: Internal Tools pod (4 engineers)

## 8. Lessons Captured at Close
- Scope held well; 2-week overrun came from Okta SCIM edge cases in a legacy tenant.
- Team familiarity with the React/Node/Postgres stack was the biggest single driver of on-time delivery.
- Recommend: for similar internal CRUD tools, default to the same stack and budget 15% buffer for SSO integration.
