# UserManagementService.get_user_details

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
                user.lastLoginAt.isoformat() if user.lastLoginAt else None,
                "last_seen_at":
                user.lastSeenAt.isoformat() if user.lastSeenAt else None,
                "current_ip":
                user.currentIP,
                "last_login_ip":
                user.lastLoginIP,
                "user_agent":
                user.userAgent,
            },
            "location": location_info,
            "profile": {
                "coins":
                user.coins,
                "xp":
                user.xp,
                "level":
                user.level,
                "current_avatar":
                str(user.currentAvatar) if user.currentAvatar else None,
                "name_color":
                user.nameColor,
                "xp_boost":
                user.xpBoost,
            },
            "achievements": {
                "count":
                len(user.achievements_v2) if user.achievements_v2 else 0,
                "details":
                achievement_details,
                "completion_percentage":
                round(
                    (
                        len(user.achievements_v2) /
                        Achievement.objects.count() * 100
                    ) if user.achievements_v2 else 0,
                    2
                ),
            },
            "shop_items": {
                "count":
                len(user.purchasedItems) if user.purchasedItems else 0,
                "details":
                shop_items_details,
                "equipment_history": [
                    change.to_dict() for change in (
                        user.equipmentHistory[-20 :] if user.
                        equipmentHistory else []
                    )
                ],
            },
            "game_stats": game_stats,
            "subscription": subscription_info,
            "platform_info": {
                "oauth_provider": user.oauth_provider,
                "needs_username": user.needs_username,
                "signup_source": user.signUpSource,
                "tags": user.tags or [],
            },
            "freemium": {
                "practice_questions_remaining":
                user.practiceQuestionsRemaining,
                "free_user_created_at":
                user.freeUserCreatedAt.isoformat()
                if user.freeUserCreatedAt else None,
            },
            "engagement": {
                "score": engagement_score,
                "factors": engagement_factors,
            },
            "admin_info": {
                "is_admin": user.is_admin,
            },
            "management_data": user_mgmt.to_dict() if user_mgmt else None,
        }
```

---

## Documentation

### Documentation for `get_user_details`

**Purpose and Behavior**
The function `get_user_details` retrieves comprehensive user information from a database, including basic profile data, activity logs, achievements, shop items, game stats, subscriptions, and engagement scores. It processes various user-related objects such as `User`, `Achievement`, and `Shop`.

**Key Implementation Details**
- **Input**: Takes a `user_id` (str) as input.
- **Output**: Returns a dictionary containing detailed information about the user.
- **Internal Logic**: 
  - Fetches user data from multiple collections (`User`, `TestAttempt`, `Achievement`, `Shop`).
  - Calculates engagement score using `UserEngagementCalculator`.
  - Handles optional fields and default values gracefully.

**When/Why to Use This Code**
Use this function when you need a complete snapshot of a user's profile, including their achievements, activity history, and game statistics. It is particularly useful for admin dashboards or detailed reports where comprehensive data is required.

**Patterns and Gotchas**
- **Database Queries**: The function makes multiple database queries, which could be optimized using eager loading or caching strategies.
- **Null Handling**: Proper null handling ensures that the function does not break when certain fields are missing. 
- **Performance Considerations**: Given its complexity (27 cyclomatic complexity), consider refactoring to reduce nested logic and improve readability.

This function is a good example of how to aggregate data from multiple sources in Python, but be mindful of performance implications with large datasets.

---

*Generated by CodeWorm on 2026-01-18 04:21*
