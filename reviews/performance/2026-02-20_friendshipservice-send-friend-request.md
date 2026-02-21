# FriendshipService.send_friend_request

**Type:** Performance Analysis
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
                requester_avatar =
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of the `send_friend_request` function is primarily determined by database queries, which are O(n). Specifically, the `User.objects.first()` and `Friendship.objects.first()` calls are costly due to potential full table scans.

#### Space Complexity
Space complexity is minimal as the function does not allocate significant additional memory. However, the use of `ObjectId` conversion within each parameter check can be optimized.

#### Bottlenecks or Inefficiencies
1. **Redundant ObjectID Conversion**: The `ObjectId` conversion is repeated for each parameter. This can be simplified by converting all parameters at once.
2. **Multiple Database Queries**: There are multiple database queries, which can lead to performance degradation if the dataset grows.

#### Optimization Opportunities
1. **Batch ObjectID Conversion**: Convert all string IDs to `ObjectId` in a single step:
   ```python
   ids = [requester_id, recipient_id, existing_friendship_id]
   for i, id in enumerate(ids):
       if isinstance(id, str):
           ids[i] = ObjectId(id)
   requester_id, recipient_id, existing_friendship_id = ids
   ```

2. **Reduce Redundant Queries**: Ensure that the `User` and `Friendship` objects are fetched only once:
   ```python
   user = User.objects(
       id=requester_id
   ).only("username", "level", "currentAvatar", "nameColor").first()
   friendship = Friendship.objects(id=existing_friendship_id).first() if existing_friendship_id else None

   # Update or create the friendship
   if friendship:
       friendship.update(set__status=FriendshipStatus.PENDING.value, set__createdAt=datetime.now(UTC))
       friendship.reload()
       SocialSocketService.emit_friend_request_received(...)
   else:
       friendship = Friendship(
           requesterUserId=requester_id,
           recipientUserId=recipient_id,
           status=FriendshipStatus.PENDING.value,
           createdAt=datetime.now(UTC)
       )
       friendship.save()
       SocialSocketService.emit_friend_request_received(...)
   ```

3. **Caching**: Consider caching frequently accessed user data to reduce database load.

4. **Indexing**: Ensure that the `id` fields are indexed in both `User` and `Friendship` collections to speed up query performance.

#### Resource Usage Concerns
- **Database Connections**: Ensure proper use of context managers or connection pools to manage database connections efficiently.
- **Socket Emissions**: The `emit_friend_request_received` call should be optimized if it involves network I/O, as it can block the main thread. Consider using asynchronous versions of such calls in async contexts.

By addressing these points, you can significantly improve the performance and efficiency of this function.

---

*Generated by CodeWorm on 2026-02-20 23:41*
