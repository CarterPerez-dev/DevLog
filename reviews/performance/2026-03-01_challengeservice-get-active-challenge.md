# ChallengeService.get_active_challenge

**Type:** Performance Analysis
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/challenge/facets/tracker/service.py
**Language:** python
**Lines:** 75-129
**Complexity:** 3.0

---

## Source Code

```python
async def get_active_challenge(
        session: AsyncSession,
    ) -> ChallengeWithStats:
        """
        Get active challenge with stats and logs
        """
        challenge = await ChallengeRepository.get_active(session)
        if not challenge:
            raise ChallengeNotFound()

        total_content, total_jobs = await ChallengeLogRepository.get_totals(
            session,
            challenge.id,
        )

        logs_response = [
            LogResponse(
                id = log.id,
                challenge_id = log.challenge_id,
                log_date = log.log_date,
                day_number = ChallengeService._calculate_day_number(
                    challenge,
                    log.log_date
                ),
                tiktok = log.tiktok,
                instagram_reels = log.instagram_reels,
                youtube_shorts = log.youtube_shorts,
                twitter = log.twitter,
                reddit = log.reddit,
                linkedin_personal = log.linkedin_personal,
                linkedin_company = log.linkedin_company,
                youtube_full = log.youtube_full,
                medium = log.medium,
                jobs_applied = log.jobs_applied,
                created_at = log.created_at,
                updated_at = log.updated_at,
            ) for log in challenge.logs
        ]

        return ChallengeWithStats(
            id = challenge.id,
            name = challenge.name,
            start_date = challenge.start_date,
            end_date = challenge.end_date,
            is_active = challenge.is_active,
            content_goal = challenge.content_goal,
            jobs_goal = challenge.jobs_goal,
            total_content = total_content,
            total_jobs = total_jobs,
            current_day = ChallengeService.
            _calculate_current_day(challenge),
            logs = logs_response,
            created_at = challenge.created_at,
            updated_at = challenge.updated_at,
        )
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity:** The time complexity of `get_active_challenge` is O(n + m), where n is the number of logs in the challenge, and m is the number of operations to calculate totals from the log repository.

**Space Complexity:** The space complexity is O(n) due to storing all logs in memory as `logs_response`.

**Bottlenecks or Inefficiencies:**
1. **N+1 Query Pattern:** The function makes a separate query for each log, leading to an N+1 query pattern.
2. **Redundant Calculations:** The `_calculate_day_number` and `_calculate_current_day` are called multiple times.

**Optimization Opportunities:**
1. **Batch Queries:** Use batch queries to fetch logs and their totals in one go, reducing the number of database calls from n to 1 + m.
2. **Memoization:** Memoize the day calculation functions to avoid redundant calculations for each log.

```python
async def get_active_challenge(
        session: AsyncSession,
) -> ChallengeWithStats:
    """
    Get active challenge with stats and logs
    """
    challenge = await ChallengeRepository.get_active(session)
    if not challenge:
        raise ChallengeNotFound()

    # Batch query to fetch logs and their totals
    logs, total_content, total_jobs = await ChallengeLogRepository.get_totals_batch(
        session,
        challenge.id,
    )

    logs_response = [
        LogResponse(
            id=log.id,
            challenge_id=log.challenge_id,
            log_date=log.log_date,
            day_number=ChallengeService._calculate_day_number(challenge, log.log_date),
            tiktok=log.tiktok,
            instagram_reels=log.instagram_reels,
            youtube_shorts=log.youtube_shorts,
            twitter=log.twitter,
            reddit=log.reddit,
            linkedin_personal=log.linkedin_personal,
            linkedin_company=log.linkedin_company,
            youtube_full=log.youtube_full,
            medium=log.medium,
            jobs_applied=log.jobs_applied,
            created_at=log.created_at,
            updated_at=log.updated_at,
        ) for log in logs
    ]

    return ChallengeWithStats(
        id=challenge.id,
        name=challenge.name,
        start_date=challenge.start_date,
        end_date=challenge.end_date,
        is_active=challenge.is_active,
        content_goal=challenge.content_goal,
        jobs_goal=challenge.jobs_goal,
        total_content=total_content,
        total_jobs=total_jobs,
        current_day=ChallengeService._calculate_current_day(challenge),
        logs=logs_response,
        created_at=challenge.created_at,
        updated_at=challenge.updated_at,
    )
```

**Resource Usage Concerns:**
- Ensure `AsyncSession` is properly managed and closed to avoid resource leaks.
- Consider using context managers or async context managers for session handling.

---

*Generated by CodeWorm on 2026-03-01 18:56*
