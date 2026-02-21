# repository

**Type:** File Overview
**Repository:** social-media-notes
**File:** backend/app/video/repository.py
**Language:** python
**Lines:** 1-124
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
repository.py
"""

from uuid import UUID
from collections.abc import Sequence

from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from config import Platform
from core.base_repository import BaseRepository
from .VideoEntry import VideoEntry


class VideoEntryRepository(BaseRepository[VideoEntry]):
    """
    Repository for VideoEntry model database operations
    """
    model = VideoEntry

    @classmethod
    async def get_by_user(
        cls,
        session: AsyncSession,
        user_id: UUID,
        skip: int = 0,
        limit: int = 100,
    ) -> Sequence[VideoEntry]:
        """
        Get all video entries for a user
        """
        result = await session.execute(
            select(VideoEntry)
            .where(VideoEntry.user_id == user_id)
            .order_by(VideoEntry.created_at.desc())
            .offset(skip)
            .limit(limit)
        )
        return result.scalars().all()

    @classmethod
    async def get_by_user_and_platform(
        cls,
        session: AsyncSession,
        user_id: UUID,
        platform: Platform,
        skip: int = 0,
        limit: int = 100,
    ) -> Sequence[VideoEntry]:
        """
        Get video entries for a user filtered by platform
        """
        result = await session.execute(
            select(VideoEntry)
            .where(
                VideoEntry.user_id == user_id,
                VideoEntry.platform == platform,
            )
            .order_by(VideoEntry.video_number.asc())
            .offset(skip)
            .limit(limit)
        )
        return result.scalars().all()

    @classmethod
    async def count_by_user(
        cls,
        session: AsyncSession,
        user_id: UUID,
        platform: Platform | None = None,
    ) -> int:
        """
        Count video entries for a user, optionally filtered by platform
        """
        from sqlalchemy import func
        query = select(func.count()).select_from(VideoEntry).where(
            VideoEntry.user_id == user_id
        )
        if platform:
            query = query.where(VideoEntry.platform == platform)
        result = await session.execute(query)
        return result.scalar_one()

    @classmethod
    async def get_next_video_number(
        cls,
        session: AsyncSession,
        user_id: UUID,
        platform: Platform,
    ) -> int:
        """
        Get the next available video number for a platform
        """
        from sqlalchemy import func
        result = await session.execute(
            select(func.max(VideoEntry.video_number))
            .where(
                VideoEntry.user_id == user_id,
                VideoEntry.platform == platform,
            )
        )
        max_num = result.scalar_one_or_none()
        return (max_num or 0) + 1

    @classmethod
    async def get_by_id_and_user(
        cls,
        session: AsyncSession,
        entry_id: UUID,
        user_id: UUID,
    ) -> VideoEntry | None:

```

---

## File Overview

### Purpose and Responsibility

This file, `repository.py`, is responsible for defining a repository class to manage database operations related to `VideoEntry` objects. It provides methods to retrieve video entries based on user IDs and platforms, count entries, determine the next available video number, and ensure that retrieved entries belong to specific users.

### Key Exports and Public Interface

- **Class: VideoEntryRepository**
  - **Methods**:
    - `get_by_user`: Fetches all video entries for a given user.
    - `get_by_user_and_platform`: Filters video entries by both user ID and platform.
    - `count_by_user`: Counts the number of video entries for a user, optionally filtered by platform.
    - `get_next_video_number`: Determines the next available video number for a specific platform.
    - `get_by_id_and_user`: Retrieves a video entry by its ID, ensuring it belongs to the specified user.

### How It Fits into the Project

This repository class is part of the broader data access layer in the `social-media-notes` project. It interacts with the database through SQLAlchemy and provides a clean interface for other parts of the application to perform common CRUD operations on video entries. By centralizing these operations, it ensures consistency and encapsulates database logic.

### Notable Design Decisions

- **Use of SQLAlchemy**: The repository leverages SQLAlchemy's `AsyncSession` for asynchronous database interactions, ensuring non-blocking operations.
- **Type Hints and Annotations**: Type hints are used extensively to improve code readability and maintainability. For example, method parameters like `user_id: UUID` and return types like `Sequence[VideoEntry]` provide clear expectations about the data being handled.
- **Platform Enum**: The use of an enumeration (`Platform`) for filtering video entries ensures that platform values are consistent and easily manageable within the codebase.

---

*Generated by CodeWorm on 2026-02-20 22:15*
