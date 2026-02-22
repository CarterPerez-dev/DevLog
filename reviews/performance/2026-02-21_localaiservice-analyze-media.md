# LocalAIService.analyze_media

**Type:** Performance Analysis
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
       
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of `analyze_media` is primarily driven by the async operations, which are generally efficient but may be impacted by network calls (e.g., fetching metadata). The main bottleneck is the potential for multiple retries (`max_retries = 2`) and the nested async calls. The overall complexity can be approximated as \(O(n \times m)\), where \(n\) is the number of retries and \(m\) is the time taken by each async operation.

#### Space Complexity
The space complexity is relatively low, with only a few variables (`description`, `audit_result`) used throughout the function. However, the use of context managers or `async with` statements could help manage resources more efficiently if file handling were involved.

#### Bottlenecks and Inefficiencies
1. **Redundant Logging**: The logging messages are repeated for each retry attempt, which can be optimized by moving them outside the loop.
2. **Multiple Progress Updates**: Publishing progress updates is done multiple times within the loop, which could be consolidated to reduce overhead.
3. **Potential N+1 Query Pattern**: If `Upload.find_by_id` or related methods make additional database calls, this could lead to an N+1 query pattern.

#### Optimization Opportunities
1. **Consolidate Logging and Progress Updates**: Move logging outside the retry loop to avoid redundant messages.
2. **Optimize Progress Updates**: Use a single progress update per attempt if possible.
3. **Caching Metadata**: Cache metadata results from `storage_service.get_upload_metadata` to reduce repeated database calls.

#### Resource Usage Concerns
- Ensure that file handles are properly managed, especially if the function is called frequently or with large files.
- Consider using context managers for any file operations to ensure proper closure and resource management.

By addressing these points, you can improve both the performance and maintainability of the `analyze_media` function.

---

*Generated by CodeWorm on 2026-02-21 21:52*
