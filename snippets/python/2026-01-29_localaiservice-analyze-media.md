# LocalAIService.analyze_media

**Repository:** vuemantics
**File:** backend/services/ai/service.py
**Language:** python
**Lines:** 104-298
**Complexity:** 15.0

---

## Source Code

```python
async def analyze_media(self, upload_id: UUID) -> None:
        """
        Analyze media file and generate embeddings

        Args:
            upload_id: Upload to process

        Updates upload record with:
        - description: Text description
        - embedding_local: 1024-dimensional vector
        - processing_status: completed or failed
        """
        upload = await Upload.find_by_id(upload_id)
        if not upload:
            logger.error(f"Upload {upload_id} not found")
            return

        try:
            await upload.update_status(ProcessingStatus.ANALYZING)
            await self._publish_progress(
                upload_id,
                ProcessingStatus.ANALYZING,
                ProcessingStage.QUEUED,
                0,
                "Starting AI analysis"
            )
            logger.info(
                f"Starting local AI analysis for upload {upload_id}"
            )

            file_path = config.settings.upload_path / upload.file_path

            description = None
            audit_result = None
            max_retries = 2

            for attempt in range(max_retries):
                if upload.file_type == "video":
                    await self._publish_progress(
                        upload_id,
                        ProcessingStatus.ANALYZING,
                        ProcessingStage.EXTRACTING_FRAMES,
                        10,
                        "Extracting video frames"
                    )

                    try:
                        metadata = await storage_service.get_upload_metadata(
                            upload.user_id,
                            upload_id
                        )
                        if metadata and metadata.get("codec"):
                            await upload.update_video_codec(
                                metadata["codec"]
                            )
                            logger.info(
                                f"Detected video codec for {upload_id}: {metadata['codec']}"
                            )
                    except Exception as codec_err:
                        logger.warning(
                            f"Failed to detect codec for {upload_id}: {codec_err}"
                        )

                await self._publish_progress(
                    upload_id,
                    ProcessingStatus.ANALYZING,
                    ProcessingStage.VISION_ANALYSIS,
                    20,
                    f"Analyzing {upload.file_type} with vision model"
                )

                if upload.file_type == "image":
                    description = await self._vision.analyze_image(
                        file_path
                    )
                else:
                    description = await self._analyze_video_with_frames(
                        upload.user_id,
                        upload_id,
                        file_path
                    )

                if not description:
                    raise VisionError(
                        "Vision model returned empty description"
                    )

                await self._publish_progress(
                    upload_id,
                    ProcessingStatus.ANALYZING,
                    ProcessingStage.DESCRIPTION_AUDIT,
                    50,
                    "Auditing description quality"
                )

                audit_result = DescriptionAuditor.audit(description)
                logger.info(
                    f"Description audit for {upload_id}: score={audit_result.score}, "
                    f"passed={audit_result.passed}, issues={audit_result.issues}"
                )

                if audit_result.passed:
                    break

                if attempt < max_retries - 1:
                    logger.warning(
                        f"Description failed audit (attempt {attempt + 1}/{max_retries}), "
                        f"retrying... Issues: {audit_result.issues}"
                    )

            if audit_result is None or description is None:
                raise VisionError("Failed to generate valid description")

            if not audit_result.passed:
                logger.warning(
                    f"Description for {upload_id} has low quality score: {audit_result.score}"
                )

            logger.info(
                f"Vision analysis complete for {upload_id}: {len(description)} chars, "
                f"audit_score={audit_result.score}"
            )

            await upload.update_status(ProcessingStatus.EMBEDDING)
            await self._publish_progress(
                upload_id,
                ProcessingStatus.EMBEDDING,
                ProcessingStage.EMBEDDING_GENERATION,
                60,
                "Generating embeddings",
                audit_score = audit_result.score
            )

            logger.info(f"Starting embedding generation for {upload_id}")
            embedding = await self._embedding.generate_embedding(
                description
            )
            logger.info(
                f"Generated embedding for {upload_id}: {len(embedding)} dimensions"
            )

            await self._publish_progress(
                upload_id,
                ProcessingStatus.EMBEDDING,
                ProcessingStage.INDEXING,
                90,
                "Updating database",
                audit_score = audit_result.score
            )

            logger.info(
                f"Updating database with analysis results for {upload_id}"
            )
            await upload.update_analysis(
                description = description,
                embedding = embedding,
                use_local = True,
                description_audit_score = audit_result.score,
            )

            # Publish completion
            completed_msg = UploadCompleted(
                upload_id = str(upload_id),
                description = description,
                audit_score = audit_result.score,
                timestamp = datetime.utcnow(),
            )
            await get_publisher().publish_progress(
                str(upload_id),
                completed_msg
            )

            logger.info(
                f"Local AI processing completed for upload {upload_id}"
            )

        except Exception as e:
            logger.error(
                f"Local AI processing failed for upload {upload_id}: {e}"
            )
            await upload.update_status(
                ProcessingStatus.FAILED,
                error_message = f"AI processing failed: {str(e)[:500]}",
            )

            # Publish failure
            failed_msg = UploadFailed(
                upload_id = str(upload_id),
                error_message = str(e)[: 500],
                timestamp = datetime.utcnow(),
            )
            await get_publisher().publish_progress(
                str(upload_id),
                failed_msg
            )
```

---

## Documentation

### Documentation for `LocalAIService.analyze_media`

**Purpose and Behavior:**
The `analyze_media` function processes media files (images or videos) by analyzing them using a vision model, generating text descriptions, auditing the quality of these descriptions, and creating embeddings. It updates the upload record with analysis results and handles retries in case of failures.

**Key Implementation Details:**
- **Asynchronous Operations:** The function is asynchronous, making use of `async`/`await` for I/O-bound tasks like file processing.
- **Progress Tracking:** Uses `_publish_progress` to log progress at various stages.
- **Error Handling:** Retries up to two times if the vision model returns an empty description. Logs detailed errors and updates the upload status accordingly.

**When/Why to Use:**
This function is essential for media analysis in applications that require automated content understanding, such as image or video tagging. It should be used when processing uploads that need AI-generated descriptions and embeddings.

**Patterns and Gotchas:**
- **Retry Logic:** The code retries up to two times if the vision model fails, which can help with transient errors but may lead to unnecessary processing.
- **Error Logging:** Detailed logging helps in debugging issues, especially during failures. Ensure proper error handling to avoid silent failures.
- **Configuration Sensitivity:** The function depends on configuration settings like `config.settings.upload_path` and `max_retries`, which should be carefully managed.

This function is a critical component for media processing pipelines, ensuring robustness through retries and detailed logging.

---

*Generated by CodeWorm on 2026-01-29 09:17*
