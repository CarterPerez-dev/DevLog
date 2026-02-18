# repository

**Type:** File Overview
**Repository:** Cybersecurity-Projects
**File:** TEMPLATES/fullstack-template/backend/app/auth/repository.py
**Language:** python
**Lines:** 1-179
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
repository.py
"""

from uuid import UUID
from datetime import UTC, datetime

from sqlalchemy import select, update
from sqlalchemy.ext.asyncio import AsyncSession

from .RefreshToken import RefreshToken
from core.base_repository import BaseRepository


class RefreshTokenRepository(BaseRepository[RefreshToken]):
    """
    Repository for RefreshToken model database operations
    """
    model = RefreshToken

    @classmethod
    async def get_by_hash(
        cls,
        session: AsyncSession,
        token_hash: str,
    ) -> RefreshToken | None:
        """
        Get refresh token by its hash
        """
        result = await session.execute(
            select(RefreshToken).where(
                RefreshToken.token_hash == token_hash
            )
        )
        return result.scalars().first()

    @classmethod
    async def get_valid_by_hash(
        cls,
        session: AsyncSession,
        token_hash: str,
    ) -> RefreshToken | None:
        """
        Get valid (not revoked, not expired) refresh token by hash
        """
        result = await session.execute(
            select(RefreshToken).where(
                RefreshToken.token_hash == token_hash,
                RefreshToken.is_revoked == False,
                RefreshToken.expires_at > datetime.now(UTC),
            )
        )
        return result.scalars().first()

    @classmethod
    async def create_token(
        cls,
        session: AsyncSession,
        user_id: UUID,
        token_hash: str,
        family_id: UUID,
        expires_at: datetime,
        device_id: str | None = None,
        device_name: str | None = None,
        ip_address: str | None = None,
    ) -> RefreshToken:
        """
        Create a new refresh token
        """
        token = RefreshToken(
            user_id = user_id,
            token_hash = token_hash,
            family_id = family_id,
            expires_at = expires_at,
            device_id = device_id,
            device_name = device_name,
            ip_address = ip_address,
        )
        session.add(token)
        await session.flush()
        await session.refresh(token)
        return token

    @classmethod
    async def revoke_token(
        cls,
        session: AsyncSession,
        token: RefreshToken,
    ) -> RefreshToken:
        """
        Revoke a single token
        """
        token.revoke()
        await session.flush()
        await session.refresh(token)
        return token

    @classmethod
    async def revoke_family(
        cls,
        session: AsyncSession,
        family_id: UUID,
    ) -> int:
        """
        Revoke all tokens in a family (for replay attack response)

        Returns count of revoked tokens
        """
        result = await session.execute(
            update(RefreshToken).where(
                RefreshToken.family_id == family_id,
                RefreshToken.is_revoked == False,
            ).values(is_revoked = True,
                     
```

---

## File Overview

### repository.py

**Purpose and Responsibility:**
This Python file defines a `RefreshTokenRepository` class, which is part of the authentication module in a full-stack application. The repository handles database operations for managing refresh tokens, including creation, revocation, validation, and cleanup.

**Key Exports or Public Interface:**
- `get_by_hash`: Retrieves a refresh token by its hash.
- `get_valid_by_hash`: Fetches valid (not revoked, not expired) refresh tokens by hash.
- `create_token`: Creates a new refresh token for a user.
- `revoke_token`: Revokes a single token.
- `revoke_family`: Revoke all tokens in a family to mitigate replay attacks.
- `revoke_all_user_tokens`: Logs out all devices associated with a user.
- `get_user_active_sessions`: Lists active sessions for a specific user.
- `cleanup_expired`: Deletes expired tokens during maintenance.

**How It Fits into the Project:**
This repository fits into the larger project by providing essential database operations for managing refresh tokens. These operations are crucial for maintaining session security and ensuring that only valid, non-expired tokens can be used to access protected resources.

**Notable Design Decisions:**
- The use of SQLAlchemy for ORM-based database interactions.
- Asynchronous methods using `AsyncSession` for handling database transactions.
- Validation checks in methods like `get_valid_by_hash` ensure that only active tokens are returned.
- Batch operations in `revoke_family` and `revoke_all_user_tokens` to efficiently handle multiple token revocations.
```

This documentation provides an overview of the file's purpose, key functionalities, integration within the project, and significant design choices.

---

*Generated by CodeWorm on 2026-02-18 10:52*
