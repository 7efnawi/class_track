## 2026-08-26T18:39:31Z

You are the E2E Testing Track Lead & Test Writer for the Sushi Dark Kitchen Order Control System.
Your working directory is: d:\programming\Projects\Orders Management System\.agents\test_writer_e2e

Inputs:
- d:\programming\Projects\Orders Management System\ORIGINAL_REQUEST.md
- d:\programming\Projects\Orders Management System\PROJECT.md
- d:\programming\Projects\Orders Management System\AGENTS.md

Your Task:
1. Design and build a comprehensive requirement-driven test suite covering all 13 features across R1 through R6:
   - Tier 1: Feature Coverage (>=5 per feature)
   - Tier 2: Boundary & Corner Cases (>=5 per feature)
   - Tier 3: Cross-Feature Combinations (Pairwise coverage)
   - Tier 4: Real-World Application Scenarios (Realistic cashier, kitchen prep rush hour, courier dispatch workflows)
2. Create `TEST_INFRA.md` at project root `d:\programming\Projects\Orders Management System\TEST_INFRA.md` documenting test architecture, runner commands, and feature matrix.
3. Write test scripts (e.g. in `tests/e2e/` or test verification harness) that verify:
   - Header active pills, persistent "+ New Order" CTA, role badges, mobile drawer.
   - Theme switching (Light, Dark, Kitchen Mode) and CSS variables.
   - Brand tokens (Flower, Mastery, Niwa, Tobiko) and Platform tokens (Talabat, elmenus, InstaShop, HarryApp, Phone).
   - POS receipt preview panel with `tabular-nums`, touch payment selectors (Cash/Visa/Online), loyalty tier calculation.
   - Orders board View Switcher (Table vs Kanban), filter persistence, 5 kanban columns, state machine adherence, and prep timer >15m pulsing alert badges.
   - Arabic RTL / English LTR layout fidelity and >=44x44px touch targets.
4. When the test infrastructure and test cases are ready, publish `d:\programming\Projects\Orders Management System\TEST_READY.md`.
5. Write your handoff report to `d:\programming\Projects\Orders Management System\.agents\test_writer_e2e\handoff.md`.

When complete, send a message to parent summarizing the test suite coverage and linking to `TEST_READY.md`.
