# UploadProgressPublisher._listen_redis

**Repository:** vuemantics
**File:** backend/core/websocket/publisher.py
**Language:** python
**Lines:** 107-198
**Complexity:** 21.0

---

## Source Code

```python
async def _listen_redis(self) -> None:
        """
        Background task that listens to Redis pub/sub

        Receives messages from Redis and
        forwards to local WebSocket connections
        """
        pubsub = None
        try:
            pubsub = await redis_pool.psubscribe("upload:*", "user:*")

            logger.info("Redis listener started (patterns: upload:*, user:*)")

            while self._running:
                try:
                    message = await pubsub.get_message(
                        ignore_subscribe_messages = True,
                        timeout = 0.1
                    )

                    if message is None:
                        continue

                    if message["type"] != "pmessage":
                        continue

                    # Note: decode_responses=True means these are already strings
                    channel = message["channel"]
                    if isinstance(channel, bytes):
                        channel = channel.decode("utf-8")

                    payload = message["data"]
                    if isinstance(payload, bytes):
                        payload = payload.decode("utf-8")

                    manager = get_manager()

                    user_ids: list[str]
                    if channel.startswith("upload:"):
                        upload_id = channel.split(":", 1)[1]
                        user_ids = list(manager.get_upload_subscriber_ids(upload_id))
                    elif channel.startswith("user:"):
                        user_id = channel.split(":", 1)[1]
                        user_ids = [user_id]
                    else:
                        continue

                    for user_id in user_ids:
                        connections = list(
                            manager.user_connections.get(user_id,
                                                         set())
                        )
                        dead_connections: set[WebSocket] = set()

                        for ws in connections:
                            try:
                                await ws.send_text(payload)
                            except Exception as e:
                                logger.debug(
                                    f"Failed to forward message to user {user_id}: {e}"
                                )
                                dead_connections.add(ws)

                        # Garbage c
                        for ws in dead_connections:
                            await manager.disconnect(user_id, ws)

                except asyncio.CancelledError:
                    break
                except Exception as e:
                    logger.error(f"Error in Redis listener: {e}")
                    if self._running:
                        await asyncio.sleep(1)

        except asyncio.CancelledError:
            logger.info("Redis listener cancelled")
        except Exception as e:
            logger.error(f"Redis listener error: {e}")
        finally:
            if pubsub:
                try:
                    await asyncio.wait_for(
                        pubsub.unsubscribe(),
                        timeout = 0.5
                    )
                    await asyncio.wait_for(pubsub.close(), timeout = 0.5)
                except TimeoutError:
                    logger.warning(
                        "Timeout closing pubsub connection (non-critical)"
                    )
                except Exception as e:
                    logger.error(f"Error closing pubsub: {e}")
```

---

## Documentation

### Purpose and Behavior

The `_listen_redis` method is an asynchronous background task that listens for messages on Redis channels `upload:*` and `user:*`. It forwards received messages to WebSocket connections associated with specific user IDs.

### Key Implementation Details

- **Redis Pub/Sub**: Uses `psubscribe` to subscribe to multiple patterns.
- **Message Handling**: Filters and decodes messages, then routes them to appropriate WebSocket connections based on the channel pattern.
- **Error Handling**: Gracefully handles exceptions, ensuring the task can be cancelled without crashing.

### When/Why to Use This Code

This code is essential for real-time updates in a web application where user actions (uploads) need to be reflected instantly across connected clients. It ensures efficient and scalable message delivery by leveraging Redis pub/sub and WebSockets.

### Patterns and Gotchas

- **Asynchronous Context Management**: Uses `asyncio.wait_for` to handle cancellations and timeouts gracefully.
- **Resource Cleanup**: Ensures proper cleanup of Redis resources even if an error occurs.
- **Error Logging**: Comprehensive logging helps in debugging issues without interrupting the task's execution.

---

*Generated by CodeWorm on 2026-01-29 08:53*
