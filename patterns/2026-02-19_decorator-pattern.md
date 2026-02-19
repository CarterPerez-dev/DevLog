# decorator_pattern

**Type:** Pattern Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/api-rate-limiter/src/fastapi_420/limiter.py
**Language:** python
**Lines:** 1-365
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
limiter.py
"""

from __future__ import annotations

import asyncio
import logging
import functools
from typing import (
    Any,
    ParamSpec,
    TypeVar,
    TYPE_CHECKING,
)
from collections.abc import Callable

from starlette.requests import Request

from fastapi_420.algorithms import (
    create_algorithm,
)
from fastapi_420.config import (
    RateLimiterSettings,
    get_settings,
)
from fastapi_420.exceptions import (
    EnhanceYourCalm,
    StorageConnectionError,
    StorageError,
)
from fastapi_420.fingerprinting import (
    CompositeFingerprinter,
)
from fastapi_420.storage import (
    MemoryStorage,
    RedisStorage,
    create_storage,
)
from fastapi_420.types import (
    Layer,
    RateLimitKey,
    RateLimitResult,
    RateLimitRule,
)

if TYPE_CHECKING:
    from fastapi_420.algorithms.base import BaseAlgorithm
    from fastapi_420.storage import Storage


logger = logging.getLogger("fastapi_420")

P = ParamSpec("P")
R = TypeVar("R")


class RateLimiter:
    """
    Main rate limiter class for FastAPI applications.

    Usage:
        limiter = RateLimiter()

        @app.get("/api/data")
        @limiter.limit("100/minute", "1000/hour")
        async def get_data(request: Request):
            return {"data": "value"}
    """
    def __init__(
        self,
        settings: RateLimiterSettings | None = None,
        storage: Storage | None = None,
    ) -> None:
        self._settings = settings or get_settings()
        self._storage = storage
        self._fallback_storage: MemoryStorage | None = None
        self._algorithm: BaseAlgorithm | None = None
        self._fingerprinter: CompositeFingerprinter | None = None
        self._initialized = False
        self._lock = asyncio.Lock()

    async def init(self) -> None:
        """
        Initialize storage, algorithm, and fingerprinter
        """
        async with self._lock:
            if self._initialized:
                return

            if self._storage is None:
                self._storage = create_storage(self._settings.storage)

            if self._settings.storage.FALLBACK_TO_MEMORY:
                self._fallback_storage = MemoryStorage.from_settings(
                    self._settings.storage
                )
                await self._fallback_storage.start_cleanup_task()

            if isinstance(self._storage, RedisStorage):
                try:
                    await self._storage.connect()
                except StorageConnectionError:
                    if self._settings.FAIL_OPEN and self._fallback_storage:
                        logger.warning(
                            "Redis unavailable, using memory fallback",
                            extra = {
                                "redis_url":
                                self._settings.storage.REDIS_URL
                            },
                        )
                        self._storage = self._fallback_storage
                        self._fallb
```

---

## Pattern Analysis

### Pattern Analysis

#### Decorator Pattern

The `RateLimiter` class in the provided code uses the **Decorator Pattern** to apply rate limits to API endpoints. Specifically, the `limit` method acts as a decorator that can be applied to FastAPI routes.

- **Implementation**: The `limit` method is defined within the `RateLimiter` class and returns another function (decorator) that wraps the original endpoint function.
- **Usage Example**:
  ```python
  @app.get("/api/data")
  @limiter.limit("100/minute", "1000/hour")
  async def get_data(request: Request):
      return {"data": "value"}
  ```

#### Benefits

- **Modularity and Flexibility**: The decorator allows for easy application of rate limiting to any function without modifying its implementation.
- **Centralized Configuration**: Rate limit rules can be defined centrally in the `RateLimiter` class, making it easier to manage and update them.

#### Deviations from Standard Pattern

- **Initialization Logic**: The `limit` method is not a standalone decorator but part of the `RateLimiter` class. This deviation ensures that rate limiting logic is managed within the same context.
- **Async Support**: The decorator uses `async def`, allowing it to handle asynchronous functions, which is crucial for modern API development.

#### Appropriate Use Cases

- **API Rate Limiting**: Ideal for applying rate limits to FastAPI endpoints where performance and security are critical.
- **Centralized Management**: Suitable when you need a centralized way to manage rate limiting rules across multiple routes or services.

This pattern is appropriate in scenarios requiring dynamic and flexible rate limiting configurations, especially in microservices architectures like those built with FastAPI.

---

*Generated by CodeWorm on 2026-02-19 14:10*
