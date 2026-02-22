# JobApplicationRepository

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/life_manager/facets/career/job_app_tracker/repository.py
**Language:** python
**Lines:** 20-177
**Complexity:** 0.0

---

## Source Code

```python
class JobApplicationRepository(BaseRepository[JobApplication]):
    """
    Repository for JobApplication operations
    """
    model = JobApplication

    @classmethod
    async def get_by_user(
        cls,
        session: AsyncSession,
        user_id: UUID,
        skip: int = 0,
        limit: int = 50,
    ) -> Sequence[JobApplication]:
        """
        Get all job applications for a user with pagination
        """
        result = await session.execute(
            select(JobApplication).where(
                JobApplication.user_id == user_id
            ).order_by(JobApplication.created_at.desc()
                       ).offset(skip).limit(limit)
        )
        return result.scalars().all()

    @classmethod
    async def count_by_user(
        cls,
        session: AsyncSession,
        user_id: UUID,
    ) -> int:
        """
        Count total job applications for a user
        """
        result = await session.execute(
            select(func.count()).select_from(JobApplication).where(
                JobApplication.user_id == user_id
            )
        )
        return result.scalar_one()

    @classmethod
    async def get_by_status(
        cls,
        session: AsyncSession,
        user_id: UUID,
        status: ApplicationStatus,
    ) -> Sequence[JobApplication]:
        """
        Get job applications by status
        """
        result = await session.execute(
            select(JobApplication).where(
                JobApplication.user_id == user_id,
                JobApplication.application_status == status,
            ).order_by(JobApplication.created_at.desc())
        )
        return result.scalars().all()

    @classmethod
    async def get_by_outcome(
        cls,
        session: AsyncSession,
        user_id: UUID,
        outcome: Outcome,
    ) -> Sequence[JobApplication]:
        """
        Get job applications by outcome
        """
        result = await session.execute(
            select(JobApplication).where(
                JobApplication.user_id == user_id,
                JobApplication.outcome == outcome,
            ).order_by(JobApplication.created_at.desc())
        )
        return result.scalars().all()

    @classmethod
    async def get_pending_followups(
        cls,
        session: AsyncSession,
        user_id: UUID,
    ) -> Sequence[JobApplication]:
        """
        Get applications with followup dates that need attention
        """
        result = await session.execute(
            select(JobApplication).where(
                JobApplication.user_id == user_id,
                JobApplication.followup_date.isnot(None),
                JobApplication.outcome == Outcome.PENDING,
            ).order_by(JobApplication.followup_date.asc())
        )
        return result.scalars().all()

    @classmethod
    async def get_stats(
        cls,
        session: AsyncSession,
        user_id: UUID,
    ) -> dict:
        """
        Get aggregated stats for a user's job appl
```

---

## Class Documentation

### JobApplicationRepository Documentation

**Class Responsibility and Purpose:**
The `JobApplicationRepository` class is responsible for managing database operations related to job applications, including retrieval, counting, filtering by status or outcome, and generating aggregated statistics. This class ensures that all job application-related queries are encapsulated within a single repository, promoting separation of concerns and making the codebase more maintainable.

**Public Interface:**
- `get_by_user`: Retrieves paginated job applications for a specific user.
- `count_by_user`: Counts total job applications for a specific user.
- `get_by_status`: Fetches job applications by status.
- `get_by_outcome`: Fetches job applications by outcome.
- `get_pending_followups`: Retrieves applications with follow-up dates that need attention.
- `get_stats`: Generates aggregated statistics for a user's job applications.

**Design Patterns Used:**
The class utilizes the **Repository Pattern**, which abstracts data access and provides a consistent interface for interacting with the database. This pattern is particularly useful in managing complex queries and ensuring that the business logic remains decoupled from the persistence layer.

**How it Fits in the Architecture:**
`JobApplicationRepository` acts as a central hub for job application operations, facilitating easy integration with other parts of the system such as user management or analytics services. By encapsulating database interactions within this repository, the code becomes more modular and easier to test. The class also adheres to Pythonic conventions, using type hints and asynchronous methods to ensure efficient and clean data handling.

---

*Generated by CodeWorm on 2026-02-22 02:29*
