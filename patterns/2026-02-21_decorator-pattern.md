# decorator_pattern

**Type:** Pattern Analysis
**Repository:** CertGames-Core
**File:** backend/api/core/auth/auth_required.py
**Language:** python
**Lines:** 1-106
**Complexity:** 0.0

---

## Source Code

```python
"""
Unified authentication decorator supporting both User and AdminUser access.
/api/core/auth/auth_required.py
"""

from typing import Any
from functools import wraps
from collections.abc import Callable
from flask import g, request, session
from flask_jwt_extended import (
    get_jwt_identity,
    verify_jwt_in_request,
    get_jwt,
)
from api.domains.account.models.User import User
from api.admin.models.users.AdminUser import AdminUser
from .unified_auth import UnifiedAuthService
from .user_cache import get_cached_user, cache_user


def auth_required(fn: Callable[..., Any]) -> Callable[..., Any]:
    """
    Unified decorator supporting authentication
    """
    @wraps(fn)
    def wrapper(*args: Any, **kwargs: Any) -> Any | tuple[dict, int]:
        user_id: str | None = None
        user: User | None = None

        try:
            verify_jwt_in_request(optional = True)
            jwt_identity = get_jwt_identity()

            if jwt_identity:
                jwt_claims = get_jwt()
                is_admin_token = jwt_claims.get("is_admin", False)

                if is_admin_token:
                    admin_user = AdminUser.objects(id = jwt_identity
                                                   ).first()
                    if admin_user:
                        user = UnifiedAuthService.get_user_for_admin(
                            admin_user
                        )
                        if user:
                            user_id = str(user.id)
                        else:
                            return {
                                "error": "Admin account not linked to user account",
                                "status": "unauthenticated",
                                "hint": "Contact administrator to link your admin account"
                            }, 401
                else:
                    user_id = jwt_identity

        except Exception:
            user_id = None

        if not user_id:
            user_id = session.get("userId")

        if not user_id:
            user_id = request.headers.get("X-User-Id")

        if not user_id:
            return {
                "error": "Authentication required",
                "status": "unauthenticated",
            }, 401

        if not user:
            try:
                user = get_cached_user(user_id)

                if not user:
                    user = User.objects(id = user_id).first()
                    if user:
                        cache_user(user)
                    else:
                        return {
                            "error": "Invalid user",
                            "status": "unauthenticated",
                        }, 401
            except Exception:
                return {
                    "error": "Authentication error",
                    "status": "unauthenticated",
                }, 401

        g.user_id = user_id
        g.user = user

        if user.is_admin:
            admin_user = Unified
```

---

## Pattern Analysis

### Pattern Analysis

**Pattern Used:** Decorator Pattern

The `auth_required` decorator in the provided code implements the **Decorator Pattern**, which allows for adding new functionality to an existing object without modifying its structure.

#### Implementation Details:
- The `auth_required` function wraps another function (`fn`) and provides authentication logic before executing it. It checks if a JWT token is present, retrieves user information from it (or session/headers), caches the user data, and sets necessary attributes in the global context (`g.user_id`, `g.user`, `g.admin_user`, `g.has_admin_access`).

#### Benefits:
- **Flexibility:** The decorator can be applied to any function that requires authentication, making the code reusable.
- **Modularity:** Authentication logic is separated from business logic, improving maintainability and testability.
- **Centralized Security Handling:** Ensures consistent security checks across multiple endpoints.

#### Deviations:
- The implementation uses `flask` context (`g`) for storing user-related data, which is a deviation from the standard decorator pattern that typically returns a new function or object without altering the original structure significantly.
- The use of exception handling and conditional logic to handle different authentication scenarios adds complexity but ensures robustness.

#### Appropriate Use Cases:
This pattern is appropriate in web applications where multiple endpoints require consistent authentication checks. It is particularly useful in frameworks like Flask, where decorators can be easily applied to routes or functions without altering their structure. However, it may not be as suitable for simpler scripts or scenarios where the overhead of a decorator might be unnecessary.

---

*Generated by CodeWorm on 2026-02-21 13:37*
