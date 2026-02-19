# AuthService

**Type:** Class Documentation
**Repository:** my-portfolio
**File:** v1/backend/app/auth/service.py
**Language:** python
**Lines:** 32-206
**Complexity:** 0.0

---

## Source Code

```python
class AuthService:
    """
    Business logic for authentication operations
    """
    def __init__(self, session: AsyncSession) -> None:
        self.session = session

    async def authenticate(
        self,
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
        user = await UserRepository.get_by_email(self.session, email)
        hashed_password = user.hashed_password if user else None

        is_valid, new_hash = await verify_password_with_timing_safety(
            password, hashed_password
        )

        if not is_valid or user is None:
            raise InvalidCredentials()

        if not user.is_active:
            raise InvalidCredentials()

        if new_hash:
            await UserRepository.update_password(self.session, user, new_hash)

        access_token = create_access_token(user.id, user.token_version)

        family_id = uuid6.uuid7()
        raw_refresh, token_hash, expires_at = create_refresh_token(user.id, family_id)

        await RefreshTokenRepository.create_token(
            self.session,
            user_id = user.id,
            token_hash = token_hash,
            family_id = family_id,
            expires_at = expires_at,
            device_id = device_id,
            device_name = device_name,
            ip_address = ip_address,
        )

        return access_token, raw_refresh, user

    async def login(
        self,
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
        access_token, refresh_token, user = await self.authenticate(
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

    async def refresh_tokens(
        self,
        refresh_token: str,
        device_id: str | None = None,
        device_name: str | None = None,
        ip_address: str | None = None,
    ) -> tuple[TokenResponse,
               str]:
        """
        Refresh access token using refresh token

        Implements token rotation with replay attack detection
        """
        token_hash = hash_token(refresh_token)
        stored_token = await RefreshTokenRepository.get_by_hash(
            self.session,
            token_hash
        )

        if stored_token is None:
            raise TokenError(message = "Invalid refresh token")

        if stored_token.is_revoked:
            await 
```

---

## Class Documentation

### AuthService Documentation

**Class Responsibility and Purpose**
The `AuthService` class handles authentication operations, including user login, token creation, and management. It ensures secure and efficient authentication processes by verifying credentials, creating tokens, and managing refresh tokens.

**Public Interface (Key Methods)**
- **`authenticate(email: str, password: str, ...)`**: Authenticates a user and generates access and refresh tokens.
- **`login(email: str, password: str, ...)`**: A convenience method that returns the `TokenWithUserResponse` along with a raw refresh token after successful authentication.
- **`refresh_tokens(refresh_token: str, ...)`**: Refreshes an access token using a valid refresh token and handles token rotation and replay protection.
- **`logout(refresh_token: str)`**: Logs out by revoking the specified refresh token. It silently succeeds if the token is already revoked or non-existent.
- **`logout_all(user: User)`**: Logs out from all devices of a user, invalidating their session tokens.

**Design Patterns Used**
- **Factory Pattern**: Implicitly used in `create_access_token` and `create_refresh_token` for generating secure tokens.
- **Observer Pattern**: The `RefreshTokenRepository` acts as an observer by managing token revocation events.
- **Strategy Pattern**: The password verification logic is encapsulated, allowing different strategies to be implemented.

**Relationship to Other Classes**
- **UserRepository**: Manages user data and operations such as getting a user by email or updating their hashed password.
- **RefreshTokenRepository**: Handles refresh token creation, retrieval, revocation, and expiration checks.
- **verify_password_with_timing_safety**: A utility function ensuring secure password verification.

**State Management Approach**
The class manages state through database interactions using `AsyncSession`. It ensures that user sessions are securely managed by updating tokens and revoking old ones as necessary.

---

*Generated by CodeWorm on 2026-02-18 19:25*
