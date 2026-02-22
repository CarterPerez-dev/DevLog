# AuthService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/auth/services/auth.py
**Language:** python
**Lines:** 32-212
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
    ) -> tuple[TokenResponse,
               str]:
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
            rai
```

---

## Class Documentation

### AuthService Documentation

**Class Responsibility and Purpose**
The `AuthService` class handles all authentication-related operations for users, including login, token creation, refresh, and logout. It ensures secure and efficient management of user sessions by implementing mechanisms such as token rotation, replay attack detection, and device tracking.

**Public Interface (Key Methods)**
- **authenticate**: Authenticates a user based on email and password, creating access and refresh tokens.
- **login**: A convenience method for logging in users and returning tokens with user data.
- **refresh_tokens**: Refreshes the access token using a valid refresh token, implementing token rotation and replay protection.
- **logout**: Revokes a specific refresh token to log out a user.
- **logout_all**: Logs out all of a user's active sessions by revoking their refresh tokens.

**Design Patterns Used**
- **Factory Pattern**: Implicitly used in the creation of access and refresh tokens through utility functions like `create_access_token` and `create_refresh_token`.
- **Observer Pattern**: Not explicitly implemented but implied through token revocation mechanisms.
- **Strategy Pattern**: Implied by the flexible handling of different authentication strategies, such as password-based verification.

**Relationship to Other Classes**
- **UserRepository**: Manages user data retrieval and updates.
- **RefreshTokenRepository**: Handles refresh token storage, validation, and revocation.
- **verify_password_with_timing_safety**: Ensures secure password comparison with timing safety.

**State Management Approach**
The class manages state through database operations, ensuring that tokens are valid and sessions are active. It uses asynchronous methods to handle database interactions efficiently, leveraging Python's `asyncio` for non-blocking I/O operations.

---

*Generated by CodeWorm on 2026-02-22 02:36*
