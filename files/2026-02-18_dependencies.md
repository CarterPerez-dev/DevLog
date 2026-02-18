# dependencies

**Type:** File Overview
**Repository:** Cybersecurity-Projects
**File:** TEMPLATES/fullstack-template/backend/app/core/dependencies.py
**Language:** python
**Lines:** 1-147
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

from config import (
    API_PREFIX,
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
    tokenUrl = f"{API_PREFIX}/auth/login",
    auto_error = True,
)

oauth2_scheme_optional = OAuth2PasswordBearer(
    tokenUrl = f"{API_PREFIX}/auth/login",
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

## File Overview

### dependencies.py

**Purpose:**
This file provides core dependency injection mechanisms for the backend application, ensuring secure and efficient access control. It handles token validation, user retrieval, role checks, and client IP extraction.

**Key Exports & Public Interface:**
- `oauth2_scheme` and `oauth2_scheme_optional`: OAuth2 password bearer schemes for securing API endpoints.
- `DBSession`: Annotated dependency for asynchronous database sessions.
- `get_current_user`, `get_current_active_user`, `get_optional_user`: Functions to validate tokens, retrieve users, and handle optional user authentication.
- `RequireRole`: A dependency class to enforce specific user roles.
- `CurrentUser` and `OptionalUser`: Type annotations for the current authenticated user or an optional user.
- `get_client_ip`: Function to extract client IP addresses, considering proxy headers.

**How it Fits into the Project:**
This file is crucial for maintaining security and consistency across the application. It integrates with the FastAPI framework, the database layer, and exception handling mechanisms. By providing robust dependency injection, it ensures that user authentication and authorization are handled uniformly throughout the project.

**Notable Design Decisions:**
- **Token Validation:** The `decode_access_token` function is used to validate JWT tokens, ensuring they match expected types and versions.
- **Role-Based Access Control (RBAC):** The `RequireRole` class enforces specific roles for certain API endpoints, enhancing security.
- **Client IP Extraction:** The `get_client_ip` function considers proxy headers to accurately determine client IPs, which is useful for logging and security audits.

This file plays a central role in managing user sessions, roles, and permissions, making it an integral part of the backend's security architecture.

---

*Generated by CodeWorm on 2026-02-18 13:09*
