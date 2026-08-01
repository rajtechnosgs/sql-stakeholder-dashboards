
# Anomaly Investigations

## Memo 1: June 14, 2026 Revenue Drop

**Observation:** On the Leadership Snapshot, June 14, 2026 shows revenue of ₹4.21L, a 72.96% drop from the prior day's ₹15.58L. This is the single largest single-day swing visible anywhere in the 90-day trend.

**Hypotheses:**

1. A genuine demand collapse (payment gateway outage, major site issue)
2. A data-pipeline cutoff — June 14 is a partial day, not a full 24 hours of data
3. A seasonal/weekday effect coinciding with a naturally low-revenue day

**Tests:** Ran `SELECT MAX(created_at) FROM ecom.orders` — confirmed June 14 is the **last date present in the entire dataset**. Checked order counts by hour on June 14 vs June 13 — June 14 has orders only through a partial window, not a full day. Checked payment failure rate on June 14 specifically — no spike (ruling out hypothesis 1).

**Conclusion:** This is a **data-cutoff artifact**, not a real business event. The dataset simply ends mid-day on June 14, so any daily aggregate for that date understates the true day's revenue. Hypothesis 2 confirmed; hypotheses 1 and 3 ruled out.

**So What:** No business action needed — this isn't a real revenue crash. Action needed on the *dashboard* itself: the Leadership Snapshot should either exclude the most recent partial day by default, or add a "data as of [last full hour]" indicator so the founder doesn't mistake this for a genuine drop on their morning check.

## Memo 2: Cohort Retention — Session Activity Not Anchored to Signup

**Observation:** On the Product Dashboard's cohort retention table, W1 retention rates hover around 25-36% for most cohorts — reasonable — but early diagnostics showed the underlying `w0_active` count (customers active during their own signup week) was only ~83% of `cohort_size`, when logically it should be closer to 100% by definition.

**Hypotheses:**

1. A broken join silently dropping customers (LEFT JOIN filter pushed into WHERE)
2. Some customers genuinely never engage at all in week 0
3. Session activity isn't strictly anchored to signup date — some sessions predate signup

**Tests:** Verified the join used `ON` correctly, not `WHERE` (ruling out hypothesis 1). Ran a diagnostic counting customers with *any* meaningful session ever (5,751 of 6,267, ~92%) versus those matching week 0 specifically — the gap didn't match "never engaged" (ruling out hypothesis 2 as the main cause). Plotted the raw distribution of session timing relative to signup and found a roughly symmetric spread from -8 to +8 weeks, confirming hypothesis 3: a meaningful share of sessions are dated *before* the customer's official signup timestamp, likely guest browsing prior to account creation.

**Conclusion:** This is a real, documented data-quality quirk — session activity in this dataset isn't strictly signup-anchored. Fixed by folding pre-signup activity into week 0 (`GREATEST(week_index, 0)`), which brought w0_active to ~5,233, closely matching the "ever active" population.

**So What:** No further action needed on this dashboard — the fix is applied and documented. Worth flagging to whoever owns the `ecom.sessions` instrumentation: pre-account-creation browsing sessions may need a separate "guest session" flag going forward, to avoid every analyst re-discovering this quirk independently.

## Final Reflection

**1. What I built and who each one is for:** Two audience-specific dashboards — a Leadership Snapshot (daily business-health view for the founder: revenue, orders, AOV, conversion, refund rate) and a Product Dashboard (behavioral view for the PM: activation, funnel, cohort retention, feature usage) — both built on `ecom`.

**2. Hardest design decision:** Deciding what to cut from the Product Dashboard. Acquisition channel breakdowns, payment method splits, and refund totals all *exist* in the data and are easy to add, but the stakeholder interview made clear the PM only cares what happens *after* users land — those metrics belong to Growth and Ops, not Product, so I left them out and defended the exclusion explicitly in the PRD.

**3. Mistake I made and corrected mid-build:** I initially wrote Field Filter conditions with explicit `table.column = {{variable}}` syntax, not realizing Field Filters generate their own condition — this silently broke every filter across five KPI queries until I caught it, fixed the syntax to bare `{{variable}}`, and removed table aliases so Metabase's generated condition could resolve correctly.

**4. What changed about the PRD after the stakeholder interview:** The Leadership Snapshot's Refund Rate KPI was almost left as a secondary metric, but the founder's answer to the cost-of-error question — that a missed refund spike is far more expensive than a revenue miscalculation — is why it's pinned at the top of the KPI row instead of buried in a breakdown.

**5. What I'd do differently with another week:** Build a dynamic "prior period" window for the breakdown charts (Revenue by Country/Channel) instead of the hardcoded 7-day comparison I used, so it correctly matches whatever date range the dashboard filter is set to, rather than always comparing to a fixed 7-day lookback.
