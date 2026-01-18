# ProfileService.get_public_profile_by_username

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
                            "_id": str(item.id),
                            "title": item.title,
                            "is_equipped": bool(is_equipped),
                            "imageUrl": item.imageUrl,
                        }
                    )

                elif item.type == "nameColor":
                    is_equipped = user.nameColor == item.value
                    inventory_name_colors.append(
                        {
                            "_id": str(item.id),
                            "title": item.title,
                            "is_equipped": is_equipped,
                            "value": item.value,
                        }
                    )

                elif item.type == "xpBoost":
                    inventory_xp_boosts.append(
                        {
                            "_id": str(item.id),
                            "title": item.title,
                            "is_equipped": False,
                            "value": item.value,
                        }
                    )

                if item.cost:
                    total_coins_spent += item.cost

        return {
            "userId":
            str(user_id),
            "username":
            user.username,
            "level":
            user.level,
            "current_role":
            user.current_role,
            "next_role":
            user.next_role,
            "currentAvatar":
            str(user.currentAvatar) if user.currentAvatar else None,
            "nameColor":
            user.nameColor,
            "xp":
            user.xp,
            "coins":
            user.coins,
            "current_streak":
            user.current_streak,
            "longest_streak":
            user.longest_streak,
            "totalFriends":
            friend_count,
            "inventory": {
                "avatars":
                inventory_avatars,
                "nameColors":
                inventory_name_colors,
                "xpBoosts":
                inventory_xp_boosts,
                "totalItems":
                len(user.purchasedItems) if user.purchasedItems else 0,
                "totalCoinsSpent":
                total_coins_spent,
            },
            "achievements":
            unlocked_achievements,
            "achievementSummary": {
                "total": unlocked_count,
                "completionPercentage": completion_percentage,
            },
            "testStats":
            cast(TestStatsDict,
                 test_stats),
            "certifications": [cert.to_dict() for cert in public_certs],
            "isOnline":
            user.isOnline,
            "lastSeenAt":
            user.lastSeenAt.isoformat() if user.lastSeenAt else None,
        }
```

---

## Documentation

### Documentation for `get_public_profile_by_username`

**Purpose and Behavior**
The function retrieves a user's public profile data, including basic information like username and level, achievements, test statistics, certifications, and inventory items (avatars, name colors, XP boosts). It also calculates the completion percentage of all available achievements.

**Key Implementation Details**
- **Database Queries**: Uses Django ORM to fetch user details and related objects.
- **Achievement Handling**: Maps user achievements to a dictionary format, filtering out non-existent or outdated ones.
- **Inventory Management**: Processes purchased items from the shop service, categorizing them into avatars, name colors, and XP boosts.

**When/Why to Use This Code**
Use this function when you need to display a comprehensive public profile for a user. It is essential in social features where detailed user information needs to be presented publicly or shared with other users.

**Patterns and Gotchas**
- **Type Hints**: The use of type hints (`PublicProfileResponse`, `AchievementDict`, etc.) ensures clear expectations about the data structure.
- **Raw MongoDB Queries**: The use of `$or` in the friendship query can be optimized for performance if used frequently.
- **Dynamic Type Checking**: The function uses `isinstance` and `.get()` to handle different types of achievements, which might lead to subtle bugs if not carefully managed.

---

*Generated by CodeWorm on 2026-01-18 02:11*
