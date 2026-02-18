# InsightsService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/analytics/facets/insights/service.py
**Language:** python
**Lines:** 37-425
**Complexity:** 0.0

---

## Source Code

```python
class InsightsService:
    """
    Service for analytics insights operations
    """

    @staticmethod
    async def get_overview_insights(
        session: AsyncSession,
    ) -> OverviewInsightsResponse:
        """Get high-level overview of all metrics"""
        metrics_data = await InsightsRepository.get_overview_metrics(session)
        videos = await InsightsRepository.get_all_videos(session)

        # Calculate average engagement rate
        total_engagement_rate = sum(
            InsightsRepository.calculate_engagement_rate(v) for v in videos
        )
        avg_engagement_rate = total_engagement_rate / len(videos) if videos else 0.0

        # Find top video
        top_video = None
        if videos:
            top_by_views = max(videos, key=lambda v: v.views)
            top_video = VideoPerformance(
                id=str(top_by_views.id),
                rank=top_by_views.rank,
                hook=top_by_views.hook,
                views=top_by_views.views,
                engagement_rate=InsightsRepository.calculate_engagement_rate(top_by_views),
                follower_conversion_rate=InsightsRepository.calculate_follower_conversion_rate(top_by_views),
                avg_watch_time=top_by_views.avg_watch_time,
                watched_full_video_percentage=top_by_views.watched_full_video_percentage,
                date_posted=top_by_views.date_posted.isoformat(),
            )

        # Determine trend (simple: compare first half vs second half of videos)
        trend = "stable"
        if len(videos) >= 4:
            mid_point = len(videos) // 2
            first_half_avg = sum(v.views for v in videos[:mid_point]) / mid_point
            second_half_avg = sum(v.views for v in videos[mid_point:]) / (len(videos) - mid_point)

            if second_half_avg > first_half_avg * 1.1:
                trend = "improving"
            elif second_half_avg < first_half_avg * 0.9:
                trend = "declining"

        metrics = OverviewMetrics(
            total_videos=metrics_data["total_videos"],
            total_views=metrics_data["total_views"],
            total_likes=metrics_data["total_likes"],
            total_comments=metrics_data["total_comments"],
            total_shares=metrics_data["total_shares"],
            total_bookmarks=metrics_data["total_bookmarks"],
            total_new_followers=metrics_data["total_new_followers"],
            avg_engagement_rate=avg_engagement_rate,
            avg_watch_time=metrics_data["avg_watch_time"],
            avg_watch_percentage=metrics_data["avg_watch_percentage"],
            date_range=(
                metrics_data["min_date"].isoformat() if metrics_data["min_date"] else "",
                metrics_data["max_date"].isoformat() if metrics_data["max_date"] else "",
            ),
        )

        return OverviewInsightsResponse(
            metrics=metrics,
            top_video=top_video,
            recent_trend=trend,
        )

    @staticmethod
    async def
```

---

## Class Documentation

### InsightsService Documentation

**Class Responsibility and Purpose:**
The `InsightsService` class provides a comprehensive set of analytics insights operations for video performance analysis. It includes methods to generate high-level overview metrics, top performing videos by various metrics, and hook effectiveness analysis.

**Public Interface (Key Methods):**
- **get_overview_insights(session)**: Retrieves an overview of all metrics including engagement rates, top-performing videos, and trend analysis.
- **get_performance_rankings(session, limit=10)**: Returns the top performing videos based on views, engagement rate, follower conversion, and average watch time.
- **get_hook_insights(session, limit=10)**: Analyzes hook effectiveness by retrieving a list of top-performing hooks.

**Design Patterns Used:**
- **Factory Method**: Implicitly used in `get_performance_rankings` where `to_video_performance` is a factory method for creating `VideoPerformance` instances.
- **Strategy Pattern**: Not explicitly implemented but could be inferred from the use of different metrics calculation functions like engagement rate and follower conversion.

**Relationship to Other Classes:**
- **InsightsRepository**: The class interacts with `InsightsRepository` to fetch data, such as videos and metrics. This relationship is crucial for data retrieval.
- **VideoPerformance & OverviewMetrics**: These classes are used to structure the response data, ensuring consistent and meaningful output formats.

**State Management Approach:**
The class manages state through method parameters (e.g., `session`, `limit`) rather than maintaining internal state. This ensures that each call is independent and does not rely on previous operations or global state.

This class fits into a broader analytics framework where detailed insights are required for performance analysis, enabling data-driven decision-making in video content strategy.

---

*Generated by CodeWorm on 2026-02-18 16:25*
