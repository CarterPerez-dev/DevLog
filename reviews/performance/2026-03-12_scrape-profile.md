# scrape_profile

**Type:** Performance Analysis
**Repository:** angelamos-operations
**File:** CarterOS-Server/scripts/scrape_tiktok.py
**Language:** python
**Lines:** 110-205
**Complexity:** 15.0

---

## Source Code

```python
async def scrape_profile(
    username: str,
    ms_token: str,
    max_count: int,
) -> list[dict]:
    """
    Scrape all videos from a TikTok user profile via direct API calls.
    """
    print(f"  Resolving secUid for @{username}...")
    sec_uid = await get_sec_uid(username, ms_token)
    print(f"  Got secUid: {sec_uid[:30]}...")

    videos = []
    cursor = 0
    has_more = True

    cookies = {"msToken": ms_token}

    async with httpx.AsyncClient(
        headers=TIKTOK_HEADERS,
        cookies=cookies,
        follow_redirects=True,
        timeout=30.0,
    ) as client:
        while has_more and len(videos) < max_count:
            print(f"  Fetching page (cursor={cursor})...")
            data = await fetch_videos_page(client, sec_uid, cursor)

            item_list = data.get("itemList", [])
            if not item_list:
                break

            for raw in item_list:
                if len(videos) >= max_count:
                    break

                stats = raw.get("stats", {})
                video_meta = raw.get("video", {})
                challenges = raw.get("challenges", [])
                create_time = raw.get("createTime", 0)

                hashtags = [
                    f"#{c.get('title', '')}" for c in challenges
                    if c.get("title")
                ]

                desc = raw.get("desc", "")
                hook = desc.split("\n")[0].strip() if desc else ""
                if not hook:
                    hook = desc[:100].strip() if desc else "No description"

                posted_date = datetime.fromtimestamp(
                    int(create_time)
                ).strftime("%Y-%m-%d") if create_time else date.today().isoformat()

                video_id = raw.get("id", "")

                mapped = {
                    "tiktok_id": video_id,
                    "rank": len(videos) + 1,
                    "date_posted": posted_date,
                    "video_url": f"https://www.tiktok.com/@{username}/video/{video_id}",
                    "views": stats.get("playCount", 0),
                    "comments": stats.get("commentCount", 0),
                    "likes": stats.get("diggCount", 0),
                    "bookmarks": stats.get("collectCount", 0),
                    "shares": stats.get("shareCount", 0),
                    "avg_watch_time": 0.0,
                    "new_followers": 0,
                    "watched_full_video_percentage": 0.0,
                    "top_comment_words": None,
                    "search_queries": None,
                    "traffic_sources": None,
                    "hook": hook if hook else "No hook",
                    "text_on_screen_hook": None,
                    "length": float(video_meta.get("duration", 0)),
                    "description": desc if desc else "No description",
                    "hashtags": hashtags if hashtags else None,
                    "cta": None,
                    "full_transcription": None,
              
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity (Big O Notation)
The time complexity of `scrape_profile` is primarily determined by the number of API calls made, which is proportional to `max_count`. The function has a nested loop structure and involves multiple string operations and dictionary lookups. The overall complexity can be approximated as \(O(n \cdot m)\), where \(n\) is the number of pages fetched and \(m\) is the average number of videos per page.

#### Bottlenecks or Inefficiencies
1. **Redundant Operations**: The `fetch_videos_page` call and subsequent processing are repeated until `has_more` becomes `False`. This can be costly if each API call takes significant time.
2. **Blocking Sleep**: The `await asyncio.sleep(1.5)` inside the loop introduces unnecessary delays, which could be removed or replaced with a more efficient backoff strategy.
3. **String Operations and List Comprehensions**: Multiple string operations like `.split()` and list comprehensions can be costly if performed repeatedly.

#### Optimization Opportunities
1. **Reduce Redundant Operations**: Ensure that `fetch_videos_page` is optimized to minimize the number of API calls by fetching only necessary data.
2. **Efficient String Handling**: Use more efficient string handling methods, such as pre-allocating lists and using built-in functions like `str.strip()` instead of custom logic.
3. **Backoff Strategy**: Implement a backoff strategy for retries rather than fixed delays to handle transient errors without blocking the event loop.

#### Resource Usage Concerns
1. **API Rate Limiting**: Ensure that API calls respect rate limits to avoid being blocked or throttled.
2. **Memory Management**: The list `videos` can grow large, especially if `max_count` is high. Consider using a generator instead of appending all results to the list.

By addressing these areas, you can significantly improve the performance and efficiency of the `scrape_profile` function.

---

*Generated by CodeWorm on 2026-03-12 19:26*
