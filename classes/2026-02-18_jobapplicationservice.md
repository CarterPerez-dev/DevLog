# JobApplicationService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/life_manager/facets/career/job_app_tracker/service.py
**Language:** python
**Lines:** 34-186
**Complexity:** 0.0

---

## Source Code

```python
class JobApplicationService:
    """
    Service for job application operations
    """

    @staticmethod
    async def get_application(
        session: AsyncSession,
        user_id: UUID,
        application_id: UUID,
    ) -> JobApplicationResponse:
        """
        Get a single job application by ID
        """
        application = await JobApplicationRepository.get_by_id(session, application_id)
        if not application:
            raise JobApplicationNotFound(application_id)
        if application.user_id != user_id:
            raise PermissionDenied()
        return JobApplicationResponse.model_validate(application)

    @staticmethod
    async def get_applications(
        session: AsyncSession,
        user_id: UUID,
        skip: int = 0,
        limit: int = 50,
    ) -> JobApplicationListResponse:
        """
        Get all job applications for a user
        """
        applications = await JobApplicationRepository.get_by_user(
            session, user_id, skip, limit
        )
        total = await JobApplicationRepository.count_by_user(session, user_id)
        return JobApplicationListResponse(
            items=[JobApplicationResponse.model_validate(a) for a in applications],
            total=total,
        )

    @staticmethod
    async def get_by_status(
        session: AsyncSession,
        user_id: UUID,
        status: ApplicationStatus,
    ) -> JobApplicationListResponse:
        """
        Get job applications filtered by status
        """
        applications = await JobApplicationRepository.get_by_status(
            session, user_id, status
        )
        return JobApplicationListResponse(
            items=[JobApplicationResponse.model_validate(a) for a in applications],
            total=len(applications),
        )

    @staticmethod
    async def get_by_outcome(
        session: AsyncSession,
        user_id: UUID,
        outcome: Outcome,
    ) -> JobApplicationListResponse:
        """
        Get job applications filtered by outcome
        """
        applications = await JobApplicationRepository.get_by_outcome(
            session, user_id, outcome
        )
        return JobApplicationListResponse(
            items=[JobApplicationResponse.model_validate(a) for a in applications],
            total=len(applications),
        )

    @staticmethod
    async def get_pending_followups(
        session: AsyncSession,
        user_id: UUID,
    ) -> JobApplicationListResponse:
        """
        Get applications needing follow-up
        """
        applications = await JobApplicationRepository.get_pending_followups(
            session, user_id
        )
        return JobApplicationListResponse(
            items=[JobApplicationResponse.model_validate(a) for a in applications],
            total=len(applications),
        )

    @staticmethod
    async def create_application(
        session: AsyncSession,
        user_id: UUID,
        data: JobApplicationCreate,
    ) -> JobApplicationResp
```

---

## Class Documentation

### JobApplicationService Documentation

**Class Responsibility and Purpose:**
The `JobApplicationService` class is responsible for managing job application operations, including retrieval, creation, updating, deletion, and statistics generation. It ensures that only authorized users can perform actions on their own applications.

**Public Interface (Key Methods):**
- **`get_application`: Retrieves a single job application by ID.**
- **`get_applications`: Fetches all job applications for a user with optional pagination.**
- **`get_by_status`: Filters applications based on status.**
- **`get_by_outcome`: Filters applications based on outcome.**
- **`get_pending_followups`: Retrieves applications needing follow-up actions.**
- **`create_application`: Creates a new job application.**
- **`update_application`: Updates an existing job application.**
- **`delete_application`: Deletes a job application.**
- **`get_stats`: Generates aggregated statistics for job applications.**

**Design Patterns Used:**
The class primarily uses the **Factory Method Pattern** through static methods to encapsulate the creation and management of job applications. It also employs validation checks to ensure data integrity and security.

**How it Fits in the Architecture:**
`JobApplicationService` acts as a service layer, providing business logic for job application operations. It interacts with `JobApplicationRepository`, which handles database interactions. This separation ensures that the repository remains agnostic of specific use cases, promoting clean architecture principles. The service also enforces authorization and validation rules, ensuring secure and controlled access to data.

---

*Generated by CodeWorm on 2026-02-18 16:47*
