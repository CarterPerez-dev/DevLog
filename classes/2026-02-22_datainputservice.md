# DataInputService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/analytics/facets/data_input/service.py
**Language:** python
**Lines:** 32-184
**Complexity:** 0.0

---

## Source Code

```python
class DataInputService:
    """
    Service for TikTok video data input operations
    """
    @staticmethod
    async def create_video(
        session: AsyncSession,
        data: TikTokVideoCreate,
    ) -> TikTokVideoResponse:
        """
        Create a TikTok video record
        """
        video = await TikTokVideoRepository.create(
            session,
            rank = data.rank,
            date_posted = data.date_posted,
            video_url = data.video_url,
            views = data.views,
            comments = data.comments,
            likes = data.likes,
            bookmarks = data.bookmarks,
            shares = data.shares,
            avg_watch_time = data.avg_watch_time,
            new_followers = data.new_followers,
            watched_full_video_percentage = data.
            watched_full_video_percentage,
            top_comment_words = data.top_comment_words,
            search_queries = data.search_queries,
            traffic_sources = data.traffic_sources,
            hook = data.hook,
            text_on_screen_hook = data.text_on_screen_hook,
            length = data.length,
            description = data.description,
            hashtags = data.hashtags,
            cta = data.cta,
            full_transcription = data.full_transcription,
            notes = data.notes,
        )
        return TikTokVideoResponse.model_validate(video)

    @staticmethod
    async def get_video(
        session: AsyncSession,
        video_id: UUID,
    ) -> TikTokVideoResponse:
        """
        Get a single video by ID
        """
        video = await TikTokVideoRepository.get_by_id(session, video_id)
        if not video:
            raise TikTokVideoNotFound(video_id)
        return TikTokVideoResponse.model_validate(video)

    @staticmethod
    async def get_all_videos(
        session: AsyncSession,
        page: int = 1,
        page_size: int = 50,
    ) -> TikTokVideoListResponse:
        """
        Get all videos with pagination
        """
        skip = (page - 1) * page_size
        videos = await TikTokVideoRepository.get_multi(
            session,
            skip = skip,
            limit = page_size
        )
        total = await TikTokVideoRepository.count(session)

        return TikTokVideoListResponse(
            items = [
                TikTokVideoResponse.model_validate(v) for v in videos
            ],
            total = total,
            page = page,
            page_size = page_size,
        )

    @staticmethod
    async def update_video(
        session: AsyncSession,
        video_id: UUID,
        data: TikTokVideoUpdate,
    ) -> TikTokVideoResponse:
        """
        Update a video record
        """
        video = await TikTokVideoRepository.get_by_id(session, video_id)
        if not video:
            raise TikTokVideoNotFound(video_id)

        update_dict = data.model_dump(exclude_unset = True)
        video = await TikTokVideoRepository.update(
            session,
          
```

---

## Class Documentation

### DataInputService Documentation

**Class Responsibility and Purpose:**
The `DataInputService` class is responsible for handling CRUD operations and advanced queries related to TikTok video data within a database. It provides a clean, static interface for interacting with `TikTokVideoRepository`, ensuring that all operations are encapsulated and easily testable.

**Public Interface (Key Methods):**
- **create_video(session: AsyncSession, data: TikTokVideoCreate) -> TikTokVideoResponse:** Creates a new TikTok video record.
- **get_video(session: AsyncSession, video_id: UUID) -> TikTokVideoResponse:** Retrieves a single video by its ID.
- **get_all_videos(session: AsyncSession, page: int = 1, page_size: int = 50) -> TikTokVideoListResponse:** Fetches all videos with pagination support.
- **update_video(session: AsyncSession, video_id: UUID, data: TikTokVideoUpdate) -> TikTokVideoResponse:** Updates an existing video record.
- **delete_video(session: AsyncSession, video_id: UUID):** Deletes a video by its ID.
- **search_videos(session: AsyncSession, query: str) -> list[TikTokVideoResponse]:** Searches videos based on a query string.
- **get_videos_by_date_range(session: AsyncSession, start_date: date, end_date: date) -> list[TikTokVideoResponse]:** Retrieves videos within a specified date range.
- **get_videos_by_min_views(session: AsyncSession, min_views: int) -> list[TikTokVideoResponse]:** Fetches videos with at least the minimum number of views.

**Design Patterns Used:**
The class leverages the **Repository Pattern** for data access, ensuring that business logic and database interactions are separated. The use of static methods promotes a service-oriented architecture where each method performs a specific task without maintaining state or having dependencies on other objects.

**How it Fits in the Architecture:**
`DataInputService` acts as a high-level facade over `TikTokVideoRepository`, providing an easy-to-use interface for various operations. It fits into the broader analytics system by offering robust, query-optimized methods that can be used to manage and analyze TikTok video data effectively. This design ensures that the service layer remains decoupled from the repository implementation details, facilitating easier maintenance and scalability.

---

*Generated by CodeWorm on 2026-02-22 01:53*
