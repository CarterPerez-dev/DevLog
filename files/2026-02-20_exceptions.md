# exceptions

**Type:** File Overview
**Repository:** my-portfolio
**File:** v1/backend/app/core/exceptions.py
**Language:** python
**Lines:** 1-249
**Complexity:** 0.0

---

## Source Code

```python
"""
ⒸAngelaMos | 2025
exceptions.py
"""

from typing import Any


class BaseAppException(Exception):
    """
    Base exception for all application specific errors
    """
    def __init__(
        self,
        message: str,
        status_code: int = 500,
        extra: dict[str,
                    Any] | None = None,
    ) -> None:
        self.message = message
        self.status_code = status_code
        self.extra = extra or {}
        super().__init__(self.message)


class ResourceNotFound(BaseAppException):
    """
    Raised when a requested resource does not exist
    """
    def __init__(
        self,
        resource: str,
        identifier: str | int,
        extra: dict[str,
                    Any] | None = None,
    ) -> None:
        super().__init__(
            message = f"{resource} with id '{identifier}' not found",
            status_code = 404,
            extra = extra,
        )
        self.resource = resource
        self.identifier = identifier


class ConflictError(BaseAppException):
    """
    Raised when an operation conflicts with existing state
    """
    def __init__(
        self,
        message: str,
        extra: dict[str,
                    Any] | None = None,
    ) -> None:
        super().__init__(message = message, status_code = 409, extra = extra)


class ValidationError(BaseAppException):
    """
    Raised when input validation fails outside of Pydantic
    """
    def __init__(
        self,
        message: str,
        field: str | None = None,
        extra: dict[str,
                    Any] | None = None,
    ) -> None:
        super().__init__(message = message, status_code = 422, extra = extra)
        self.field = field


class AuthenticationError(BaseAppException):
    """
    Raised when authentication fails
    """
    def __init__(
        self,
        message: str = "Authentication failed",
        extra: dict[str,
                    Any] | None = None,
    ) -> None:
        super().__init__(message = message, status_code = 401, extra = extra)


class TokenError(AuthenticationError):
    """
    Raised for JWT token specific errors
    """
    def __init__(
        self,
        message: str = "Invalid or expired token",
        extra: dict[str,
                    Any] | None = None,
    ) -> None:
        super().__init__(message = message, extra = extra)


class TokenRevokedError(TokenError):
    """
    Raised when a revoked token is used
    """
    def __init__(self, extra: dict[str, Any] | None = None) -> None:
        super().__init__(message = "Token has been revoked", extra = extra)


class PermissionDenied(BaseAppException):
    """
    Raised when user lacks required permissions
    """
    def __init__(
        self,
        message: str = "Permission denied",
        required_permission: str | None = None,
        extra: dict[str,
                    Any] | None = None,
    ) -> None:
        super().__init__(message = message, status_code = 403, extra = extra)
 
```

---

## File Overview

### Purpose and Responsibility
This file defines a suite of custom exceptions for the application, ensuring consistent error handling across the codebase. Each exception class provides specific context and status codes relevant to different failure scenarios.

### Key Exports and Public Interface
- **BaseAppException**: The base class for all application-specific exceptions.
- **ResourceNotFound**: Raised when a requested resource does not exist.
- **ConflictError**: Indicates an operation conflicts with the existing state.
- **ValidationError**: Thrown when input validation fails outside of Pydantic.
- **AuthenticationError**: Base exception for authentication failures, including `TokenError`, `TokenRevokedError`.
- **PermissionDenied**: Raised when a user lacks required permissions.
- **RateLimitExceeded**: Indicates that the rate limit has been exceeded.
- **UserNotFound**: A specific case of `ResourceNotFound` for users.
- **EmailAlreadyExists**: Conflicts when attempting to register with an existing email.
- **InvalidCredentials**: Thrown during failed login attempts.
- **InactiveUser**: Raised when an inactive user tries to authenticate.

### How It Fits in the Project
This file is crucial for maintaining a consistent error handling strategy throughout the application. Exceptions are used not only for error reporting but also for controlling flow and enforcing business rules. They integrate seamlessly with the Flask framework, providing clear status codes and messages that can be easily handled by both the backend and frontend.

### Notable Design Decisions
- **Inheritance**: All exceptions inherit from `BaseAppException`, ensuring a uniform structure.
- **Type Hints**: Use of `typing` for type annotations to enhance code readability and maintainability.
- **Contextual Information**: Each exception includes additional context via the `extra` parameter, which can be useful for logging or detailed error messages.
- **Status Codes**: Custom status codes are assigned based on the nature of the failure, improving HTTP response consistency.

---

*Generated by CodeWorm on 2026-02-20 07:37*
