# repository

**Type:** File Overview
**Repository:** my-portfolio
**File:** v1/backend/app/user/repository.py
**Language:** python
**Lines:** 1-110
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
repository.py
"""
from uuid import UUID

from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from config import UserRole
from .User import User
from core.base_repository import BaseRepository


class UserRepository(BaseRepository[User]):
    """
    Repository for User model database operations
    """
    model = User

    @classmethod
    async def get_by_email(
        cls,
        session: AsyncSession,
        email: str,
    ) -> User | None:
        """
        Get user by email address
        """
        result = await session.execute(select(User).where(User.email == email))
        return result.scalars().first()

    @classmethod
    async def get_by_id(
        cls,
        session: AsyncSession,
        id: UUID,
    ) -> User | None:
        """
        Get user by ID
        """
        return await session.get(User, id)

    @classmethod
    async def email_exists(
        cls,
        session: AsyncSession,
        email: str,
    ) -> bool:
        """
        Check if email is already registered
        """
        result = await session.execute(
            select(User.id).where(User.email == email)
        )
        return result.scalars().first() is not None

    @classmethod
    async def create_user(
        cls,
        session: AsyncSession,
        email: str,
        hashed_password: str,
        full_name: str | None = None,
        role: UserRole = UserRole.USER,
    ) -> User:
        """
        Create a new user
        """
        user = User(
            email = email,
            hashed_password = hashed_password,
            full_name = full_name,
            role = role,
        )
        session.add(user)
        await session.flush()
        await session.refresh(user)
        return user

    @classmethod
    async def update_password(
        cls,
        session: AsyncSession,
        user: User,
        hashed_password: str,
    ) -> User:
        """
        Update user password and increment token version
        """
        user.hashed_password = hashed_password
        user.increment_token_version()
        await session.flush()
        await session.refresh(user)
        return user

    @classmethod
    async def increment_token_version(
        cls,
        session: AsyncSession,
        user: User,
    ) -> User:
        """
        Invalidate all user tokens
        """
        user.increment_token_version()
        await session.flush()
        await session.refresh(user)
        return user

```

---

## File Overview

### Purpose and Responsibility

This file, `repository.py`, is responsible for defining a repository class that encapsulates database operations for the `User` model. It provides methods to interact with user data such as retrieving users by email or ID, checking if an email is already registered, creating new users, updating passwords, and incrementing token versions.

### Key Exports and Public Interface

- **BaseRepository**: Inherits from this base class.
- **get_by_email(session: AsyncSession, email: str) -> User | None**: Retrieves a user by their email address.
- **get_by_id(session: AsyncSession, id: UUID) -> User | None**: Retrieves a user by their unique ID.
- **email_exists(session: AsyncSession, email: str) -> bool**: Checks if an email is already registered in the database.
- **create_user(session: AsyncSession, email: str, hashed_password: str, full_name: str | None = None, role: UserRole = UserRole.USER) -> User**: Creates a new user with specified details.
- **update_password(session: AsyncSession, user: User, hashed_password: str) -> User**: Updates the password and increments the token version for an existing user.
- **increment_token_version(session: AsyncSession, user: User) -> User**: Invalidates all tokens associated with a user by incrementing their token version.

### How It Fits into the Project

This repository class is part of the `User` module in the broader project structure. It adheres to the Single Responsibility Principle by focusing solely on database operations related to users, making it easier to manage and test these functionalities. The methods are designed to be reusable across different parts of the application, ensuring consistency in user data handling.

### Notable Design Decisions

- **Use of SQLAlchemy**: Leverages SQLAlchemy for asynchronous database interactions, providing a robust and efficient way to handle CRUD operations.
- **Type Hints and Annotations**: Utilizes Python's type hints and annotations to ensure clear and maintainable code. This is particularly useful in an async context where session management can be complex.
- **Inheritance from BaseRepository**: Inherits from `BaseRepository` to standardize common repository functionalities, promoting DRY principles and reducing redundancy.

---

*Generated by CodeWorm on 2026-02-20 08:37*
