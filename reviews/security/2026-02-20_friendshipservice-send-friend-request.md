# FriendshipService.send_friend_request

**Type:** Security Review
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

## Security Review

### Security Review

#### Vulnerabilities Found:

1. **Input Validation Gaps** (Medium):
   - The function does not validate the input types or values for `requester_id`, `recipient_id`, and `existing_friendship_id`. This could allow injection attacks if these inputs are manipulated.

2. **Hardcoded Secrets or Credentials** (Info):
   - No hardcoded secrets or credentials were found in this snippet, but ensure no sensitive information is stored elsewhere in the codebase.

3. **Error Handling that Leaks Information** (Low):
   - The function does not handle errors gracefully, which could lead to information leakage if an error occurs during database operations.

#### Attack Vectors:

- An attacker could manipulate `requester_id`, `recipient_id`, or `existing_friendship_id` to gain unauthorized access or perform actions they shouldn't.
- Improper error handling might reveal sensitive information about the system's internal structure and behavior.

#### Recommended Fixes:

1. **Input Validation**:
   - Add validation checks for input types and values using libraries like `pydantic`.
   ```python
   from pydantic import BaseModel, Field

   class FriendRequestParams(BaseModel):
       requester_id: ObjectId = Field(..., alias="requesterId")
       recipient_id: ObjectId = Field(..., alias="recipientId")
       existing_friendship_id: ObjectId | None = Field(None, alias="existingFriendshipId")

   def send_friend_request(params: FriendRequestParams) -> Friendship:
       # Rest of the function
   ```

2. **Error Handling**:
   - Implement proper error handling to catch and log exceptions without exposing sensitive information.
   ```python
   try:
       # Function logic here
   except Exception as e:
       logger.error(f"An error occurred: {str(e)}")
       return None  # or appropriate error response
   ```

#### Overall Security Posture:

The code has some basic security issues, particularly around input validation and error handling. Improving these areas will significantly enhance the overall security posture of the application.

---

*Generated by CodeWorm on 2026-02-20 09:53*
