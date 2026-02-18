# RefreshTokenRepository

**Type:** Class Documentation
**Repository:** Cybersecurity-Projects
**File:** TEMPLATES/fullstack-template/backend/app/auth/repository.py
**Language:** python
**Lines:** 16-178
**Complexity:** 0.0

---

## Source Code

```python
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
                     revoked_at = datetime.now(UTC))
        )
        await session.flush()
        return result.rowcount or 0

    @classmethod
    async def revoke_all_user_tokens(
        cls,
        session: AsyncSession,
        user_id: UUID,
    ) -> int:
        """
        Revoke all
```

---

## Class Documentation

### RefreshTokenRepository Documentation

**Class Responsibility and Purpose:**
The `RefreshTokenRepository` class is responsible for managing database operations related to the `RefreshToken` model. It provides a structured way to interact with refresh tokens, including creating, revoking, and querying them.

**Public Interface (Key Methods):**
- **get_by_hash**: Retrieves a refresh token by its hash.
- **get_valid_by_hash**: Fetches a valid (not revoked, not expired) refresh token by hash.
- **create_token**: Creates a new refresh token for a given user.
- **revoke_token**: Revokes a single token.
- **revoke_family**: Revoke all tokens in a family to handle replay attacks.
- **revoke_all_user_tokens**: Logs out all devices associated with a user by revoking their tokens.
- **get_user_active_sessions**: Lists all active sessions for a specific user.
- **cleanup_expired**: Cleans up expired tokens during maintenance.

**Design Patterns Used:**
The class employs the **Repository Pattern**, which abstracts database access and provides a clean interface to interact with data. It also uses **CRUD (Create, Read, Update, Delete) operations** typical in repository implementations.

**How it Fits in the Architecture:**
`RefreshTokenRepository` is part of the backend's authentication module. It interacts directly with the database through SQLAlchemy sessions to manage refresh tokens. This class ensures that token-related operations are consistent and secure, supporting features like user logout, session management, and security against replay attacks. It integrates seamlessly into the broader application architecture by providing a reliable interface for handling refresh tokens across various services.

---

*Generated by CodeWorm on 2026-02-18 08:09*
