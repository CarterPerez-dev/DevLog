# repository_pattern

**Type:** Pattern Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/bug-bounty-platform/backend/app/program/service.py
**Language:** python
**Lines:** 1-376
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
service.py
"""

from uuid import UUID

from sqlalchemy.ext.asyncio import AsyncSession

from core.exceptions import (
    AssetNotFound,
    NotProgramOwner,
    ProgramNotFound,
    SlugAlreadyExists,
)
from user.User import User
from .schemas import (
    AssetCreate,
    AssetResponse,
    AssetUpdate,
    ProgramCreate,
    ProgramDetailResponse,
    ProgramListResponse,
    ProgramResponse,
    ProgramUpdate,
    RewardTierCreate,
    RewardTierResponse,
)
from .repository import (
    AssetRepository,
    ProgramRepository,
    RewardTierRepository,
)


class ProgramService:
    """
    Business logic for program operations
    """
    def __init__(self, session: AsyncSession) -> None:
        self.session = session

    async def create_program(
        self,
        user: User,
        program_data: ProgramCreate,
    ) -> ProgramResponse:
        """
        Create a new bug bounty program
        """
        if await ProgramRepository.slug_exists(self.session,
                                               program_data.slug):
            raise SlugAlreadyExists(program_data.slug)

        program = await ProgramRepository.create(
            self.session,
            company_id = user.id,
            name = program_data.name,
            slug = program_data.slug,
            description = program_data.description,
            rules = program_data.rules,
            response_sla_hours = program_data.response_sla_hours,
            visibility = program_data.visibility,
        )
        return ProgramResponse.model_validate(program)

    async def get_program_by_slug(
        self,
        slug: str,
    ) -> ProgramDetailResponse:
        """
        Get program by slug with full details
        """
        program = await ProgramRepository.get_by_slug_with_details(
            self.session,
            slug
        )
        if not program:
            raise ProgramNotFound(slug)

        return ProgramDetailResponse(
            id = program.id,
            created_at = program.created_at,
            updated_at = program.updated_at,
            company_id = program.company_id,
            name = program.name,
            slug = program.slug,
            description = program.description,
            rules = program.rules,
            response_sla_hours = program.response_sla_hours,
            status = program.status,
            visibility = program.visibility,
            assets = [
                AssetResponse.model_validate(a) for a in program.assets
            ],
            reward_tiers = [
                RewardTierResponse.model_validate(r)
                for r in program.reward_tiers
            ],
        )

    async def get_program_by_id(
        self,
        program_id: UUID,
    ) -> ProgramResponse:
        """
        Get program by ID
        """
        program = await ProgramRepository.get_by_id(
            self.session,
            program_id
        )
        if not program:
            
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used: Repository Pattern**

The `ProgramService` class in the provided code implements the **Repository Pattern**, which encapsulates data access logic and abstracts it from the business logic. This is evident through the use of `ProgramRepository`, `AssetRepository`, and `RewardTierRepository` to handle database operations.

- **Implementation**: The service methods delegate data retrieval and manipulation tasks to repository methods, such as `create_program`, `get_program_by_slug`, and `list_public_programs`. For example, `create_program` checks for slug uniqueness using `ProgramRepository.slug_exists` and creates a new program with `ProgramRepository.create`.

- **Benefits**: 
  - **Separation of Concerns**: The service layer focuses on business logic while the repository handles data access.
  - **Testability**: Repository methods can be easily mocked or replaced, making unit testing more straightforward.

- **Deviations**:
  - The pattern is slightly modified as the service class directly interacts with the session object. Typically, a dedicated repository class would handle sessions and transactions.
  - Some business logic (e.g., checking if the user owns the program) is implemented in the service layer rather than the repository.

- **Appropriateness**:
  - This pattern is appropriate for applications where data access logic needs to be abstracted from business rules. It enhances maintainability and testability by clearly separating concerns.
  - However, the direct session handling could be refactored into a more generic repository class that manages sessions internally.

This analysis highlights how the Repository Pattern is effectively used in this code snippet while noting some potential improvements for better adherence to pattern standards.

---

*Generated by CodeWorm on 2026-02-19 15:39*
