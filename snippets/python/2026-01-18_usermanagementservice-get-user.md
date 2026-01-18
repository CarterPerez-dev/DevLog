# UserManagementService.get_user_analytics_summary

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
            "engagement_metrics": {
                "login_frequency":
                login_frequency,
                "days_active":
                user.achievement_counters.get("daysActive",
                                              0),
                "last_active":
                user.lastSeenAt.isoformat() if user.lastSeenAt else None,
            },
            "achievement_progress": {
                "unlocked":
                len(user.achievements_v2) if user.achievements_v2 else 0,
                "total":
                Achievement.objects.count(),
                "percentage":
                round(
                    (
                        len(user.achievements_v2) /
                        Achievement.objects.count() * 100
                    ) if user.achievements_v2 else 0,
                    2
                ),
            },
            "test_performance": {
                "tests_taken": test_summary.get("total_tests",
                                                0),
                "average_score":
                round(test_summary.get("avg_score",
                                       0),
                      2),
                "perfect_tests": test_summary.get("perfect_tests",
                                                  0),
                "total_study_time": test_summary.get("total_time",
                                                     0),
            },
        }

        if user_mgmt and user_mgmt.analytics:
            response["analytics"] = {
                "totalTestsCompleted":
                user_mgmt.analytics.totalTestsCompleted,
                "perfectTestsCount":
                user_mgmt.analytics.perfectTestsCount,
                "totalQuestionsAnswered":
                user_mgmt.analytics.totalQuestionsAnswered,
                "achievementCount":
                user_mgmt.analytics.achievementCount,
                "shopItemCount":
                user_mgmt.analytics.shopItemCount,
                "lastActiveAt":
                user_mgmt.analytics.lastActiveAt.isoformat()
                if user_mgmt.analytics.lastActiveAt else None,
                "engagementScore":
                user_mgmt.analytics.engagementScore,
                "lifetimeValue":
                user_mgmt.analytics.lifetimeValue,
                "avgTestScore":
                user_mgmt.analytics.avgTestScore,
                "favoriteCategory":
                user_mgmt.analytics.favoriteCategory,
                "daysActive":
                user_mgmt.analytics.daysActive,
                "lastUpdated":
                user_mgmt.analytics.lastUpdated.isoformat()
                if user_mgmt.analytics.lastUpdated else None,
            }
            response["engagement_score"
                     ] = user_mgmt.analytics.engagementScore

        return response
```

---

## Documentation

### Documentation for `get_user_analytics_summary`

**Purpose and Behavior**
The function `get_user_analytics_summary` retrieves a comprehensive analytics summary for a given user, including engagement metrics, test performance, and additional data from the `UserManagement` object. It processes various aspects such as login frequency, lifetime value, and achievement progress.

**Key Implementation Details**
- **Querying Data**: The function queries the database to fetch user details, test attempts, and analytics data.
- **Aggregation**: Aggregates test attempt data to calculate total tests taken, average score, perfect tests, and study time.
- **Engagement Metrics**: Computes login frequency based on days active and achievement counters.
- **Response Construction**: Constructs a detailed response dictionary containing various metrics.

**When/Why to Use This Code**
Use this function when you need a comprehensive analytics report for a user. It is particularly useful in administrative or analytical contexts where detailed insights into user behavior are required.

**Patterns and Gotchas**
- The function uses MongoDB aggregation pipelines effectively but requires careful handling of timezone conversions.
- Ensure that the `User` and `TestAttempt` models correctly define fields like `score`, `examTimerSeconds`, etc., to avoid runtime errors.
- The logic for calculating login frequency can be complex; consider testing with different user activity scenarios.

---

*Generated by CodeWorm on 2026-01-18 13:22*
