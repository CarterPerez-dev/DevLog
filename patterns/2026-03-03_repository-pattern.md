# repository_pattern

**Type:** Pattern Analysis
**Repository:** social-media-notes
**File:** backend/app/video/service.py
**Language:** python
**Lines:** 1-205
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

from config import Platform
from core.exceptions import NotFoundError
from .VideoEntry import VideoEntry
from .repository import VideoEntryRepository
from .schemas import (
    VideoEntryCreate,
    VideoEntryUpdate,
    VideoEntryResponse,
    VideoEntryListResponse,
)


class VideoEntryService:
    """
    Business logic for video entry operations
    """
    def __init__(self, session: AsyncSession) -> None:
        self.session = session

    async def create_entry(
        self,
        user_id: UUID,
        data: VideoEntryCreate,
    ) -> VideoEntryResponse:
        """
        Create a new video entry
        """
        video_number = data.video_number
        if video_number == 0:
            video_number = await VideoEntryRepository.get_next_video_number(
                self.session,
                user_id,
                data.platform,
            )

        entry = await VideoEntryRepository.create(
            self.session,
            user_id=user_id,
            platform=data.platform,
            video_number=video_number,
            description=data.description,
            youtube_description=data.youtube_description,
            scheduled_time=data.scheduled_time,
        )
        return VideoEntryResponse.model_validate(entry)

    async def get_entry(
        self,
        entry_id: UUID,
        user_id: UUID,
    ) -> VideoEntryResponse:
        """
        Get a video entry by ID
        """
        entry = await VideoEntryRepository.get_by_id_and_user(
            self.session,
            entry_id,
            user_id,
        )
        if not entry:
            raise NotFoundError("Video entry not found")
        return VideoEntryResponse.model_validate(entry)

    async def update_entry(
        self,
        entry_id: UUID,
        user_id: UUID,
        data: VideoEntryUpdate,
    ) -> VideoEntryResponse:
        """
        Update a video entry
        """
        entry = await VideoEntryRepository.get_by_id_and_user(
            self.session,
            entry_id,
            user_id,
        )
        if not entry:
            raise NotFoundError("Video entry not found")

        update_dict = data.model_dump(exclude_unset=True)
        updated = await VideoEntryRepository.update(
            self.session,
            entry,
            **update_dict,
        )
        return VideoEntryResponse.model_validate(updated)

    async def delete_entry(
        self,
        entry_id: UUID,
        user_id: UUID,
    ) -> None:
        """
        Delete a video entry
        """
        entry = await VideoEntryRepository.get_by_id_and_user(
            self.session,
            entry_id,
            user_id,
        )
        if not entry:
            raise NotFoundError("Video entry not found")
        await VideoEntryRepository.delete(self.session, entry)

    async def list_entries(
        self,
     
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Repository Pattern

The `VideoEntryService` class in this code implements the **Repository Pattern**, which encapsulates data access logic and provides a clean interface for business logic to interact with data.

#### Implementation:
- The service layer (`VideoEntryService`) interacts with the repository layer through methods like `create_entry`, `get_entry`, etc.
- These methods delegate data operations (like creating, updating, deleting) to the `VideoEntryRepository` class.

#### Benefits:
1. **Separation of Concerns:** Business logic is separated from data access concerns, making the code more maintainable and testable.
2. **Flexibility:** Easier to change storage mechanisms or add caching without affecting business logic.
3. **Encapsulation:** Data access methods are encapsulated within the repository, reducing complexity in service layer.

#### Deviations:
- The `VideoEntryService` class directly interacts with session objects (`AsyncSession`), which is not ideal as it tightly couples the service to the database session.
- Some methods like `_shorten_description` are mixed into the service logic, deviating slightly from pure repository pattern principles.

#### Appropriateness:
This pattern is appropriate for this scenario because it clearly separates business logic from data access. However, consider refactoring to use dependency injection for session objects and move utility functions out of the service class for better adherence to the pattern.

```python
class VideoEntryService:
    def __init__(self, repository: VideoEntryRepository) -> None:
        self.repository = repository

# Usage in a service method
async def create_entry(self, user_id: UUID, data: VideoEntryCreate):
    video_number = await self.repository.get_next_video_number(
        user_id,
        data.platform,
    )
    entry = await self.repository.create(
        user_id=user_id,
        platform=data.platform,
        video_number=video_number,
        description=data.description,
        youtube_description=data.youtube_description,
        scheduled_time=data.scheduled_time,
    )
    return VideoEntryResponse.model_validate(entry)
```

This refactoring enhances the pattern's purity and maintainability.

---

*Generated by CodeWorm on 2026-03-03 11:35*
