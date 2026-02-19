# HumanLikeTrigger

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/scheduler/scheduler.py
**Language:** python
**Lines:** 85-228
**Complexity:** 0.0

---

## Source Code

```python
class HumanLikeTrigger(BaseTrigger):
    """
    APScheduler trigger that generates human-like scheduling patterns
    """
    def __init__(
        self,
        min_commits: int = 12,
        max_commits: int = 18,
        min_gap_minutes: int = 30,
        prefer_hours: list[int] | None = None,
        avoid_hours: list[int] | None = None,
        weekend_reduction: float = 0.7,
        timezone: str = "UTC",
    ) -> None:
        """
        Initialize the human-like trigger
        """
        self.min_commits = min_commits
        self.max_commits = max_commits
        self.min_gap_minutes = min_gap_minutes
        self.prefer_hours = prefer_hours or list(range(9, 18))
        self.avoid_hours = avoid_hours or [3, 4, 5, 6]
        self.weekend_reduction = weekend_reduction
        self.timezone = ZoneInfo(timezone)
        self._last_run: datetime | None = None
        self._daily_times: list[datetime] = []
        self._current_day: datetime | None = None

    def get_next_fire_time(
        self,
        _previous_fire_time: datetime | None,
        now: datetime
    ) -> datetime | None:
        """
        Calculate the next fire time
        """
        now_local = now.astimezone(self.timezone)
        today = now_local.date()

        if self._current_day != today or not self._daily_times:
            self._generate_daily_schedule(now_local)
            self._current_day = today

        for scheduled_time in self._daily_times:
            if scheduled_time > now_local:
                return scheduled_time.astimezone(ZoneInfo("UTC"))

        tomorrow = now_local + timedelta(days = 1)
        tomorrow_start = tomorrow.replace(
            hour = 0,
            minute = 0,
            second = 0,
            microsecond = 0
        )
        self._generate_daily_schedule(tomorrow_start)
        self._current_day = tomorrow.date()

        if self._daily_times:
            return self._daily_times[0].astimezone(ZoneInfo("UTC"))

        return None

    def _generate_daily_schedule(self, day: datetime) -> None:
        """
        Generate commit times for a day
        """
        is_weekend = day.weekday() >= 5
        commit_count = random.randint(self.min_commits, self.max_commits)

        if is_weekend:
            commit_count = int(commit_count * self.weekend_reduction)
            commit_count = max(commit_count, 3)

        self._daily_times = self._generate_times(day, commit_count)

        logger.debug(
            "daily_schedule_generated",
            date = day.date().isoformat(),
            commit_count = len(self._daily_times),
            is_weekend = is_weekend,
        )

    def _generate_times(self, day: datetime, count: int) -> list[datetime]:
        """
        Generate random times weighted by hour preferences
        """
        times: list[datetime] = []
        weights = self._build_hour_weights()
        hours = list(range(24))

        attempts = 0
        max_attempts = count * 10

        while len(t
```

---

## Class Documentation

### HumanLikeTrigger Class Documentation

**Class Responsibility and Purpose:**
The `HumanLikeTrigger` class is an APScheduler trigger designed to simulate human-like scheduling behavior for tasks. It generates a dynamic schedule of task execution times based on configurable parameters such as minimum and maximum commits, preferred hours, avoided hours, weekend reduction, and timezone.

**Public Interface (Key Methods):**
- **`__init__`:** Initializes the trigger with various configuration parameters.
- **`get_next_fire_time`:** Calculates the next fire time for a task, considering the current schedule and time constraints.
- **`_generate_daily_schedule`:** Generates commit times for a day based on preferences and avoidance rules.
- **`_generate_times`:** Creates random times weighted by hour preferences.
- **`_build_hour_weights`:** Constructs weights for hours to favor preferred times and avoid certain hours.
- **`_is_valid_time`:** Validates if a candidate time is valid given existing scheduled times.

**Design Patterns Used:**
The class leverages the **Strategy Pattern** through the `_generate_times` method, which dynamically adjusts based on preferences and avoidance rules. The use of randomization in scheduling also incorporates an element of **Randomness**, ensuring unpredictability similar to human behavior.

**How It Fits in the Architecture:**
In the CodeWorm architecture, `HumanLikeTrigger` is part of the scheduler module responsible for task execution timing. It integrates with APScheduler to dynamically adjust task schedules based on predefined rules and preferences, making it suitable for simulating real-world scenarios where tasks are executed at times that mimic human behavior. This class interacts with other components like the main scheduler instance and logging mechanisms to ensure seamless integration and effective task management.

---

*Generated by CodeWorm on 2026-02-19 12:30*
