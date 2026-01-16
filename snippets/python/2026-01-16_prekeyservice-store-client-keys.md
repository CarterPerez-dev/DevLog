# PrekeyService.store_client_keys

**Repository:** Cybersecurity-Projects
**File:** PROJECTS/encrypted-p2p-chat/backend/app/services/prekey_service.py
**Language:** python
**Lines:** 45-150
**Complexity:** 8.0

---

## Source Code

```python
async def store_client_keys(
        self,
        session: AsyncSession,
        user_id: UUID,
        identity_key: str,
        identity_key_ed25519: str,
        signed_prekey: str,
        signed_prekey_signature: str,
        one_time_prekeys: list[str]
    ) -> IdentityKey:
        """
        Stores client-generated public keys for E2E encryption.
        Only stores PUBLIC keys - private keys remain on client.
        """
        statement = select(User).where(User.id == user_id)
        result = await session.execute(statement)
        user = result.scalar_one_or_none()

        if not user:
            logger.error("User not found: %s", user_id)
            raise UserNotFoundError("User not found")

        existing_ik_statement = select(IdentityKey).where(
            IdentityKey.user_id == user_id
        )
        existing_ik_result = await session.execute(existing_ik_statement)
        existing_ik = existing_ik_result.scalar_one_or_none()

        if existing_ik:
            existing_ik.public_key = identity_key
            existing_ik.public_key_ed25519 = identity_key_ed25519
            logger.info("Updated existing identity key for user %s", user_id)
        else:
            existing_ik = IdentityKey(
                user_id = user_id,
                public_key = identity_key,
                private_key = "",
                public_key_ed25519 = identity_key_ed25519,
                private_key_ed25519 = ""
            )
            session.add(existing_ik)
            logger.info("Created identity key for user %s", user_id)

        old_spks_statement = select(SignedPrekey).where(
            SignedPrekey.user_id == user_id,
            SignedPrekey.is_active
        )
        old_spks_result = await session.execute(old_spks_statement)
        old_spks = old_spks_result.scalars().all()

        for old_spk in old_spks:
            old_spk.is_active = False

        max_key_id_statement = select(SignedPrekey.key_id).where(
            SignedPrekey.user_id == user_id
        ).order_by(SignedPrekey.key_id.desc()).limit(1)
        max_key_id_result = await session.execute(max_key_id_statement)
        max_key_id = max_key_id_result.scalar_one_or_none()
        new_spk_key_id = (max_key_id + 1) if max_key_id is not None else 1

        expires_at = datetime.now(UTC) + timedelta(
            hours = SIGNED_PREKEY_ROTATION_HOURS
        )

        new_spk = SignedPrekey(
            user_id = user_id,
            key_id = new_spk_key_id,
            public_key = signed_prekey,
            private_key = "",
            signature = signed_prekey_signature,
            is_active = True,
            expires_at = expires_at
        )
        session.add(new_spk)

        max_opk_key_id_statement = select(OneTimePrekey.key_id).where(
            OneTimePrekey.user_id == user_id
        ).order_by(OneTimePrekey.key_id.desc()).limit(1)
        max_opk_key_id_result = await session.execute(max_opk_key_id_statement)
        max_opk_key_id = max_opk_key_id_result.scalar_one_or_none()
        next_opk_key_id = (max_opk_key_id + 1) if max_opk_key_id is not None else 1

        for i, opk_public in enumerate(one_time_prekeys):
            new_opk = OneTimePrekey(
                user_id = user_id,
                key_id = next_opk_key_id + i,
                public_key = opk_public,
                private_key = "",
                is_used = False
            )
            session.add(new_opk)

        try:
            await session.commit()
            await session.refresh(existing_ik)
            logger.info(
                "Stored client keys for user %s: IK + SPK + %s OPKs",
                user_id,
                len(one_time_prekeys)
            )
        except IntegrityError as e:
            await session.rollback()
            logger.error("Database error storing client keys: %s", e)
            raise DatabaseError("Failed to store client keys") from e

        return existing_ik
```

---

## Documentation

### Documentation for `store_client_keys`

**Purpose and Behavior:**
The function `store_client_keys` in the `PrekeyService` class handles storing client-generated public keys (identity key, signed prekey, and one-time prekeys) for end-to-end encryption. It updates or creates identity keys and manages signed and one-time prekeys.

**Key Implementation Details:**
1. **Identity Key Management:** Updates existing identity keys if they exist; otherwise, creates new ones.
2. **Signed Prekey Handling:** Deactivates old active signed prekeys and adds a new one with an incremented key ID.
3. **One-Time Prekeys:** Adds multiple one-time prekeys with sequentially increasing key IDs.
4. **Database Transactions:** Uses `session.execute` to query the database and `session.add`, `session.commit`, and `session.refresh` for updates.

**When/Why to Use:**
Use this function when setting up or updating client keys in a secure, end-to-end encryption system. It ensures that user keys are properly stored and managed, enhancing security by rotating prekeys periodically.

**Patterns/Gotchas:**
- **Database Integrity:** The code handles `IntegrityError` by rolling back the transaction to prevent partial updates.
- **Key Management:** Efficiently manages key IDs for signed and one-time prekeys, ensuring no gaps in sequence.
- **Logging:** Comprehensive logging helps with debugging and auditing.

---

*Generated by CodeWorm on 2026-01-16 13:06*
