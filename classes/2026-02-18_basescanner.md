# BaseScanner

**Type:** Class Documentation
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/intermediate/api-security-scanner/backend/scanners/base_scanner.py
**Language:** python
**Lines:** 21-289
**Complexity:** 0.0

---

## Source Code

```python
class BaseScanner(ABC):
    """
    Abstract base class for all security scanners

    Provides common HTTP functionality, request spacing, retry logic,
    and evidence collection. Specific scanners inherit and implement scan().
    """
    def __init__(
        self,
        target_url: str,
        auth_token: str | None = None,
        max_requests: int | None = None,
    ):
        """
        Initialize scanner with target and configuration

        Args:
            target_url: Base URL of API to scan
            auth_token: Optional authentication token
            max_requests: Optional limit on requests (from settings if None)
        """
        self.target_url = target_url.rstrip("/")
        self.auth_token = auth_token
        self.max_requests = max_requests or settings.DEFAULT_MAX_REQUESTS
        self.session = self._create_session()
        self.last_request_time = 0.0
        self.request_count = 0

    def _create_session(self) -> requests.Session:
        """
        Create persistent HTTP session with proper headers

        Returns:
            requests.Session: Configured session object
        """
        session = requests.Session()

        session.headers.update(
            {
                "User-Agent":
                f"{settings.APP_NAME}/{settings.VERSION}",
                "Accept": "application/json",
            }
        )

        if self.auth_token:
            session.headers.update(
                {"Authorization": f"Bearer {self.auth_token}"}
            )

        return session

    def _wait_before_request(
        self,
        jitter_ms: int | None = None
    ) -> None:
        """
        Implement request spacing to avoid overwhelming target

        Based on research: production-safe scanning requires spacing
        requests to avoid triggering rate limits or affecting service.

        Args:
            jitter_ms: Random jitter in milliseconds to add (DEFAULT_JITTER_MS)
        """
        if jitter_ms is None:
            jitter_ms = settings.DEFAULT_JITTER_MS

        required_delay = 1.0 / (
            self.max_requests /
            settings.SCANNER_RATE_LIMIT_WINDOW_SECONDS
        )
        jitter = random.uniform(0, jitter_ms / 1000.0)

        elapsed = time.time() - self.last_request_time

        if elapsed < required_delay:
            time.sleep(required_delay - elapsed + jitter)
        else:
            time.sleep(jitter)

        self.last_request_time = time.time()

    def make_request(
        self,
        method: str,
        endpoint: str,
        **kwargs: Any,
    ) -> requests.Response:
        """
        Make HTTP request with retry logic and rate limit handling

        Implements exponential backoff for server errors and respects
        Retry-After headers for 429 responses.

        Args:
            method: HTTP method (GET, POST, etc.)
            endpoint: Endpoint path (will be joined with target_url)
            **kwargs: Additional arguments passed to reque
```

---

## Class Documentation

### BaseScanner Class Documentation

**Class Responsibility and Purpose:**
The `BaseScanner` class serves as an abstract base class for all security scanners within the project. It provides a common framework for handling HTTP requests, managing rate limits, implementing retry logic, and collecting evidence during scans. Specific scanner classes inherit from this base class and implement the core scanning logic.

**Public Interface (Key Methods):**
- `__init__(self, target_url: str, auth_token: str | None = None, max_requests: int | None = None)`: Initializes the scanner with a target URL and optional authentication token.
- `_create_session(self) -> requests.Session`: Creates a persistent HTTP session with proper headers configured.
- `_wait_before_request(self, jitter_ms: int | None = None)`: Implements request spacing to avoid overwhelming the target server.
- `make_request(self, method: str, endpoint: str, **kwargs: Any) -> requests.Response`: Makes an HTTP request with retry logic and rate limit handling.

**Design Patterns Used:**
- **Abstract Base Class (ABC)**: Ensures that derived classes implement specific methods like `scan()`.
- **Factory Method**: `_create_session` encapsulates the creation of a session, providing a consistent interface.
- **Strategy Pattern**: The base class handles common tasks while allowing derived classes to define their own scanning logic.

**How it Fits in the Architecture:**
The `BaseScanner` class is part of the backend module for an API security scanner. It acts as a foundational component that provides essential functionalities such as HTTP request handling, rate limiting, and retry mechanisms. Specific scanner implementations can focus on defining the unique behavior required for different types of scans while leveraging the common infrastructure provided by this base class. This design promotes modularity and maintainability in the overall architecture.

---

*Generated by CodeWorm on 2026-02-18 09:50*
