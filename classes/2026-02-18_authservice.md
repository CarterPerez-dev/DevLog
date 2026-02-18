# AuthService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/auth/services/auth.py
**Language:** python
**Lines:** 32-211
**Complexity:** 0.0

---

## Source Code

```python
class AuthService:
    """
    Business logic for authentication operations
    """
    @staticmethod
    async def authenticate(
        session: AsyncSession,
        email: str,
        password: str,
        device_id: str | None = None,
        device_name: str | None = None,
        ip_address: str | None = None,
    ) -> tuple[str,
               str,
               User]:
        """
        Authenticate user and create tokens
        """
        user = await UserRepository.get_by_email(session, email)
        hashed_password = user.hashed_password if user else None

        is_valid, new_hash = await verify_password_with_timing_safety(
            password, hashed_password
        )

        if not is_valid or user is None:
            raise InvalidCredentials()

        if not user.is_active:
            raise InvalidCredentials()

        if new_hash:
            await UserRepository.update_password(session, user, new_hash)

        access_token = create_access_token(user.id, user.token_version)

        family_id = uuid6.uuid7()
        raw_refresh, token_hash, expires_at = create_refresh_token(user.id, family_id)

        await RefreshTokenRepository.create_token(
            session,
            user_id = user.id,
            token_hash = token_hash,
            family_id = family_id,
            expires_at = expires_at,
            device_id = device_id,
            device_name = device_name,
            ip_address = ip_address,
        )

        return access_token, raw_refresh, user

    @staticmethod
    async def login(
        session: AsyncSession,
        email: str,
        password: str,
        device_id: str | None = None,
        device_name: str | None = None,
        ip_address: str | None = None,
    ) -> tuple[TokenWithUserResponse,
               str]:
        """
        Login and return tokens with user data
        """
        access_token, refresh_token, user = await AuthService.authenticate(
            session,
            email,
            password,
            device_id,
            device_name,
            ip_address,
        )

        response = TokenWithUserResponse(
            access_token = access_token,
            user = UserResponse.model_validate(user),
        )
        return response, refresh_token

    @staticmethod
    async def refresh_tokens(
        session: AsyncSession,
        refresh_token: str,
        device_id: str | None = None,
        device_name: str | None = None,
        ip_address: str | None = None,
    ) -> tuple[TokenResponse, str]:
        """
        Refresh access token using refresh token

        Implements token rotation with replay attack detection

        Returns:
            Tuple of (TokenResponse, new_raw_refresh_token)
        """
        token_hash = hash_token(refresh_token)
        stored_token = await RefreshTokenRepository.get_by_hash(
            session,
            token_hash
        )

        if stored_token is None:
            raise TokenError(m
```

---

## Class Documentation

### AuthService Documentation

**Class Responsibility and Purpose:**
The `AuthService` class handles authentication operations, including user login, token creation, refresh, and logout. It ensures secure and efficient management of user sessions by implementing password verification, token generation, and token revocation mechanisms.

**Public Interface (Key Methods):**
- **`authenticate(session: AsyncSession, email: str, password: str, ...)`**: Authenticates a user and generates access and refresh tokens.
- **`login(session: AsyncSession, email: str, password: str, ...)`**: Convenience method to log in a user and return tokens with user data.
- **`refresh_tokens(session: AsyncSession, refresh_token: str, ...)`**: Refreshes the access token using a valid refresh token.
- **`logout(session: AsyncSession, refresh_token: str)`**: Logs out by revoking a specific refresh token.
- **`logout_all(session: AsyncSession, user: User)`**: Logs out all tokens associated with a user.

**Design Patterns Used:**
The class utilizes the **Strategy Pattern** for password verification through `verify_password_with_timing_safety`, ensuring secure handling of passwords. It also employs the **Observer Pattern** via token revocation mechanisms to manage session states effectively.

**Relationship to Other Classes:**
- **`UserRepository`**: Manages user data retrieval and updates.
- **`RefreshTokenRepository`**: Handles refresh token creation, storage, and revocation.
- **`create_access_token`, `create_refresh_token`**: Utility functions for generating secure tokens.

**State Management Approach:**
The class manages state through database operations, ensuring that each authentication or token operation is recorded and can be audited. Token states are updated to reflect current session statuses, such as revoking expired or revoked tokens.

---

*Generated by CodeWorm on 2026-02-18 17:03*
