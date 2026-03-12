# scrape_profile

**Type:** Security Review
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

## Security Review

### Security Review for `scrape_profile` Function

#### Vulnerabilities Found:

1. **Hardcoded Secrets**:
   - **Severity: High**
   - **Line**: N/A
   - **Fix**: Do not hardcode secrets like `ms_token`. Use environment variables or a secure vault.

2. **Input Validation Gaps**:
   - **Severity: Medium**
   - **Line**: 10-13, 45-68
   - **Fix**: Validate and sanitize all inputs to prevent injection attacks (e.g., `username`, `sec_uid`).

3. **Error Handling**:
   - **Severity: Low**
   - **Line**: N/A
   - **Fix**: Implement proper error handling to avoid information leakage.

4. **Race Conditions/TOCTOU Bugs**:
   - **Severity: Medium**
   - **Line**: 51-68
   - **Fix**: Ensure consistent state management and use atomic operations where necessary.

#### Attack Vectors:

- **Injection Attacks**: Unvalidated inputs can lead to injection attacks.
- **Information Leakage**: Improper error handling could expose sensitive information.

#### Recommended Fixes:

1. **Hardcoded Secrets**:
   - Replace hardcoded `ms_token` with environment variables or a secure vault.
   
2. **Input Validation Gaps**:
   - Validate and sanitize all inputs, especially `username`, `sec_uid`, and other dynamic values to prevent injection attacks.
   - Example: Use `re` for pattern matching in `username`.

3. **Error Handling**:
   - Implement try-except blocks to handle exceptions gracefully without exposing sensitive information.

4. **Race Conditions/TOCTOU Bugs**:
   - Ensure consistent state management, especially when dealing with asynchronous operations and shared resources.
   - Use context managers where appropriate for resource cleanup.

#### Overall Security Posture:

The code has some critical issues that could be exploited if not properly addressed. Addressing the hardcoded secrets and input validation gaps will significantly improve the security posture of this function. Proper error handling and race condition management are also essential to prevent information leakage and ensure robustness.

---

*Generated by CodeWorm on 2026-03-12 00:04*
