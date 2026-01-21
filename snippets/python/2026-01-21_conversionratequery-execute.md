# ConversionRateQuery.execute

**Repository:** CertGames-Core
**File:** backend/api/admin/domains/revenue/queries.py
**Language:** python
**Lines:** 629-727
**Complexity:** 13.0

---

## Source Code

```python
def execute(self) -> dict[str, Any]:
        """
        Calculate conversion rates
        """
        total_users = User.objects(
            createdAt__gte = self.start_date,
            createdAt__lte = self.end_date
        ).count()

        converted_users = SubscriptionMetrics.objects(
            signupCohort__gte = self.start_date,
            signupCohort__lte = self.end_date,
            conversionDate__exists = True
        )

        if self.dimension == 'platform':
            breakdown = {}
            for platform in ['web', 'ios', 'android']:
                platform_total = User.objects(
                    createdAt__gte = self.start_date,
                    createdAt__lte = self.end_date,
                    signUpSource = platform
                ).count()

                platform_converted = converted_users.filter(
                    originalSignupSource = platform
                ).count()

                if platform_total > 0:
                    breakdown[platform] = {
                        'total_users':
                        platform_total,
                        'converted_users':
                        platform_converted,
                        'conversion_rate':
                        round(
                            platform_converted / platform_total * 100,
                            2
                        )
                    }

        elif self.dimension == 'days_since_signup':
            breakdown = {
                '0-7_days': {
                    'converted': 0,
                    'rate': 0
                },
                '8-30_days': {
                    'converted': 0,
                    'rate': 0
                },
                '31-90_days': {
                    'converted': 0,
                    'rate': 0
                },
                '90+_days': {
                    'converted': 0,
                    'rate': 0
                }
            }

            for metric in converted_users:
                if metric.daysToConvert is not None:
                    if metric.daysToConvert <= 7:
                        breakdown['0-7_days']['converted'] += 1
                    elif metric.daysToConvert <= 30:
                        breakdown['8-30_days']['converted'] += 1
                    elif metric.daysToConvert <= 90:
                        breakdown['31-90_days']['converted'] += 1
                    else:
                        breakdown['90+_days']['converted'] += 1

            total_converted = converted_users.count()
            for _key, value in breakdown.items():
                if total_converted > 0:
                    value['rate'] = round(
                        value['converted'] / total_converted * 100,
                        2
                    )

        else:
            breakdown = {}

        overall_rate = (
            converted_users.count() / total_users * 100
        ) if total_users > 0 else 0

        return {
            'period': {
                'start': self.start_date.isoformat(),
                'end': self.end_date.isoformat()
            },
            'total_users': total_users,
            'converted_users': converted_users.count(),
            'overall_conversion_rate': round(overall_rate,
                                             2),
            'breakdown': breakdown,
            'dimension': self.dimension
        }
```

---

## Documentation

### Documentation for `execute` Method in `ConversionRateQuery`

**Purpose and Behavior:**
The `execute` method calculates conversion rates based on user data from a specified date range, segmented by either platform (`'web', 'ios', 'android'`) or days since signup. It returns a detailed breakdown of total users, converted users, overall conversion rate, and per-dimension metrics.

**Key Implementation Details:**
- The method uses Django ORM to query `User` and `SubscriptionMetrics` objects based on date ranges.
- Breakdown is calculated differently for each dimension:
  - **Platform:** Segments by platform source.
  - **Days since signup:** Bins users into 4-day categories (0-7, 8-30, 31-90, >90 days).
- Overall conversion rate is computed as a percentage of converted to total users.

**When/Why to Use:**
Use this method when you need to analyze user conversion rates over specific time periods and dimensions. It's particularly useful for understanding which platforms or signup cohorts are most effective in driving conversions.

**Patterns & Gotchas:**
- The method handles edge cases, such as zero users in a segment, by setting the rate to 0.
- Be cautious with large datasets; filtering and counting operations can be resource-intensive.
- Ensure that `User` and `SubscriptionMetrics` models are correctly defined to avoid runtime errors.

---

*Generated by CodeWorm on 2026-01-21 11:14*
