# publisher

**Type:** File Overview
**Repository:** vuemantics
**File:** backend/core/websocket/publisher.py
**Language:** python
**Lines:** 1-223
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2026
publisher.py
"""

import asyncio
import logging

from fastapi import WebSocket

from core.redis import redis_pool
from core.websocket.manager import get_manager
from core.websocket.messages import ServerMessage


logger = logging.getLogger(__name__)


class UploadProgressPublisher:
    """
    Publishes upload progress events via Redis pub/sub

    Handles cross instance message delivery and local distribution
    Listens to Redis channels and forwards to connected WebSocket clients
    """
    def __init__(self) -> None:
        """
        Initialize publisher
        """
        self._listener_task: asyncio.Task | None = None
        self._running = False

    async def start(self) -> None:
        """
        Start Redis listener background task
        """
        if self._running:
            return

        self._running = True
        self._listener_task = asyncio.create_task(self._listen_redis())
        logger.info("UploadProgressPublisher started")

    async def stop(self) -> None:
        """
        Stop Redis listener
        """
        self._running = False

        if self._listener_task:
            self._listener_task.cancel()
            try:
                await asyncio.wait_for(self._listener_task, timeout = 2.0)
            except asyncio.CancelledError:
                pass
            except TimeoutError:
                logger.warning(
                    "Redis listener task did not stop within timeout, forcing shutdown"
                )
            except Exception as e:
                logger.error(f"Error stopping Redis listener: {e}")

        logger.info("UploadProgressPublisher stopped")

    async def publish_progress(
        self,
        upload_id: str,
        message: ServerMessage
    ) -> None:
        """
        Publish upload progress to Redis

        Args:
            upload_id: Upload's ID
            message: Progress message to broadcast
        """
        channel = f"upload:{upload_id}"
        payload = message.model_dump_json()

        try:
            await redis_pool.publish(channel, payload)
            logger.debug(f"Published progress for upload {upload_id}")
        except Exception as e:
            logger.error(f"Failed to publish to Redis: {e}")

    async def publish_to_user(
        self,
        user_id: str,
        message: ServerMessage
    ) -> None:
        """
        Publish message directly to a user

        Args:
            user_id: User's ID
            message: Message to broadcast
        """
        channel = f"user:{user_id}"
        payload = message.model_dump_json()

        try:
            await redis_pool.publish(channel, payload)
            logger.debug(f"Published message to user {user_id}")
        except Exception as e:
            logger.error(f"Failed to publish to Redis: {e}")

    async def _listen_redis(self) -> None:
        """
        Background task that listens to Redis pub/sub

        Receives messages from Redis and
 
```

---

## File Overview

### publisher.py

**Purpose:**
This Python source file is responsible for publishing upload progress events and user-specific messages via Redis pub/sub. It ensures cross-instance message delivery and local distribution to WebSocket clients.

**Key Exports & Public Interface:**
- `UploadProgressPublisher`: A class that manages the lifecycle of a Redis listener, publishes messages, and forwards them to WebSocket connections.
  - `start()`: Starts the Redis listener background task.
  - `stop()`: Stops the Redis listener gracefully.
  - `publish_progress(upload_id: str, message: ServerMessage)`: Publishes upload progress events to Redis.
  - `publish_to_user(user_id: str, message: ServerMessage)`: Publishes messages directly to a specific user.

**How it Fits in the Project:**
This file is part of the WebSocket management system within the backend. It integrates with the core WebSocket manager and Redis for real-time communication between server instances and clients. The `UploadProgressPublisher` class ensures that upload progress updates are efficiently broadcasted to relevant clients, enhancing the user experience.

**Notable Design Decisions:**
- **AsyncIO**: Utilizes asynchronous programming for efficient handling of multiple WebSocket connections.
- **Redis Pub/Sub**: Implements a publish-subscribe model using Redis to ensure real-time message delivery across server instances.
- **Graceful Shutdown**: The `stop` method ensures that the Redis listener is stopped gracefully, preventing abrupt disconnections and potential data loss.
- **Error Handling**: Robust error handling mechanisms are in place to manage exceptions during message publishing and WebSocket communication.
```

This documentation provides a high-level overview of the file's purpose, key functionalities, integration within the project, and significant design choices.

---

*Generated by CodeWorm on 2026-02-20 12:38*
