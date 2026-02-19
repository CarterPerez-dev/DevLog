# FileValidator

**Type:** Class Documentation
**Repository:** vuemantics
**File:** backend/core/validators/file.py
**Language:** python
**Lines:** 65-170
**Complexity:** 0.0

---

## Source Code

```python
class FileValidator:
    """
    Validates uploaded files for size, MIME type, and determines file type
    """
    @classmethod
    def validate(
        cls,
        filename: str,
        mime_type: str,
        file_size: int
    ) -> FileValidationResult:
        """
        Validate uploaded file

        Args:
            filename: Original filename
            mime_type: MIME type
            file_size: Size in bytes

        Returns:
            FileValidationResult with file_type and extension

        Raises:
            FileTooLargeError: If file exceeds size limit
            UnsupportedFileTypeError: If MIME type not allowed
        """
        cls._validate_size(file_size)
        cls._validate_mime_type(mime_type)

        file_type = cls._determine_file_type(mime_type)
        extension = cls._get_extension(filename, mime_type)

        return FileValidationResult(
            file_type = file_type,
            extension = extension
        )

    @classmethod
    def _validate_size(cls, file_size: int) -> None:
        """
        Validate file size is within limits
        """
        if file_size > config.settings.max_upload_size:
            raise FileTooLargeError(
                f"File size {file_size} exceeds limit of "
                f"{config.settings.max_upload_size} bytes"
            )

    @classmethod
    def _validate_mime_type(cls, mime_type: str) -> None:
        """
        Validate MIME type is allowed
        """
        if mime_type not in ALLOWED_MIME_TYPES:
            raise UnsupportedFileTypeError(
                f"MIME type {mime_type} is not supported"
            )

    @classmethod
    def _determine_file_type(cls, mime_type: str) -> str:
        """
        Determine if file is image or video based on MIME type
        """
        if mime_type in ALLOWED_IMAGE_MIMES:
            return "image"
        return "video"

    @classmethod
    def _get_extension(cls, filename: str, mime_type: str) -> str:
        """
        Get file extension from filename or MIME type

        Args:
            filename: Original filename
            mime_type: MIME type

        Returns:
            File extension (without dot)
        """
        if "." in filename:
            ext = filename.rsplit(".", 1)[-1].lower()
            if ext in VALID_EXTENSIONS:
                return ext

        return MIME_TO_EXTENSION.get(mime_type, "bin")

    @classmethod
    def is_valid_extension(cls, extension: str) -> bool:
        """
        Check if extension is valid
        """
        return extension.lower() in VALID_EXTENSIONS

    @classmethod
    def is_image_mime(cls, mime_type: str) -> bool:
        """
        Check if MIME type is an image
        """
        return mime_type in ALLOWED_IMAGE_MIMES

    @classmethod
    def is_video_mime(cls, mime_type: str) -> bool:
        """
        Check if MIME type is a video
        """
        return mime_type in ALLOWED_VIDEO_MIMES
```

---

## Class Documentation

### FileValidator Class Documentation

**Class Responsibility and Purpose:**
The `FileValidator` class is responsible for validating uploaded files based on size, MIME type, and file type. It ensures that only valid files are processed by raising appropriate errors if any validation fails.

**Public Interface (Key Methods):**
- **`validate(filename: str, mime_type: str, file_size: int) -> FileValidationResult`:** Validates the file and returns a `FileValidationResult` object containing the determined file type and extension.
- **`_validate_size(file_size: int) -> None`:** Checks if the file size is within the allowed limit.
- **`_validate_mime_type(mime_type: str) -> None`:** Ensures that the MIME type of the file is supported.
- **`_determine_file_type(mime_type: str) -> str`:** Determines whether the file is an image or a video based on its MIME type.
- **`_get_extension(filename: str, mime_type: str) -> str`:** Retrieves the file extension from either the filename or the MIME type.
- **`is_valid_extension(extension: str) -> bool`:** Checks if a given extension is valid.
- **`is_image_mime(mime_type: str) -> bool`:** Determines if the provided MIME type corresponds to an image.
- **`is_video_mime(mime_type: str) -> bool`:** Determines if the provided MIME type corresponds to a video.

**Design Patterns Used:**
The class does not explicitly use any design patterns, but it follows Pythonic conventions and best practices. The methods are organized in a way that promotes separation of concerns, with validation logic encapsulated within specific methods.

**How It Fits in the Architecture:**
`FileValidator` is part of the backend core module, specifically designed for file validation. It interacts with other modules by providing validated file metadata (type and extension) to higher-level services or storage systems. The class ensures that only valid files proceed through the system, contributing to overall data integrity and security.

---

*Generated by CodeWorm on 2026-02-19 02:10*
