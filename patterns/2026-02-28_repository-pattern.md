# repository_pattern

**Type:** Pattern Analysis
**Repository:** my-portfolio
**File:** v1/backend/app/core/dependencies.py
**Language:** python
**Lines:** 1-159
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
dependencies.py
"""

from __future__ import annotations

from typing import Annotated
from uuid import UUID

import jwt
from fastapi import Depends, Request
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession

import config
from config import (
    TokenType,
    UserRole,
)
from .database import get_db_session
from .exceptions import (
    InactiveUser,
    PermissionDenied,
    TokenError,
    TokenRevokedError,
    UserNotFound,
)
from user.User import User
from .security import decode_access_token
from user.repository import UserRepository


oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl = f"{config.API_PREFIX}/auth/login",
    auto_error = True,
)

oauth2_scheme_optional = OAuth2PasswordBearer(
    tokenUrl = f"{config.API_PREFIX}/auth/login",
    auto_error = False,
)

DBSession = Annotated[AsyncSession, Depends(get_db_session)]


async def get_current_user(
    token: Annotated[str,
                     Depends(oauth2_scheme)],
    db: DBSession,
) -> User:
    """
    Validate access token and return current user
    """
    try:
        payload = decode_access_token(token)
    except jwt.InvalidTokenError as e:
        raise TokenError(message = str(e)) from e

    if payload.get("type") != TokenType.ACCESS.value:
        raise TokenError(message = "Invalid token type")

    user_id = UUID(payload["sub"])
    user = await UserRepository.get_by_id(db, user_id)

    if user is None:
        raise UserNotFound(identifier = str(user_id))

    if payload.get("token_version") != user.token_version:
        raise TokenRevokedError()

    return user


async def get_current_active_user(
    user: Annotated[User,
                    Depends(get_current_user)],
) -> User:
    """
    Ensure user is active
    """
    if not user.is_active:
        raise InactiveUser()
    return user


async def get_optional_user(
    token: Annotated[str | None,
                     Depends(oauth2_scheme_optional)],
    db: DBSession,
) -> User | None:
    """
    Return current user if authenticated, None otherwise
    """
    if token is None:
        return None

    try:
        payload = decode_access_token(token)
        if payload.get("type") != TokenType.ACCESS.value:
            return None
        user_id = UUID(payload["sub"])
        user = await UserRepository.get_by_id(db, user_id)
        if user and user.token_version == payload.get("token_version"):
            return user
    except (jwt.InvalidTokenError, ValueError):
        pass

    return None


class RequireRole:
    """
    Dependency class to check user role
    """
    def __init__(self, *allowed_roles: UserRole) -> None:
        self.allowed_roles = allowed_roles

    async def __call__(
        self,
        user: Annotated[User,
                        Depends(get_current_active_user)],
    ) -> User:
        if user.role not in self.allowed_roles:
            raise PermissionDenied(
                message =
   
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Dependency Injection with Repository Pattern

#### Implementation in Code
The code implements a dependency injection framework using FastAPI's `Depends` mechanism and integrates it with a repository pattern for database operations. The `get_current_user`, `get_current_active_user`, and `get_optional_user` functions are used to manage authentication and authorization, while the `RequireRole` class ensures that users have specific roles.

#### Benefits
1. **Modularity:** Functions like `get_current_user` and `get_current_active_user` encapsulate authentication logic, making the code easier to maintain.
2. **Reusability:** These functions can be reused across different endpoints without duplicating code.
3. **Testability:** Dependency injection makes it easier to mock dependencies during testing.

#### Deviations
1. **Custom Annotations:** The use of custom annotations like `CurrentUser`, `OptionalUser`, and `ClientIP` is a deviation from the standard pattern, providing more type safety and clarity in function parameters.
2. **Role Checking:** The `RequireRole` class adds an additional layer of role-based access control, which is not part of the traditional repository pattern but enhances security.

#### Appropriateness
This pattern is highly appropriate for web applications that require robust authentication and authorization mechanisms. It ensures that sensitive operations are protected and that dependencies are managed effectively. However, it might be overkill for simpler applications where basic dependency injection suffices.

By using this pattern, the code becomes more organized, maintainable, and secure, making it a suitable choice for complex web applications like the one described.

---

*Generated by CodeWorm on 2026-02-28 01:36*
