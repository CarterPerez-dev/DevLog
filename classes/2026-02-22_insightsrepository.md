# InsightsRepository

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/analytics/facets/insights/repository.py
**Language:** python
**Lines:** 16-469
**Complexity:** 0.0

---

## Source Code

```python
class InsightsRepository:
    """
    Repository for analytics insights calculations
    """
    @staticmethod
    async def get_all_videos(
        session: AsyncSession
    ) -> Sequence[TikTokVideo]:
        """Get all videos for analysis"""
        result = await session.execute(
            select(TikTokVideo).order_by(TikTokVideo.date_posted.desc())
        )
        return result.scalars().all()

    @staticmethod
    def calculate_engagement_rate(video: TikTokVideo) -> float:
        """Calculate engagement rate for a video"""
        if video.views == 0:
            return 0.0
        total_engagement = video.likes + video.comments + video.shares + video.bookmarks
        return (total_engagement / video.views) * 100

    @staticmethod
    def calculate_follower_conversion_rate(video: TikTokVideo) -> float:
        """Calculate follower conversion rate"""
        if video.views == 0:
            return 0.0
        return (video.new_followers / video.views) * 100

    @staticmethod
    def convert_length_to_seconds(length: float) -> int:
        """Convert TikTok format length to seconds (1.32 -> 92 seconds)"""
        minutes = int(length)
        seconds = int((length % 1) * 100)
        return minutes * 60 + seconds

    @staticmethod
    def get_length_range_label(seconds: int) -> str:
        """Get human-readable length range label"""
        ranges = [
            (30,
             "0:00-0:30"),
            (60,
             "0:30-1:00"),
            (90,
             "1:00-1:30"),
            (120,
             "1:30-2:00"),
            (150,
             "2:00-2:30"),
            (180,
             "2:30-3:00"),
        ]
        for max_sec, label in ranges:
            if seconds <= max_sec:
                return label
        return "3:00+"

    @staticmethod
    def get_day_of_week(date_posted: date) -> str:
        """Get day of week from date"""
        return date_posted.strftime("%A")

    @staticmethod
    async def get_overview_metrics(session: AsyncSession) -> dict:
        """Calculate overall performance metrics"""
        result = await session.execute(
            select(
                func.count(TikTokVideo.id).label("total_videos"),
                func.sum(TikTokVideo.views).label("total_views"),
                func.sum(TikTokVideo.likes).label("total_likes"),
                func.sum(TikTokVideo.comments).label("total_comments"),
                func.sum(TikTokVideo.shares).label("total_shares"),
                func.sum(TikTokVideo.bookmarks).label("total_bookmarks"),
                func.sum(TikTokVideo.new_followers
                         ).label("total_new_followers"),
                func.avg(TikTokVideo.avg_watch_time
                         ).label("avg_watch_time"),
                func.avg(TikTokVideo.watched_full_video_percentage
                         ).label("avg_watch_percentage"),
                func.min(TikTokVideo.date_posted).label("min_date"),
                func.max(TikTokVideo.dat
```

---

## Class Documentation

### InsightsRepository Documentation

**Class Responsibility and Purpose:**
The `InsightsRepository` class serves as a repository for analytics insights calculations, providing static methods to retrieve and process TikTok video data. It is designed to support various analytical tasks such as calculating engagement rates, follower conversion rates, and overall performance metrics.

**Public Interface (Key Methods):**
- `get_all_videos(session: AsyncSession) -> Sequence[TikTokVideo]`: Retrieves all videos for analysis.
- `calculate_engagement_rate(video: TikTokVideo) -> float`: Calculates the engagement rate for a given video.
- `calculate_follower_conversion_rate(video: TikTokVideo) -> float`: Calculates the follower conversion rate for a given video.
- `convert_length_to_seconds(length: float) -> int`: Converts TikTok format length to seconds.
- `get_length_range_label(seconds: int) -> str`: Returns a human-readable label for the video length range.
- `get_day_of_week(date_posted: date) -> str`: Determines the day of the week from the posted date.
- `get_overview_metrics(session: AsyncSession) -> dict`: Calculates overall performance metrics across all videos.

**Design Patterns Used:**
The class employs several design patterns:
- **Factory Method**: The static methods provide a factory-like approach for creating and processing video insights without exposing the underlying implementation details.
- **Strategy Pattern**: Methods like `calculate_engagement_rate` and `calculate_follower_conversion_rate` encapsulate different strategies for calculating metrics.

**How It Fits in the Architecture:**
The `InsightsRepository` class is part of a larger analytics system, handling data retrieval and processing. It interacts with an asynchronous database session to fetch video data and performs various calculations based on this data. The results can be used by other components of the application for generating reports or making informed decisions.

This design ensures that the repository remains focused on its core responsibilities while maintaining flexibility through static methods and adherence to Pythonic patterns.

---

*Generated by CodeWorm on 2026-02-22 01:55*
