# InsightsRepository

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/analytics/facets/insights/repository.py
**Language:** python
**Lines:** 16-418
**Complexity:** 0.0

---

## Source Code

```python
class InsightsRepository:
    """
    Repository for analytics insights calculations
    """

    @staticmethod
    async def get_all_videos(session: AsyncSession) -> Sequence[TikTokVideo]:
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
            (30, "0:00-0:30"),
            (60, "0:30-1:00"),
            (90, "1:00-1:30"),
            (120, "1:30-2:00"),
            (150, "2:00-2:30"),
            (180, "2:30-3:00"),
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
                func.sum(TikTokVideo.new_followers).label("total_new_followers"),
                func.avg(TikTokVideo.avg_watch_time).label("avg_watch_time"),
                func.avg(TikTokVideo.watched_full_video_percentage).label("avg_watch_percentage"),
                func.min(TikTokVideo.date_posted).label("min_date"),
                func.max(TikTokVideo.date_posted).label("max_date"),
            )
        )
        row = result.one()
        return {
            "total_videos": row.total_videos or 0,
            "total_vi
```

---

## Class Documentation

### InsightsRepository Documentation

**Class Responsibility and Purpose:**
The `InsightsRepository` class is responsible for calculating various analytics metrics related to TikTok videos, such as engagement rates, follower conversion rates, video length conversions, and overall performance metrics. It provides a static interface for querying and processing data from the database.

**Public Interface (Key Methods):**
- `get_all_videos(session: AsyncSession) -> Sequence[TikTokVideo]`: Retrieves all videos in descending order by date posted.
- `calculate_engagement_rate(video: TikTokVideo) -> float`: Computes the engagement rate for a given video.
- `calculate_follower_conversion_rate(video: TikTokVideo) -> float`: Calculates the follower conversion rate based on views and new followers.
- `convert_length_to_seconds(length: float) -> int`: Converts TikTok's length format to seconds.
- `get_length_range_label(seconds: int) -> str`: Returns a human-readable label for video length ranges.
- `get_day_of_week(date_posted: date) -> str`: Determines the day of the week from a given date.
- `get_overview_metrics(session: AsyncSession) -> dict`: Aggregates overall performance metrics across all videos.

**Design Patterns Used:**
The class employs several design patterns:
- **Factory Method**: Implicitly used in methods like `get_all_videos` and `aggregate_performance`, where the repository acts as a factory for video data.
- **Strategy Pattern**: The calculation of engagement rates and follower conversion rates uses different strategies based on the video's attributes.

**How It Fits in the Architecture:**
The `InsightsRepository` class is part of the analytics layer, providing essential metrics to higher-level services or dashboards. It interacts with the database through an asynchronous session (`AsyncSession`) to fetch data efficiently and processes it using static methods. This design ensures that the repository remains stateless and focused on its specific responsibilities, making it easy to integrate into a larger application architecture.

---

*Generated by CodeWorm on 2026-02-18 16:29*
