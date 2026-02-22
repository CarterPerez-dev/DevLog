# ChallengeService

**Type:** Class Documentation
**Repository:** angelamos-operations
**File:** CarterOS-Server/src/aspects/challenge/facets/tracker/service.py
**Language:** python
**Lines:** 49-358
**Complexity:** 0.0

---

## Source Code

```python
class ChallengeService:
    """
    Business logic for challenge operations
    """
    @staticmethod
    def _calculate_day_number(challenge: Challenge, log_date: date) -> int:
        """
        Calculate the day number (1-30) for a given date
        """
        delta = log_date - challenge.start_date
        return delta.days + 1

    @staticmethod
    def _calculate_current_day(challenge: Challenge) -> int:
        """
        Calculate the current day number based on today's date
        """
        today = date.today()
        if today < challenge.start_date:
            return 0
        if today > challenge.end_date:
            return 30
        delta = today - challenge.start_date
        return delta.days + 1

    @staticmethod
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

    @staticmethod
    async def start_challenge(
        session: AsyncSession,
        data: ChallengeStart,
    ) -> ChallengeWithStats:
        """
        Start a new challenge (deactivates previous)
        """
        await Challeng
```

---

## Class Documentation

### ChallengeService

**Class Responsibility and Purpose:**
The `ChallengeService` class encapsulates business logic for managing challenges, including their creation, activation, logging, and history retrieval. It ensures that operations related to challenge management are centralized and consistent.

**Public Interface (Key Methods):**
- **`get_active_challenge(session: AsyncSession) -> ChallengeWithStats:`**: Retrieves the active challenge with associated stats and logs.
- **`start_challenge(session: AsyncSession, data: ChallengeStart) -> ChallengeWithStats:`**: Starts a new challenge by deactivating previous ones and creating a new one.
- **`get_history(session: AsyncSession, page: int = 1, size: int = 10) -> ChallengeHistoryResponse:`**: Fetches historical challenges with pagination support.
- **`create_or_update_log(session: AsyncSession, data: LogCreate) -> LogResponse:`**: Creates or updates a log entry for the active challenge.

**Design Patterns Used:**
- **Factory Pattern**: Implicitly used in `start_challenge`, where it creates and manages new challenges by deactivating old ones.
- **Repository Pattern**: Utilized through `ChallengeRepository` to interact with the database, ensuring separation of concerns.

**Relationship to Other Classes:**
- **ChallengeRepository**: Manages operations related to challenge data storage and retrieval.
- **ChallengeLogRepository**: Handles log entries for challenges.
- **ChallengeWithStats**, `ChallengeResponse`, `ChallengeHistoryResponse`: Models used to represent challenges and their associated data in a structured format.

This class plays a crucial role in the application's architecture by providing a centralized point of control for challenge-related operations, ensuring that all interactions with challenges are managed through this service.

---

*Generated by CodeWorm on 2026-02-22 01:16*
