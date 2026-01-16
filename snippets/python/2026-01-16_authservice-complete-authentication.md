# AuthService.complete_authentication

**Repository:** Cybersecurity-Projects
**File:** PROJECTS/encrypted-p2p-chat/backend/app/services/auth_service.py
**Language:** python
**Lines:** 488-598
**Complexity:** 11.0

---

## Source Code

```python
async def complete_authentication(
        self,
        session: AsyncSession,
        request: AuthenticationCompleteRequest,
    ) -> UserResponse:
        """
        Complete WebAuthn passkey authentication
        """
        credential_id = request.credential.get("id")
        if not credential_id:
            raise InvalidDataError("Missing credential ID")

        credential = await self.get_credential_by_id(
            session = session,
            credential_id = credential_id,
        )

        if not credential:
            logger.warning(
                "Authentication with unknown credential: %s...",
                credential_id[: 16]
            )
            raise CredentialNotFoundError("Credential not found")

        user = await self.get_user_by_id(
            session = session,
            user_id = credential.user_id,
        )

        if not user:
            logger.error(
                "User not found for credential: %s...",
                credential_id[: 16]
            )
            raise UserNotFoundError("User not found")

        if not user.is_active:
            logger.warning(
                "Authentication attempt for inactive user: %s",
                user.username
            )
            raise UserInactiveError("User account is inactive")

        expected_challenge = await redis_manager.get_authentication_challenge(
            user_id = user.username
        )

        if not expected_challenge:
            logger.warning(
                "Authentication challenge not found for user: %s",
                user.username
            )
            raise ChallengeExpiredError(
                "Challenge expired or not found - please restart authentication"
            )

        try:
            verified = passkey_manager.verify_authentication(
                credential = request.credential,
                expected_challenge = expected_challenge,
                credential_public_key = base64url_to_bytes(credential.public_key),
                credential_current_sign_count = credential.sign_count,
            )
        except ValueError as e:
            logger.error("Authentication verification failed: %s", e)
            raise CredentialVerificationError(str(e)) from e
        except Exception as e:
            logger.error("Unexpected error during authentication: %s", e)
            raise CredentialVerificationError(
                "Authentication verification failed"
            ) from e

        await self.update_credential_counter(
            session = session,
            credential_id = credential.credential_id,
            new_count = verified.new_sign_count,
        )

        if (credential.backup_state != verified.backup_state
                or credential.backup_eligible != verified.backup_eligible):
            await self.update_backup_state(
                session = session,
                credential_id = credential.credential_id,
                backup_state = verified.backup_state,
                backup_eligible = verified.backup_eligible,
            )

        ik_statement = select(IdentityKey).where(IdentityKey.user_id == user.id)
        ik_result = await session.execute(ik_statement)
        existing_ik = ik_result.scalar_one_or_none()

        if not existing_ik:
            logger.info(
                "Initializing encryption keys for existing user: %s",
                user.username
            )
            await prekey_service.initialize_user_keys(
                session = session,
                user_id = user.id,
            )

        logger.info("Authentication successful for user: %s", user.username)

        return UserResponse(
            id = str(user.id),
            username = user.username,
            display_name = user.display_name,
            is_active = user.is_active,
            is_verified = user.is_verified,
            created_at = user.created_at.isoformat(),
        )
```

---

## Documentation

### Documentation for `complete_authentication`

**Purpose and Behavior:**
The `complete_authentication` method in the `AuthService` class handles the final steps of WebAuthn passkey authentication. It verifies the user's credential, updates counters, checks backup states, and initializes encryption keys if necessary.

**Key Implementation Details:**
- **Asynchronous Nature:** The function is marked as `async`, allowing for non-blocking I/O operations.
- **Credential Verification:** Uses `passkey_manager.verify_authentication` to validate the passkey. It handles exceptions like `ValueError` and generic errors, logging appropriate messages.
- **Session Management:** Utilizes an `AsyncSession` object passed from the caller to manage database transactions asynchronously.
- **Database Interaction:** Queries the database using SQLAlchemy to fetch user and credential data.

**When/Why to Use This Code:**
This method is crucial for ensuring secure passkey authentication in web applications. It should be used whenever a user attempts to authenticate with their WebAuthn passkey, especially when integrating multi-factor authentication (MFA) into your application.

**Patterns and Gotchas:**
- **Error Handling:** The function employs robust error handling, logging detailed messages that can aid debugging.
- **Asynchronous Decorators:** Ensure that any decorators used on this method are compatible with asynchronous operations to avoid runtime errors.
- **Credential State Management:** Regularly updates the credential state (e.g., sign count, backup state) to maintain security and prevent replay attacks.

---

*Generated by CodeWorm on 2026-01-16 11:11*
