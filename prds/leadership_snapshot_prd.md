
# Leadership Snapshot PRD

Dashboard: Leadership Snapshot
User: The Founder/CEO
Refresh cadence: Daily
Primary device: Phone
Database: ecom

Job-to-be-done (1 sentence):
"When the founder starts their day, they need to know within 3 minutes
whether yesterday was a good day or a bad one, so they can decide
whether to escalate anything before their first meeting."

Top 3 decisions enabled:

1. Is this week materially worse than last week? (Triggers a "what happened" conversation.)
2. Is there a red flag to escalate today? (Payment failures, refund spike, country collapse.)
3. Are we on track for the quarter?

Stakeholder interview (from Phase 1):
Q1 (decision): When you open this dashboard, what do you decide based on it, and how often?
A1: I check it every morning on my phone, before coffee, sometimes again before a
meeting where someone might ask "how are we doing." I decide three things:
is today materially worse than the same day last week (triggers a "what happened"
conversation), is there a red flag to escalate today (payment failures, refund
spike), and — checked once or twice a week, not daily — are we on track for the quarter.

Q2 (cost of error): Which metric is the most expensive to be wrong on?
A2: Refund rate. If it's miscalculated and actually spiking while the dashboard shows
normal, I could miss a customer-trust crisis until it's already too late. A revenue
number being briefly off is recoverable; missing a refund spike means we catch a
product-quality or fraud issue days late, after customers are already frustrated.

Q3 (context): Who else looks at this, and what would they want to see that's different?
A3: My Head of Growth also checks it, but she wants channel-level breakdowns — I don't
need that much detail, just the overall number. I also screenshot it for investor
updates sometimes — for them it needs to look simple and trustworthy, no jargon.

Metrics on this dashboard:
Top row (5-second read, 5 numbers):
- Revenue (today + WoW%)
- Orders (today + WoW%)
- AOV (today + WoW%)
- Conversion Rate (today + WoW%)
- Refund Rate (last 7 days + WoW%) — kept at the top per stakeholder interview (A2):
a refund spike is the single most expensive thing to miss

Middle (trends):
- Revenue trend: 90-day daily line with 7-day prior-period overlay (dotted)
- Orders trend: 90-day daily line

Bottom (breakdowns):
- Revenue by country: top 5 with WoW delta
- Revenue by acquisition channel: top 5 with WoW delta
- Diagnostic table: last 14 days day-by-day — Revenue, Orders, AOV, CR, Refund Rate

What's NOT in this dashboard, and why:

- Channel-level funnel breakdown — excluded because the founder explicitly said (A3)
they want the overall number, not channel detail; that's the Growth dashboard's job.
- Feature adoption / product engagement metrics — excluded because this is a business-
health view, not a product-behavior view; conflating the two would slow the 5-second read.
- Payment-method-level failure breakdown — excluded because it's operationally actionable
detail (Ops' job), not a founder-level decision; the founder only needs to know a
refund/payment red flag exists, not which specific method is failing.

Known caveats / limitations:

- Revenue keys off payment_status = 'paid'; orders.status has case variants (shipped/
SHIPPED/Shipped) so any fulfillment-state chart must use LOWER(status), not payment_status.
- customers.country has NULL, '', and 'N/A' variants — standardized via
COALESCE(NULLIF(TRIM(country), ''), 'Unknown') before the country breakdown.
- Most recent day may look artificially low if the data snapshot cuts off mid-day.
