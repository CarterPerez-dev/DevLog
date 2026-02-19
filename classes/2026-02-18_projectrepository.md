# ProjectRepository

**Type:** Class Documentation
**Repository:** my-portfolio
**File:** v1/backend/app/project/repository.py
**Language:** python
**Lines:** 18-119
**Complexity:** 0.0

---

## Source Code

```python
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

    @classmethod
    async def slug_exists(
        cls,
        session: AsyncSession,
        slug: str,
    ) -> bool:
        """
        Check if a project slug exists
        """
        result = await session.execute(
            select(Project.id).where(Project.slug == slug).limit(1)
    
```

---

## Class Documentation

### ProjectRepository Documentation

**Class Responsibility and Purpose:**
The `ProjectRepository` class is responsible for handling database operations related to the `Project` model. It provides a clean, abstracted layer over SQLAlchemy queries, ensuring that all project-related data access logic is centralized.

**Public Interface (Key Methods):**
- **`get_by_slug_and_language`:** Retrieves a single project by its slug and language.
- **`get_visible_by_language`:** Fetches all visible projects for a specific language, ordered by `display_order`.
- **`get_featured_by_language`:** Retrieves featured projects for a given language, limited to a specified number.
- **`count_visible_by_language`:** Counts the total number of visible projects for pagination purposes.
- **`get_all_slugs`:** Returns all unique project slugs, useful for generating navigation or validating slugs.
- **`slug_exists`:** Checks if a project slug is already in use.

**Design Patterns Used:**
The class utilizes the **Repository Pattern**, which encapsulates data access logic and abstracts it from the business logic. This pattern ensures that the repository can be easily replaced with different implementations, such as an in-memory or external database backend.

**How It Fits in the Architecture:**
`ProjectRepository` is a core component of the application's data layer. It interacts directly with the database through SQLAlchemy sessions and provides methods for common CRUD operations tailored to project management. By centralizing these operations, it enhances maintainability and testability while decoupling business logic from data access concerns. This class also supports pagination and filtering, making it suitable for various use cases such as public project pages, featured projects, and admin interfaces.

---

*Generated by CodeWorm on 2026-02-18 19:48*
