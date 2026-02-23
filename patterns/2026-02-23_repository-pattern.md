# repository_pattern

**Type:** Pattern Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/bug-bounty-platform/backend/app/program/repository.py
**Language:** python
**Lines:** 1-233
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
repository.py
"""

from collections.abc import Sequence
from uuid import UUID

from sqlalchemy import func, select
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.orm import selectinload

from config import ProgramStatus, ProgramVisibility
from core.base_repository import BaseRepository
from .Program import Program
from .Asset import Asset
from .RewardTier import RewardTier


class ProgramRepository(BaseRepository[Program]):
    """
    Repository for Program model database operations
    """
    model = Program

    @classmethod
    async def get_by_slug(
        cls,
        session: AsyncSession,
        slug: str,
    ) -> Program | None:
        """
        Get program by slug
        """
        result = await session.execute(
            select(Program).where(Program.slug == slug)
        )
        return result.scalars().first()

    @classmethod
    async def get_by_slug_with_details(
        cls,
        session: AsyncSession,
        slug: str,
    ) -> Program | None:
        """
        Get program by slug with assets and reward tiers
        """
        result = await session.execute(
            select(Program).where(Program.slug == slug).options(
                selectinload(Program.assets),
                selectinload(Program.reward_tiers),
            )
        )
        return result.scalars().first()

    @classmethod
    async def get_by_id_with_details(
        cls,
        session: AsyncSession,
        program_id: UUID,
    ) -> Program | None:
        """
        Get program by ID with assets and reward tiers
        """
        result = await session.execute(
            select(Program).where(Program.id == program_id).options(
                selectinload(Program.assets),
                selectinload(Program.reward_tiers),
            )
        )
        return result.scalars().first()

    @classmethod
    async def slug_exists(
        cls,
        session: AsyncSession,
        slug: str,
    ) -> bool:
        """
        Check if slug is already taken
        """
        result = await session.execute(
            select(Program.id).where(Program.slug == slug)
        )
        return result.scalars().first() is not None

    @classmethod
    async def get_public_programs(
        cls,
        session: AsyncSession,
        skip: int = 0,
        limit: int = 20,
    ) -> Sequence[Program]:
        """
        Get active public programs
        """
        result = await session.execute(
            select(Program).where(
                Program.status == ProgramStatus.ACTIVE,
                Program.visibility == ProgramVisibility.PUBLIC,
            ).order_by(Program.created_at.desc()
                       ).offset(skip).limit(limit)
        )
        return result.scalars().all()

    @classmethod
    async def count_public_programs(
        cls,
        session: AsyncSession,
    ) -> int:
        """
        Count active public programs
        """
        result = await
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Repository Pattern

#### Implementation in Code
The `ProgramRepository`, `AssetRepository`, and `RewardTierRepository` classes are implementations of the repository pattern. Each class inherits from a base repository (`BaseRepository`) and provides methods for database operations specific to their respective models (e.g., `Program`, `Asset`, `RewardTier`). These methods include fetching entities by ID or slug, checking existence, getting public programs, counting records, and more.

#### Benefits
1. **Encapsulation**: The pattern encapsulates all data access logic within the repository classes, making it easier to switch between different database systems.
2. **Separation of Concerns**: It separates business logic from data access logic, improving code maintainability and testability.
3. **Consistency**: Ensures a consistent approach to handling CRUD operations across multiple models.

#### Deviations
- The `BaseRepository` is not explicitly defined in the provided snippet, but it likely abstracts common repository functionalities like session management or basic CRUD operations.
- Some methods (e.g., `get_by_company`, `count_by_company`) are specific to certain conditions and might require additional logic depending on the context.

#### Appropriateness
This pattern is highly appropriate for this scenario because:
1. **Complex Queries**: The code handles complex queries involving multiple joins and filters, which are common in database operations.
2. **Scalability**: It can easily accommodate more models or operations as the application grows.
3. **Maintenance**: By centralizing data access logic, it simplifies maintenance and reduces the risk of introducing bugs.

Overall, the repository pattern is well-suited for managing database interactions in a structured and scalable manner.

---

*Generated by CodeWorm on 2026-02-23 07:41*
