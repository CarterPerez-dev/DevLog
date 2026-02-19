# UserManagementService.get_user_analytics_summary

**Type:** Security Review
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

## Security Review

### Security Review for `get_user_analytics_summary`

#### Vulnerabilities Found:

1. **Information Disclosure** - **Severity: Info**
   - The function does not have any obvious injection vulnerabilities, but it could potentially leak information about the user's analytics and subscription status.

2. **Hardcoded Secrets or Credentials** - **Severity: Low**
   - No hardcoded secrets or credentials are found in the provided code snippet.

3. **Input Validation Gaps** - **Severity: Medium**
   - The `user_id` parameter is not validated for type or format, which could lead to unexpected behavior if an invalid ID is passed.

4. **Error Handling** - **Severity: Info**
   - Error handling is minimal and does not provide detailed error messages that could be exploited.

#### Attack Vectors:

- An attacker could pass a malicious `user_id` to potentially trigger unexpected behavior or access unauthorized data.
- Detailed error messages in production could inadvertently reveal sensitive information about the database schema or user analytics.

#### Recommended Fixes:

1. **Input Validation** - Validate and sanitize `user_id` before using it in queries:
   ```python
   if not isinstance(user_id, str) or len(user_id) < 1:
       raise ValueError("Invalid user ID")
   ```

2. **Enhance Error Handling** - Use try-except blocks to handle errors gracefully without revealing sensitive information:
   ```python
   try:
       user = User.objects(id=user_id).first()
       if not user:
           raise NotFoundError(f"User not found: {user_id}")
   except Exception as e:
       logger.error(f"An error occurred: {str(e)}")
       raise InternalServerError("Internal server error")
   ```

3. **Logging and Monitoring** - Implement logging to track any unusual activities or errors.

#### Overall Security Posture:

The current code has a moderate security posture with some potential areas for improvement, particularly in input validation and error handling. Addressing these issues will help mitigate risks associated with unexpected inputs and improve the overall robustness of the application.

---

*Generated by CodeWorm on 2026-02-19 01:35*
