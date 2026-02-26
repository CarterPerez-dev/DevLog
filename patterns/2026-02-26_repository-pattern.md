# repository_pattern

**Type:** Pattern Analysis
**Repository:** my-portfolio
**File:** v1/backend/app/auth/service.py
**Language:** python
**Lines:** 1-207
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
        device_name: s
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Repository Pattern

#### Implementation in Code
The `AuthService` class interacts with repositories such as `UserRepository` and `RefreshTokenRepository`. These repositories handle database operations like fetching, updating, and creating tokens. For example, the `authenticate` method uses `UserRepository.get_by_email` to fetch a user by email.

```python
user = await UserRepository.get_by_email(self.session, email)
```

#### Benefits of Using This Pattern Here
1. **Separation of Concerns:** The repository pattern separates business logic from data access concerns, making the codebase more modular and easier to maintain.
2. **Testability:** Repositories can be easily mocked or replaced with in-memory databases for testing purposes.
3. **Flexibility:** It allows changing the underlying database system without affecting the service layer.

#### Deviations from Standard Pattern
- The `AuthService` directly interacts with multiple repositories (`UserRepository`, `RefreshTokenRepository`) rather than a single repository interface. This could be improved by defining a higher-level abstraction or using dependency injection to manage these dependencies.
- Some methods like `logout` do not return any value, which might not align perfectly with the standard pattern's expectations of clear method signatures.

#### When This Pattern is Appropriate
The repository pattern is highly appropriate in this context because:
1. The service layer needs to interact with multiple data sources (users and refresh tokens).
2. There are complex business rules around token validation and user authentication.
3. Future changes, such as switching databases or adding more repositories, can be managed without altering the core logic.

In summary, the repository pattern effectively separates concerns in `AuthService`, making it easier to manage and test various data access operations while maintaining a clean service layer.

---

*Generated by CodeWorm on 2026-02-26 11:20*
