# CodeWormScheduler

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/scheduler/scheduler.py
**Language:** python
**Lines:** 231-346
**Complexity:** 0.0

---

## Source Code

```python
class CodeWormScheduler:
    """
    Manages scheduling of documentation tasks
    """
    def __init__(self, settings: ScheduleSettings) -> None:
        """
        Initialize scheduler with settings
        """
        self.settings = settings
        self.timezone = ZoneInfo(settings.timezone)
        self._scheduler = BackgroundScheduler(timezone = self.timezone)
        self._task_callback: Callable[[], None] | None = None
        self._daily_schedule: DailySchedule | None = None

    def set_task_callback(self, callback: Callable[[], None]) -> None:
        """
        Set the callback function for scheduled tasks
        """
        self._task_callback = callback

    def start(self) -> None:
        """
        Start the scheduler
        """
        if not self.settings.enabled:
            logger.info("scheduler_disabled")
            return

        trigger = HumanLikeTrigger(
            min_commits = self.settings.min_commits_per_day,
            max_commits = self.settings.max_commits_per_day,
            min_gap_minutes = self.settings.min_gap_minutes,
            prefer_hours = self.settings.prefer_hours,
            avoid_hours = self.settings.avoid_hours,
            weekend_reduction = self.settings.weekend_reduction,
            timezone = self.settings.timezone,
        )

        self._scheduler.add_job(
            self._execute_task,
            trigger = trigger,
            id = "documentation_task",
            replace_existing = True,
            misfire_grace_time = 3600,
            coalesce = True,
        )

        self._scheduler.start()
        logger.info(
            "scheduler_started",
            min_commits = self.settings.min_commits_per_day,
            max_commits = self.settings.max_commits_per_day,
            timezone = self.settings.timezone,
        )

    def stop(self, wait: bool = True) -> None:
        """
        Stop the scheduler
        """
        if self._scheduler.running:
            self._scheduler.shutdown(wait = wait)
            logger.info("scheduler_stopped")

    def _execute_task(self) -> None:
        """
        Execute a scheduled documentation task
        """
        if self._task_callback:
            try:
                logger.info("executing_scheduled_task")
                self._task_callback()
            except Exception as e:
                logger.exception("scheduled_task_failed", error = str(e))
        else:
            logger.warning("no_task_callback_set")

    def get_next_run_time(self) -> datetime | None:
        """
        Get the next scheduled run time
        """
        job = self._scheduler.get_job("documentation_task")
        if job:
            return job.next_run_time
        return None

    def get_schedule_preview(self, days: int = 1) -> list[dict]:
        """
        Preview upcoming scheduled times
        """
        trigger = HumanLikeTrigger(
            min_commits = self.settings.min_commits_per_day,
            max_commits = self.set
```

---

## Class Documentation

### CodeWormScheduler Documentation

**Class Responsibility and Purpose:**
The `CodeWormScheduler` class manages the scheduling of documentation tasks based on predefined settings. It uses a background scheduler to execute tasks at appropriate times, ensuring that these tasks are performed according to specified conditions such as commit frequency and preferred hours.

**Public Interface (Key Methods):**
- **`__init__(self, settings: ScheduleSettings) -> None`:** Initializes the scheduler with given settings.
- **`set_task_callback(self, callback: Callable[[], None]) -> None`:** Sets a callback function for scheduled tasks.
- **`start(self) -> None`:** Starts the scheduler if enabled and sets up a job to execute documentation tasks based on the provided trigger.
- **`stop(self, wait: bool = True) -> None`:** Stops the scheduler gracefully.
- **`_execute_task(self) -> None`:** Executes the scheduled task using the set callback function.
- **`get_next_run_time(self) -> datetime | None`:** Returns the next scheduled run time for the documentation task.
- **`get_schedule_preview(self, days: int = 1) -> list[dict]`:** Provides a preview of upcoming scheduled times over a specified number of days.

**Design Patterns Used:**
The class employs the **Observer pattern** through the use of a callback function (`_task_callback`) that can be set to receive notifications when tasks are executed. It also uses the **Strategy pattern** via the `HumanLikeTrigger` object, which dynamically generates scheduling times based on various conditions.

**How it Fits in the Architecture:**
In the CodeWorm architecture, `CodeWormScheduler` acts as a central component responsible for managing and executing documentation tasks. It interacts with other modules that provide settings and task callbacks, ensuring that documentation activities are performed efficiently according to predefined schedules. The scheduler's design allows for flexibility in task execution times and conditions, making it adaptable to different operational scenarios.

---

*Generated by CodeWorm on 2026-02-19 12:38*
