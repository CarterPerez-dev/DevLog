# authenticate_websocket

**Type:** Performance Analysis
**Repository:** vuemantics
**File:** backend/auth/dependencies.py
**Language:** python
**Lines:** 41-125
**Complexity:** 13.0

---

## Source Code

```python
async def authenticate_websocket(websocket: WebSocket) -> str | None:
    """
    Authenticate WebSocket connection via first message pattern

    Returns:
        User ID if authentication successful, None otherwise
    """
    origin = websocket.headers.get("origin", "")
    if origin and origin not in config.settings.cors_origins:
        logger.warning(f"WebSocket from unauthorized origin: {origin}")
        return None

    await websocket.accept()

    try:
        auth_data = await asyncio.wait_for(
            websocket.receive_json(),
            timeout = config.WEBSOCKET_AUTH_TIMEOUT
        )
    except TimeoutError:
        await websocket.send_json(
            AuthError(message = "Authentication timeout").model_dump()
        )
        await websocket.close(
            code = config.WEBSOCKET_CLOSE_AUTH_TIMEOUT,
            reason = "Authentication timeout"
        )
        return None
    except WebSocketDisconnect:
        logger.debug(
            "WebSocket disconnected before auth (likely React StrictMode)"
        )
        return None
    except Exception as e:
        logger.error(f"Error receiving auth message: {e}")
        with contextlib.suppress(RuntimeError):
            await websocket.close(
                code = config.WEBSOCKET_CLOSE_INVALID_MESSAGE,
                reason = "Invalid message format"
            )
        return None

    if auth_data.get("type") != "auth" or not auth_data.get("token"):
        await websocket.send_json(
            AuthError(message = "Authentication required").model_dump()
        )
        await websocket.close(
            code = config.WEBSOCKET_CLOSE_AUTH_REQUIRED,
            reason = "Authentication required"
        )
        return None

    try:
        payload = decode_token(
            auth_data["token"],
            expected_type = TokenType.ACCESS
        )

        user_id_raw = payload.get("sub")
        if not user_id_raw:
            raise AuthenticationError("Missing user ID in token")

        user_id: str = str(user_id_raw)
        user = await User.find_by_id(UUID(user_id))
        if not user or not user.is_active:
            raise AuthenticationError("User not found or inactive")

        token_version = payload.get("token_version", 0)
        if token_version != user.token_version:
            raise AuthenticationError("Token has been invalidated")

        await websocket.send_json(
            AuthSuccess(user_id = user_id).model_dump()
        )

        return user_id

    except AuthenticationError as e:
        logger.warning(f"WebSocket auth failed: {e}")
        await websocket.send_json(AuthError(message = str(e)).model_dump())
        await websocket.close(
            code = config.WEBSOCKET_CLOSE_INVALID_TOKEN,
            reason = "Invalid token"
        )
        return None
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of `authenticate_websocket` is primarily determined by the `decode_token`, `User.find_by_id`, and exception handling. The overall complexity is O(1) for each request, but the `decode_token` function could be costly if it involves cryptographic operations.

#### Space Complexity and Memory Allocation
- **Memory Usage**: The function uses a few local variables (`auth_data`, `user_id_raw`, `payload`, `user`) which are bounded by the size of the input data. However, the `User.find_by_id` call might involve database queries, leading to potential memory overhead.
- **Resource Management**: Proper use of context managers and exception handling ensures that resources like websockets are closed correctly.

#### Bottlenecks or Inefficiencies
1. **Redundant Logging**: The function logs multiple times for the same error conditions (e.g., `WebSocketDisconnect`), which can be optimized by centralizing logging.
2. **Exception Handling Overhead**: Multiple exception handlers increase overhead, especially if exceptions are infrequent.

#### Optimization Opportunities
1. **Centralize Error Responses**: Combine similar error responses to reduce redundancy and improve readability.
   ```python
   async def handle_error(websocket, e):
       await websocket.send_json(AuthError(message=str(e)).model_dump())
       await websocket.close(
           code=config.WEBSOCKET_CLOSE_INVALID_MESSAGE,
           reason="Invalid message format"
       )
   ```

2. **Optimize Exception Handling**: Use a single `try-except` block to handle common exceptions.
   ```python
   try:
       # Authentication logic
   except (WebSocketDisconnect, TimeoutError) as e:
       await handle_error(websocket, e)
       return None
   except Exception as e:
       logger.error(f"Unexpected error: {e}")
       with contextlib.suppress(RuntimeError):
           await websocket.close(
               code=config.WEBSOCKET_CLOSE_INVALID_MESSAGE,
               reason="Internal server error"
           )
       return None
   ```

#### Resource Usage Concerns
- **Database Queries**: Ensure `User.find_by_id` is optimized and uses indexes.
- **WebSocket Management**: Properly manage WebSocket connections to avoid leaks. The current implementation should be sufficient, but consider using a connection pool if handling many concurrent connections.

By centralizing error responses and optimizing exception handling, the function can become more efficient and maintainable.

---

*Generated by CodeWorm on 2026-02-21 22:22*
