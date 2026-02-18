# SocialSocketService

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/websockets/social/service.py
**Language:** python
**Lines:** 23-336
**Complexity:** 0.0

---

## Source Code

```python
class SocialSocketService:
    """
    Service for emitting social-related socket events
    """
    @staticmethod
    def emit_friend_request_received(
        friendship_id: str,
        *,
        requester_id: str,
        requester_username: str,
        requester_level: int,
        requester_avatar: str | None,
        requester_name_color: str | None,
        recipient_id: str,
        created_at: datetime
    ) -> bool:
        """
        Emit event when user receives a friend request
        """
        event_data: FriendRequestReceivedEvent = {
            "friendshipId": friendship_id,
            "requesterId": requester_id,
            "requesterUsername": requester_username,
            "requesterLevel": requester_level,
            "requesterAvatar": requester_avatar,
            "requesterNameColor": requester_name_color,
            "createdAt": created_at.isoformat(),
        }

        return SocketManager.emit_to_user(
            user_id = recipient_id,
            event = "friend_request_received",
            data = event_data
        )

    @staticmethod
    def emit_friend_request_accepted(
        friendship_id: str,
        *,
        requester_id: str,
        recipient_id: str,
        recipient_username: str,
        recipient_level: int,
        recipient_avatar: str | None,
        recipient_name_color: str | None,
        recipient_role: str,
        recipient_xp: float,
        recipient_coins: float,
        accepted_at: datetime
    ) -> tuple[bool,
               bool]:
        """
        Emit event when friend request is accepted (to both users)
        """
        event_data: FriendRequestAcceptedEvent = {
            "friendshipId": friendship_id,
            "userId": recipient_id,
            "username": recipient_username,
            "level": recipient_level,
            "currentAvatar": recipient_avatar,
            "nameColor": recipient_name_color,
            "currentRole": recipient_role,
            "xp": recipient_xp,
            "coins": recipient_coins,
            "acceptedAt": accepted_at.isoformat(),
        }

        result_requester = SocketManager.emit_to_user(
            user_id = requester_id,
            event = "friend_request_accepted",
            data = event_data
        )

        result_recipient = SocketManager.emit_to_user(
            user_id = recipient_id,
            event = "friend_list_updated",
            data = {"friendshipId": friendship_id}
        )

        return result_requester, result_recipient

    @staticmethod
    def emit_friend_request_rejected(
        friendship_id: str,
        requester_id: str,
        recipient_id: str,
        recipient_username: str
    ) -> bool:
        """
        Emit event when friend request is rejected
        """
        event_data: FriendRequestRejectedEvent = {
            "friendshipId": friendship_id,
            "recipientId": recipient_id,
            "recipientUsername": recipient_username,
        }

        r
```

---

## Class Documentation

### SocialSocketService Documentation

**Class Responsibility and Purpose:**
The `SocialSocketService` class is responsible for emitting socket events related to social interactions within the application, such as friend requests, challenges, and friendships. It ensures that relevant users are notified of these events through real-time communication.

**Public Interface (Key Methods):**
- **emit_friend_request_received**: Emits an event when a user receives a friend request.
- **emit_friend_request_accepted**: Emits events for both the requester and recipient when a friend request is accepted.
- **emit_friend_request_rejected**: Emits an event when a friend request is rejected.
- **emit_friend_removed**: Emits events to both users involved in removing a friendship.
- **emit_challenge_received**: Emits an event when a user receives a challenge.

**Design Patterns Used:**
The class leverages the **Observer Pattern** by emitting events that can be observed by connected clients. It uses static methods for each type of event, adhering to the Single Responsibility Principle (SRP) by ensuring each method handles one specific task.

**How it Fits in the Architecture:**
`SocialSocketService` is part of the WebSocket service layer within the application's architecture. It interacts with the `SocketManager` to send events to specific users based on their IDs, facilitating real-time communication between clients and the server. This class ensures that social interactions are promptly communicated to all relevant parties, enhancing user engagement and interaction.

This documentation provides a clear understanding of the class's role in the application, its methods, design patterns used, and how it integrates with other components.

---

*Generated by CodeWorm on 2026-02-18 13:21*
