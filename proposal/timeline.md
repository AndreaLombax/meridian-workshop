# Delivery Timeline

**RFP #MC-2026-0417 — Meridian Components Inventory Dashboard Modernization**

---

Engagement duration: **10 weeks** from contract execution. Phased delivery with formal acceptance at each milestone.

---

## Phase 1 — Foundation (Weeks 1–3)

**R4: Architecture review + R1: Reports audit**

We begin with two parallel workstreams: architecture documentation and a systematic audit of the Reports module. These are read-before-write activities — we produce no new features until we understand what we're working with.

- Week 1: Codebase review, architecture documentation draft, Reports defect mapping
- Week 2: Reports fixes begin (filter wiring, API patterns); architecture doc finalized
- Week 3: Reports remediation complete; Playwright test suite initialized; first tests written

**Milestone 1 (end of Week 3):** R4 (current-state) delivered; R1 delivered with fix log; baseline Playwright suite passing. Acceptance criteria for Week 10 final QA agreed with Meridian IT at this milestone.

---

## Phase 2 — Core build (Weeks 4–7)

**R2: Restocking recommendations + R3: Test coverage**

With the Reports module stable and a test suite in place, we build the Restocking view. Tests are authored alongside feature development — not after.

- Week 4: Backend endpoint for restocking recommendations; algorithm implementation. **End of Week 4: algorithm logic review with VP Operations (1-hour session). Frontend build begins only after approval.**
- Week 5: Frontend Restocking view; filter and budget input UI; test cases authored in parallel
- Week 6: Integration, edge cases, export functionality; test cases authored in parallel
- Week 7: Full Playwright suite integrated and passing across all modules; regression complete

**Milestone 2 (end of Week 7):** R2 delivered; R3 complete with full suite passing.

---

## Phase 3 — Internationalization + handoff (Weeks 8–10)

**D2: i18n + final QA + delivery**

With all required items delivered, we extend i18n to remaining modules and prepare for handoff.

- Week 8: Vue i18n setup (if not already present); Japanese locale for Inventory and Orders
- Week 9: Japanese locale extended to Reports and Restocking; full application review
- Week 10: Final QA pass per agreed acceptance criteria; R4 revised to reflect R2 and D2 changes; documentation handoff; walkthrough with Meridian IT

**Milestone 3 (end of Week 10):** D2 delivered; all Playwright tests green; zero P1/P2 defects in acceptance checklist. Acceptance by Meridian IT within 3 business days of delivery. Engagement closed.

---

## Scope boundaries

| In scope | Out of scope / change request trigger |
|---|---|
| All defects identified during Week 1 Reports audit | Defects discovered after Week 1 audit sign-off |
| Algorithm logic as approved at Week 4 checkpoint | Changes to recommendation logic post-approval |
| Technical integration of i18n strings | Translation accuracy (Meridian provides or approves translations) |
| Architecture doc reflecting codebase at Week 2 + final delta at Week 10 | Rearchitecting beyond component-level changes |

Any out-of-scope item discovered during the engagement is estimated separately within 5 business days of identification, with no work proceeding until Meridian approves.

---

## Assumptions

- Contract execution by **May 26, 2026** (estimated, pending RFP award)
- **Meridian IT provides staging environment access by Week 1, Day 3.** If delayed, milestone dates shift day-for-day.
- To protect the timeline, milestone reviews are scheduled proactively. We send materials 2 days before each review so your team can prepare feedback efficiently. We carry a 3-day internal buffer per phase; client delays beyond 5 business days trigger a schedule revision.
- Codebase reviewed during proposal phase; no blockers identified. Any architectural issue requiring rearchitecting beyond component-level changes is treated as a change request per the scope table above.

---

## Optional extensions (post-engagement)

| Item | Estimated duration |
|---|---|
| D1 — UI modernization | +2 weeks |
| D3 — Dark mode | +1 week |
