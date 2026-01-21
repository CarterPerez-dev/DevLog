# RevenueBySourceQuery.execute

**Repository:** CertGames-Core
**File:** backend/api/admin/domains/revenue/queries.py
**Language:** python
**Lines:** 172-265
**Complexity:** 13.0

---

## Source Code

```python
def execute(self) -> list[dict[str, Any]]:
        """
        Calculate revenue breakdown by source
        """
        pipeline = [
            {
                '$match': {
                    'conversionDate': {
                        '$gte': self.start_date,
                        '$lte': self.end_date
                    },
                    'monthlyRevenue': {
                        '$exists': True,
                        '$gt': 0
                    }
                }
            },
            {
                '$group': {
                    '_id': {
                        'source':
                        '$originalSignupSource',
                        'platform':
                        '$conversionPlatform'
                        if self.group_by_platform else None
                    },
                    'user_count': {
                        '$sum': 1
                    },
                    'total_monthly_revenue': {
                        '$sum': '$monthlyRevenue'
                    },
                    'total_lifetime_revenue': {
                        '$sum': '$totalRevenue'
                    },
                    'avg_monthly_revenue': {
                        '$avg': '$monthlyRevenue'
                    },
                    'avg_days_to_convert': {
                        '$avg': '$daysToConvert'
                    }
                }
            },
            {
                '$project': {
                    '_id': 0,
                    'signup_source': '$_id.source',
                    'conversion_platform': '$_id.platform',
                    'metrics': {
                        'user_count': '$user_count',
                        'mrr': {
                            '$round': ['$total_monthly_revenue',
                                       2]
                        },
                        'lifetime_revenue': {
                            '$round': ['$total_lifetime_revenue',
                                       2]
                        },
                        'arpu': {
                            '$round': ['$avg_monthly_revenue',
                                       2]
                        },
                        'avg_days_to_convert': {
                            '$round': ['$avg_days_to_convert',
                                       1]
                        }
                    }
                }
            },
            {
                '$sort': {
                    'metrics.mrr': -1
                }
            }
        ]

        results = list(SubscriptionMetrics.objects.aggregate(pipeline))

        total_mrr = sum(r['metrics']['mrr'] for r in results)
        total_users = sum(r['metrics']['user_count'] for r in results)

        for result in results:
            result['metrics']['mrr_percentage'] = round(
                (result['metrics']['mrr'] / total_mrr *
                 100) if total_mrr > 0 else 0,
                2
            )
            result['metrics']['user_percentage'] = round(
                (result['metrics']['user_count'] / total_users *
                 100) if total_users > 0 else 0,
                2
            )

        return results
```

---

## Documentation

### Documentation for `execute` Method

**Purpose and Behavior:**
The `execute` method calculates revenue breakdown by source over a specified date range, aggregating data from the `SubscriptionMetrics` collection. It groups metrics by signup source and optionally conversion platform, providing detailed statistics such as user count, monthly revenue, lifetime revenue, average monthly revenue, and average days to convert.

**Key Implementation Details:**
- Utilizes MongoDB aggregation pipeline for complex data processing.
- Aggregates multiple metrics including user count, MRR (Monthly Recurring Revenue), LTV (Lifetime Value), ARPU (Average Revenue Per User), and average days to conversion.
- Rounds off revenue values to two decimal places using `$round` operator.
- Sorts results by MRR in descending order.
- Calculates percentage contributions of each source's MRR and user count relative to the total.

**When/Why to Use:**
Use this method when you need a detailed breakdown of revenue sources over a specific period, helping identify which sources contribute most significantly. It is particularly useful for financial analysis or reporting purposes in subscription-based businesses.

**Patterns and Gotchas:**
- The use of MongoDB aggregation framework requires careful handling of pipeline stages.
- Ensure `SubscriptionMetrics` model exists with appropriate fields like `conversionDate`, `monthlyRevenue`, etc.
- Be cautious with large datasets as the aggregation process can be resource-intensive. Consider optimizing queries for performance, especially when dealing with extensive data volumes.

---

*Generated by CodeWorm on 2026-01-21 00:45*
