
# Product Dashboard PRD

Dashboard: Product Dashboard
User: Rahul, Product Manager
Refresh cadence: Weekly (checked 2-3x/week around sprint planning)
Primary device: Desktop
Database: ecom

Job-to-be-done (1 sentence):
"When a step in the user journey degrades versus baseline, Rahul needs
to know which step, by how much, and which segment is most affected,
so the right fix lands in next sprint's planning."

Top 3 decisions enabled:

1. Which step in the user journey is bleeding the most users? Is it a recent regression?
2. Which feature/screen is underperforming and worth a redesign?
3. Are new users activating? Is cohort retention healthy or quietly eroding?

Stakeholder interview (from Phase 1):
Q1 (decision): When you open this dashboard, what do you decide based on it, and how often?
A1: I check it 2-3 times a week, mostly on desktop, around sprint planning. I decide
three things: which step in the user journey is losing the most users and whether
it's a recent regression (that becomes next sprint's priority), which feature/screen
is underperforming enough to redesign or sunset, and whether new users are
activating and whether weekly cohort retention is healthy or quietly eroding.

Q2 (cost of error): Which metric is the most expensive to be wrong on?
A2: Activation rate. If it's miscalculated — say, inflated when it's actually declining
— I'd set the wrong sprint priorities; the team would focus on something else while
activation actually crashes. Feature-usage numbers being slightly off matters less;
activation is a leading indicator, so getting it wrong sends the whole roadmap in
the wrong direction.

Q3 (context): Who else looks at this, and what would they want to see that's different?
A3: Engineering leads check it occasionally when there's a performance/bug-related drop
— they want more technical granularity (exact timestamp a drop started). Design
checks it when proposing a redesign — they want before/after comparisons around a
feature launch. I personally only need trend and magnitude, not that level of detail.

Metrics on this dashboard:
Top row (5-second read, 3 numbers):
- DAU / WAU / MAU
- DAU/MAU stickiness ratio
- Activation rate (first paid order within 7 days of signup)

Middle (trends):
- Product funnel (session_events): product_view → add_to_cart → begin_checkout →
purchase, per-step drop-off % with 4-week trend (scoped to sessions on/after
2026-04-19, when event instrumentation launched)
- Cohort retention table: signup-week cohorts, % returning at W1/W2/W4/W8

Bottom (breakdowns):
- Feature usage matrix: top 10 features by % of WAU
- Cart abandonment rate + time-in-cart distribution
- Review-rating trend by category (product_reviews) — flags product-quality regressions
- Power user %: share of MAU placing 3+ orders/month, trend

What's NOT in this dashboard, and why:

- Acquisition channel breakdown — excluded because that's Growth's job (A1); Rahul cares
what happens after users land, not where they came from.
- Payment method splits — excluded unless it signals checkout friction; otherwise it's
an Ops concern, not a product-behavior concern.
- Refund totals — excluded; only the return *reason* would matter here if it signals a
product issue, and that's a stretch addition, not core.
- Revenue-by-country — excluded because it isn't actionable for a PM; it's a Leadership/
Growth lens, not a product-journey lens.

Known caveats / limitations:

- product_reviews rating is skewed positive (50% 5-star, 30% 4-star) — raw average rating
is not used as a hero metric on its own; only the trend/direction is surfaced, per the
brief's explicit warning about this skew.
- session_events instrumentation launched 2026-04-19 — funnel and cohort queries exclude
or clearly flag sessions before that date as uninstrumented, not inactive.
- loyalty_accounts/loyalty_transactions used for power-user segmentation, not as a
standalone hero metric.
