# CycleStats

**Type:** Class Documentation
**Repository:** CodeWorm
**File:** codeworm/daemon.py
**Language:** python
**Lines:** 44-122
**Complexity:** 0.0

---

## Source Code

```python
class CycleStats:
    """
    Tracks cycle statistics for monitoring and backoff logic
    """
    total_cycles: int = 0
    successful_cycles: int = 0
    failed_cycles: int = 0
    skipped_cycles: int = 0
    consecutive_failures: int = 0
    consecutive_ollama_failures: int = 0
    consecutive_push_failures: int = 0
    last_success: datetime | None = None
    last_failure: datetime | None = None
    last_failure_reason: str = ""
    repos_exhausted: set = field(default_factory = set)

    @property
    def success_rate(self) -> float:
        if self.total_cycles == 0:
            return 0.0
        return self.successful_cycles / self.total_cycles * 100

    def record_success(self) -> None:
        self.total_cycles += 1
        self.successful_cycles += 1
        self.consecutive_failures = 0
        self.consecutive_ollama_failures = 0
        self.last_success = datetime.now()
        self.repos_exhausted.clear()

    def record_failure(self, reason: str) -> None:
        self.total_cycles += 1
        self.failed_cycles += 1
        self.consecutive_failures += 1
        self.last_failure = datetime.now()
        self.last_failure_reason = reason

    def record_ollama_failure(self) -> None:
        self.consecutive_ollama_failures += 1

    def record_ollama_recovery(self) -> None:
        self.consecutive_ollama_failures = 0

    def record_skip(self, reason: str) -> None:
        self.total_cycles += 1
        self.skipped_cycles += 1
        self.last_failure_reason = reason

    def record_repo_exhausted(self, repo_name: str) -> None:
        self.repos_exhausted.add(repo_name)

    def get_backoff_seconds(self) -> int:
        if self.consecutive_failures <= 1:
            return 0
        return min(300, 30 * (2**(self.consecutive_failures - 1)))

    def get_ollama_wait_seconds(self) -> int:
        if self.consecutive_ollama_failures <= 1:
            return 10
        return min(300, 10 * (2**(self.consecutive_ollama_failures - 1)))

    def to_dict(self) -> dict:
        return {
            "total_cycles":
            self.total_cycles,
            "successful_cycles":
            self.successful_cycles,
            "failed_cycles":
            self.failed_cycles,
            "skipped_cycles":
            self.skipped_cycles,
            "success_rate":
            round(self.success_rate,
                  1),
            "consecutive_failures":
            self.consecutive_failures,
            "last_success":
            self.last_success.isoformat() if self.last_success else None,
        }
```

---

## Class Documentation

### CycleStats Class Documentation

**Class Responsibility and Purpose:**
The `CycleStats` class is responsible for tracking various statistics related to cycles, including success rates, consecutive failures, skipped cycles, and repository exhaustion. It provides methods to record different outcomes of cycles (success, failure, skip) and calculates backoff times based on the number of consecutive failures.

**Public Interface:**
- **Properties:** `total_cycles`, `successful_cycles`, `failed_cycles`, `skipped_cycles`, `consecutive_failures`, `consecutive_ollama_failures`, `last_success`, `last_failure_reason`, `repos_exhausted`.
- **Methods:**
  - `record_success()`: Records a successful cycle.
  - `record_failure(reason)`: Records a failure with an optional reason.
  - `record_ollama_failure()`: Increments the consecutive Ollama failures count.
  - `record_ollama_recovery()`: Resets the consecutive Ollama failures count.
  - `record_skip(reason)`: Records a skipped cycle with an optional reason.
  - `record_repo_exhausted(repo_name)`: Adds a repository to the set of exhausted repositories.
  - `get_backoff_seconds()`: Calculates and returns the backoff time in seconds based on consecutive failures.
  - `get_ollama_wait_seconds()`: Calculates and returns the Ollama wait time in seconds based on consecutive Ollama failures.
  - `to_dict()`: Converts the object's state to a dictionary for easy serialization.

**Design Patterns Used:**
- **Strategy Pattern:** The class uses methods like `record_success`, `record_failure`, etc., as different strategies for handling cycle outcomes.
- **Property Decorator:** Properties like `success_rate` are used to encapsulate complex calculations and provide a clean interface.

**Relationship to Other Classes:**
The `CycleStats` class is part of the broader monitoring and backoff logic in the CodeWorm daemon. It interacts with other classes that handle cycle execution, failure handling, and repository management. The statistics tracked by this class are crucial for making informed decisions about when to retry cycles or take other actions.

**State Management Approach:**
The class maintains a stateful approach using instance variables to track various metrics over time. This allows for dynamic adjustments in backoff logic based on historical cycle outcomes.

---

*Generated by CodeWorm on 2026-02-28 20:52*
