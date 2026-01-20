# FriendshipService.search_users_for_friends

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
                }
            )

        return result
```

---

## Documentation

### Documentation for `search_users_for_friends`

**Purpose and Behavior:**
The function `search_users_for_friends` searches for users by username, returning a list of potential friends along with their friendship status relative to the user making the request. It filters out the current user from search results and checks for existing friendships or rejection history.

**Key Implementation Details:**
- **Type Hints:** The function accepts `user_id`, `search_term`, and an optional `limit` parameter, returning a list of dictionaries containing user information.
- **ObjectID Handling:** Converts `user_id` to `ObjectId` if it's provided as a string.
- **Query Filtering:** Uses PyMongo queries to find users matching the search term and exclude the current user. It also retrieves existing friendships related to the user.
- **Friendship Status Mapping:** Maps each friend to their status, whether they requested or received the friendship request, and checks if the user can send a new request based on recent rejection history.

**When/Why to Use:**
Use this function when implementing autocomplete features for adding friends in social applications. It ensures that users see relevant suggestions while considering existing friendships and recent interactions.

**Patterns/Gotchas:**
- **Type Safety:** Utilizes type hints (`str | ObjectId`, `list[SearchUserDict]`) for better code readability.
- **Date Handling:** Uses UTC timezone for date comparisons, which is crucial for accurate time calculations.
- **Potential Performance Issues:** The function involves multiple database queries and list comprehensions, which could impact performance with large datasets. Consider optimizing if used frequently.

This function encapsulates complex logic in a clear, reusable manner, making it suitable for integration into larger applications.

---

*Generated by CodeWorm on 2026-01-20 00:41*
