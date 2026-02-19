# ProfileService.get_public_profile_by_username

**Type:** Performance Analysis
**Repository:** CertGames-Core
**File:** backend/api/domains/social/services/profile_ops.py
**Language:** python
**Lines:** 35-203
**Complexity:** 20.0

---

## Source Code

```python
def get_public_profile_by_username(
        username: str
    ) -> PublicProfileResponse:
        """
        Get public profile data by username
        Returns user info, achievements, test stats, public certifications, and inventory
        """
        user = User.objects(username = username).first()
        if not user:
            raise NotFoundError("User", username)

        user_id = user.id

        friend_count = Friendship.objects(
            status = FriendshipStatus.ACCEPTED.value
        ).filter(
            __raw__ = {
                "$or": [
                    {
                        "requesterUserId": user_id
                    },
                    {
                        "recipientUserId": user_id
                    }
                ]
            }
        ).count()

        test_stats = TestAnalysisService.get_comprehensive_test_stats_for_user(
            user_id
        )

        public_certs = CertificationService.get_public_certifications_by_user(
            user_id
        )

        all_achievements = AchievementService.get_all()
        achievement_map = {
            ach.achievementId: ach
            for ach in all_achievements
        }

        user_achievement_ids = []
        if user.achievements_v2:
            for ach in user.achievements_v2:
                if isinstance(ach, dict):
                    user_achievement_ids.append(ach.get('achievementId'))
                else:
                    user_achievement_ids.append(ach.achievementId)

        unlocked_achievements: list[AchievementDict] = []
        for ach_id in user_achievement_ids:
            if ach_id in achievement_map:
                ach = achievement_map[ach_id]
                unlocked_achievements.append(
                    {
                        "achievementId": ach.achievementId,
                        "title": ach.title,
                        "description": ach.description,
                        "xpReward": ach.xpReward,
                        "coinReward": ach.coinReward,
                        "category": ach.category,
                    }
                )

        total_achievements = len(all_achievements)
        unlocked_count = len(unlocked_achievements)
        completion_percentage = round(
            (unlocked_count / total_achievements * 100),
            2
        ) if total_achievements > 0 else 0

        inventory_avatars: list[AvatarItemDict] = []
        inventory_name_colors: list[NameColorItemDict] = []
        inventory_xp_boosts: list[XPBoostItemDict] = []
        total_coins_spent = 0.0

        if user.purchasedItems:
            shop_items = Shop.objects(id__in = user.purchasedItems)
            for item in shop_items:
                if item.type == "avatar":
                    is_equipped = user.currentAvatar and str(
                        user.currentAvatar
                    ) == str(item.id)
                    inventory_avatars.append(
                        {
                 
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of the `get_public_profile_by_username` function is dominated by database queries, which are O(n) for each query. Specifically:
- The `Friendship.objects` query is O(f), where f is the number of friendships.
- The `Shop.objects` query is O(s), where s is the number of purchased items.

#### Space Complexity
The space complexity is primarily influenced by the storage of intermediate results, such as `user_achievement_ids`, `unlocked_achievements`, and `inventory_items`. The overall space complexity is O(a + c + v), where a is the number of achievements, c is the number of certifications, and v is the number of purchased items.

#### Bottlenecks or Inefficiencies
1. **Multiple Database Queries**: There are multiple database queries (`User`, `Friendship`, `Shop`), leading to potential N+1 query patterns.
2. **Redundant Iterations**: The loop over `user_achievements_v2` and the subsequent loop for `unlocked_achievements` can be optimized.
3. **Unnecessary Type Checks**: The type checks in the `user_achievement_ids` list comprehension are redundant.

#### Optimization Opportunities
1. **Batch Queries**: Use batch queries to reduce the number of database calls. For example, use `User.objects().only(...)` and `Shop.objects(id__in=user.purchasedItems).only(...)`.
2. **Cache Intermediate Results**: Cache `all_achievements` if they don't change frequently.
3. **Simplify Type Checks**: Use type hints to avoid redundant checks.

#### Resource Usage Concerns
- Ensure that database connections are properly managed using context managers or connection pools to prevent resource leaks.
- Consider implementing caching mechanisms for frequently accessed data like achievements and certifications to reduce the load on the database.

By addressing these issues, you can significantly improve the performance of this function.

---

*Generated by CodeWorm on 2026-02-19 18:42*
