# UploadProgressPublisher._listen_redis

**Type:** Performance Analysis
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
            logger.error(f"Redis listener err
```

---

## Performance Analysis

### Performance Analysis

#### Time and Space Complexity
The time complexity of `_listen_redis` is primarily driven by the `get_message`, `send_text`, and `unsubscribe` operations, which are O(1) for each call but can accumulate over many iterations. The space complexity is O(n) due to the storage of connections and dead_connections sets.

#### Bottlenecks or Inefficiencies
1. **Redundant Decoding**: Each message's channel and data are decoded multiple times.
2. **Unnecessary Iterations**: The `for ws in connections` loop iterates over all connections, even if only a few fail to send the message.
3. **Blocking Calls**: `asyncio.wait_for` with timeouts can block the event loop.

#### Optimization Opportunities
1. **Decode Messages Once**: Decode messages once and store them as strings.
2. **Filter Connections Before Iteration**: Filter out dead connections before iterating over them to avoid unnecessary try-except blocks.
3. **Reduce Timeout Usage**: Use more efficient error handling without blocking calls.

```python
async def _listen_redis(self) -> None:
    pubsub = await redis_pool.psubscribe("upload:*", "user:*")
    logger.info("Redis listener started (patterns: upload:*, user:*)")

    while self._running:
        try:
            message = await pubsub.get_message(
                ignore_subscribe_messages=True,
                timeout=0.1
            )

            if not message or message["type"] != "pmessage":
                continue

            channel, payload = message["channel"].decode("utf-8"), message["data"].decode("utf-8")

            user_ids: list[str]
            if channel.startswith("upload:"):
                upload_id = channel.split(":", 1)[1]
                user_ids = get_manager().get_upload_subscriber_ids(upload_id)
            elif channel.startswith("user:"):
                user_id = channel.split(":", 1)[1]
                user_ids = [user_id]
            else:
                continue

            for user_id in user_ids:
                connections = set(get_manager().user_connections.get(user_id, []))
                dead_connections = set()

                for ws in connections.copy():
                    try:
                        await ws.send_text(payload)
                    except Exception as e:
                        logger.debug(f"Failed to forward message to user {user_id}: {e}")
                        dead_connections.add(ws)

                for ws in dead_connections:
                    await get_manager().disconnect(user_id, ws)
        except asyncio.CancelledError:
            break
        except Exception as e:
            logger.error(f"Error in Redis listener: {e}")
            if self._running:
                await asyncio.sleep(1)

    try:
        await pubsub.unsubscribe()
        await pubsub.close()
    except Exception as e:
        logger.error(f"Error closing pubsub: {e}")
```

#### Resource Usage Concerns
- Ensure `pubsub` is properly closed to avoid resource leaks.
- Use context managers or `async with` for better resource management.

---

*Generated by CodeWorm on 2026-02-21 22:06*
