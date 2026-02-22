# process_upload_with_retry

**Type:** Performance Analysis
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

## Performance Analysis

### Performance Analysis

**Time Complexity**: The function has a time complexity of \(O(T + A)\), where \(T\) is the number of thumbnail generation attempts (1) and \(A\) is the number of AI service analysis attempts (2). The `monitor_ai_progress` loop runs indefinitely until the upload status changes or is deleted, making it potentially unbounded.

**Space Complexity**: The primary space concern is the `monitor_task` created within the function. While this task is cancelled at the end, there's a risk of resource leaks if the function exits prematurely without cancelling all tasks properly.

**Bottlenecks and Inefficiencies**:
1. **Redundant Operations**: The thumbnail generation logic runs only on the first attempt, but it’s still executed in every iteration.
2. **Blocking Calls in Async Contexts**: `await ai_service.analyze_media(upload.id)` is a blocking call within an async function, which can lead to context switching overhead and potential deadlocks if not managed properly.
3. **Unnecessary Iterations**: The `monitor_ai_progress` loop runs indefinitely until the upload status changes or is deleted, leading to unnecessary polling.

**Optimization Opportunities**:
1. **Remove Redundant Operations**: Ensure that thumbnail generation logic only runs once per upload by moving it outside the retry loop.
2. **Use Async Context Managers**: Wrap `ai_service.analyze_media(upload.id)` in an async context manager to ensure proper cleanup and avoid blocking calls.

**Resource Usage Concerns**:
1. **Task Management**: Ensure all tasks are properly cancelled and awaited to prevent resource leaks, especially if the function is called concurrently.
2. **Polling Frequency**: Reduce the polling frequency from 300ms to a more efficient interval based on real-time needs.

By addressing these points, you can improve both the performance and reliability of your code.

---

*Generated by CodeWorm on 2026-02-21 22:05*
