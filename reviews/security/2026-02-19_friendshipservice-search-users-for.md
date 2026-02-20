# FriendshipService.search_users_for_friends

**Type:** Security Review
**Repository:** CertGames-Core
**File:** backend/api/domains/social/services/friendship_ops.py
**Language:** python
**Lines:** 393-478
**Complexity:** 11.0

---

## Source Code

```python
def search_users_for_friends(
        user_id: str | ObjectId,
        search_term: str,
        limit: int = 20
    ) -> list[SearchUserDict]:
        """
        Search users by username (autocomplete)
        Returns users with their friendship status relative to the searcher
        """
        user_id = ObjectId(user_id
                           ) if isinstance(user_id,
                                           str) else user_id

        users = User.objects(
            username__icontains = search_term,
            id__ne = user_id
        ).limit(limit)

        existing_friendships = Friendship.objects().filter(
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
        )

        friendship_map = {}
        for friendship in existing_friendships:
            if friendship.requesterUserId == user_id:
                friend_id = str(friendship.recipientUserId)
            else:
                friend_id = str(friendship.requesterUserId)

            can_send = True
            if friendship.status == FriendshipStatus.REJECTED.value and friendship.requesterUserId == user_id:
                thirty_days_ago = datetime.now(UTC) - timedelta(days = 30)
                recent_rejections = [
                    dt for dt in friendship.rejectionHistory if (
                        dt.replace(tzinfo = UTC) if dt.tzinfo is
                        None else dt
                    ) >= thirty_days_ago
                ]
                can_send = len(recent_rejections) < 6

            friendship_map[friend_id] = {
                "status": friendship.status,
                "friendshipId": str(friendship.id),
                "iRequested": friendship.requesterUserId == user_id,
                "canSendRequest": can_send,
            }

        result: list[SearchUserDict] = []
        for user in users:
            friend_data = friendship_map.get(str(user.id), {})
            result.append(
                {
                    "userId":
                    str(user.id),
                    "username":
                    user.username,
                    "level":
                    user.level,
                    "currentAvatar":
                    str(user.currentAvatar)
                    if user.currentAvatar else None,
                    "nameColor":
                    user.nameColor,
                    "friendshipStatus":
                    friend_data.get("status"),
                    "friendshipId":
                    friend_data.get("friendshipId"),
                    "iRequested":
                    friend_data.get("iRequested",
                                    False),
                    "canSendRequest":
                    friend_data.get("canSendRequest",
                                    True),
     
```

---

## Security Review

### Security Review

#### Vulnerabilities Found:

1. **Hardcoded Secrets or Credentials:**
   - Severity: **Info**
   - The code does not contain any hardcoded secrets, but it's recommended to ensure no sensitive data is stored in plain text elsewhere.

2. **Input Validation Gaps:**
   - Severity: **Medium**
   - Line 6: `user_id` could be a string or an `ObjectId`. Ensure that the input validation for `search_term` and `limit` parameters is consistent across all functions to prevent injection attacks.

3. **Error Handling:**
   - Severity: **Info**
   - The code does not provide detailed error handling, which might leak information about the application's internal state if an exception occurs.

#### Attack Vectors:

- **Injection Attacks:** If `search_term` is derived from user input without proper validation, it could lead to injection attacks.
- **Information Leakage:** Improper error handling can expose sensitive information about the database structure or application logic.

#### Recommended Fixes:

1. **Input Validation:**
   - Ensure all inputs are properly validated and sanitized (e.g., using `validate_search_term` function).
   ```python
   from bson.objectid import ObjectId

   def validate_search_term(term):
       if not isinstance(term, str) or len(term.strip()) == 0:
           raise ValueError("Invalid search term")
       return term
   ```

2. **Error Handling:**
   - Implement proper error handling to catch and log exceptions without exposing sensitive information.
   ```python
   try:
       # Code execution
   except Exception as e:
       logger.error(f"An error occurred: {str(e)}", exc_info=True)
       return []
   ```

3. **Security Posture:**
   - Maintain a consistent security posture by applying these fixes across the application.
   - Consider using environment variables for configuration settings and secrets.

Overall, the code is relatively secure but could benefit from improved input validation and robust error handling to prevent potential vulnerabilities.

---

*Generated by CodeWorm on 2026-02-19 22:59*
