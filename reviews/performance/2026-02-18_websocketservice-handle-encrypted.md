# WebSocketService.handle_encrypted_message

**Type:** Performance Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/encrypted-p2p-chat/backend/app/services/websocket_service.py
**Language:** python
**Lines:** 87-179
**Complexity:** 11.0

---

## Source Code

```python
async def handle_encrypted_message(
        self,
        user_id: UUID,
        message: dict[str,
                      Any]
    ) -> None:
        """
        Process client-encrypted message and forward to recipient (pass-through)
        """
        try:
            recipient_id = UUID(message.get("recipient_id"))
            room_id = message.get("room_id")
            ciphertext = message.get("ciphertext")
            nonce = message.get("nonce")
            header = message.get("header")
            temp_id = message.get("temp_id", "")

            if not ciphertext or not nonce or not header:
                logger.error("Missing encryption fields in message from %s", user_id)
                return

            if not room_id:
                logger.error("Missing room_id in message from %s", user_id)
                return

            async with async_session_maker() as session:
                result = await message_service.store_encrypted_message(
                    session,
                    user_id,
                    recipient_id,
                    ciphertext,
                    nonce,
                    header,
                    room_id,
                )

            ws_message = EncryptedMessageWS(
                message_id = result.id if hasattr(result, 'id') else "unknown",
                sender_id = str(user_id),
                recipient_id = str(recipient_id),
                room_id = room_id,
                content = "",
                ciphertext = ciphertext,
                nonce = nonce,
                header = header,
                sender_username = result.sender_username if hasattr(result, 'sender_username') else ""
            )

            is_recipient_connected = connection_manager.is_user_connected(recipient_id)
            logger.debug(
                "Sending to recipient %s - connected: %s",
                recipient_id,
                is_recipient_connected
            )

            await connection_manager.send_message(
                recipient_id,
                ws_message.model_dump(mode = "json")
            )
            logger.debug("Message sent to recipient %s", recipient_id)

            confirmation = MessageSentWS(
                temp_id = temp_id,
                message_id = result.id if hasattr(result, 'id') else "unknown",
                room_id = room_id,
                status = "sent",
                created_at = result.created_at if hasattr(result, 'created_at') else datetime.now(UTC)
            )

            await connection_manager.send_message(
                user_id,
                confirmation.model_dump(mode = "json")
            )

            logger.info(
                "Encrypted message forwarded: %s -> %s in room %s",
                user_id,
                recipient_id,
                room_id
            )

        except ValueError as e:
            logger.error(
                "Invalid UUID in encrypted message from %s: %s",
                
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of the function is \(O(1)\) for most operations, as it involves a fixed number of checks and database calls. The primary bottleneck is the database query `await message_service.store_encrypted_message`, which could be costly if not optimized.

#### Space Complexity
Space complexity is also \(O(1)\), assuming that the input parameters and local variables do not grow with the size of the input. However, the function creates several objects like `ws_message` and `confirmation`, which can consume memory, especially in high-traffic scenarios.

#### Bottlenecks or Inefficiencies
1. **Redundant Checks**: The checks for `ciphertext`, `nonce`, and `header` are redundant since they are already validated when the message is received.
2. **Multiple JSON Dumps**: Converting objects to JSON multiple times (`ws_message.model_dump(mode='json')`) can be optimized by doing it once and reusing the string.
3. **Logging Overhead**: Frequent logging, especially in error handling, can introduce significant overhead.

#### Optimization Opportunities
1. **Reduce Redundant Checks**: Remove checks for `ciphertext`, `nonce`, and `header` if they are guaranteed to be present by the client.
2. **Minimize JSON Dumps**: Cache the JSON string of `ws_message` and reuse it in subsequent calls.
3. **Optimize Logging**: Use structured logging with log levels to reduce overhead, e.g., only log errors at the error level.

#### Resource Usage Concerns
- Ensure that database connections are properly managed using context managers (`async with async_session_maker() as session`) to avoid resource leaks.
- Consider caching frequently accessed data like `recipient_id` and `user_id` to reduce database hits.

---

*Generated by CodeWorm on 2026-02-18 23:36*
