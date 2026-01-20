# FriendshipService.send_friend_request

**Repository:** CertGames-Core
**File:** backend/api/domains/social/services/friendship_ops.py
**Language:** python
**Lines:** 29-112
**Complexity:** 10.0

---

## Source Code

```python
def send_friend_request(
        requester_id: str | ObjectId,
        recipient_id: str | ObjectId,
        existing_friendship_id: str | ObjectId | None = None
    ) -> Friendship:
        """
        Create a new friend request or reuse existing rejected one
        """
        requester_id = ObjectId(requester_id) if isinstance(
            requester_id,
            str
        ) else requester_id
        recipient_id = ObjectId(recipient_id) if isinstance(
            recipient_id,
            str
        ) else recipient_id

        if existing_friendship_id is not None:
            existing_friendship_id = ObjectId(existing_friendship_id
                                              ) if isinstance(
                                                  existing_friendship_id,
                                                  str
                                              ) else existing_friendship_id

            friendship = Friendship.objects(id = existing_friendship_id
                                            ).first()
            if friendship:
                friendship.update(
                    set__status = FriendshipStatus.PENDING.value,
                    set__createdAt = datetime.now(UTC)
                )
                friendship.reload()

                requester = User.objects(
                    id = requester_id
                ).only("username",
                       "level",
                       "currentAvatar",
                       "nameColor").first()

                if requester:
                    SocialSocketService.emit_friend_request_received(
                        friendship_id = str(friendship.id),
                        requester_id = str(requester_id),
                        requester_username = requester.username,
                        requester_level = requester.level,
                        requester_avatar = str(requester.currentAvatar)
                        if requester.currentAvatar else None,
                        requester_name_color = requester.nameColor,
                        recipient_id = str(recipient_id),
                        created_at = friendship.createdAt
                    )

                return friendship

        friendship = Friendship(
            requesterUserId = requester_id,
            recipientUserId = recipient_id,
            status = FriendshipStatus.PENDING.value,
            createdAt = datetime.now(UTC)
        )
        friendship.save()

        requester = User.objects(
            id = requester_id
        ).only("username",
               "level",
               "currentAvatar",
               "nameColor").first()

        if requester:
            SocialSocketService.emit_friend_request_received(
                friendship_id = str(friendship.id),
                requester_id = str(requester_id),
                requester_username = requester.username,
                requester_level = requester.level,
                requester_avatar = str(requester.currentAvatar)
                if requester.currentAvatar else None,
                requester_name_color = requester.nameColor,
                recipient_id = str(recipient_id),
                created_at = friendship.createdAt
            )

        return friendship
```

---

## Documentation

### Documentation for `send_friend_request` Method

**Purpose and Behavior**
The `send_friend_request` method in the `FriendshipService` class handles creating a new friend request or reusing an existing rejected one. It updates the status of the friendship to "PENDING" if it already exists, otherwise, it creates a new pending friendship.

**Key Implementation Details**
- **Type Hints**: The function uses type hints for input parameters and return types.
- **ObjectId Conversion**: Converts `requester_id`, `recipient_id`, and optionally `existing_friendship_id` to `ObjectId` instances if they are strings.
- **Friendship Update/Creation**: Checks if an existing friendship exists; updates its status to "PENDING" and re-saves it. If no existing friendship, creates a new one with the "PENDING" status.
- **Emit Event**: Emits a socket event `friend_request_received` when a friend request is sent or reused.

**When/Why to Use This Code**
Use this method whenever you need to send a friend request in your application. It handles both creating new requests and reusing existing rejected ones, ensuring consistency in the state of friendships.

**Patterns/Gotchas**
- Ensure that `ObjectId` instances are correctly handled as strings or `ObjectId` objects.
- The method assumes the existence of `Friendship`, `User`, and `SocialSocketService` classes and their associated methods. Misconfigurations here can lead to errors.
- The use of `only()` with specific fields in queries helps optimize database performance by reducing the amount of data fetched.

---

*Generated by CodeWorm on 2026-01-20 12:44*
