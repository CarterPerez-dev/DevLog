# repository_pattern

**Type:** Pattern Analysis
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/life_manager/facets/notes/service.py
**Language:** python
**Lines:** 1-324
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2026
service.py
"""

from uuid import UUID

from sqlalchemy.ext.asyncio import AsyncSession

from core.exceptions import ResourceNotFound
from aspects.life_manager.facets.notes.models import Note
from aspects.life_manager.facets.notes.repository import (
    NoteFolderRepository,
    NoteRepository,
)
from aspects.life_manager.facets.notes.schemas import (
    NoteFolderCreate,
    NoteFolderUpdate,
    NoteFolderResponse,
    NoteCreate,
    NoteUpdate,
    NoteResponse,
    NotesListResponse,
    DeletedNotesListResponse,
)


class NoteFolderNotFound(ResourceNotFound):
    """
    Raised when folder not found
    """
    def __init__(self, folder_id: UUID) -> None:
        super().__init__(
            resource = "NoteFolder",
            identifier = str(folder_id)
        )


class NoteNotFound(ResourceNotFound):
    """
    Raised when note not found
    """
    def __init__(self, note_id: UUID) -> None:
        super().__init__(resource = "Note", identifier = str(note_id))


class NotesService:
    """
    Service for notes operations
    """
    @staticmethod
    async def get_all_notes(
        session: AsyncSession,
    ) -> NotesListResponse:
        """
        Get all folders and notes
        """
        folders = await NoteFolderRepository.get_all(session)
        notes = await NoteRepository.get_all(session)
        return NotesListResponse(
            folders = [
                NoteFolderResponse.model_validate(f) for f in folders
            ],
            notes = [NoteResponse.model_validate(n) for n in notes],
        )

    @staticmethod
    async def create_folder(
        session: AsyncSession,
        data: NoteFolderCreate,
    ) -> NoteFolderResponse:
        """
        Create a folder
        """
        folder = await NoteFolderRepository.create(
            session,
            name = data.name,
            parent_id = data.parent_id,
            sort_order = data.sort_order,
        )
        return NoteFolderResponse.model_validate(folder)

    @staticmethod
    async def update_folder(
        session: AsyncSession,
        folder_id: UUID,
        data: NoteFolderUpdate,
    ) -> NoteFolderResponse:
        """
        Update a folder
        """
        folder = await NoteFolderRepository.get_by_id(session, folder_id)
        if not folder:
            raise NoteFolderNotFound(folder_id)

        update_dict = data.model_dump(exclude_unset = True)
        folder = await NoteFolderRepository.update(
            session,
            folder,
            **update_dict
        )
        return NoteFolderResponse.model_validate(folder)

    @staticmethod
    async def delete_folder(
        session: AsyncSession,
        folder_id: UUID,
    ) -> None:
        """
        Soft delete a folder and all its notes
        """
        folder = await NoteFolderRepository.get_by_id(session, folder_id)
        if not folder:
            raise NoteFolderNotFound(folder_id)

        notes_in_folder = await Not
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Repository Pattern

The `NotesService` class in this code implements the **Repository Pattern**, which decouples business logic from data access by providing a uniform interface to retrieve and store domain objects.

- **Implementation**: The `NoteFolderRepository` and `NoteRepository` classes handle CRUD operations for `NoteFolder` and `Note` entities, respectively. These repositories are injected into the service methods via the `session` parameter.
  
  ```python
  @staticmethod
  async def get_all_notes(session: AsyncSession) -> NotesListResponse:
      folders = await NoteFolderRepository.get_all(session)
      notes = await NoteRepository.get_all(session)
      return NotesListResponse(
          folders=[NoteFolderResponse.model_validate(f) for f in folders],
          notes=[NoteResponse.model_validate(n) for n in notes],
      )
  ```

- **Benefits**: 
  - **Encapsulation**: The service layer is decoupled from the database, making it easier to switch data storage mechanisms.
  - **Testability**: Repositories can be easily mocked or replaced with in-memory databases during testing.

- **Deviations**:
  - The `NotesService` class uses static methods instead of instance methods, which might not align perfectly with the standard pattern where services are often instances that maintain state.
  - Error handling is done using custom exceptions (`NoteFolderNotFound`, `NoteNotFound`) rather than generic ones like `ResourceNotFound`.

- **Appropriateness**:
  - This pattern is highly appropriate in this context as it effectively separates concerns, making the code more modular and easier to manage. It’s particularly useful when dealing with complex data access logic or when integrating with different types of storage systems.

This implementation closely follows the Repository Pattern while providing a clear separation between business logic and data access.

---

*Generated by CodeWorm on 2026-02-25 19:59*
