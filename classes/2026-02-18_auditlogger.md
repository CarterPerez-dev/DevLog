# AuditLogger

**Type:** Class Documentation
**Repository:** CertGames-Core
**File:** backend/api/core/services/logging/audit.py
**Language:** python
**Lines:** 13-215
**Complexity:** 0.0

---

## Source Code

```python
class AuditLogger:
    """
    Centralized audit logging service using MongoEngine
    """
    @staticmethod
    def _extract_request_info() -> dict[str, Any]:
        """
        Extract information from the current request
        """
        try:
            if not request or not request.method:
                return {}
        except RuntimeError:
            return {}

        info = {
            "ip_address": request.remote_addr,
            "user_agent": request.headers.get("User-Agent",
                                              "")[: 500],
            "endpoint": request.endpoint,
            "method": request.method,
            "request_id": getattr(g,
                                  'request_id',
                                  None),
        }

        try:
            if session:
                info["session_id"] = session.get('session_id')
        except Exception:
            info["session_id"] = None

        return info

    @staticmethod
    def _calculate_response_time(provided_time: int | None) -> int | None:
        """
        Calculate response time if not provided
        """
        if provided_time is not None:
            return provided_time

        if hasattr(g, "start_time"):
            return int(
                (datetime.now(UTC).timestamp() - g.start_time) * 1000
            )

        return None

    @staticmethod
    def _build_audit_data(
        base_data: dict[str,
                        Any] | None,
        request_info: dict[str,
                           Any],
        response_time: int | None,
        status_code: int | None,
    ) -> dict[str,
              Any]:
        """
        Build the audit data dictionary
        """
        audit_data = base_data or {}

        skip_fields = {
            "ip_address",
            "user_agent",
            "endpoint",
            "method",
            "request_id",
            "session_id"
        }

        for key, value in request_info.items():
            if value and key not in skip_fields:
                audit_data[key] = value

        if response_time is not None:
            audit_data["response_time_ms"] = response_time
        if status_code is not None:
            audit_data["status_code"] = status_code

        return audit_data

    @staticmethod
    def log_action(
        action: str,
        user_id: str | None = None,
        data: dict[str,
                   Any] | None = None,
        status_code: int | None = None,
        response_time: int | None = None,
    ) -> AuditLog | None:
        """
        Log an action to the audit trail.
        """
        try:
            request_info = AuditLogger._extract_request_info()

            actual_response_time = AuditLogger._calculate_response_time(
                response_time
            )

            audit_data = AuditLogger._build_audit_data(
                data,
                request_info,
                actual_response_time,
                status_code
            
```

---

## Class Documentation

### AuditLogger Class Documentation

**Class Responsibility and Purpose:**
The `AuditLogger` class is responsible for centralizing audit logging within a Python application using MongoEngine. It extracts relevant information from HTTP requests, calculates response times, and logs actions to an audit trail.

**Public Interface (Key Methods):**
- `_extract_request_info()`: Extracts request-related metadata.
- `_calculate_response_time(provided_time: int | None)`: Calculates the response time if not provided.
- `_build_audit_data(base_data: dict[str, Any] | None, request_info: dict[str, Any], response_time: int | None, status_code: int | None)`: Builds the audit data dictionary.
- `log_action(action: str, user_id: str | None = None, data: dict[str, Any] | None = None, status_code: int | None = None, response_time: int | None = None) -> AuditLog | None`: Logs an action to the audit trail.
- `log_database_timing(collection_name: str, operation: str, duration: float) -> AuditLog | None`: Logs database operation timing.
- `log_auth_event(event_type: str, user_id: str | None = None, success: bool = True, details: dict[str, Any] | None = None) -> AuditLog | None`: Logs authentication events.
- `log_api_call(endpoint: str, user_id: str | None = None, response_time: int | None = None) -> AuditLog | None`: Logs API calls.

**Design Patterns Used:**
The class uses a **Strategy pattern** for the `_build_audit_data` method, allowing flexible data inclusion based on provided parameters. It also employs **Static Methods**, making it suitable for utility functions without needing an instance of the class.

**Relationship to Other Classes:**
- `AuditLog`: The `log_action` method interacts with this class to create and log audit records.
- `request`: Utilizes request-related information from Flask's `g` object, indicating its integration within a web framework context.
- `session`: Accesses session data for additional logging details.

**State Management Approach:**
The class relies on the global `g` object and Flask’s `session` to store temporary state (e.g., request ID) during the request lifecycle. This approach ensures that relevant information is available throughout the request handling process.

---

*Generated by CodeWorm on 2026-02-18 13:04*
