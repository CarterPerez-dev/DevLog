# ChecklistService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/life_manager/facets/checklist/service.py
**Language:** python
**Lines:** 70-250
**Complexity:** 0.0

---

## Source Code

```python
class ChecklistService:
    """
    Service for checklist operations
    """
    @staticmethod
    async def get_items(
        session: AsyncSession
    ) -> ChecklistItemListResponse:
        """
        Get all active checklist items
        """
        items = await ChecklistItemRepository.get_active(session)
        return ChecklistItemListResponse(
            items = [
                ChecklistItemResponse.model_validate(i) for i in items
            ]
        )

    @staticmethod
    async def create_item(
        session: AsyncSession,
        data: ChecklistItemCreate,
    ) -> ChecklistItemResponse:
        """
        Create a new checklist item
        """
        item = await ChecklistItemRepository.create(
            session,
            title = data.title,
            sort_order = data.sort_order,
        )
        return ChecklistItemResponse.model_validate(item)

    @staticmethod
    async def update_item(
        session: AsyncSession,
        item_id: UUID,
        data: ChecklistItemUpdate,
    ) -> ChecklistItemResponse:
        """
        Update a checklist item
        """
        item = await ChecklistItemRepository.get_by_id(session, item_id)
        if not item:
            raise ChecklistItemNotFound(item_id)

        update_dict = data.model_dump(exclude_unset = True)
        item = await ChecklistItemRepository.update(
            session,
            item,
            **update_dict
        )
        return ChecklistItemResponse.model_validate(item)

    @staticmethod
    async def delete_item(
        session: AsyncSession,
        item_id: UUID,
    ) -> None:
        """
        Soft-delete a checklist item (is_active=False)
        """
        item = await ChecklistItemRepository.get_by_id(session, item_id)
        if not item:
            raise ChecklistItemNotFound(item_id)
        await ChecklistItemRepository.soft_delete(session, item)

    @staticmethod
    async def get_day(
        session: AsyncSession,
        log_date: date,
    ) -> ChecklistDayResponse:
        """
        Get checklist for a day. Auto-inits log rows if first visit.
        """
        existing = await ChecklistLogRepository.get_by_date(
            session,
            log_date
        )
        active_items = await ChecklistItemRepository.get_active(session)

        existing_item_ids = {log.item_id for log in existing}
        new_items = [
            i for i in active_items if i.id not in existing_item_ids
        ]

        for item in new_items:
            await ChecklistLogRepository.create(
                session,
                item_id = item.id,
                log_date = log_date,
                completed = False,
            )

        logs = await ChecklistLogRepository.get_by_date(session, log_date)
        entries = [_to_log_response(log) for log in logs]

        return ChecklistDayResponse(
            date = log_date,
            entries = entries,
            completed_count = sum(1 for e in entries if e.com
```

---

## Class Documentation

### ChecklistService

**Class Responsibility and Purpose:**
The `ChecklistService` class is responsible for managing operations related to checklists, including fetching items, creating, updating, deleting, and generating statistics. It acts as a service layer that interacts with repositories to perform CRUD (Create, Read, Update, Delete) operations on checklist items and logs.

**Public Interface:**
- `get_items(session)`: Retrieves all active checklist items.
- `create_item(session, data)`: Creates a new checklist item.
- `update_item(session, item_id, data)`: Updates an existing checklist item.
- `delete_item(session, item_id)`: Soft-deletes a checklist item by setting its `is_active` flag to False.
- `get_day(session, log_date)`: Fetches the checklist for a specific day and auto-initializes missing logs if necessary.
- `update_log(session, log_id, data)`: Updates the completion status and note of a log entry.
- `get_stats(session)`: Computes streaks, item completion rates, and year heatmap.

**Design Patterns Used:**
The class employs several design patterns:
- **Factory Pattern**: Implicitly used through repository methods for creating and updating items.
- **Observer Pattern**: Not explicitly implemented but implied by the interaction between checklist items and logs.
- **Strategy Pattern**: Implied in handling different update operations (e.g., toggling completion status).

**Relationship to Other Classes:**
`ChecklistService` interacts with `ChecklistItemRepository` and `ChecklistLogRepository` for data retrieval and manipulation. It also uses models like `ChecklistItemListResponse`, `ChecklistDayResponse`, and `ChecklistStatsResponse` to structure the responses.

This class fits into a broader architecture where it serves as a service layer, providing a clean interface between business logic and database operations.

---

*Generated by CodeWorm on 2026-02-22 19:05*
