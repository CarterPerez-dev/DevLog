# UserAchievementService

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/domains/progression/services/achievement_ops.py
**Language:** python
**Lines:** 154-320
**Complexity:** 0.0

---

## Source Code

```python
class UserAchievementService:
    """
    Service class for user achievement operations
    """
    @staticmethod
    def count_all_users() -> int:
        """
        Get total user count (used for achievement statistics)
        """
        return int(User.objects().count())

    @staticmethod
    def count_users_with_achievement(achievement_id: str) -> int:
        """
        Count users who have a specific achievement
        """
        return int(
            User.objects(achievements_v2__achievementId = achievement_id
                         ).count()
        )

    @staticmethod
    def get_achievement_ids(user: User) -> list[str]:
        """
        Get list of achievement IDs from v2 format with timestamps
        """
        achievement_ids = []
        for ach in user.achievements_v2:
            if isinstance(ach, dict):
                achievement_ids.append(ach.get('achievementId', ''))
            else:
                achievement_ids.append(ach.achievementId)
        return [aid for aid in achievement_ids if aid]

    @staticmethod
    def has_achievement(user: User, achievement_id: str) -> bool:
        """
        Check if user has a specific achievement
        """
        return achievement_id in UserAchievementService.get_achievement_ids(
            user
        )

    @staticmethod
    def get_unlock_times_map(user: User) -> dict[str, str | None]:
        """
        Get a mapping of achievement IDs to their unlock times
        """
        unlock_times: dict[str, str | None] = {}
        if user.achievements_v2:
            for user_ach in user.achievements_v2:
                if isinstance(user_ach, dict):
                    achievement_id = user_ach.get('achievementId')
                    unlocked_at = user_ach.get('unlockedAt')
                    if achievement_id:
                        unlock_times[achievement_id] = (
                            unlocked_at.isoformat() if unlocked_at
                            and hasattr(unlocked_at,
                                        'isoformat') else None
                        )
                else:
                    unlock_times[user_ach.achievementId] = (
                        user_ach.unlockedAt.isoformat()
                        if user_ach.unlockedAt else None
                    )
        return unlock_times

    @staticmethod
    def get_completion_stats(user: User,
                             total_achievements: int) -> dict[str,
                                                              int | float]:
        """
        Calculate achievement completion statistics for a user
        """
        unlocked_count = len(
            UserAchievementService.get_achievement_ids(user)
        )
        locked_count = total_achievements - unlocked_count
        completion_percentage = (
            round((unlocked_count / total_achievements * 100),
                  2) if total_achievements > 0 else 0
        )

        return {
            "total": total_ac
```

---

## Class Documentation

### UserAchievementService

**Class Responsibility and Purpose:**
The `UserAchievementService` class is a service layer responsible for managing user achievements within the application. It provides static methods to count users with specific achievements, retrieve achievement details, calculate completion statistics, and format achievement data for display.

**Public Interface (Key Methods):**
- `count_all_users()`: Returns the total number of users.
- `count_users_with_achievement(achievement_id)`: Counts users who have a specific achievement.
- `get_achievement_ids(user)`: Retrieves a list of achievement IDs from user objects.
- `has_achievement(user, achievement_id)`: Checks if a user has a specific achievement.
- `get_unlock_times_map(user)`: Returns a mapping of achievement IDs to their unlock times.
- `get_completion_stats(user, total_achievements)`: Calculates the completion statistics for a user.
- `format_achievement_display(achievement, is_unlocked, unlock_time)`: Formats achievement data for display, handling hidden achievements.
- `add_achievement(user, achievement_id, xp_reward=0, coin_reward=0)`: Adds an achievement to a user with a timestamp.

**Design Patterns Used:**
The class utilizes static methods to encapsulate functionality without requiring instance creation. It does not explicitly use design patterns like Factory, Observer, or Strategy but follows Pythonic practices such as type hints and context managers where applicable.

**Relationship to Other Classes:**
- **User**: The `User` model is referenced for fetching user data.
- **Achievement**: Assumed to be another class that holds achievement details such as title, description, category, etc. This relationship is inferred from the method signatures.

This service class fits into a broader architecture where it acts as a utility layer providing methods to interact with and manipulate user achievements efficiently.

---

*Generated by CodeWorm on 2026-02-18 13:34*
