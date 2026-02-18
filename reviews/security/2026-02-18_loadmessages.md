# loadMessages

**Type:** Security Review
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/encrypted-p2p-chat/frontend/src/services/room.service.ts
**Language:** typescript
**Lines:** 51-137
**Complexity:** 11.0

---

## Source Code

```typescript
async function loadMessages(
  roomId: string,
  limit: number = 50,
  offset: number = 0
): Promise<Message[]> {
  try {
    const localMessages = await getDecryptedMessages(roomId, limit)
    const localMessageIds = new Set(localMessages.map((m) => m.id))

    if (localMessages.length > 0) {
      setRoomMessages(roomId, localMessages)
    }

    const response = await api.rooms.getMessages(roomId, limit, offset)
    const serverMessages = response.messages.reverse()

    const newMessages: Message[] = []

    const currentUserId = $userId.get()

    for (const msg of serverMessages) {
      if (localMessageIds.has(msg.id)) {
        continue
      }

      let content = "[Encrypted - from another session]"
      const isOwnMessage = msg.sender_id === currentUserId

      if (isOwnMessage) {
        const localCopy = await getDecryptedMessage(msg.id)
        if (localCopy) {
          content = localCopy.content
        } else {
          content = "[Your message - not stored locally]"
        }
      } else {
        try {
          content = await cryptoService.decrypt(
            msg.sender_id,
            msg.ciphertext,
            msg.nonce,
            msg.header
          )
        } catch {
          content = "[Encrypted - from another session]"
        }
      }

      const decryptedMessage: Message = {
        id: msg.id,
        room_id: msg.room_id,
        sender_id: msg.sender_id,
        sender_username: msg.sender_username,
        content,
        status: "delivered" as const,
        is_encrypted: true,
        encrypted_content: msg.ciphertext,
        nonce: msg.nonce,
        header: msg.header,
        created_at: msg.created_at,
        updated_at: msg.created_at,
      }

      if (!content.startsWith("[Encrypted") && !content.startsWith("[Your message")) {
        void saveDecryptedMessage(decryptedMessage)
      }

      newMessages.push(decryptedMessage)
    }

    const allMessages = [...localMessages, ...newMessages]
    const uniqueMessages = Array.from(
      new Map(allMessages.map((m) => [m.id, m])).values()
    ).sort((a, b) => new Date(a.created_at).getTime() - new Date(b.created_at).getTime())

    setRoomMessages(roomId, uniqueMessages)
    setHasMore(roomId, response.has_more)
    return uniqueMessages
  } catch (err) {
    console.error("[RoomService] Failed to load messages:", err)
    const localMessages = await getDecryptedMessages(roomId, limit)
    if (localMessages.length > 0) {
      setRoomMessages(roomId, localMessages)
    }
    return localMessages
  }
}
```

---

## Security Review

### Security Review for `loadMessages` Function

#### Vulnerabilities Found:

1. **Information Disclosure via Error Handling** - **Severity: Medium**
   - The error message in the catch block is logged to the console, which could potentially leak information about the application's internal structure or state.
   
2. **Hardcoded Secrets or Credentials** - **Severity: Info**
   - No hardcoded secrets are found directly in this code snippet.

3. **Input Validation Gaps** - **Severity: Low**
   - The function parameters (`roomId`, `limit`, `offset`) are not validated for type and range, which could lead to unexpected behavior if invalid values are passed.

4. **Error Handling** - **Severity: Medium**
   - The error handling in the catch block logs sensitive information and falls back to local messages without further validation or logging.

#### Attack Vectors:

- An attacker could exploit the error logging by injecting malicious input, leading to potential information leakage.
- Unvalidated parameters might allow injection attacks if not properly sanitized.

#### Recommended Fixes:

1. **Secure Error Handling:**
   - Use structured logging to avoid exposing sensitive information in logs.
   ```typescript
   catch (err) {
     console.error("[RoomService] Failed to load messages:", { roomId, limit, offset });
     const localMessages = await getDecryptedMessages(roomId, limit);
     if (localMessages.length > 0) {
       setRoomMessages(roomId, localMessages);
     }
     return localMessages;
   }
   ```

2. **Parameter Validation:**
   - Validate and sanitize `roomId`, `limit`, and `offset` parameters to prevent unexpected behavior.
   ```typescript
   if (!roomId || typeof limit !== 'number' || typeof offset !== 'number') {
     throw new Error('Invalid input');
   }
   ```

3. **Overall Security Posture:**
   - Ensure that all sensitive operations, especially those involving decryption and error handling, are thoroughly reviewed for security implications.
   - Implement a comprehensive logging strategy to avoid exposing sensitive information.

By addressing these issues, the overall security posture of the application can be significantly improved.

---

*Generated by CodeWorm on 2026-02-18 11:46*
