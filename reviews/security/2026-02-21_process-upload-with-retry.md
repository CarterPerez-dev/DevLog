# process_upload_with_retry

**Type:** Security Review
**Repository:** vuemantics
**File:** backend/workers/batch_helpers/_process_upload_with_retry.py
**Language:** python
**Lines:** 23-169
**Complexity:** 12.0

---

## Source Code

```python
async def process_upload_with_retry(
    ai_service: LocalAIService,
    upload: Upload,
    batch: UploadBatch,
) -> bool:
    """
    Process a single upload with retry logic

    Args:
        ai_service: AI service instance
        upload: Upload to process
        batch: UploadBatch instance for progress updates

    Returns:
        True if successful, False if failed after retry

    Design:
    - Try once, retry once on failure (max 2 attempts)
    - Failures don't stop batch processing
    - Upload marked as failed after 2 attempts
    - Publishes real-time progress by monitoring AI service stages
    """
    max_attempts = 2

    for attempt in range(1, max_attempts + 1):
        try:
            logger.info(
                f"Processing upload {upload.id} (attempt {attempt}/{max_attempts})"
            )

            # Generate thumbnail (only on first attempt)
            if attempt == 1 and not upload.thumbnail_path:
                try:
                    logger.info(f"Generating thumbnail for upload {upload.id}")
                    await publish_file_progress(batch, upload, status="processing", progress=5)
                    thumbnail_path = await storage_service.generate_thumbnail(
                        user_id = upload.user_id,
                        upload_id = upload.id,
                        file_type = upload.file_type,
                        extension = upload.file_path.split('.')[-1],
                    )
                    if thumbnail_path:
                        await upload.update_thumbnail(thumbnail_path)
                        logger.info(f"Thumbnail generated for upload {upload.id}")
                    await publish_file_progress(batch, upload, status="processing", progress=10)
                except Exception as thumb_error:
                    logger.warning(
                        f"Thumbnail generation failed for {upload.id}: {thumb_error}"
                    )
                    # Continue processing even if thumbnail fails

            # Monitor AI progress with real stages
            async def monitor_ai_progress() -> None:
                """Poll upload status and publish real progress from AI service"""
                last_progress = 10
                start_time = asyncio.get_event_loop().time()

                while True:
                    await asyncio.sleep(0.3)  # Poll every 300ms

                    try:
                        await upload.refresh()
                    except ValueError:
                        # Upload was deleted mid-processing (user action)
                        logger.warning(f"Upload {upload.id} was deleted during processing, stopping monitor")
                        break
                    except Exception as e:
                        logger.warning(f"Failed to refresh upload {upload.id}: {e}")
                        await asyncio.sleep(1.0)  # Back off and retry
                        continue

                    elapsed = asyncio.get_event_loop(
```

---

## Security Review

### Security Review for `process_upload_with_retry`

#### Vulnerabilities Found:

1. **Information Disclosure** - **Severity: Low**
   - The function logs detailed error messages and status updates, which could potentially leak information about the processing state.
   - **Attack Vector:** An attacker could exploit this to infer upload statuses or service states.

2. **Potential Race Conditions** - **Severity: Medium**
   - The `monitor_task` is cancelled in a `finally` block without ensuring it has completed before cancellation, which might lead to race conditions.
   - **Attack Vector:** A malicious user could manipulate the state of an upload while it's being processed.

3. **Hardcoded Secrets** - **Severity: Info**
   - No hardcoded secrets or credentials are found in the provided code snippet.

4. **Input Validation Gaps** - **Severity: Low**
   - The function does not validate input parameters like `upload`, `batch`, and `ai_service` for type safety, though this is more of a Pythonic issue rather than a security vulnerability.
   - **Attack Vector:** If these inputs are manipulated, it could lead to unexpected behavior.

#### Recommended Fixes:

1. **Information Disclosure:**
   - Use structured logging with levels (e.g., `logging.error`, `logging.warning`) and avoid sensitive information in logs.
   
2. **Race Conditions:**
   - Ensure the `monitor_task` is properly awaited before cancellation by using a flag or condition variable to signal completion.

3. **Input Validation:**
   - Add type hints and validation for function parameters to ensure they are of expected types.

#### Overall Security Posture:

The code has some minor security concerns, but overall maintains a reasonable security posture. Addressing the information disclosure and race conditions will further enhance its robustness.

---

*Generated by CodeWorm on 2026-02-21 13:28*
