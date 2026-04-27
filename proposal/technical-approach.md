# Technical Approach

**RFP #MC-2026-0417 — Meridian Components Inventory Dashboard Modernization**

---

## Our approach

We start from the code, not from assumptions. Before estimating or committing to any timeline, we reviewed the source code and the previous vendor's handoff notes provided with this RFP. That review informed our scope estimate and shaped the sequencing below.

Our general principle is incremental delivery with test coverage at every step. We do not deliver a working feature and then add tests — we establish tests as part of each fix or build, so that Meridian IT has a growing safety net from day one, not a retrospective artifact at the end.

---

## R1 — Reports module remediation

We will perform a systematic audit of the Reports module, mapping each filter (Time Period, Warehouse, Category, Order Status) against the backend API to verify correct wiring. From our review of the vendor handoff, we know to expect: incomplete filter wiring, internationalization gaps in displayed strings, and API pattern inconsistencies (some components use the older Vue Options API rather than Composition API, which the rest of the codebase has adopted).

Each defect will be documented with: the observed behavior, the root cause, and the fix applied. Meridian will receive a fix log alongside the corrected module, so your IT team has a clear record of what changed and why.

**Deliverable:** Fully functional Reports module + written fix log.

---

## R3 — Automated browser testing

We will use Playwright for end-to-end browser test coverage. Playwright is already present in the repository; we will build on the existing setup rather than introduce new tooling.

Tests will cover the critical user flows across all modules:

- **Inventory:** browsing, filtering by warehouse and category
- **Orders:** filtering by status, warehouse, and time period
- **Reports:** all filter combinations (this is where we expect the most regression risk)
- **Restocking:** budget input, recommendation generation, result display

Critically, we write tests *during* R1, not after. Each Reports fix gets a corresponding regression test before we move on. By the time R2 is complete, the suite covers the whole application.

**Deliverable:** Playwright test suite, documented setup instructions, and a passing run in CI — so your IT team can execute it as a gate before approving any future change.

---

## R2 — Restocking recommendations

The Restocking view will take three inputs: current stock levels (per warehouse and category), the demand forecast already exposed by the `/api/demand` endpoint, and an operator-supplied budget ceiling entered directly in the UI.

The recommendation algorithm prioritizes items with the lowest stock-to-demand ratio — those at highest risk of stockout — and proposes order quantities up to the budget ceiling. If the ceiling is reached, remaining items are ranked and listed so the operator can make an informed cut.

The UI will be a new "Restocking" view integrated into the existing navigation, using the same design tokens, layout patterns, and filter components as the current modules. The output is a table showing: item, recommended quantity, estimated cost, and warehouse. The table will be exportable.

**Deliverable:** Restocking view, backend endpoint, and Playwright tests covering the full flow.

---

## R4 — Architecture documentation

We will produce a current-state architecture overview covering:

- **Frontend:** Vue 3 component structure, routing, API client patterns, i18n setup
- **Backend:** FastAPI endpoint inventory, filtering logic, Pydantic data models
- **Data layer:** JSON file structure in `server/data/`, how mock data is loaded and filtered

The deliverable will be an HTML document with an interactive architecture diagram, written for Meridian IT staff — not for developers. We will also flag any technical debt observed during the engagement so your team has a clear picture of what was inherited, what was fixed, and what remains.

**Deliverable:** `architecture.html` — standalone document, no external dependencies.

---

## D2 — Internationalization (included in fixed fee)

We will verify whether Vue i18n is already configured in the project. If not, we will introduce it. We will then map all hard-coded strings in Vue templates that are not yet externalized, and add a Japanese locale (`ja`) with full translations.

Priority order: Inventory and Orders views first (highest daily use by Tokyo team), then Reports after R1 remediation, then Restocking once R2 is complete. The structure will be designed to support additional locales without further engineering effort.

**Deliverable:** i18n-enabled application with Japanese locale, all modules covered.

---

## What is not included

UI modernization (D1) and dark mode (D3) are not included in the fixed fee. Both are available as separate options; see the Pricing section.
