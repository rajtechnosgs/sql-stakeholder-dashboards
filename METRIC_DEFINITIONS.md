
# Metric Definitions

| Metric | SQL Fragment / Filter Logic | Question Reference |
|---|---|---|
| Total Revenue | `SUM(orders.total) WHERE orders.payment_status = 'paid'` — cancelled orders excluded (they never reach `payment_status = 'paid'`). | leadership_kpi_revenue_daily |
| Orders | `COUNT(orders.order_id) WHERE orders.payment_status = 'paid'` | leadership_kpi_orders_daily |
| AOV | `SUM(orders.total) / COUNT(orders.order_id) WHERE orders.payment_status = 'paid'` | leadership_kpi_aov_daily |
| Conversion Rate | `COUNT(DISTINCT orders.session_id) FILTER (WHERE orders.payment_status = 'paid') / COUNT(DISTINCT sessions.session_id)`. **Denominator = all sessions in the period** (not just sessions that reached checkout) — this is total-funnel conversion, not checkout-only conversion. Both sides filtered by the same date range. | leadership_kpi_conversion_rate_daily |
| Refund Rate | `COUNT(refunds.refund_id) / COUNT(orders.order_id) WHERE orders.payment_status = 'paid'`. **Period asymmetry:** the refund is counted in the period `refunds.created_at` falls in, while the order in the denominator is counted in the period `orders.created_at` falls in — these can be different periods for the same transaction (e.g. an order placed May 28, refunded June 3, contributes to June's refund count against June's order count, not May's). | leadership_kpi_refund_rate_daily |
| Revenue by Country | `SUM(orders.total) WHERE orders.payment_status = 'paid' GROUP BY COALESCE(NULLIF(TRIM(customers.country), ''), 'Unknown')` | leadership_breakdown_revenue_by_country |
| Revenue by Channel | `SUM(orders.total) WHERE orders.payment_status = 'paid' GROUP BY COALESCE(session_channels.channel, 'direct')` | leadership_breakdown_revenue_by_channel |
| DAU / WAU / MAU | `COUNT(DISTINCT sessions.customer_id) WHERE sessions.started_at::date = max_date` (DAU) / `WHERE sessions.started_at::date > max_date - INTERVAL '7 days'` (WAU) / `... - INTERVAL '30 days'` (MAU). `max_date` = `MAX(sessions.started_at)::date` in the dataset, not calendar "today". | product_kpi_dau_wau_mau |
| Stickiness Ratio | `DAU / NULLIF(MAU, 0)` | product_kpi_dau_mau_stickiness |
| Activation Rate | `COUNT(*) FILTER (WHERE first_meaningful_action_at BETWEEN signup_at AND signup_at + INTERVAL '7 days') / COUNT(*)`, where `first_meaningful_action_at = MIN(session_events.occurred_at) FILTER (WHERE event_type IN ('add_to_cart','begin_checkout','purchase'))`. Scoped to `customers.created_at >= '2026-04-19'`. Reused from Task 2 E1. | product_kpi_activation_rate |
| Product Funnel Drop-off % | `MAX(CASE WHEN event_type='purchase' THEN 4 WHEN event_type='begin_checkout' THEN 3 WHEN event_type='add_to_cart' THEN 2 WHEN event_type='product_view' THEN 1 ELSE 0 END)` per session, then `(COUNT(*) FILTER (WHERE max_step >= N) - COUNT(*) FILTER (WHERE max_step >= N+1)) / COUNT(*) FILTER (WHERE max_step >= N)` per step. Scoped to `sessions.started_at >= '2026-04-19'`. | product_trend_funnel_weekly |
| Cohort Retention (W1/W2/W4/W8) | `week_index = FLOOR(EXTRACT(EPOCH FROM (session_events.occurred_at - customers.created_at)) / (86400*7))`, clamped to `GREATEST(week_index, 0)`. `COUNT(DISTINCT customer_id) FILTER (WHERE week_index = N) / COUNT(DISTINCT customer_id) [cohort_size]`. Reused from Task 2 E3. | product_trend_cohort_retention |
| Feature Usage % | `COUNT(DISTINCT loyalty_transactions.customer_id) / COUNT(DISTINCT WAU customer_id) GROUP BY loyalty_transactions.reason`. **Proxy metric** — no `features` table exists in `ecom`; `reason` (earned_order/redeemed_reward/bonus_*) used as an engagement-feature stand-in. | product_breakdown_feature_usage_matrix |
| Cart Abandonment Rate | `(atc_sessions - purchased_sessions) / atc_sessions`, where `atc_sessions = COUNT(DISTINCT session_id) FROM session_events WHERE event_type='add_to_cart'`, bucketed by `SUM(quantity*unit_price)` per session. Reused from Task 2 E5. | product_breakdown_cart_abandonment |
| Review Rating Trend | `AVG(product_reviews.rating) GROUP BY DATE_TRUNC('month', product_reviews.created_at), categories.category_name`. Known skew: ~50% of all ratings are 5-star — trend direction is the signal, not the absolute value. | product_breakdown_review_rating_trend |
| Power User % | `COUNT(DISTINCT customer_id) FILTER (WHERE orders_this_month >= 3) / COUNT(DISTINCT customer_id) WHERE orders.payment_status = 'paid' GROUP BY DATE_TRUNC('month', orders.created_at)` | product_breakdown_power_user_pct |

## Cross-Dashboard Consistency Audit

**Method:** May 2026, no other filters, checked the "paid orders" definition across both dashboards.

**Test:**
```sql
-- Leadership-style
SELECT COUNT(*) FROM ecom.orders
WHERE payment_status = 'paid' AND DATE_TRUNC('month', created_at) = '2026-05-01';
-- = 9,716

-- Product-style (sum of per-customer monthly counts from Power User % query)
SELECT SUM(orders_this_month) FROM (
    SELECT DATE_TRUNC('month', created_at) AS month, customer_id, COUNT(*) AS orders_this_month
    FROM ecom.orders WHERE payment_status = 'paid' GROUP BY 1, 2
) x WHERE month = '2026-05-01';
-- = 9,716
```

**Result:** Exact match (9,716 = 9,716). Both dashboards apply the identical `payment_status = 'paid'` filter with no drift.
