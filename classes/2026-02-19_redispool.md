# RedisPool

**Type:** Class Documentation
**Repository:** vuemantics
**File:** backend/core/redis.py
**Language:** python
**Lines:** 19-148
**Complexity:** 0.0

---

## Source Code

```python
class RedisPool:
    """
    Manages Redis connection pool for caching and pub/sub

    Provides methods for caching operations and pub/sub messaging
    Used for WebSocket cross-instance communication
    """
    def __init__(self) -> None:
        """
        Initialize Redis pool instance
        """
        self._pool: redis.ConnectionPool | None = None
        self._client: redis.Redis | None = None
        self._lock = asyncio.Lock()

    async def connect(self) -> None:
        """
        Initialize Redis connection pool
        """
        if self._pool is not None:
            return

        async with self._lock:
            if self._pool is not None:
                return  # type: ignore[unreachable]  # Double-check locking pattern

            try:
                self._pool = redis.ConnectionPool.from_url(
                    config.settings.redis_url,
                    decode_responses = config.settings.
                    redis_decode_responses,
                    max_connections = config.settings.redis_pool_max_size,
                )

                self._client = redis.Redis(connection_pool = self._pool)

                await self._client.ping()

                logger.info(
                    "Redis pool created successfully",
                    extra = {
                        "max_connections":
                        config.settings.redis_pool_max_size,
                    },
                )

            except Exception as e:
                logger.error(f"Failed to create Redis pool: {e}")
                raise

    async def disconnect(self) -> None:
        """
        Close all Redis connections
        """
        if self._client is not None:
            await self._client.close()
            self._client = None

        if self._pool is not None:
            await self._pool.disconnect()
            self._pool = None

        logger.info("Redis pool closed")

    @asynccontextmanager
    async def acquire(self) -> AsyncIterator[redis.Redis]:
        """
        Acquire a Redis connection from the pool
        """
        if self._client is None:
            raise RuntimeError(
                "Redis pool is not initialized. Call connect() first."
            )

        yield self._client

    async def publish(self, channel: str, message: str) -> int:
        """
        Publish message to Redis channel

        Args:
            channel: Channel name
            message: Message to publish

        Returns:
            Number of subscribers that received the message
        """
        async with self.acquire() as client:
            return await client.publish(channel, message)

    async def subscribe(self, *channels: str) -> redis.client.PubSub:
        """
        Subscribe to Redis channels

        Args:
            channels: Channel names to subscribe to

        Returns:
            PubSub object for receiving messages
        """
        if self._client is None:
            raise RuntimeError("
```

---

## Class Documentation

### RedisPool Documentation

**Class Responsibility and Purpose**
The `RedisPool` class manages a connection pool for Redis, providing caching operations and pub/sub messaging capabilities. It ensures thread-safe initialization and management of Redis connections across multiple instances, facilitating WebSocket cross-instance communication.

**Public Interface (Key Methods)**
- **`__init__()`**: Initializes the Redis pool instance with private attributes for managing the connection pool and client.
- **`connect()`**: Initializes the Redis connection pool and client. Ensures thread-safe initialization using a lock to prevent multiple concurrent connections.
- **`disconnect()`**: Closes all Redis connections, ensuring proper cleanup.
- **`acquire()`**: Acquires a Redis connection from the pool with an asynchronous context manager.
- **`publish()`**: Publishes a message to a specified channel and returns the number of subscribers that received the message.
- **`subscribe()`**: Subscribes to one or more channels and returns a `PubSub` object for receiving messages.
- **`psubscribe()`**: Subscribes to channel patterns and returns a `PubSub` object for receiving messages.

**Design Patterns Used**
The class employs several design patterns:
- **Factory Pattern**: Implicitly used through the `connect()` method, which initializes the Redis pool and client.
- **Observer Pattern**: Implemented via the `subscribe()` and `psubscribe()` methods, allowing clients to receive notifications when a message is published.

**How It Fits in the Architecture**
`RedisPool` serves as a central component for managing Redis connections within the application. By providing a connection pool and pub/sub functionality, it ensures efficient resource management and facilitates real-time communication between different parts of the system, particularly useful for WebSocket-based applications. The class integrates seamlessly with other services that require caching or event-driven messaging, enhancing the overall architecture's scalability and performance.

---

*Generated by CodeWorm on 2026-02-19 01:35*
