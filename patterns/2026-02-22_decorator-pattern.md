# decorator_pattern

**Type:** Pattern Analysis
**Repository:** CertGames-Core
**File:** backend/api/core/decorators/unified.py
**Language:** python
**Lines:** 1-198
**Complexity:** 0.0

---

## Source Code

```python
"""
Unified API Endpoint Decorator
/api/core/decorators/unified.py
"""

import time
import traceback
from typing import Any
from functools import wraps
from flask import g, request
from collections.abc import Callable

from ..services.logging import AuditLogger
from ..auth.auth_required import (
    auth_required as _auth_required,
)
from ..auth.subscription_required import (
    subscription_required as _subscription_required,
)
from ...domains.progression.services.streak_ops import StreakService
from ...domains.progression.services.progression_ops import ProgressionService


def api_endpoint(
    *,
    auth: bool = True,
    subscription: bool = False,
    audit: bool = True,
    audit_action: str | None = None,
    audit_data: dict | None = None,
    json_body: bool | None = None,
) -> Callable[[Callable[...,
                        Any]],
              Callable[...,
                       Any]]:
    """
    Unified decorator for API endpoints combining common middleware.
    """
    def decorator(f: Callable[..., Any]) -> Callable[..., Any]:
        @wraps(f)
        def decorated_function(*args: Any, **kwargs: Any) -> Any:
            """
            Used as @api_endpoint - audit | auth | subscription | json body
            """
            start_time: float = time.time()
            if json_body is None:
                expects_json: bool = request.method in [
                    "POST",
                    "PUT",
                    "PATCH"
                ]
            else:
                expects_json = json_body

            if expects_json:
                try:
                    g.json = request.get_json(
                        force = True,
                        silent = True
                    ) or {}
                except Exception:
                    g.json = {}
            else:
                g.json = {}

            if auth:

                @_auth_required
                def _temp_auth_check() -> None:
                    pass

                auth_result = _temp_auth_check()
                if auth_result is not None:

                    return auth_result

            if subscription:

                @_subscription_required
                def _temp_sub_check() -> None:
                    pass

                sub_result = _temp_sub_check()
                if sub_result is not None:
                    return sub_result

            if auth and hasattr(g, 'user') and g.user:
                try:
                    streak_result = StreakService.update_streak(g.user)
                    if streak_result["streak_updated"]:
                        daily_reward = StreakService.calculate_daily_reward(
                            streak_result["current_streak"]
                        )
                        ProgressionService.update_xp_and_coins(
                            g.user,
                            daily_reward["xp"],
                            daily_reward["coins"]
                        )
          
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Decorator Pattern

The `api_endpoint` decorator combines multiple middleware functionalities such as authentication, subscription validation, logging, and JSON body handling into a single unified decorator. This implementation uses Python's `functools.wraps` to preserve the metadata of the decorated function.

**Implementation Details:**
- The `api_endpoint` takes various parameters like `auth`, `subscription`, `audit`, etc., which control whether specific middleware should be applied.
- It wraps the original function with a nested decorator, allowing for conditional execution based on these parameters.
- Middleware functions (like `_temp_auth_check` and `_temp_sub_check`) are temporarily defined within the decorator to ensure they only run when needed.

**Benefits:**
1. **Modularity:** The ability to selectively enable or disable middleware components makes the code more modular and easier to maintain.
2. **Reusability:** A single decorator can handle multiple common tasks, reducing redundancy in API endpoint definitions.
3. **Flexibility:** Different types of endpoints (e.g., public vs. private) can be easily configured by adjusting the parameters.

**Deviations:**
- The use of temporary functions (`_temp_auth_check` and `_temp_sub_check`) to conditionally apply middleware is a deviation from the standard decorator pattern, which typically applies middleware directly.
- The handling of `g.json` and error logging involves additional context management that isn't strictly part of the core decorator functionality.

**Appropriateness:**
This pattern is highly appropriate for APIs where common functionalities like authentication and logging need to be applied consistently across multiple endpoints. It provides a clean, maintainable way to manage these concerns without cluttering each endpoint definition with boilerplate code.

---

*Generated by CodeWorm on 2026-02-22 19:08*
