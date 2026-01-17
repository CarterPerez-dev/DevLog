# MessageService.send_encrypted_message

**Repository:** Cybersecurity-Projects
**File:** PROJECTS/encrypted-p2p-chat/backend/app/services/message_service.py
**Language:** python
**Lines:** 316-402
**Complexity:** 5.0

---

## Source Code

```python
async def send_encrypted_message(
        self,
        session: AsyncSession,
        sender_id: UUID,
        recipient_id: UUID,
        plaintext: str,
        room_id: str | None = None,
    ) -> Any:
        """
        [DEPRECATED] Server-side encryption - kept for backwards compatibility
        Encrypts message with Double Ratchet and stores in SurrealDB
        """
        ratchet_state_statement = select(RatchetState).where(
            RatchetState.user_id == sender_id,
            RatchetState.peer_user_id == recipient_id
        )
        ratchet_state_result = await session.execute(ratchet_state_statement)
        ratchet_state_db = ratchet_state_result.scalar_one_or_none()

        if not ratchet_state_db:
            logger.warning(
                "No ratchet state for %s -> %s, initializing",
                sender_id,
                recipient_id
            )
            ratchet_state_db = await self.initialize_conversation(
                session,
                sender_id,
                recipient_id
            )

        dr_state = await self._load_ratchet_state_from_db(ratchet_state_db)

        sender_user_statement = select(User).where(User.id == sender_id)
        sender_user_result = await session.execute(sender_user_statement)
        sender_user = sender_user_result.scalar_one_or_none()

        if not sender_user:
            raise UserNotFoundError("Sender not found")

        associated_data = f"{sender_id}:{recipient_id}".encode()

        try:
            encrypted_msg = double_ratchet.encrypt_message(
                dr_state,
                plaintext.encode(),
                associated_data
            )
        except Exception as e:
            logger.error("Encryption failed: %s", e)
            raise EncryptionError(f"Failed to encrypt message: {str(e)}") from e

        await self._save_ratchet_state_to_db(session, ratchet_state_db, dr_state)

        message_header = {
            "dh_public_key": bytes_to_base64url(encrypted_msg.dh_public_key),
            "message_number": encrypted_msg.message_number,
            "previous_chain_length": encrypted_msg.previous_chain_length
        }

        from datetime import UTC, datetime

        now = datetime.now(UTC)
        surreal_message = {
            "sender_id": str(sender_id),
            "recipient_id": str(recipient_id),
            "room_id": room_id,
            "ciphertext": bytes_to_base64url(encrypted_msg.ciphertext),
            "nonce": bytes_to_base64url(encrypted_msg.nonce),
            "header": json.dumps(message_header),
            "sender_username": sender_user.username,
            "created_at": now.isoformat(),
            "updated_at": now.isoformat(),
        }

        try:
            result = await surreal_db.create_message(surreal_message)
            logger.info(
                "Sent encrypted message: %s -> %s (msg #%s)",
                sender_id,
                recipient_id,
                encrypted_msg.message_number
            )
            return result
        except Exception as e:
            logger.error("Failed to store encrypted message: %s", e)
            raise DatabaseError(f"Failed to store message: {str(e)}") from e
```

---

## Documentation

### Documentation for `send_encrypted_message`

**Purpose and Behavior:**
The function `send_encrypted_message` handles the encryption and storage of messages in a secure, asynchronous manner using Double Ratchet Algorithm (DRA). It retrieves necessary ratchet state from the database, encrypts the plaintext message, updates the ratchet state, and stores the encrypted message along with metadata.

**Key Implementation Details:**
- **Asynchronous Operations:** Uses `async` functions for database interactions.
- **Error Handling:** Catches exceptions during encryption and storage processes, logging errors and re-raising them as custom exceptions (`EncryptionError`, `DatabaseError`).
- **Ratchet State Management:** Loads ratchet state from the database, initializes if not present, and updates it after encryption.

**When/Why to Use:**
This function is used for secure messaging in a P2P chat application where messages need to be encrypted using DRA. It ensures backward compatibility while maintaining security by handling both existing and new conversations.

**Patterns and Gotchas:**
- **Deprecation Notice:** The docstring indicates this method is deprecated, suggesting it should not be used for new development.
- **Custom Exceptions:** Custom exceptions are raised for specific error cases, making debugging easier.
- **Asynchronous Context Management:** Proper use of `async`/`await` ensures non-blocking database operations.

This function is crucial for maintaining the security and integrity of messages in a P2P chat application.

---

*Generated by CodeWorm on 2026-01-17 16:04*
