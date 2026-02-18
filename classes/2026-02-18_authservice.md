# AuthService

**Type:** Class Documentation
**Repository:** Cybersecurity-Projects
**File:** TEMPLATES/fullstack-template/backend/app/auth/service.py
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
            await UserRepository.update_password(
                self.session,
                user,
                new_hash
            )

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
    ) -> tuple[TokenResponse, str]:
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

       
```

---

## Class Documentation

### AuthService Documentation

**Class Responsibility and Purpose**
The `AuthService` class is responsible for handling authentication operations, including user login, token generation, and management. It ensures secure and efficient authentication by verifying credentials, generating access and refresh tokens, and managing token revocation.

**Public Interface (Key Methods)**
- **authenticate**: Authenticates a user based on email and password, creating new or updating existing tokens.
- **login**: Logs in a user and returns the necessary tokens with user data.
- **refresh_tokens**: Refreshes an access token using a valid refresh token while handling token rotation and replay attacks.
- **logout**: Revokes a specific refresh token to log out a user from a particular session.
- **logout_all**: Logs out a user from all devices by revoking their existing tokens.

**Design Patterns Used**
The class utilizes several design patterns:
- **Factory Pattern**: Implicitly used in the creation of access and refresh tokens through `create_access_token` and `create_refresh_token`.
- **Observer Pattern**: Indirectly, as token revocation events can be observed and handled by other components.
- **Strategy Pattern**: The password verification logic is encapsulated within `verify_password_with_timing_safety`, allowing for flexible implementation of different strategies.

**How It Fits in the Architecture**
`AuthService` acts as a central hub for authentication operations. It interacts with the `UserRepository` and `RefreshTokenRepository` to manage user data and tokens, ensuring that all authentication-related logic is encapsulated within this service class. This design promotes separation of concerns and makes the system more modular and testable.

---

*Generated by CodeWorm on 2026-02-18 07:57*
