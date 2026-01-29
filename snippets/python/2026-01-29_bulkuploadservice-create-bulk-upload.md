# BulkUploadService.create_bulk_upload

**Repository:** vuemantics
**File:** backend/services/bulk_upload_service.py
**Language:** python
**Lines:** 97-237
**Complexity:** 13.0

---

## Source Code

```python
async def create_bulk_upload(
        user_id: UUID,
        files: list[UploadFile],
    ) -> BulkUploadResult:
        """
        Create and queue a bulk upload batch

        Args:
            user_id: User's UUID
            files: List of files to upload

        Returns:
            BulkUploadResult with batch info

        Process:
        1. Validate bulk upload request
        2. Create batch record
        3. Save each file to storage
        4. Create upload records linked to batch
        5. Queue batch for background processing
        6. Return batch ID and results
        """
        # Validate bulk upload
        BulkUploadService._validate_bulk_upload(files)

        total_size = sum(file.size or 0 for file in files)
        logger.info(
            f"Creating bulk upload for user {user_id}: "
            f"{len(files)} files, {total_size / (1024 * 1024):.2f} MB total"
        )

        batch = await UploadBatch.create(
            user_id = user_id,
            total_uploads = len(files),
        )

        logger.info(f"Created batch {batch.id} with {len(files)} uploads")

        upload_ids = []
        failed_files = []

        for file in files:
            try:
                if not file.filename:
                    failed_files.append(
                        {
                            "filename": "unknown",
                            "error": "No filename provided",
                        }
                    )
                    continue

                upload_id = uuid4()

                file_content = await file.read()
                file_size = len(file_content)

                await file.seek(0)

                file_type, extension = await storage_service.validate_file(
                    filename = file.filename,
                    mime_type = file.content_type or "application/octet-stream",
                    file_size = file_size,
                )

                file_path = await storage_service.save_upload(
                    file_content = file.file,
                    user_id = user_id,
                    upload_id = upload_id,
                    extension = extension,
                )

                upload = await Upload.create(
                    user_id = user_id,
                    batch_id = batch.id,
                    filename = file.filename,
                    file_path = file_path,
                    file_type = file_type,
                    file_size = file_size,
                    mime_type = file.content_type or "application/octet-stream",
                    metadata = {
                        "original_filename": file.filename,
                        "upload_source": "bulk",
                        "batch_id": str(batch.id),
                    },
                    upload_id = upload_id,
                )

                upload_ids.append(str(upload.id))
                logger.debug(
                    f"Created upload {upload.id} in batch {batch.id}"
                )

            except (FileTooLargeError, UnsupportedFileTypeError) as e:
                logger.warning(
                    f"File validation failed for {file.filename}: {e}"
                )
                failed_files.append(
                    {
                        "filename": file.filename or "unknown",
                        "error": str(e),
                    }
                )
            except Exception as e:
                logger.error(
                    f"Error processing file {file.filename}: {e}",
                    exc_info = True,
                )
                failed_files.append(
                    {
                        "filename": file.filename or "unknown",
                        "error": "Internal error during file processing",
                    }
                )

        if len(upload_ids) != batch.total_uploads:
            batch.total_uploads = len(upload_ids)
            await batch.save()
            logger.info(
                f"Updated batch {batch.id} total uploads to {len(upload_ids)}"
            )

        if len(upload_ids) > 0:
            process_batch.send(str(batch.id))
            logger.info(
                f"Queued batch {batch.id} for processing ({len(upload_ids)} uploads)"
            )

        logger.info(
            f"Bulk upload completed for batch {batch.id}: "
            f"{len(upload_ids)} queued, {len(failed_files)} failed"
        )

        return BulkUploadResult(
            batch_id = batch.id,
            total_files = len(files),
            queued = len(upload_ids),
            failed = len(failed_files),
            upload_ids = upload_ids,
            failed_files = failed_files,
        )
```

---

## Documentation

### Documentation for `create_bulk_upload` Function

**Purpose and Behavior:**
The `create_bulk_upload` function processes a bulk upload request by validating files, creating batch records, saving files to storage, and queuing the batch for background processing. It returns detailed information about the uploads.

**Key Implementation Details:**
- **Async/Await:** The function is asynchronous, using `await` to handle I/O operations like file reading and database queries.
- **Error Handling:** Uses try-except blocks to manage errors during file validation and processing, logging warnings and errors as appropriate.
- **Logging:** Extensive use of logging for informational, debug, warning, and error messages.

**When/Why to Use:**
Use this function when handling bulk file uploads in a web application. It ensures that files are processed efficiently and securely, providing detailed feedback on the upload process.

**Patterns/Gotchas:**
- **Asynchronous I/O:** The function relies heavily on asynchronous operations for efficient file processing.
- **File Validation:** Ensures only valid files are uploaded by checking size and type, logging errors for invalid files.
- **Batch Management:** Efficiently manages batch creation, file storage, and background processing to handle large numbers of files.

---

*Generated by CodeWorm on 2026-01-29 08:41*
