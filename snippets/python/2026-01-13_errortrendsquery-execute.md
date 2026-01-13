# ErrorTrendsQuery.execute

**Repository:** CertGames-Core
**File:** backend/api/admin/domains/analytics/queries.py
**Language:** python
**Lines:** 1198-1302
**Complexity:** 10.0

---

## Source Code

```python
def execute(self) -> dict:
        """
        Get error trends with breakdowns by severity and category.
        """
        if self.interval == 'hourly':
            date_format = "%Y-%m-%d %H:00"
        elif self.interval == 'weekly':
            date_format = "%Y-W%V"
        else:
            date_format = "%Y-%m-%d"

        pipeline = [
            {
                "$match": {
                    "timestamp": {
                        "$gte": self.start_date
                    }
                }
            },
            {
                "$group": {
                    "_id": {
                        "period": {
                            "$dateToString": {
                                "format": date_format,
                                "date": "$timestamp"
                            }
                        },
                        "severity": "$severity",
                        "category": "$category"
                    },
                    "count": {
                        "$sum": 1
                    },
                    "unique_users": {
                        "$addToSet": "$user_id"
                    },
                    "unique_errors": {
                        "$addToSet": "$error_type"
                    }
                }
            },
            {
                "$project": {
                    "_id": 0,
                    "period": "$_id.period",
                    "severity": "$_id.severity",
                    "category": "$_id.category",
                    "count": 1,
                    "unique_users": {
                        "$size": "$unique_users"
                    },
                    "unique_error_types": {
                        "$size": "$unique_errors"
                    }
                }
            },
            {
                "$sort": {
                    "period": 1,
                    "severity": 1
                }
            }
        ]

        results = list(ErrorLog.objects.aggregate(pipeline))

        trends: dict[str, dict[str, Any]] = {}
        for item in results:
            period = item['period']
            if period not in trends:
                trends[period] = {
                    'total': 0,
                    'by_severity': {},
                    'by_category': {},
                    'unique_users': 0,
                    'unique_error_types': 0
                }

            trends[period]['total'] += item['count']

            severity = item['severity']
            if severity not in trends[period]['by_severity']:
                trends[period]['by_severity'][severity] = 0
            trends[period]['by_severity'][severity] += item['count']

            category = item['category']
            if category not in trends[period]['by_category']:
                trends[period]['by_category'][category] = 0
            trends[period]['by_category'][category] += item['count']

            trends[period]['unique_users'] = max(
                trends[period]['unique_users'],
                item['unique_users']
            )
            trends[period]['unique_error_types'] = max(
                trends[period]['unique_error_types'],
                item['unique_error_types']
            )

        return {
            'interval': self.interval,
            'start_date': self.start_date.isoformat(),
            'trends': trends
        }
```

---

## Documentation

### Documentation for `execute` Method in `ErrorTrendsQuery`

**Purpose and Behavior:**
The `execute` method retrieves error trends from a database collection named `ErrorLog`. It aggregates errors by date, severity, and category, calculating counts, unique users, and unique error types. The results are formatted into a dictionary that includes total counts and breakdowns by severity and category.

**Key Implementation Details:**
- **Aggregation Pipeline:** Uses MongoDB aggregation framework to process the data.
- **Date Formatting:** Dynamically sets date format based on `interval` parameter ('hourly', 'weekly', or default 'daily').
- **Result Processing:** Aggregates results into a structured dictionary, updating counts and unique values as needed.

**When/Why to Use:**
Use this method when you need to analyze error trends over specific time intervals (hourly, weekly) for detailed insights. It’s particularly useful in monitoring system health or identifying patterns in error occurrences.

**Patterns & Gotchas:**
- **Date Handling:** Ensure `interval` is correctly set; incorrect values will lead to wrong date formats.
- **Aggregation Pipeline Complexity:** The pipeline can be complex and may require adjustments based on specific use cases. 
- **Resource Management:** Be mindful of the size of the dataset, as large datasets could impact performance.

This method provides a robust way to analyze error trends with detailed breakdowns, making it invaluable for analytics and debugging purposes.

---

*Generated by CodeWorm on 2026-01-13 15:01*
