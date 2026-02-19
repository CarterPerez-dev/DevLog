# UserManagementService.get_user_details

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis

#### Time Complexity (Big O Notation)
The time complexity of this function is dominated by the database queries, which are `O(n)` for each query executed. Specifically:
- Retrieving user details: `O(1)`
- Calculating days since signup: `O(1)`
- Counting test attempts: `O(1)`
- Achievements and shop items queries: `O(m + k)`, where `m` is the number of achievements and `k` is the number of purchased items.
- Retrieving game stats, subscription info, and location info: `O(1)` each.

#### Bottlenecks or Inefficiencies
1. **Multiple Database Queries**: The function performs multiple database queries (e.g., for achievements, shop items, game stats) without caching, leading to potential performance issues.
2. **Redundant Operations**: Calculating `days_since_signup` and counting test attempts are redundant if these values are already cached or precomputed.
3. **Unnecessary Iterations**: The loop over `user.equipmentHistory` is unnecessary if only the last 20 changes are needed.

#### Optimization Opportunities
1. **Caching**: Cache user-related data (achievements, shop items) to reduce database hits. Use a caching layer like Redis or Memcached.
2. **Precompute Values**: Precompute and store `days_since_signup` and test attempts count in the database.
3. **Optimize Loops**: Only retrieve the last 20 changes from `user.equipmentHistory`.

#### Resource Usage Concerns
1. **Database Connections**: Ensure that connections are managed properly using context managers or connection pooling to avoid resource leaks.
2. **Memory Allocation**: Be mindful of memory usage, especially when handling large datasets.

### Suggested Optimizations

- Implement caching for frequently accessed data.
- Use `@cached_property` decorators to precompute and cache values like `days_since_signup`.
- Optimize loops by slicing directly in the query or using a more efficient approach.

---

*Generated by CodeWorm on 2026-02-19 18:47*
