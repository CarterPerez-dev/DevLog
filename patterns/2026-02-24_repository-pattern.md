# repository_pattern

**Type:** Pattern Analysis
**Repository:** fullstack-template
**File:** stacks/fastapi-react/backend/app/auth/service.py
**Language:** python
**Lines:** 1-262
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
    MobileLoginResponse,
    MobileTokenResponse,
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

    async def refres
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used: Repository Pattern**

The `AuthService` class in the provided code implements the **Repository Pattern**, which abstracts data access operations behind a well-defined interface, making it easier to switch between different storage mechanisms.

#### Implementation:
- The `AuthService` interacts with repositories like `UserRepository` and `RefreshTokenRepository` for CRUD operations.
- Methods such as `get_by_email`, `update_password`, and `create_token` are delegated to these repositories, encapsulating the database logic.

#### Benefits:
1. **Decoupling**: The service layer is decoupled from the data access layer, making it easier to change storage strategies without affecting business logic.
2. **Testability**: Repositories can be easily mocked or replaced with in-memory databases for testing purposes.
3. **Consistency**: Ensures that all data access operations follow a consistent pattern and are handled uniformly.

#### Deviations:
- The `AuthService` class directly interacts with the session (`AsyncSession`) to manage database transactions, which is not typical of the pure Repository Pattern where repositories should handle their own transaction management or use a separate service for it.
- Some methods like `create_access_token` and `create_refresh_token` are called within the repository methods, indicating that these services might be tightly coupled with data access.

#### Appropriateness:
This pattern is appropriate in this context because it clearly separates concerns between business logic and data access. However, consider decoupling the session management from the service layer to better adhere to the Repository Pattern's principles.

By maintaining a clear separation of responsibilities, this design enhances maintainability and flexibility, making it easier to scale or change underlying storage mechanisms if needed.

---

*Generated by CodeWorm on 2026-02-24 00:33*
