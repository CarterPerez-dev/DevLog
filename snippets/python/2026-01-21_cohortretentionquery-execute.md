# CohortRetentionQuery.execute

**Repository:** CertGames-Core
**File:** backend/api/admin/domains/revenue/queries.py
**Language:** python
**Lines:** 407-504
**Complexity:** 13.0

---

## Source Code

```python
def execute(self) -> list[dict[str, Any]]:
        """
        Calculate cohort retention rates
        """
        pipeline = [
            {
                '$match': {
                    'signupCohort': {
                        '$gte': self.start_date,
                        '$lte': self.end_date
                    }
                }
            },
            {
                '$group': {
                    '_id': '$cohortMonth',
                    'user_ids': {
                        '$push': '$userId'
                    },
                    'cohort_size': {
                        '$sum': 1
                    }
                }
            },
            {
                '$match': {
                    'cohort_size': {
                        '$gte': self.min_cohort_size
                    }
                }
            },
            {
                '$sort': {
                    '_id': 1
                }
            }
        ]

        cohorts = list(SubscriptionMetrics.objects.aggregate(pipeline))

        retention_data = []

        for cohort in cohorts:
            cohort_month = cohort['_id']
            cohort_date = datetime.strptime(cohort_month,
                                            '%Y-%m').replace(tzinfo = UTC)
            user_ids = cohort['user_ids']
            cohort_size = cohort['cohort_size']

            retention_months = []
            current_date = cohort_date
            month_number = 0

            while current_date <= self.end_date and month_number <= 12:
                if month_number == 0:
                    retained_count = cohort_size
                else:
                    month_end = (current_date + timedelta(days = 32)
                                 ).replace(day = 1) - timedelta(days = 1)

                    churned_users = SubscriptionEvent.objects(
                        userId__in = [ObjectId(uid) for uid in user_ids],
                        event__in = [
                            'subscription_cancelled',
                            'subscription_expired'
                        ],
                        timestamp__lte = month_end
                    ).distinct('userId')

                    retained_count = cohort_size - len(churned_users)

                retention_rate = (
                    retained_count / cohort_size * 100
                ) if cohort_size > 0 else 0

                retention_months.append(
                    {
                        'month': month_number,
                        'retained_users': retained_count,
                        'retention_rate': round(retention_rate,
                                                2),
                        'churned_users': cohort_size - retained_count
                    }
                )

                current_date = (current_date +
                                timedelta(days = 32)).replace(day = 1)
                month_number += 1

            retention_data.append(
                {
                    'cohort': cohort_month,
                    'cohort_size': cohort_size,
                    'retention_curve': retention_months
                }
            )

        return retention_data
```

---

## Documentation

### Documentation for `execute` Method in `CohortRetentionQuery`

**Purpose and Behavior:**
The `execute` method calculates cohort retention rates by processing subscription data using MongoDB aggregation pipelines and Python logic. It matches users within a specified date range, groups them by month, filters cohorts based on size, sorts the results, and then iterates through each cohort to compute monthly retention rates.

**Key Implementation Details:**
- **Aggregation Pipeline:** The method uses an aggregation pipeline to match, group, and filter subscription data.
- **Retention Calculation:** For each cohort, it calculates the number of retained users over 12 months by checking for churn events (e.g., cancellations or expirations).
- **Date Handling:** Uses `datetime` and `timedelta` to manage date calculations accurately.

**When/Why to Use:**
This method is ideal for analyzing user retention trends within specific signup cohorts. It helps in understanding how users are retained over time, which is crucial for product and marketing strategy decisions.

**Patterns/Gotchas:**
- **Date Parsing:** Ensure the `cohort_month` format matches the expected date string.
- **Churn Event Filtering:** The method assumes certain event types (`subscription_cancelled`, `subscription_expired`) to identify churned users. Customize these if your application uses different events.
- **Performance Considerations:** Large datasets may impact performance due to multiple database queries and date calculations.

This code is particularly useful for backend analytics in subscription-based services where cohort analysis is essential.

---

*Generated by CodeWorm on 2026-01-21 02:25*
