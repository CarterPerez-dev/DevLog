# SharedRateLimiterBase

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/core/limiters/shared_rate_limiter.py
**Language:** python
**Lines:** 25-439
**Complexity:** 0.0

---

## Source Code

```python
class SharedRateLimiterBase:
    """
    Base class with shared functionality for all rate limiters.
    Now uses Redis for high-performance storage instead of MongoDB.
    """
    PENALTY_PERIODS = Security.RATE_LIMIT_PENALTY_PERIODS

    def __init__(self, limiter_type: str, limits: dict[str, Any]) -> None:
        """
        Initialize the rate limiter with specific type limits.
        """
        self.limiter_type: str = limiter_type
        self.limits: dict[str, Any] = limits

        if limits.get("period", 0) <= 0:
            raise ValueError("Rate limit period must be positive")

        self.redis = current_app.extensions.get("redis_client")
        if not self.redis:
            logger.warning(
                "Redis client not available - rate limiting may not work correctly"
            )

    def _get_redis_key(self, client_id: str, suffix: str = "calls") -> str:
        """
        Generate Redis key for rate limit data
        """
        return f"rate_limit:{self.limiter_type}:{client_id}:{suffix}"

    def _check_if_blocked(self,
                          client_id: str,
                          now: datetime) -> tuple[bool,
                                                  int] | None:
        """
        Check if client is currently blocked using Redis
        """
        if not self.redis:
            return None

        try:
            block_key = self._get_redis_key(client_id, "block")
            block_until_ts = self.redis.get(block_key)

            if block_until_ts:
                block_until = datetime.fromtimestamp(
                    float(block_until_ts),
                    tz = UTC
                )
                if block_until > now:
                    retry_after = int((block_until - now).total_seconds())
                    return True, retry_after
                self.redis.delete(block_key)

            return None
        except Exception as e:
            logger.warning("Error checking block status: %s", e)
            return None

    def _calculate_valid_calls(self,
                               client_id: str,
                               now: datetime) -> tuple[int,
                                                       int]:
        """
        Calculate valid calls within the rate limit period using Redis sorted set
        """
        if not self.redis:
            return 0, self.limits["calls"]

        try:
            calls_key = self._get_redis_key(client_id, "calls")

            jitter_factor = 1 + (
                secrets.SystemRandom().random() - 0.5
            ) / 10
            period_with_jitter = self.limits["period"] * jitter_factor
            window_start = now - timedelta(seconds = period_with_jitter)
            window_start_ts = window_start.timestamp()

            self.redis.zremrangebyscore(calls_key, 0, window_start_ts)

            used_calls = self.redis.zcount(
                calls_key,
                window_start_ts,
                float('inf')
          
```

---

## Class Documentation

### SharedRateLimiterBase

**Responsibility and Purpose:**
The `SharedRateLimiterBase` class serves as a base for rate limiting mechanisms, providing shared functionality across various types of rate limiters. It leverages Redis for high-performance storage, replacing MongoDB to enhance speed and reliability.

**Public Interface (Key Methods):**
- **Initialization (`__init__`)**: Initializes the rate limiter with specific type limits.
- **_get_redis_key**: Generates a Redis key based on client ID and suffix.
- **_check_if_blocked**: Checks if a client is blocked using Redis.
- **_calculate_valid_calls**: Calculates valid calls within the rate limit period using Redis sorted sets.
- **_calculate_reset_time**: Determines when the rate limit will reset using Redis.
- **_apply_violation_penalty**: Applies penalties for repeated violations.

**Design Patterns Used:**
- **Factory Method Pattern**: Not explicitly used, but the class can be extended to create different types of rate limiters.
- **Strategy Pattern**: The class provides a strategy for handling rate limits and penalties.

**Relationship to Other Classes:**
This class is part of a broader rate limiting system in `CertGames-Core`. It interacts with other classes that inherit from it, such as specific rate limiter implementations (e.g., `UserRateLimiter`, `IPRateLimiter`). These subclasses can override or extend methods to implement custom behavior.

**State Management Approach:**
The class manages state through instance variables like `limiter_type` and `limits`. It also uses Redis for persistent storage, ensuring that rate limit data is retained across requests. The `_check_if_blocked`, `_calculate_valid_calls`, and `_apply_violation_penalty` methods handle the core logic of rate limiting, including blocking clients, calculating remaining calls, and applying penalties.

This class fits into the architecture by providing a robust foundation for rate limiting mechanisms, ensuring that rate limits are enforced efficiently and consistently across different types of clients.

---

*Generated by CodeWorm on 2026-02-18 13:21*
