# NotesService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/life_manager/facets/notes/service.py
**Language:** python
**Lines:** 47-323
**Complexity:** 0.0

---

## Source Code

```python
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

        notes_in_folder = await NoteRepository.get_all(
            session,
            folder_id = folder_id
        )
        for note in notes_in_folder:
            await NoteRepository.soft_delete(session, note)

        await NoteFolderRepository.soft_delete(session, folder)

    @staticmethod
    async def create_note(
        session: AsyncSession,
        data: NoteCreate,
    ) -> NoteResponse:
        """
        Create a note
        """
        if data.folder_id:
            folder = await NoteFolderRepository.get_by_id(
                session,
                data.folder_id
            )
            if not folder:
                raise NoteFolderNotFound(data.folder_id)

        note = await NoteRepository.create(
            session,
            title = data.title,
            content = data.content,
            folder_id = data.folder_id,
            sort_order = data.sort_order,
        )
        return NoteResponse.model_validate(note)

    @staticmethod
    async def get_note(
        session: AsyncSession,
    
```

---

## Class Documentation

### NotesService Documentation

**Class Responsibility and Purpose:**
The `NotesService` class serves as a service layer for managing notes and folders within an application. It encapsulates operations such as creating, updating, deleting (soft and hard), retrieving, and listing both active and deleted notes and folders.

**Public Interface:**
- **get_all_notes**: Retrieves all folders and notes.
- **create_folder**: Creates a new folder with specified data.
- **update_folder**: Updates an existing folder by ID.
- **delete_folder**: Soft deletes a folder along with its associated notes.
- **create_note**: Creates a new note, optionally associating it with a folder.
- **get_note**: Retrieves a note by ID.
- **update_note**: Updates an existing note.
- **delete_note**: Soft deletes a note.
- **get_deleted_notes**: Lists all soft-deleted notes and folders.

**Design Patterns Used:**
- **Factory Method Pattern**: Implicitly used in the creation of `NoteFolderResponse` and `NoteResponse` objects via model validation.
- **Repository Pattern**: Utilized through `NoteFolderRepository` and `NoteRepository` to abstract data access logic.

**Relationship to Other Classes:**
The class interacts with `NoteFolderRepository` and `NoteRepository` for CRUD operations on folders and notes, respectively. It also handles the soft deletion of entities by marking them as deleted in the database without physically removing them.

**State Management Approach:**
The service layer ensures that all state changes are managed through repository methods, maintaining a clean separation between business logic and data access. This approach promotes testability and maintainability by encapsulating complex operations within well-defined methods.

---

*Generated by CodeWorm on 2026-02-22 02:10*
