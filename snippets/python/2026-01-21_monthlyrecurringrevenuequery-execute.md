# MonthlyRecurringRevenueQuery.execute

**Repository:** CertGames-Core
**File:** backend/api/admin/domains/revenue/queries.py
**Language:** python
**Lines:** 294-381
**Complexity:** 13.0

---

## Source Code

```python
def execute(self) -> dict[str, Any]:
        """
        Calculate comprehensive MRR metrics
        """
        active_users = User.objects(
            subscriptionActive = True,
            subscriptionEndDate__gte = self.as_of_date
        )

        current_mrr = 0.0
        platform_breakdown = {'stripe': 0.0, 'apple': 0.0, 'promo': 0.0}

        for user in active_users:
            metrics = SubscriptionMetrics.objects(userId = user.id).first()
            if metrics and metrics.monthlyRevenue:
                monthly_rev = metrics.monthlyRevenue
            else:
                monthly_rev = 9.99 if user.subscriptionPlatform in (
                    'stripe',
                    'apple'
                ) else 0.0

            current_mrr += monthly_rev
            if user.subscriptionPlatform:
                platform_breakdown[user.subscriptionPlatform] = \
                    platform_breakdown.get(user.subscriptionPlatform, 0.0) + monthly_rev

        new_subs = SubscriptionEvent.objects(
            event = 'subscription_created',
            timestamp__gte = self.month_start,
            timestamp__lt = self.as_of_date
        )

        new_mrr = 0.0
        for event in new_subs:
            metrics = SubscriptionMetrics.objects(userId = event.userId
                                                  ).first()
            if metrics and metrics.monthlyRevenue:
                new_mrr += metrics.monthlyRevenue

        churned_subs = SubscriptionEvent.objects(
            event__in = ['subscription_cancelled',
                         'subscription_expired'],
            timestamp__gte = self.month_start,
            timestamp__lt = self.as_of_date
        )

        churned_mrr = 0.0
        for event in churned_subs:
            metrics = SubscriptionMetrics.objects(userId = event.userId
                                                  ).first()
            if metrics and metrics.monthlyRevenue:
                churned_mrr += metrics.monthlyRevenue

        prev_month_mrr = current_mrr - new_mrr + churned_mrr

        net_mrr_change = new_mrr - churned_mrr
        growth_rate = (
            (current_mrr - prev_month_mrr) / prev_month_mrr * 100
        ) if prev_month_mrr > 0 else 0

        arpu = current_mrr / active_users.count() if active_users.count(
        ) > 0 else 0

        return {
            'current_mrr': round(current_mrr,
                                 2),
            'new_mrr': round(new_mrr,
                             2),
            'churned_mrr': round(churned_mrr,
                                 2),
            'net_mrr_change': round(net_mrr_change,
                                    2),
            'previous_mrr': round(prev_month_mrr,
                                  2),
            'growth_rate': round(growth_rate,
                                 2),
            'active_subscriptions': active_users.count(),
            'arpu': round(arpu,
                          2),
            'platform_breakdown': {
                platform: round(amount,
                                2)
                for platform, amount in platform_breakdown.items()
            },
            'as_of_date': self.as_of_date.isoformat(),
            'month': self.month_start.strftime('%Y-%m')
        }
```

---

## Documentation

### Documentation for `MonthlyRecurringRevenueQuery.execute`

**Purpose and Behavior:**
This function calculates comprehensive MRR (Monthly Recurring Revenue) metrics, including current MRR, new MRR, churned MRR, net MRR change, growth rate, ARPU (Average Revenue Per User), and platform-specific breakdowns. It processes user subscriptions from a database, categorizing them by subscription platforms like Stripe, Apple, or promo.

**Key Implementation Details:**
- **Query Filtering:** Uses Django ORM to filter active users based on their subscription status and end date.
- **Metrics Calculation:** Iterates over filtered users and events to calculate MRR components. It handles different revenue amounts based on the user's platform.
- **Conditional Logic:** Utilizes conditional statements to determine monthly revenue for each user, considering default values if metrics are missing.

**When/Why to Use:**
Use this function when you need detailed MRR analytics for a specific date range. It is particularly useful for financial reporting and business intelligence purposes in subscription-based services.

**Patterns and Gotchas:**
- **Performance:** The function performs multiple database queries, which can be inefficient if not optimized.
- **Default Values:** Default revenue values (e.g., 9.99) are hardcoded; consider using a configuration or settings file for flexibility.
- **Type Hints:** While the return type is annotated as `dict[str, Any]`, this could benefit from more specific types to improve code readability and maintainability.

This function provides a robust framework for calculating MRR metrics but may require adjustments based on specific business needs.

---

*Generated by CodeWorm on 2026-01-21 01:38*
