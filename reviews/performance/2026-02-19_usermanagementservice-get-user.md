# UserManagementService.get_user_analytics_summary

**Type:** Performance Analysis
**Repository:** CertGames-Core
**File:** backend/api/admin/domains/users/services.py
**Language:** python
**Lines:** 379-531
**Complexity:** 21.0

---

## Source Code

```python
def get_user_analytics_summary(user_id: str) -> dict[str, Any]:
        """
        Get user analytics summary
        """
        user = User.objects(id = user_id).first()
        if not user:
            raise NotFoundError(f"User not found: {user_id}")

        user_mgmt = UserManagement.objects(userId = user.id).first()

        test_stats = TestAttempt.objects(
            userId = user.id,
            finished = True
        ).aggregate(
            [
                {
                    "$group": {
                        "_id": None,
                        "total_tests": {
                            "$sum": 1
                        },
                        "avg_score": {
                            "$avg": "$score"
                        },
                        "perfect_tests": {
                            "$sum": {
                                "$cond": [{
                                    "$eq": ["$score",
                                            100]
                                },
                                          1,
                                          0]
                            }
                        },
                        "total_time": {
                            "$sum": "$examTimerSeconds"
                        },
                    }
                }
            ]
        )

        test_stats_list = list(test_stats) if test_stats else []
        test_summary = test_stats_list[0] if test_stats_list else {
            "total_tests": 0,
            "avg_score": 0,
            "perfect_tests": 0,
            "total_time": 0
        }

        lifetime_value = 0.0
        if user.subscriptionActive and user.subscriptionStartDate:
            subscription_start = user.subscriptionStartDate.replace(
                tzinfo = UTC
            ) if user.subscriptionStartDate.tzinfo is None else user.subscriptionStartDate
            months_subscribed = (
                datetime.now(UTC) - subscription_start
            ).days / 30
            lifetime_value = months_subscribed * 29.99

        login_frequency = "unknown"
        if user.lastLoginAt and user.createdAt:
            created_at = user.createdAt.replace(
                tzinfo = UTC
            ) if user.createdAt.tzinfo is None else user.createdAt
            days_active = (datetime.now(UTC) - created_at).days
            if days_active > 0:
                login_rate = user.achievement_counters.get(
                    "daysActive",
                    0
                ) / days_active
                if login_rate > 0.7:
                    login_frequency = "daily"
                elif login_rate > 0.3:
                    login_frequency = "regular"
                elif login_rate > 0.1:
                    login_frequency = "occasional"
                else:
                    login_frequency = "rare"

        response = {
            "lifetime_value": round(lifetime_value,
                                    2),
      
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The primary bottleneck is the MongoDB aggregation query, which has a time complexity of \(O(n)\), where \(n\) is the number of test attempts for the user. Additionally, calculating `login_frequency` involves iterating over achievements and performing date calculations.

#### Space Complexity
Space complexity is \(O(1)\) for the dictionary operations, but the MongoDB aggregation query can consume significant memory if there are many test attempts.

#### Bottlenecks or Inefficiencies
1. **MongoDB Aggregation Query**: The aggregation operation could be optimized by using more efficient queries.
2. **Date Calculations**: Repeated date manipulations in `login_frequency` calculations can be costly.
3. **Redundant Operations**: Checking if `user_mgmt` and `user_mgmt.analytics` are not `None` before accessing their attributes is redundant.

#### Optimization Opportunities
1. **Optimize Aggregation Query**: Use `$match` to filter test attempts first, then perform aggregation operations.
2. **Cache Date Calculations**: Store the result of date calculations in variables to avoid repeated computations.
3. **Simplify Conditional Checks**: Use `if-else` statements directly instead of redundant checks.

#### Resource Usage Concerns
1. **Unclosed Connections**: Ensure that database connections are properly closed using context managers or connection pools.
2. **Memory Leaks**: Be cautious with large data sets; consider using `.limit()` to avoid loading too much data into memory.

### Suggested Optimizations

```python
def get_user_analytics_summary(user_id: str) -> dict[str, Any]:
    user = User.objects(id=user_id).first()
    if not user:
        raise NotFoundError(f"User not found: {user_id}")

    test_stats = TestAttempt.objects(
        userId=user.id,
        finished=True
    ).only('score', 'examTimerSeconds')

    # Optimize aggregation query
    test_summary = (
        test_stats.aggregate(
            {
                "$group": {
                    "_id": None,
                    "total_tests": {"$sum": 1},
                    "avg_score": {"$avg": "$score"},
                    "perfect_tests": {
                        "$sum": {
                            "$cond": [{"$eq": ["$score", 100]}, 1, 0]
                        }
                    },
                    "total_time": {"$sum": "$examTimerSeconds"}
                }
            }
        )
    )

    # Simplify date calculations
    subscription_start = user.subscriptionStartDate.astimezone(UTC)
    months_subscribed = (datetime.now(UTC) - subscription_start).days / 30

    lifetime_value = (
        months_subscribed * 29.99 if user.subscriptionActive and user.subscriptionStartDate else 0.0
    )

    # Simplify conditional checks
    login_frequency = "unknown"
    if user.lastLoginAt and user.createdAt:
        created_at = user.createdAt.astimezone(UTC)
        days_active = (datetime.now(UTC) - created_at).days

        if days_active > 0:
            login_rate = user.achievement_counters.get("daysActive", 0) / days_active
            if login_rate > 0.7:
                login_frequency = "daily"
            elif login_rate > 0.3:
                login_frequency = "regular"
            elif login_rate > 0.1:
                login_frequency = "occasional"
            else:
                login_frequency = "rare"

    response = {
        # ... (existing code)
    }

    return response
```

These changes should improve the performance and readability of your function.

---

*Generated by CodeWorm on 2026-02-19 19:01*
