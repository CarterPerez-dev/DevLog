# repository_pattern

**Type:** Pattern Analysis
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/life_manager/facets/checklist/repository.py
**Language:** python
**Lines:** 1-138
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2026
repository.py
"""

from collections.abc import Sequence
from datetime import date
from uuid import UUID

import sqlalchemy as sa
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession

from core.foundation.repositories.base import BaseRepository
from aspects.life_manager.facets.checklist.models import ChecklistItem, ChecklistLog


class ChecklistItemRepository(BaseRepository[ChecklistItem]):
    """
    Repository for ChecklistItem operations
    """
    model = ChecklistItem

    @classmethod
    async def get_active(
        cls,
        session: AsyncSession,
    ) -> Sequence[ChecklistItem]:
        """
        Get all active items ordered by sort_order
        """
        result = await session.execute(
            select(ChecklistItem).where(
                ChecklistItem.is_active == sa.true()
            ).order_by(ChecklistItem.sort_order)
        )
        return result.scalars().all()

    @classmethod
    async def soft_delete(
        cls,
        session: AsyncSession,
        item: ChecklistItem,
    ) -> ChecklistItem:
        """
        Set is_active=False instead of deleting
        """
        item.is_active = False
        await session.flush()
        await session.refresh(item)
        return item


class ChecklistLogRepository(BaseRepository[ChecklistLog]):
    """
    Repository for ChecklistLog operations
    """
    model = ChecklistLog

    @classmethod
    async def get_by_date(
        cls,
        session: AsyncSession,
        log_date: date,
    ) -> Sequence[ChecklistLog]:
        """
        Get all log entries for a date, joined with item for title/sort
        """
        result = await session.execute(
            select(ChecklistLog).join(ChecklistLog.item).where(
                ChecklistLog.log_date == log_date
            ).order_by(ChecklistItem.sort_order)
        )
        return result.scalars().all()

    @classmethod
    async def get_by_item_and_date(
        cls,
        session: AsyncSession,
        item_id: UUID,
        log_date: date,
    ) -> ChecklistLog | None:
        """
        Get a specific log entry
        """
        result = await session.execute(
            select(ChecklistLog).where(
                ChecklistLog.item_id == item_id,
                ChecklistLog.log_date == log_date,
            )
        )
        return result.scalar_one_or_none()

    @classmethod
    async def get_heatmap_data(
        cls,
        session: AsyncSession,
        start_date: date,
        end_date: date,
    ) -> Sequence[sa.Row]:
        """
        Get (log_date, completed_count, total_count) aggregates for a date range
        """
        result = await session.execute(
            select(
                ChecklistLog.log_date,
                func.sum(sa.cast(ChecklistLog.completed,
                                 sa.Integer)).label("completed_count"),
                func.count(ChecklistLog.id).label("total_count"),
            ).where(

```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Repository Pattern

The `ChecklistItemRepository` and `ChecklistLogRepository` classes implement the repository pattern, which encapsulates data access logic to provide a consistent interface for interacting with database operations.

- **Implementation**: Each repository class inherits from `BaseRepository`, defining methods like `get_active`, `soft_delete`, etc., that perform specific CRUD operations on their respective models (`ChecklistItem` and `ChecklistLog`). These methods abstract the underlying database interactions, making them reusable and testable.
  
- **Benefits**:
  - **Encapsulation**: Data access logic is encapsulated within repository classes, reducing coupling between business logic and data storage mechanisms.
  - **Testability**: Methods are isolated, allowing for easier unit testing without needing a live database.
  - **Consistency**: A consistent interface across repositories ensures uniformity in how operations are performed.

- **Deviations**:
  - The `BaseRepository` class is not shown, but it likely provides common methods and attributes. The provided classes only implement specific methods relevant to their models.
  - Some methods return sequences of model instances or aggregated data (`sa.Row`), which may require additional processing in the calling code.

- **Appropriateness**:
  - This pattern is highly appropriate for applications that need to abstract database interactions, especially when working with SQLAlchemy and async operations. It ensures a clean separation of concerns between business logic and data access layers.

This implementation effectively leverages the repository pattern to manage ChecklistItem and ChecklistLog operations in an organized and maintainable manner.

---

*Generated by CodeWorm on 2026-02-25 21:45*
