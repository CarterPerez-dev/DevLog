# SocketManager

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/websockets/socket_manager.py
**Language:** python
**Lines:** 12-122
**Complexity:** 0.0

---

## Source Code

```python
class SocketManager:
    """
    Global socket emission utility for all domains
    """
    @staticmethod
    def _get_socketio() -> SocketIO | None:
        """
        Get SocketIO instance from Flask app extensions
        """
        return current_app.extensions.get("socketio")

    @staticmethod
    def get_user_room(user_id: str) -> str:
        """
        Get consistent user room name
        """
        return f"user_{user_id}"

    @staticmethod
    def emit_to_user(
        user_id: str,
        event: str,
        data: Mapping[str,
                      Any],
        include_self: bool = True
    ) -> bool:
        """
        Emit socket event to a specific user's personal room
        """
        socketio = SocketManager._get_socketio()
        if not socketio:
            return False

        room = SocketManager.get_user_room(user_id)

        try:
            socketio.emit(
                event,
                data,
                room = room,
                include_self = include_self
            )
            return True
        except Exception:
            return False

    @staticmethod
    def emit_to_users(
        user_ids: list[str],
        event: str,
        data: Mapping[str,
                      Any] | None = None,
        user_specific_data: dict[str,
                                 Mapping[str,
                                         Any]] | None = None
    ) -> int:
        """
        Emit socket event to multiple users
        """
        if not user_ids:
            return 0

        socketio = SocketManager._get_socketio()
        if not socketio:
            current_app.logger.warning(
                f"[SocketManager] SocketIO not available for event '{event}'"
            )
            return 0

        success_count = 0

        for user_id in user_ids:
            room = SocketManager.get_user_room(user_id)

            user_data = data or {}
            if user_specific_data and user_id in user_specific_data:
                user_data = {**user_data, **user_specific_data[user_id]}

            try:
                socketio.emit(event, user_data, room = room)
                success_count += 1
            except Exception:
                pass

        return success_count

    @staticmethod
    def emit_to_room(
        room: str,
        event: str,
        data: Mapping[str,
                      Any],
        include_self: bool = True
    ) -> bool:
        """
        Emit socket event to a specific room
        """
        socketio = SocketManager._get_socketio()
        if not socketio:
            return False

        try:
            socketio.emit(
                event,
                data,
                room = room,
                include_self = include_self
            )
            return True
        except Exception:
            return False
```

---

## Class Documentation

### SocketManager Class Documentation

**Class Responsibility and Purpose:**
The `SocketManager` class serves as a utility for emitting socket events to specific users or rooms within a Flask application. It ensures consistent handling of socket communications, abstracting away the complexities of obtaining the `SocketIO` instance.

**Public Interface (Key Methods):**
- `_get_socketio()`: Static method to retrieve the `SocketIO` instance from the Flask app extensions.
- `get_user_room(user_id: str) -> str`: Returns a consistent user room name based on the provided user ID.
- `emit_to_user(user_id: str, event: str, data: Mapping[str, Any], include_self: bool = True) -> bool`: Emits an event to a specific user's personal room.
- `emit_to_users(user_ids: list[str], event: str, data: Mapping[str, Any] | None = None, user_specific_data: dict[str, Mapping[str, Any]] | None = None) -> int`: Emits an event to multiple users, handling individual data for each user if needed.
- `emit_to_room(room: str, event: str, data: Mapping[str, Any], include_self: bool = True) -> bool`: Emits an event to a specific room.

**Design Patterns Used:**
The class does not explicitly use any design patterns but follows the Singleton pattern implicitly by ensuring that socket operations are centralized through static methods. It also uses exception handling for robust error management.

**How it Fits in the Architecture:**
`SocketManager` acts as a central hub for all socket-related operations within the application, making it easier to manage and maintain socket communications across different parts of the system. By abstracting away the `SocketIO` instance retrieval, it promotes loose coupling between components that need to emit events and the underlying WebSocket infrastructure.

This class is crucial for ensuring consistent and reliable communication in a real-time application, fitting well into the overall architecture by providing a clear interface for socket operations.

---

*Generated by CodeWorm on 2026-02-18 13:53*
