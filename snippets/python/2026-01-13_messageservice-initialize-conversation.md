# MessageService.initialize_conversation

**Repository:** Cybersecurity-Projects
**File:** PROJECTS/encrypted-p2p-chat/backend/app/services/message_service.py
**Language:** python
**Lines:** 48-166
**Complexity:** 10.0

---

## Source Code

```python
async def initialize_conversation(
        self,
        session: AsyncSession,
        sender_id: UUID,
        recipient_id: UUID
    ) -> RatchetState:
        """
        Performs X3DH key exchange and initializes Double Ratchet for new conversation
        """
        if sender_id == recipient_id:
            raise InvalidDataError("Cannot start conversation with yourself")

        existing_state_statement = select(RatchetState).where(
            RatchetState.user_id == sender_id,
            RatchetState.peer_user_id == recipient_id
        )
        existing_state_result = await session.execute(existing_state_statement)
        existing_state = existing_state_result.scalar_one_or_none()

        if existing_state:
            logger.warning(
                "Ratchet state already exists for %s -> %s",
                sender_id,
                recipient_id
            )
            return existing_state

        sender_ik_statement = select(IdentityKey).where(
            IdentityKey.user_id == sender_id
        )
        sender_ik_result = await session.execute(sender_ik_statement)
        sender_ik = sender_ik_result.scalar_one_or_none()

        if not sender_ik:
            logger.error("Sender identity key not found: %s", sender_id)
            raise InvalidDataError(
                "Sender has no identity key - initialize encryption first"
            )

        recipient_bundle = await prekey_service.get_prekey_bundle(
            session,
            recipient_id
        )

        recipient_ik_statement = select(IdentityKey).where(
            IdentityKey.user_id == recipient_id
        )
        recipient_ik_result = await session.execute(recipient_ik_statement)
        recipient_ik = recipient_ik_result.scalar_one_or_none()

        if not recipient_ik:
            logger.error("Recipient identity key not found: %s", recipient_id)
            raise InvalidDataError("Recipient has no identity key")

        try:
            x3dh_result = x3dh_manager.perform_x3dh_sender(
                alice_identity_private_x25519 = sender_ik.private_key,
                bob_bundle = recipient_bundle,
                bob_identity_public_ed25519 = recipient_ik.public_key_ed25519
            )
        except Exception as e:
            logger.error("X3DH key exchange failed: %s", e)
            raise KeyExchangeError(f"Key exchange failed: {str(e)}") from e

        recipient_spk_public_bytes = base64url_to_bytes(
            recipient_bundle.signed_prekey
        )

        dr_state = double_ratchet.initialize_sender(
            shared_key = x3dh_result.shared_key,
            peer_public_key = recipient_spk_public_bytes
        )

        dh_private_bytes = dr_state.dh_private_key.private_bytes(
            encoding = serialization.Encoding.Raw,
            format = serialization.PrivateFormat.Raw,
            encryption_algorithm = serialization.NoEncryption()
        ) if dr_state.dh_private_key else b''

        dh_public_bytes = dr_state.dh_private_key.public_key().public_bytes(
            encoding = serialization.Encoding.Raw,
            format = serialization.PublicFormat.Raw
        ) if dr_state.dh_private_key else b''

        ratchet_state = RatchetState(
            user_id = sender_id,
            peer_user_id = recipient_id,
            dh_private_key = bytes_to_base64url(dh_private_bytes),
            dh_public_key = bytes_to_base64url(dh_public_bytes),
            dh_peer_public_key = bytes_to_base64url(dr_state.dh_peer_public_key)
            if dr_state.dh_peer_public_key else None,
            root_key = bytes_to_base64url(dr_state.root_key),
            sending_chain_key = bytes_to_base64url(dr_state.sending_chain_key),
            receiving_chain_key = bytes_to_base64url(
                dr_state.receiving_chain_key
            ),
            sending_message_number = dr_state.sending_message_number,
            receiving_message_number = dr_state.receiving_message_number,
            previous_sending_chain_length = (
                dr_state.previous_sending_chain_length
            )
        )

        session.add(ratchet_state)

        try:
            await session.commit()
            await session.refresh(ratchet_state)
            logger.info(
                "Initialized conversation: %s -> %s (X3DH complete)",
                sender_id,
                recipient_id
            )
        except IntegrityError as e:
            await session.rollback()
            logger.error("Database error saving ratchet state: %s", e)
            raise DatabaseError("Failed to initialize conversation") from e

        return ratchet_state
```

---

## Documentation

### Documentation for `initialize_conversation`

**Purpose and Behavior:**
The function `initialize_conversation` in the `MessageService` class performs an X3DH key exchange to establish a secure Double Ratchet state for initiating a new conversation between two users. It handles identity keys, prekey bundles, and ensures no self-conversations are initiated.

**Key Implementation Details:**
- **X3DH Key Exchange:** Utilizes `x3dh_manager.perform_x3dh_sender` for key exchange.
- **Database Interaction:** Uses SQLAlchemy's `AsyncSession` to interact with the database. It checks for existing ratchet states and adds new ones.
- **Error Handling:** Logs errors and raises specific exceptions like `InvalidDataError`, `KeyExchangeError`, and `DatabaseError`.

**When/Why to Use:**
This function should be used when starting a new conversation between two users in an encrypted P2P chat application. It ensures secure key exchange and state initialization, critical for maintaining the integrity of the communication.

**Patterns and Gotchas:**
- **Asynchronous Operations:** The function is designed to work asynchronously using `async`/`await`.
- **Error Propagation:** Exceptions are caught and re-raised with detailed error messages.
- **Database Integrity:** Ensures no duplicate states by checking for existing ratchet states before adding new ones.

---

*Generated by CodeWorm on 2026-01-13 11:45*
