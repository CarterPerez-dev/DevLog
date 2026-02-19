# request_handlers

**Type:** File Overview
**Repository:** CertGames-Core
**File:** backend/app/handlers/request_handlers.py
**Language:** python
**Lines:** 1-305
**Complexity:** 0.0

---

## Source Code

```python
"""
Request Lifecycle Handlers
/app/handlers/request_handlers.py
"""

import time
import uuid
import logging
import hashlib
from flask import (
    g,
    Flask,
    Response,
    request,
    session,
)
from flask_limiter import Limiter
from datetime import datetime, UTC

from flask_jwt_extended import (
    verify_jwt_in_request,
    get_jwt_identity,
)
from api.core.limiters.fallback_rate_limiter import (
    apply_fallback_rate_limiting,
)
from api.domains.account.models.User import User
from api.core.services.logging import AuditLogger
from api.admin.models.monitoring import RequestLog


logger = logging.getLogger(__name__)


def setup_request_handlers(app: Flask, limiter: Limiter) -> None:
    """
    Request lifecycle handlers
    """
    @app.before_request
    def fix_remote_addr() -> None:
        """
        Fix the remote_addr to use X-Forwarded-For if available
        """
        forwarded_for = request.headers.get("X-Forwarded-For")
        if forwarded_for:
            forwarded_ips = forwarded_for.split(",")
            request.environ["REMOTE_ADDR"] = forwarded_ips[0].strip()
            g.real_ip = forwarded_ips[0].strip()
        else:
            g.real_ip = request.remote_addr

    @app.before_request
    def log_request_start() -> None:
        """
        Track request timing and generate request ID
        """
        g.request_start_time = time.time()
        g.db_time_accumulator = 0.0
        g.request_id = str(uuid.uuid4())

        if 'session_id' not in session:
            session['session_id'] = str(uuid.uuid4())
            session.permanent = True

    app.before_request(apply_fallback_rate_limiting())

    @app.before_request
    def log_request_info() -> None:
        """
        Basic request logging
        """
        logger.info(
            "Handling request to %s with method %s",
            request.path,
            request.method
        )

    @app.before_request
    def log_api_request() -> Response | tuple[Response, int] | None:
        """
        API request logging and user tracking
        """
        if request.path.startswith("/static/"
                                   ) or request.path == "/health":
            return None

        g.request_log_data = {
            "method": request.method,
            "path": request.path,
            "ip_address": getattr(g,
                                  'real_ip',
                                  request.remote_addr),
            "user_agent": request.headers.get("User-Agent",
                                              "Unknown"),
            "query_params": dict(request.args),
            "body_size": request.content_length or 0,
            "user_id": getattr(g,
                               "user_id",
                               None),
            "is_admin": request.path.startswith("/cracked/"),
        }

        if not request.path.startswith("/cracked/"):
            _track_unique_user_request()

        return None

    @app.after_re
```

---

## File Overview

# Request Lifecycle Handlers

## Purpose and Responsibility
This Python file handles various aspects of the request lifecycle in a Flask application, including fixing remote addresses, logging requests, applying rate limiting, and tracking performance metrics.

## Key Exports or Public Interface
- `setup_request_handlers(app: Flask, limiter: Limiter)`: Configures request handlers for an Flask app.
  - `fix_remote_addr()`: Fixes the `REMOTE_ADDR` to use X-Forwarded-For if available.
  - `log_request_start()`: Tracks request timing and generates a unique request ID.
  - `apply_fallback_rate_limiting()`: Applies fallback rate limiting before each request.
  - `log_request_info()`: Logs basic information about the incoming request.
  - `log_api_request()`: Logs API requests, tracks user activity, and handles unique user tracking.
  - `log_request_end(response: Response)`: Logs performance metrics after request completion.

## How It Fits in the Project
This file is crucial for maintaining the integrity of the request handling process. It integrates with other modules like JWT authentication, rate limiting, and logging to ensure secure and efficient request processing. The handlers are applied globally via `setup_request_handlers`, making them accessible throughout the application.

## Notable Design Decisions
- **Flask Context**: Utilizes Flask's context (`g`) for storing per-request data.
- **Rate Limiting**: Implements fallback rate limiting using a custom module.
- **Logging**: Uses structured logging with `AuditLogger` and `RequestLog` to track requests and user activities comprehensively.
- **Performance Metrics**: Tracks response time, database interaction times, and other performance metrics post-request completion.
```

This documentation provides an overview of the file's purpose, key components, integration within the project, and notable design choices.

---

*Generated by CodeWorm on 2026-02-19 11:07*
