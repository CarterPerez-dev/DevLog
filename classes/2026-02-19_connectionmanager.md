# ConnectionManager

**Type:** Class Documentation
**Repository:** vuemantics
**File:** backend/core/websocket/manager.py
**Language:** python
**Lines:** 17-177
**Complexity:** 0.0

---

## Source Code

```python
class ConnectionManager:
    """
    Manages WebSocket connections and subscriptions

    Handles connection lifecycle and routing messages to subscribers
    Does NOT handle Redis pub/sub (that's in publisher.py)
    """
    def __init__(self) -> None:
        """
        Initialize connection manager
        """
        self.user_connections: dict[str, set[WebSocket]] = defaultdict(set)
        self.upload_subscribers: dict[str, set[str]] = defaultdict(set)
        self.user_subscriptions: dict[str, set[str]] = defaultdict(set)

    async def connect(self, user_id: str, websocket: WebSocket) -> None:
        """
        Register new WebSocket connection

        Args:
            user_id: User's ID
            websocket: WebSocket connection
        """
        self.user_connections[user_id].add(websocket)
        logger.info(
            f"User {user_id} connected "
            f"(total connections: {len(self.user_connections[user_id])})"
        )

    async def disconnect(self, user_id: str, websocket: WebSocket) -> None:
        """
        Unregister WebSocket connection and clean up subscriptions

        Args:
            user_id: User's ID
            websocket: WebSocket connection
        """
        self.user_connections[user_id].discard(websocket)

        if not self.user_connections[user_id]:
            del self.user_connections[user_id]

            # Clean up all subscriptions for this user
            for upload_id in list(self.user_subscriptions.get(user_id,
                                                              [])):
                await self.unsubscribe_upload(user_id, upload_id)

        logger.info(f"User {user_id} disconnected")

    async def subscribe_upload(self, user_id: str, upload_id: str) -> None:
        """
        Subscribe user to upload progress updates

        Args:
            user_id: User's ID
            upload_id: Upload's ID
        """
        self.upload_subscribers[upload_id].add(user_id)
        self.user_subscriptions[user_id].add(upload_id)
        logger.debug(f"User {user_id} subscribed to upload {upload_id}")

    async def unsubscribe_upload(
        self,
        user_id: str,
        upload_id: str
    ) -> None:
        """
        Unsubscribe user from upload updates

        Args:
            user_id: User's ID
            upload_id: Upload's ID
        """
        self.upload_subscribers[upload_id].discard(user_id)
        self.user_subscriptions[user_id].discard(upload_id)

        if not self.upload_subscribers[upload_id]:
            del self.upload_subscribers[upload_id]
        if not self.user_subscriptions[user_id]:
            del self.user_subscriptions[user_id]

    async def send_to_user(
        self,
        user_id: str,
        message: ServerMessage
    ) -> None:
        """
        Send message to all connections for a specific user

        Args:
            user_id: User's ID
            message: Message to send
        """
        connections = self.user_co
```

---

## Class Documentation

### ConnectionManager Documentation

**Class Responsibility and Purpose**
The `ConnectionManager` class manages WebSocket connections and subscriptions, ensuring that messages are routed correctly to connected users or upload subscribers. It does not handle Redis pub/sub functionality, which is managed by another module (`publisher.py`).

**Public Interface (Key Methods)**
- **`__init__()`**: Initializes the connection manager with data structures for managing user connections, upload subscribers, and user subscriptions.
- **`connect(user_id: str, websocket: WebSocket)`**: Registers a new WebSocket connection for a given user.
- **`disconnect(user_id: str, websocket: WebSocket)`**: Unregisters a WebSocket connection and cleans up associated subscriptions if the user has no remaining connections.
- **`subscribe_upload(user_id: str, upload_id: str)`**: Subscribes a user to receive updates about an upload.
- **`unsubscribe_upload(user_id: str, upload_id: str)`**: Unsubscribes a user from receiving updates for a specific upload.
- **`send_to_user(user_id: str, message: ServerMessage)`**: Sends a message to all WebSocket connections associated with a given user.
- **`send_to_upload_subscribers(upload_id: str, message: ServerMessage)`**: Sends a message to all users subscribed to an upload.
- **`get_upload_subscriber_ids(upload_id: str) -> set[str]`**: Returns the set of user IDs subscribed to a specific upload.
- **`disconnect_all()`**: Forces the disconnection of all WebSocket connections during application shutdown.

**Design Patterns Used**
- **Observer Pattern**: The class observes WebSocket connections and subscriptions, notifying subscribers when messages are sent or users disconnect.
- **Strategy Pattern**: Although not explicitly used here, the handling of different types of messages could be abstracted using a strategy pattern if needed in future enhancements.

**How it Fits in the Architecture**
`ConnectionManager` is a central component responsible for managing WebSocket connections and subscriptions. It integrates with other parts of the application by providing methods to send messages to specific users or groups, ensuring that real-time updates are delivered efficiently. This class acts as a bridge between the WebSocket layer and higher-level services, such as user management and upload tracking.

---

*Generated by CodeWorm on 2026-02-19 01:51*
