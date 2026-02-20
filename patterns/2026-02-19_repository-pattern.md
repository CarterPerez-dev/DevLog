# repository_pattern

**Type:** Pattern Analysis
**Repository:** Cybersecurity-Projects
**File:** TEMPLATES/fullstack-template/backend/app/auth/service.py
**Language:** python
**Lines:** 1-213
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
service.py
"""

import uuid6
from sqlalchemy.ext.asyncio import (
    AsyncSession,
)

from core.exceptions import (
    InvalidCredentials,
    TokenError,
    TokenRevokedError,
)
from core.security import (
    hash_token,
    create_access_token,
    create_refresh_token,
    verify_password_with_timing_safety,
)
from user.User import User
from user.repository import UserRepository
from .repository import RefreshTokenRepository
from .schemas import (
    TokenResponse,
    TokenWithUserResponse,
)
from user.schemas import UserResponse


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
        refresh_token: str
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Repository Pattern

The `AuthService` class in the provided code implements the **Repository Pattern**, which abstracts data access operations behind a common interface. This pattern is used to encapsulate database interactions within the `UserRepository` and `RefreshTokenRepository` classes.

#### Implementation Details:
- The `authenticate`, `login`, `refresh_tokens`, and `logout` methods interact with these repositories through their respective interfaces.
- For example, in the `authenticate` method, `UserRepository.get_by_email` is called to fetch a user by email, and `RefreshTokenRepository.create_token` is used to store new tokens.

#### Benefits:
1. **Decoupling:** The service layer is decoupled from the database implementation details.
2. **Testability:** Repositories can be easily mocked or replaced for unit testing.
3. **Maintainability:** Changes in data storage (e.g., switching from SQLAlchemy to another ORM) are isolated.

#### Deviations:
- The `AuthService` class directly instantiates repositories and sessions, which is not a typical pattern. Normally, these dependencies would be injected via constructor parameters or a dependency injection framework.
- The `logout` method does not return any value but still adheres to the pattern by performing an operation (revoking tokens).

#### Appropriateness:
This pattern is highly appropriate in this context as it clearly separates business logic from data access. However, for better testability and maintainability, consider using dependency injection to pass repositories and session objects into `AuthService`.

By following these guidelines, you can ensure that the service layer remains clean and focused on its core responsibilities while abstracting away complex database interactions.

---

*Generated by CodeWorm on 2026-02-19 21:05*
