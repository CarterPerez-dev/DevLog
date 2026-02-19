# UserManagementService.get_user_details

**Type:** Security Review
**Repository:** CertGames-Core
**File:** backend/api/admin/domains/users/services.py
**Language:** python
**Lines:** 217-376
**Complexity:** 27.0

---

## Source Code

```python
def get_user_details(user_id: str) -> dict[str, Any]:
        """
        Get comprehensive user information
        """
        user = User.objects(id = user_id).first()
        if not user:
            raise NotFoundError(f"User not found: {user_id}")

        user_mgmt = UserManagement.objects(userId = user.id).first()

        days_since_signup = (
            datetime.now(UTC) - (
                user.createdAt.replace(tzinfo = UTC) if user.createdAt
                and user.createdAt.tzinfo is None else user.createdAt
            )
        ).days if user.createdAt else 0
        test_attempts = TestAttempt.objects(
            userId = user.id,
            finished = True
        ).count()

        calculator = UserEngagementCalculator(
            user,
            test_attempts,
            days_since_signup
        )
        engagement_score, engagement_factors = calculator.calculate_engagement_score()

        achievement_details = []
        if user.achievements_v2:
            achievement_ids = [
                a.achievementId for a in user.achievements_v2
            ]
            achievements = Achievement.objects(
                achievementId__in = achievement_ids
            )
            achievement_map = {a.achievementId: a for a in achievements}

            for user_achievement in user.achievements_v2:
                if achievement := achievement_map.get(
                        user_achievement.achievementId):
                    achievement_details.append(
                        {
                            "achievement":
                            achievement.to_dict(),
                            "unlockedAt":
                            user_achievement.unlockedAt.isoformat()
                            if user_achievement.unlockedAt else None
                        }
                    )

        shop_items_details = []
        if user.purchasedItems:
            items = Shop.objects(id__in = user.purchasedItems)
            shop_items_details = [item.to_dict() for item in items]

        game_stats = UserManagementService._get_user_game_stats(user)

        subscription_info = UserManagementService._get_subscription_info(
            user
        )

        location_info = UserManagementService._get_location_info(
            user.currentIP or user.lastLoginIP
        )

        return {
            "basic_info": {
                "user_id":
                str(user.id),
                "username":
                user.username,
                "email":
                user.email,
                "created_at":
                user.createdAt.isoformat() if user.createdAt else None,
                "updated_at":
                user.updatedAt.isoformat() if user.updatedAt else None,
                "days_since_signup":
                days_since_signup,
            },
            "activity": {
                "is_online":
                user.isOnline,
                "last_login_at":
                user.lastLogi
```

---

## Security Review

### Security Review for `get_user_details` Function

#### Vulnerabilities Found:

1. **Information Disclosure** - **Severity: Low**
   - The function returns detailed user information, which could be sensitive data.
   - **Attack Vector:** An attacker with access to this endpoint could retrieve a user's profile and activity details.

2. **Hardcoded Secrets or Credentials** - **Severity: Info**
   - No hardcoded secrets or credentials are found in the provided code snippet.

3. **Input Validation Gaps** - **Severity: Low**
   - The function accepts `user_id` as a string, but no validation is performed to ensure it's a valid user ID.
   - **Attack Vector:** An attacker could provide an invalid or malicious input leading to unexpected behavior.

4. **Error Handling** - **Severity: Info**
   - Error handling is minimal; exceptions are raised without providing detailed error messages that could leak information.

#### Recommended Fixes:

1. **Input Validation:**
   ```python
   if not user_id.isnumeric():
       raise ValueError("Invalid user ID")
   ```

2. **Error Handling:**
   ```python
   try:
       # Function logic here
   except Exception as e:
       logger.error(f"An error occurred: {str(e)}")
       raise InternalServerError from e
   ```

3. **Sensitive Information Masking:**
   - Ensure that sensitive information is not logged or returned in error messages.

4. **Authorization Checks:**
   - Implement checks to ensure only authorized users can access this function.
   ```python
   @login_required
   def get_user_details(user_id: str) -> dict[str, Any]:
       # Function logic here
   ```

#### Overall Security Posture:

The overall security posture is moderate. The code does not contain severe vulnerabilities but has room for improvement in input validation and error handling. Implementing the recommended fixes will enhance the security of the function.

---

This review focuses on specific issues found within the provided code snippet, offering concrete steps to improve its security.

---

*Generated by CodeWorm on 2026-02-19 00:34*
