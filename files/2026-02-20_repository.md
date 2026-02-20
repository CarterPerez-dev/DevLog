# repository

**Type:** File Overview
**Repository:** my-portfolio
**File:** v1/backend/app/project/repository.py
**Language:** python
**Lines:** 1-120
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
repository.py
"""

from collections.abc import Sequence

from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession

import config
from config import Language
from core.base_repository import BaseRepository

from .Project import Project


class ProjectRepository(BaseRepository[Project]):
    """
    Repository for Project model database operations
    """
    model = Project

    @classmethod
    async def get_by_slug_and_language(
        cls,
        session: AsyncSession,
        slug: str,
        language: Language,
    ) -> Project | None:
        """
        Get a single project by slug and language
        Primary lookup for public project pages
        """
        result = await session.execute(
            select(Project).where(Project.slug == slug
                                  ).where(Project.language == language)
        )
        return result.scalars().first()

    @classmethod
    async def get_visible_by_language(
        cls,
        session: AsyncSession,
        language: Language,
        skip: int = config.PAGINATION_DEFAULT_SKIP,
        limit: int = config.PAGINATION_DEFAULT_LIMIT,
    ) -> Sequence[Project]:
        """
        Get all visible (complete) projects for a language
        Ordered by display_order for consistent nav/listing
        """
        result = await session.execute(
            select(Project).where(Project.language == language).order_by(
                Project.display_order
            ).offset(skip).limit(limit)
        )
        return result.scalars().all()

    @classmethod
    async def get_featured_by_language(
        cls,
        session: AsyncSession,
        language: Language,
        limit: int = config.PAGINATION_FEATURED_LIMIT,
    ) -> Sequence[Project]:
        """
        Get featured projects for a language.
        Used for overview page highlights.
        """
        result = await session.execute(
            select(Project).where(Project.language == language).where(
                Project.is_complete == True
            ).where(Project.is_featured == True).order_by(
                Project.display_order
            ).limit(limit)
        )
        return result.scalars().all()

    @classmethod
    async def count_visible_by_language(
        cls,
        session: AsyncSession,
        language: Language,
    ) -> int:
        """
        Count visible projects for pagination
        """
        result = await session.execute(
            select(func.count()
                   ).select_from(Project).where(Project.language == language)
        )
        return result.scalar_one()

    @classmethod
    async def get_all_slugs(
        cls,
        session: AsyncSession,
    ) -> Sequence[str]:
        """
        Get all unique project slugs.
        Useful for generating nav or validating slugs
        """
        result = await session.execute(select(Project.slug).distinct())
        return result.scalars().all()

    @classm
```

---

## File Overview

### Purpose and Responsibility

This file, `repository.py`, is responsible for defining a repository class to manage database operations related to the `Project` model. It provides methods to retrieve projects based on various criteria such as slug, language, visibility status, and featured status.

### Key Exports and Public Interface

- **Class**: `ProjectRepository`
  - **Methods**:
    - `get_by_slug_and_language`: Fetches a single project by its slug and language.
    - `get_visible_by_language`: Retrieves all visible projects for a given language.
    - `get_featured_by_language`: Fetches featured projects for a specific language.
    - `count_visible_by_language`: Counts the number of visible projects for pagination.
    - `get_all_slugs`: Returns all unique project slugs.
    - `slug_exists`: Checks if a project slug exists.

### How It Fits in the Project

`ProjectRepository` is part of the data access layer, specifically designed to interact with the database. It adheres to the repository pattern by encapsulating database operations and providing a clean interface for other parts of the application to use. This class interacts directly with `BaseRepository`, inheriting common functionality while implementing specific methods tailored to project management.

### Notable Design Decisions

- **Type Hints**: The use of type hints ensures that method parameters and return types are explicitly defined, enhancing code readability and maintainability.
- **Async Operations**: All methods are asynchronous, leveraging SQLAlchemy’s `AsyncSession` for database operations, which is crucial for handling concurrent requests efficiently.
- **Pagination Support**: Methods like `get_visible_by_language` and `count_visible_by_language` support pagination through configurable `skip` and `limit` parameters, ensuring efficient data retrieval even with large datasets.
- **SQLAlchemy ORM**: The repository uses SQLAlchemy’s ORM to interact with the database, providing a flexible and powerful way to manage project data.

---

*Generated by CodeWorm on 2026-02-20 08:12*
