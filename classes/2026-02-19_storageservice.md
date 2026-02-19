# StorageService

**Type:** Class Documentation
**Repository:** vuemantics
**File:** backend/services/storage_service.py
**Language:** python
**Lines:** 26-522
**Complexity:** 0.0

---

## Source Code

```python
class StorageService:
    """
    Handles file storage operations with cloud-ready interface

    Currently uses local filesystem but designed for easy migration
    to cloud storage services like S3 or Google Cloud Storage
    """
    def __init__(self, base_path: Path | None = None):
        """
        Initialize storage service

        Args:
            base_path: Base directory for uploads (defaults to settings)
        """
        self.base_path = base_path or config.settings.upload_path
        logger.info(
            f"Storage service initialized with base path: {self.base_path}"
        )

    def _get_upload_dir(self, user_id: UUID, upload_id: UUID) -> Path:
        """
        Get directory path for specific upload.

        Structure: base_path/user_id/upload_id/
        """
        return self.base_path / str(user_id) / str(upload_id)

    async def validate_file(
        self,
        filename: str,
        mime_type: str,
        file_size: int
    ) -> tuple[str,
               str]:
        """
        Validate uploaded file using FileValidator.

        Args:
            filename: Original filename
            mime_type: MIME type
            file_size: Size in bytes

        Returns:
            Tuple of (file_type, extension)
        """
        result = FileValidator.validate(filename, mime_type, file_size)
        return result.file_type, result.extension

    async def save_upload(
        self,
        file_content: BinaryIO,
        user_id: UUID,
        upload_id: UUID,
        extension: str,
    ) -> str:
        """
        Save uploaded file to storage

        Args:
            file_content: File content as binary stream
            user_id: User's ID
            upload_id: Upload's ID
            extension: File extension

        Returns:
            Relative path to saved file
        """
        # Create dir structure
        upload_dir = self._get_upload_dir(user_id, upload_id)
        upload_dir.mkdir(parents = True, exist_ok = True)

        # Save original file
        filename = f"original.{extension}"
        file_path = upload_dir / filename

        async with aiofiles.open(file_path, "wb") as f:
            # Read chunks to handle large files
            while chunk := file_content.read(config.FILE_UPLOAD_CHUNK_SIZE
                                             ):
                await f.write(chunk)

        relative_path = file_path.relative_to(self.base_path)
        logger.info(f"Saved upload to: {relative_path}")

        return str(relative_path)

    async def generate_thumbnail(
        self,
        user_id: UUID,
        upload_id: UUID,
        file_type: str,
        extension: str,
    ) -> str | None:
        """
        Generate thumbnail for upload (async).

        Args:
            user_id: User's ID
            upload_id: Upload's ID
            file_type: 'image' or 'video'
            extension: Original file extension

        Returns:
            Relative path to thumbnail or No
```

---

## Class Documentation

### StorageService Documentation

**Class Responsibility and Purpose:**
The `StorageService` class is responsible for handling file storage operations, currently using the local filesystem but designed to be easily migrated to cloud storage services like S3 or Google Cloud Storage. It provides a consistent interface for uploading, validating, saving, and generating thumbnails for files.

**Public Interface (Key Methods):**
- **`__init__(self, base_path: Path | None = None)`**: Initializes the storage service with an optional `base_path`.
- **`validate_file(self, filename: str, mime_type: str, file_size: int) -> tuple[str, str]`**: Validates uploaded files using a `FileValidator`.
- **`save_upload(self, file_content: BinaryIO, user_id: UUID, upload_id: UUID, extension: str) -> str`**: Saves an uploaded file to the storage.
- **`generate_thumbnail(self, user_id: UUID, upload_id: UUID, file_type: str, extension: str) -> str | None`**: Generates a thumbnail for an image or video.

**Design Patterns Used:**
The class uses the **Strategy Pattern** through `FileValidator.validate()` to handle different validation logic. The use of asynchronous methods (`async def`) and context managers (e.g., `aiofiles.open()`, `Image.open()`) ensures efficient handling of large files and image processing, respectively.

**Relationship to Other Classes:**
- **`config.settings.upload_path`**: Configures the base path for file uploads.
- **`FileValidator.validate()`**: Validates uploaded files based on filename, MIME type, and size.
- **`Image.open()`**: Used for synchronous image thumbnail generation.

This class fits into a larger architecture where it provides robust file management capabilities, ensuring that the application can handle both local and cloud storage seamlessly.

---

*Generated by CodeWorm on 2026-02-19 01:01*
