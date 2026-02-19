# ProfileService.get_public_profile_by_username

**Type:** Security Review
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

## Security Review

### Security Review for `get_public_profile_by_username`

#### Vulnerabilities Found:

1. **Input Validation Gaps** (Line 5-6, Severity: Medium):
   - The function accepts a raw `username` string without any validation or sanitization.
   - This could lead to injection attacks if the username is crafted maliciously.

2. **Hardcoded Secrets or Credentials** (Severity: Info):
   - No hardcoded secrets are found in this snippet, but ensure that all credentials and sensitive information are properly managed elsewhere in the codebase.

3. **Insecure Deserialization** (Line 48-51, Severity: Low):
   - The `user_achievement_ids` list construction assumes a specific structure for `ach`. Ensure robust handling of deserialized data to prevent unexpected behavior.

#### Attack Vectors:

- **Injection Attacks**: A malicious user could inject SQL or command injection payloads by crafting a username that triggers unintended database queries.
- **Deserialization Risks**: If `user.achievements_v2` contains serialized data, improper validation could lead to execution of arbitrary code.

#### Recommended Fixes:

1. **Input Validation**:
   - Validate and sanitize the `username` parameter before using it in database queries (Line 5-6).
     ```python
     if not re.match(r'^[a-zA-Z0-9_]+$', username):
         raise ValueError("Invalid username")
     ```

2. **Secure Deserialization**:
   - Ensure that any deserialized data is validated and sanitized before use.
     ```python
     for ach in user.achievements_v2:
         if isinstance(ach, dict) and 'achievementId' in ach:
             user_achievement_ids.append(ach['achievementId'])
         elif isinstance(ach, Achievement):
             user_achievement_ids.append(ach.achievementId)
     ```

#### Overall Security Posture:

The code snippet is relatively secure but could benefit from additional input validation and deserialization checks. Ensure that all database interactions are protected against injection attacks and that sensitive data handling follows best practices.

---

*Generated by CodeWorm on 2026-02-19 00:14*
